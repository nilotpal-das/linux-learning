# Day 1: Linux Filesystem & Navigation (18.11.2026)

I am learning the Linux terminal. In Linux, `~/` means the home directory (`/home/username`). When we open the terminal, it starts in this directory.

---

## Core Linux Directories

### Critical System & Boot Directories
- **`/`**: Root directory. The top-most directory; every file and directory starts here.
- **`/bin`**: Essential command binaries needed for single-user mode and basic system repair (`ls`, `cp`, `rm`, etc.).
- **`/sbin`**: System binaries meant for administration by the root user (`fdisk`, `reboot`, `iptables`).
- **`/lib` & `/lib64`**: Essential shared libraries and kernel modules required by binaries in `/bin` and `/sbin`.
- **`/boot`**: Files required to boot the system (including bootloader and Linux kernel).
- **`/etc`**: System-wide configuration files and shell scripts for services and apps.

### Hardware, Kernel & Runtime Directories
- **`/dev`**: Physical and virtual hardware devices represented as files (disks, terminals, USB devices, etc.).
- **`/proc`**: Virtual filesystem providing a live window into the kernel and running processes.
- **`/var`**: Variable data that changes constantly (system logs, databases, mail).
- **`/run`**: Volatile runtime data (process IDs, sockets) generated since the last boot.

### Application & User Data
- **`/usr`**: Majority of user-installed applications, utilities, and read-only system resources.
- **`/home`**: Personal directories for regular users (`/home/username`).
- **`/root`**: Dedicated home directory for the system administrator (`root` user).
- **`/opt`**: Optional, self-contained third-party software packages.
- **`/srv`**: Site-specific data served by the system (web or FTP server files).

### Mounts & Temporary Storage
- **`/media`**: Automatic mount points for removable media like USB drives and DVDs.
- **`/mnt`**: Blank directory reserved for system administrators to manually mount temporary filesystems.
- **`/tmp`**: Scratchpad for temporary files created by apps and users (typically wiped on every reboot).

---

## Basic Commands

Entering `man <command>` displays a detailed manual for that command.

### Navigation & Listing (`ls`, `cd`, `pwd`)
- `ls` — Lists non-hidden files and directories in the current working directory.
- `ls -a` — Lists all files and directories, including hidden ones starting with `.` (e.g., `.games`).
- `ls -la` — Lists all files with permissions, sizes, and modification dates.
- `ls -lh` — Same as `-la`, with human-readable file sizes (`KB`, `MB`, `GB`).
- `pwd` — Displays the current working directory path.
- `cd <path>` — Changes the working directory:
  ```bash
  cd ~/Documents   # Move to Documents inside home directory
  cd ..            # Move up one level
  cd ~             # Jump to personal home directory (or just type `cd`)
  ```

### Creating, Removing & Copying (`mkdir`, `rm`, `rmdir`, `cp`)
```bash
mkdir games                  # Create a directory named 'games'
mkdir -p games/racing        # Create parent and nested directories together

rmdir games                  # Delete 'games' ONLY if it is empty
rm -r games                  # Recursively delete 'games' and everything inside it

cp notes.txt notes2.txt                  # Copy a file in the current directory
cp ~/notes.txt ~/Documents/notes2.txt    # Copy a file to another directory
cp -r games pictures                     # Copy 'games' inside 'pictures' (pictures/games)
cp -r games/. pictures/                  # Merge contents (including hidden files) into 'pictures'
```

### Finding & Searching (`find`, `grep`)
```bash
find /etc -name "*.conf"     # Search inside /etc for files ending in .conf
find ~ -type d               # Find only directories inside home directory
find ~/Documents -type f     # Find only files inside Documents directory

grep "error" app.log         # Print lines containing "error"
grep -i "error" app.log      # Case-insensitive search (Error, ERROR, etc.)
grep -r "TODO" ./src/        # Recursively search all files in ./src/ for "TODO"
grep -iv "debug" app.log     # Print lines that do NOT contain "debug"
```

### Viewing & Combining Files (`cat`, `less`, `head`, `tail`)
```bash
cat file.txt                              # Display entire file contents
cat file1.txt file2.txt                   # Display file1 followed by file2
cat file1.txt file2.txt > combined.txt    # Combine files and save into a new file
cat > newfile.txt                         # Create a file from terminal input (Ctrl + D to save/exit)
cat file1.txt >> existing.txt             # Append file1 to the end of existing.txt without overwriting

less file.txt    # Read large files page by page
head file.txt    # View the first 10 lines
tail file.txt    # View the last 10 lines
```

### Common Command Flags
Flags modify how commands behave and can be combined (e.g., `ls -la` or `ls -a -l`):
- `-h` — Human-readable sizes (`ls`)
- `-r` — Recursive operation (`cp`, `rm`, `grep`)
- `-a` — All files, including hidden ones (`ls`, `cp`)

