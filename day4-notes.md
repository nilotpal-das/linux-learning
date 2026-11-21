# Day 4: Linux File Permissions, `sudoers` & OverTheWire Bandit (Levels 11–14)
**Date:** 21-11-2026

---

## 1. Understanding Linux File Permissions (`rwx`)
When running `ls -l`, the first 10 characters show the file type and permissions across three categories: **User/Owner (`u`)**, **Group (`g`)**, and **Others (`o`)**.

```text
-rwxr-xr-- 1 nilotpal wheel 4096 Nov 21 20:15 script.sh
```


* **1st character:** Item type (`-` = regular file, `d` = directory, `l` = symbolic link).
* **Characters 2–4 (`u`):** Permissions for the file's **owner**.
* **Characters 5–7 (`g`):** Permissions for members of the file's **group**.
* **Characters 8–10 (`o`):** Permissions for **all other users** on the system.

### How `r`, `w`, and `x` Differ on Files vs. Directories

| Permission | Numeric Value | On a Regular File | On a Directory (Folder) |
| :--- | :---: | :--- | :--- |
| **`r` (Read)** | `4` | View/copy file contents (`cat`, `less`). | List filenames inside the folder (`ls`). |
| **`w` (Write)** | `2` | Modify or overwrite the file's contents. | Create, rename, or delete files inside the folder. |
| **`x` (Execute)** | `1` | Run the file as a program or shell script (`./script.sh`). | Enter/traverse the directory (`cd`) and access file metadata/contents. |

> **Key Takeaway:** Having `r` (read) on a directory *without* `x` (execute) lets you list filenames inside it, but blocks you from `cd`-ing into it or reading the files inside.

---

## 2. Changing Permissions (`chmod`)

### Numeric (Octal) Mode
Add the permission values (`r=4`, `w=2`, `x=1`, `-=0`) for each trio (**User**, **Group**, **Others**):

| Octal Mode | Symbolic | Breakdown | Common Use Case |
| :---: | :---: | :--- | :--- |
| **`755`** | `rwxr-xr-x` | Owner: `7` (`rwx`), Group: `5` (`r-x`), Others: `5` (`r-x`) | Standard directories and executable scripts. |
| **`644`** | `rw-r--r--` | Owner: `6` (`rw-`), Group: `4` (`r--`), Others: `4` (`r--`) | Regular text files and configuration files. |
| **`600`** | `rw-------` | Owner: `6` (`rw-`), Group: `0` (`---`), Others: `0` (`---`) | SSH private keys (`id_ed25519`) and sensitive secrets. |
| **`440`** | `r--r-----` | Owner: `4` (`r--`), Group: `4` (`r--`), Others: `0` (`---`) | System security files like `/etc/sudoers`. |

```bash
chmod 755 deploy.sh          # Full access for owner; read & execute for everyone else
chmod 644 config.txt         # Read/write for owner; read-only for everyone else
chmod 600 ~/.ssh/id_ed25519  # Strict private key permissions (required by SSH)
```

### Symbolic Mode
Modify specific permissions without recalculating all three numbers:
* **Who:** `u` (user/owner), `g` (group), `o` (others), `a` (all)
* **Action:** `+` (add), `-` (remove), `=` (set exact)

```bash
chmod u+x script.sh          # Add execute permission for the owner
chmod g-w shared.txt         # Remove write permission from the group
chmod o= secret.txt          # Strip all permissions from others
chmod -R 755 ./project/      # Recursively apply permissions to a directory
```

---

## 3. Privilege Escalation & `/etc/sudoers`
`sudo` allows authorized users to run commands with elevated (`root`) privileges without sharing the root password.

### Golden Rules for `sudoers`
* **Never edit `/etc/sudoers` directly** with `nano` or `vim`. A single syntax error can break `sudo` across the entire system.
* **Always use `visudo`** (`sudo EDITOR=vim visudo` or `sudo EDITOR=nano visudo`), which validates syntax before saving.
* **Drop-in Configs (`/etc/sudoers.d/`):** Modern Linux distros (including Arch Linux) keep `/etc/sudoers` clean by loading custom user/group rules from `/etc/sudoers.d/` via `@includedir /etc/sudoers.d`.

