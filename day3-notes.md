# Day 3: Linux Users, Groups, Ownership & OverTheWire Bandit (Levels 6–10)
**Date:** 20-11-2026

---

## 1. Linux Users & `/etc/passwd`
Linux keeps its user account information in a plain-text file named `passwd` inside `/etc`.

```bash
cat /etc/passwd         # To print its content
```

**Format:** `name:password:UID:GID:GECOS:directory:shell`

* **`password`:** Stored as `x`; the real encrypted password is stored in `/etc/shadow`.
* **`UID` (User ID):** A unique numeric identifier:
  * `0` — Reserved for `root`
  * `1–99` — Predefined system accounts
  * `100–999` — System/service accounts
  * `1000+` — Standard users
* **`GID` (Group ID):** The numeric primary group ID assigned to the user (defined in `/etc/group`).
* **`GECOS`:** Optional descriptive data such as the user's full name.
* **`directory`:** The absolute path to the user's home directory (`/home/username` for users).
* **`shell`:** The absolute path to the user's default login shell (`/bin/bash`).

---

## 2. Linux Groups & `/etc/group`
In Linux, groups manage shared permissions for users. Each group can have zero or more users, and the file lives in `/etc` as well.

```bash
cat /etc/group
```

**Format:** `group_name:password:GID:user_list`

* **`user_list`:** Comma-separated list of users in that group.
* **Example:** `sudo:x:27:alice,bob`

---

## 3. Essential User and Group Commands

| Command | Description |
| :--- | :--- |
| `id` or `id user_name` | Shows the UID, GID, and all groups that `user_name` belongs to. |
| `sudo useradd -m -s /bin/bash user_name` | Creates a new user (`-m` creates the user's home directory, `-s /bin/bash` sets the shell). |
| `sudo passwd user_name` | Sets or changes a user's password. |
| `sudo groupadd group_name` | Creates a new group. |
| `sudo usermod -aG group1,group2 user_name` | Appends (`-aG`) a user to specific secondary groups. |
| `sudo userdel -r username` | Deletes a user along with their home folder (`-r`). |
| `sudo groupdel group_name` | Deletes a group. |

---

## 4. File Ownership (`chown` & `chgrp`)
Every file and directory has two owners: **User owner** and **Group owner**.

`ls -l` shows the contents of a directory in long list format:
```text
-rw-r--r-- 1 user group 4096 Nov 20 18:10 example.txt
```

* **1st character:** Indicates item type (`-` for file, `d` for directory).
* **Other 9 characters:** Grouped into 3 sets of 3 (**owner**, **group**, **others**).
  * `r` = readable, `w` = writable, `x` = executable or accessible.

### Changing Ownership
```bash
sudo chown user:group filename       # Change both user and group
sudo chown user filename             # Change only user
sudo chown :group filename           # Change only group
sudo chgrp group filename            # Change only group (alternative)
sudo chown -R user:group /path       # Change entire directory and everything inside
```


---

## 5. OverTheWire Bandit Walkthrough (Levels 6–10)

### Bandit Level 6 (`bandit6 -> bandit7`)
Search the root directory (`/`) for a file of size `33c` owned by user `bandit7` and group `bandit6`, adding `2>/dev/null` at the end.
* `1` means standard output (`stdout`), while `2` means errors (`stderr`) like `Permission denied`.
* `2>/dev/null` sends error messages to `/dev/null`, where they are automatically discarded.

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```


**Another way:**
```bash
find / -user bandit7 -group bandit6 -size 33c 2>&1 | grep -v "Permission denied"
```
* `2>&1` merges errors with standard output and pipes them into `grep -v`, which filters out the quoted messages.

---

### Bandit Level 7 (`bandit7 -> bandit8`)
Find the password next to the word `millionth` inside `data.txt`:

```bash
grep "millionth" data.txt
```


---

### Bandit Level 8 (`bandit8 -> bandit9`)
Combine `sort` and `uniq` to find the single unique line:
* `sort data.txt` — Puts all identical lines next to each other.
* `uniq` — Used for reporting, filtering, or counting adjacent duplicate lines in a sorted text file (almost always paired with `sort`).
  * `uniq -u` — Prints lines that appear only once.
  * `uniq -d` — Prints lines that appear more than once.

```bash
sort data.txt | uniq -u
```


---

### Bandit Level 9 (`bandit9 -> bandit10`)
The file contains binary data alongside the password, so regular `cat` or `grep` won't work cleanly. Use `strings`:
* `strings data.txt` — Scans the binary file and extracts only sequences of printable, human-readable characters while ignoring raw binary gibberish.

```bash
strings data.txt | grep "==="
```


---

### Bandit Level 10 (`bandit10 -> bandit11`)
The password is encoded in Base64, so decode it using `-d`:

```bash
base64 -d data.txt
```


