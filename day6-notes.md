# Week 2, Day 6: Standard Streams, Redirection & Text Filtering Pipelines

## 1. Standard File Descriptors (Streams)
Every Linux process opens three default file descriptors:
- **`0` (`stdin`):** Standard input (keyboard or `<` file input).
- **`1` (`stdout`):** Standard output (normal output printed to terminal).
- **`2` (`stderr`):** Standard error (diagnostic/permission error messages, isolated so they don't corrupt piped data).

### Redirection Operators

| Operator | Explicit Form | Behavior | SRE Use Case |
| :--- | :--- | :--- | :--- |
| `> file` | `1> file` | Overwrites `file` with `stdout` | Saving fresh diagnostic snapshots |
| `>> file` | `1>> file` | Appends `stdout` to `file` | Logging ongoing checks without wiping history |
| `2> file` | `2> file` | Redirects only `stderr` | Sending `Permission denied` errors to `/dev/null` |
| `2>&1` | `2>&1` | Merges `stderr` into `stdout`'s target | Capturing full output + errors in logs or pipes |
| `&> file` | `> file 2>&1` | Shorthand for redirecting both streams | Quick ad-hoc debugging in Bash/Zsh |
| `< file` | `0< file` | Feeds `file` into `stdin` | Passing input to `stdin`-only tools (`tr`) or getting raw `wc -l` counts |
| `cmd1 \| cmd2` | — | Pipes `stdout` of `cmd1` into `stdin` of `cmd2` | Chaining small CLI utilities into log analyzers |

**Key Gotchas:**
- Never add a space between the FD number and `>` (`1 > file` passes `1` as an argument instead of redirecting FD 1).
- Pipes (`|`) only pass `stdout` (`1`). Use `2>&1` before `|` if you want `stderr` included in the pipeline.
- `wc -l < file` outputs a clean integer without the filename (ideal for Bash variables), whereas `wc -l file` appends the filename.

---

## 2. Text Filtering & Aggregation Toolkit
- **`grep`:** Searches text line by line.
  - `-i`: Case-insensitive match.
  - `-v`: Invert match (exclude matching lines).
  - `-E`: Extended regex (`|`, `+`, `?`).
  - `-o`: Print **only** the matched substring instead of the entire line.
  - `-r` / `-n`: Recursive directory search / show line numbers.
- **`sort`:** Orders lines (`-n` for numeric sort, `-r` for reverse/descending order).
- **`uniq`:** Collapses **adjacent** duplicate lines (`-c` prefixes occurrence count). Always `sort` before `uniq`.
- **`wc`:** Counts lines (`-l`), words (`-w`), or bytes (`-c`).

---

## 3. Hands-On Labs & SRE One-Liners

### Split `stdout` and `stderr` Cleanly
```bash
grep -ri "failed" /var/log/ > valid_matches.txt 2> /dev/null
wc -l < valid_matches.txt
```
- Separates valid log hits from noisy `Permission denied` errors without risking accidental data loss from `grep -v`.

### Strip Comments & Blank Lines from Configs
```bash
grep -vE '^\s*#|^\s*\$' /etc/nginx/nginx.conf
```
- Reveals only active configuration directives during incident triage.

### Top HTTP Status Codes from Nginx Access Logs
```bash
sudo grep -h "HTTP/1.1" /var/log/nginx/access.log | grep -oE '" [0-9]{3} ' | sort | uniq -c | sort -nr
```
- Verified locally against Nginx (`6` x `200`, `3` x `404`). Turns thousands of raw log lines into an instant health summary.