### Rule Anatomy
```text
username ALL=(ALL:ALL) ALL
%wheel   ALL=(ALL:ALL) ALL
```
* **`username` / `%wheel`:** Applies to a specific user, or a group when prefixed with `%`.
* **`ALL` (1st):** Hosts where the rule applies.
* **`(ALL:ALL)`:** Target users and groups the user can run commands as.
* **`ALL` (2nd):** Commands allowed (can be restricted to specific binaries like `/usr/bin/systemctl` for least privilege).

### Shell Expansion vs. `sudo` Gotcha
Running `sudo cat /etc/sudoers.d/*` fails with *"no matches"* because the unprivileged shell tries to expand the `*` wildcard **before** `sudo` runs, and regular users cannot read inside `/etc/sudoers.d/` (`0750`).

* **Fix:** List the exact filename first with `sudo ls -la /etc/sudoers.d`, then read it directly with `sudo cat /etc/sudoers.d/<filename>`.

---

## 4. OverTheWire Bandit Walkthrough (Levels 11–14)

### Bandit Level 11 (`bandit11 -> bandit12`)
`data.txt` contained text rotated by 13 characters (**ROT13**). Decoded it using `tr` (translate):
```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

### Bandit Level 12 (`bandit12 -> bandit13`)
`data.txt` was a hexdump of a file compressed repeatedly with `gzip`, `bzip2`, and `tar`. Worked inside a temporary directory in `/tmp`:
```bash
mktemp -d
cd /tmp/tmp.XXXXXXXXXX
cp ~/data.txt .
xxd -r data.txt > data.bin      # Convert hexdump back into binary
```
Repeatedly checked the outer layer with `file <filename>` and unpacked step-by-step:

* `gzip compressed data` → `mv <file> <file>.gz && gunzip <file>.gz`
* `bzip2 compressed data` → `mv <file> <file>.bz2 && bunzip2 <file>.bz2`
* `POSIX tar archive` → `tar -xvf <file>`
* `ASCII text` → `cat <file>` to read the Level 13 password.

### Bandit Level 13 (`bandit13 -> bandit14`)
The home directory held an SSH private key (`sshkey.private`) instead of a password. Since OverTheWire blocks `localhost` SSH connections on the game server, copied the key to my local machine (`sandbox/bandit14.key`), locked down permissions to `600`, and authenticated with `-i`:
```bash
scp -P 2220 bandit13@bandit.labs.overthewire.org:sshkey.private ~/linux-learning/sandbox/bandit14.key
chmod 600 ~/linux-learning/sandbox/bandit14.key
ssh -i ~/linux-learning/sandbox/bandit14.key bandit14@bandit.labs.overthewire.org -p 2220
```
Once logged into `bandit14`, read `/etc/bandit_pass/bandit14` for the Level 14 password.

### Bandit Level 14 (`bandit14 -> bandit15`)
Sent the `bandit14` password to a local TCP service listening on port `30000` using **Netcat (`nc`)**:
```bash
cat /etc/bandit_pass/bandit14 | nc localhost 30000
```

### Bandit Level 15 (`bandit15 -> bandit16`)
Sent the `bandit15` password to a local **SSL/TLS-encrypted** service listening on port `30001` using `openssl s_client` (with `-ign_eof` so the connection stays open long enough to receive the response):
```bash
cat /etc/bandit_pass/bandit15 | openssl s_client -connect localhost:30001 -ign_eof
```



---

## 5. Takeaways & Mistakes Avoided
* SSH strictly rejects private key files if permissions are broader than `600` (`-rw-------`).
* Wildcard globbing (`*`) is evaluated by the current user's shell *before* `sudo` elevates privileges.
* Keep private keys (`bandit14.key`) and wargame passwords inside the `.gitignore`-protected `sandbox/` directory.

