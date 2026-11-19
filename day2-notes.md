# Day 2: File Viewing, Terminal Editors & OverTheWire Bandit 0–5 (19.11.2026)

Today I practiced monitoring live system logs, editing files directly inside the terminal using `nano` and `vim`, inspecting unknown files safely, and solving OverTheWire Bandit Levels 0 to 5 over SSH.

---

## Live Log Monitoring (`tail -f` & `journalctl`)

- **`tail -f /var/log/pacman.log`**: Watches a log file in real time and streams new entries as they happen (tested live while installing `htop`, `btop`, and `tree` on Arch Linux).
- **`Ctrl + C`**: Stops following the live log output and returns to the terminal prompt.
- **`journalctl -f`**: Systemd command for following live system and service logs.

---

## Terminal Text Editors (`nano` & `vim`)

### `nano` (Beginner-Friendly Editor)
- `nano <filename>` — Open or create a file.
- `Ctrl + O`, then `Enter` — Save changes.
- `Ctrl + X` — Exit `nano`.

### `vim` (Modal Terminal Editor)
Vim separates navigating/commanding from typing text using modes:
- **Normal Mode** (default on open): Press `Esc` from anywhere to return here.
- **Insert Mode**: Press `i` to type and edit text normally.
- **Command-Line Mode** (typed while in Normal Mode):
  - `:w` — Save (write) the file.
  - `:q` — Quit `vim`.
  - `:wq` — Save and quit.
  - `:q!` — Force quit **without** saving changes.

---

## Inspecting Unknown Files & Disk Usage (`file` & `du`)

- **`file <filename>`**: Checks a file's actual type (`ASCII text`, `data`, or escape sequences) **before** running `cat`. Prevents printing raw binary data that can garble or freeze the terminal.
- **`du -ab`**: Calculates the exact size in bytes of every file and subdirectory.
- **`du -sh *`**: Displays a human-readable summary total (`KB`, `MB`, `GB`) for each item in the current directory.

---

## OverTheWire Bandit Wargame (Levels 0–5)

Connected to the remote lab server using SSH with a keep-alive interval so the session wouldn't time out:
```bash
ssh -o ServerAliveInterval=30 bandit0@bandit.labs.overthewire.org -p 2220
```

### Key Techniques Used
- **Level 0 → 1**: Used `ls -la` and `cat readme` to read the initial password file.
- **Level 1 → 2 (Dashed Filename)**: Read a file named `-` using `cat ./-` so Linux wouldn't treat `-` as a command flag.
- **Level 2 → 3 (Spaces in Filename)**: Handled filenames containing spaces by wrapping them in double quotes (`cat "spaces in this filename"`) or using `Tab` auto-completion.
- **Level 3 → 4 (Hidden Files)**: Navigated into `inhere` and used `ls -la` to reveal and read a hidden dot-file (`...Hiding-From-You`).
- **Level 4 → 5 (Human-Readable vs. Binary)**: Ran `file ./*` across 10 files (`-file00` to `-file09`) to identify the single `ASCII text` file (`./-file07`) without opening binary `data` files.
- **Level 5 → 6 (Targeted File Search)**: Located a 1033-byte, non-executable, human-readable file buried across multiple subdirectories using either:
  ```bash
  find . -type f -size 1033c ! -executable
  # or
  du -ab | grep 1033
  ```

---

## Takeaways & Mistakes Avoided
- Always run `file <filename>` before `cat` on unknown files to avoid dumping binary garbage into the terminal.
- Never store wargame passwords inside public GitHub repositories—keep them inside a `.gitignore`-protected local folder (`sandbox/`).

