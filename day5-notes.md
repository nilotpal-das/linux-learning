# Day 5 (22-11-26): Linux Process Management, `systemd` Services & Week 1 Capstone Lab

**Environment:** Arch Linux (Konsole, Zsh)  

---

## 1. Core Concepts: Processes, `systemd` & Daemons

* **Process & PID:** Every running program in Linux is a process identified by a unique **PID** (Process ID) and managed by the kernel.
* **`systemd` (PID 1):** The core system and service manager for most Linux distributions. It acts as the first user-space process started by the Linux kernel at boot and oversees all other system processes until shutdown.
* **Daemon:** A background program that runs continuously without user interaction or a graphical interface. Daemons start automatically at boot or on demand to quietly handle core system tasks, network requests, and scheduled jobs (e.g., `nginx`, `sshd`).

---

## 2. Inspecting Processes (`ps`, `top`, `htop`)

### `ps` (Static Process Snapshots)

| Command | Description |
| :--- | :--- |
| `ps aux` | Displays processes from all users with CPU and memory percentages. |
| `ps aux --sort=-%mem \| head -10` | Lists the top 10 memory-consuming processes. |
| `ps aux \| grep [n]ginx` | Finds details for a specific service (brackets filter out the `grep` process itself). |
| `ps auxf` | Prints an ASCII tree showing parent-child process relationships. |

### Common Process States (`STAT` Column)

* **`R` (Running):** Actively executing on CPU or sitting in the run queue.
* **`S` (Interruptible Sleep):** Waiting for an event (network I/O, timer, user input).
* **`D` (Uninterruptible Sleep):** Waiting on hardware/disk I/O; cannot be killed even with `kill -9` until I/O finishes.
* **`T` (Stopped):** Suspended/paused (e.g., via `Ctrl+Z`).
* **`Z` (Zombie / Defunct):** Process finished executing, but its parent hasn't read its exit code yet.

### `top` (Built-in Real-Time Monitor)
Provides a live dashboard split into a **top summary** (system uptime, load average, CPU, and RAM usage) and a **dynamic process table** sorted by CPU usage:

* **`P` / `M`:** Sort the process list by CPU usage (`Shift + P`) or Memory usage (`Shift + M`).
* **`k`:** Kill a process by entering its PID when prompted.
* **`r`:** Renice (change the scheduling priority of) a running process.
* **`q`:** Quit and return to the terminal.

### `htop` (Interactive Visual Process Viewer)
A modern, user-friendly upgrade to `top` (not installed by default; install via `sudo pacman -S htop`) that visualizes per-core CPU, RAM, and swap usage with color-coded bars:

* **`F3` / `F4`:** Search (`F3`) or filter (`F4`) processes by name in real time.
* **`F5`:** Toggle tree view to trace child processes back to their parent.
* **`F6`:** Choose any column (`CPU`, `MEM`, `USER`, `PID`) to sort by.
* **`F9`:** Kill the highlighted process and pick a specific signal (`SIGTERM`, `SIGKILL`, etc.).

---

## 3. Process Signals & Termination (`kill`, `kill -9`)

`kill` sends a signal to a PID rather than forcefully terminating it by default:

| Signal | Number | Command | Behavior |
| :--- | :--- | :--- | :--- |
| **`SIGTERM`** | `15` | `kill <PID>` | **Default.** Politely asks the process to shut down, close files, and release ports/locks cleanly. |
| **`SIGINT`** | `2` | `Ctrl+C` | Interrupt signal from keyboard for active foreground processes. |
| **`SIGHUP`** | `1` | `kill -1 <PID>` | Hangup; used by daemons like Nginx to reload configuration files without dropping connections. |
| **`SIGKILL`** | `9` | `kill -9 <PID>` | **Nuclear option.** Terminates the process immediately at the kernel level; cannot be caught or ignored, and can leave corrupted files or stale locks. |

---

## 4. CPU Priority (`nice` / `renice`) & Job Control

### Priority (`nice` & `renice`)
`nice` starts a **new** process with a custom CPU scheduling priority (`-20` = highest priority, `+19` = lowest priority, `0` = default), while `renice` changes the priority of an **already running** process:

* **Lower priority for a backup:** `nice -n 10 tar -czf backup.tar.gz /home/user/data`
* **Raise priority on launch (requires `sudo`):** `sudo nice -n -5 <command>`
* **Lower priority of a running process (PID 1234):** `renice -n 10 -p 1234`
* **Raise priority of a running process (requires `sudo`):** `sudo renice -n -10 -p 1234`
* **Adjust all processes for a user:** `sudo renice -n 5 -u username`

### Terminal Job Control

* **`<command> &`:** Launch a command immediately in the background.
* **`Ctrl+Z`:** Suspend/pause the active foreground job (`SIGTSTP`).
* **`jobs`:** List background and stopped jobs in the current terminal session.
* **`bg %1`:** Resume stopped job `%1` in the background.
* **`fg %1`:** Bring job `%1` back into the interactive foreground.

---

## 5. Service Management (`systemctl` & `journalctl`)

* `sudo systemctl start <service>` — Start a service right now (does not persist across reboots).
* `sudo systemctl stop <service>` — Stop a running service cleanly.
* `sudo systemctl restart <service>` — Full stop and start (drops active connections).
* `sudo systemctl reload <service>` — Gracefully reload config files (`SIGHUP`) with zero downtime.
* `sudo systemctl enable <service>` — Create boot symlink in `/etc/systemd/system/multi-user.target.wants/`.
* `sudo systemctl enable --now <service>` — Enable on boot **and** start immediately.
* `sudo systemctl status <service> --no-pager` — Check runtime state, Main PID, CGroup processes, and recent logs without opening a pager.
* `journalctl -u nginx -f` / `journalctl -xe` — Follow unit logs live or jump to latest logs with explanatory error context.

---

## 6. Week 1 Capstone Lab Summary

1. **Users & Shared Group Directory (`/srv/webops`):**
    * Created users `alice` and `bob`, created shared group `webops`, and added both users to `webops`.
    * Set ownership and permissions via `sudo chown root:webops /srv/webops` and **`sudo chmod 2770 /srv/webops`** (`rwxrws---`).
    * **Takeaway:** Using `2770` instead of `770` activates the **`setgid`** bit (`2`), ensuring new files created inside `/srv/webops` automatically inherit the `webops` group rather than the creator's primary private group, while `0` blocks non-`webops` users.
2. **Nginx Setup & Verification on Arch Linux:**
    * Installed Nginx (`sudo pacman -S nginx`), enabled/started it (`sudo systemctl enable --now nginx`), tested config syntax (`sudo nginx -t`), and verified HTTP responses (`curl -I http://localhost` -> `200 OK`).
    * **Takeaway:** In `ps aux | grep nginx`, the `master process` runs as `root` solely to bind to privileged port `80` (`< 1024`), while the `worker process` drops privileges to the unprivileged `http` user for security isolation.

