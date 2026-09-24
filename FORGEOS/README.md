#  Linux Kernel Monitor

**Operating Systems and Systems Programming (25CS2104E) — 2026–27, Term-I**
Koneru Lakshmaiah Education Foundation (KL University), Hyderabad

A command-line system monitoring tool written in **C** that reads information directly from the Linux **`/proc` virtual filesystem** and shows CPU usage, memory utilization, system uptime, load average and running processes in one place.

---

##  Team Details

| Section | Team | Project Title |
|---------|------|---------------|
| 8 | 13 | Kernel Monitor |

| Roll Number | Name | Responsibility |
|-------------|------|----------------|
| 2520030531 | D. Venya Sri | CPU & System Monitoring (CPU usage, CPU info, kernel info, uptime, load average) |
| 2520030527 | K. Vansika Reddy | Memory Monitoring (total, used, free, available memory and usage %) |
| 2520030212 | N. Rishika Chowdary | Process Monitoring (PID, process name, process state) |
| 2520030060 | J. Hansika | Integration & Testing (menu/UI, live monitoring, testing, debugging, documentation) |

---

## Overview

Linux keeps a lot of information about system resources and processes, but it is spread across different commands (`top`, `free`, `uptime`, `ps`, `cat /proc/...`). This makes it hard, especially for beginners, to see the overall system status at once.

**Kernel Monitor** solves this by collecting the important details in a single, easy-to-read command-line dashboard. It also shows how a **user-space program talks to the kernel** through the `/proc` interface using plain file I/O system calls.

---

##  Features

-  **CPU monitoring** – live CPU usage %, CPU model / cores, kernel version
-  **Memory monitoring** – total, used, free and available memory with utilization %
-  **System uptime** – time since the system booted
-  **Load average** – 1, 5 and 15 minute averages
-  **Process monitoring** – running processes with PID, name and state
-  **Live monitoring mode** – screen refreshes automatically at regular intervals (using POSIX threads)
-  **Menu-driven interface** – choose which statistics to view

---

##  Operating System Concepts Used

| OS Concept / API | Purpose in the Project |
|------------------|------------------------|
| User space vs Kernel space | The monitor runs in user space and reads data exported by the kernel |
| `/proc` virtual filesystem | Source of CPU, memory, uptime, load and process information |
| `open()` | Opens `/proc` files |
| `read()` | Reads the data from the opened files |
| `close()` | Closes the file descriptors after reading |
| File descriptors | Access to `/proc` files through Linux file I/O |
| Process management | Listing processes, PIDs, names and states |
| Memory management | Calculating total / used / available memory |
| CPU utilization | Calculating CPU usage from `/proc/stat` |
| POSIX threads (`pthread`) | Continuous background refresh in live mode |

---

## /proc Files Used

| File | Information |
|------|-------------|
| `/proc/stat` | CPU time counters (used to calculate CPU usage) |
| `/proc/cpuinfo` | CPU model name, cores, frequency |
| `/proc/meminfo` | `MemTotal`, `MemFree`, `MemAvailable`, buffers, cache |
| `/proc/uptime` | System uptime in seconds |
| `/proc/loadavg` | 1 / 5 / 15 minute load average |
| `/proc/version` | Kernel version information |
| `/proc/[pid]/stat` and `/proc/[pid]/status` | Process name, state and other details |

---

##  How It Works

1. **Collect** – open the relevant `/proc` file with `open()` and read it with `read()`.
2. **Process** – parse the text using C string functions and calculate values.
3. **Display** – print the formatted output in the terminal menu/dashboard.
4. **Refresh** – in live mode, a thread repeats steps 1–3 every few seconds.

### CPU Usage Calculation

The first line of `/proc/stat` gives cumulative CPU time in jiffies:

```
cpu  user nice system idle iowait irq softirq steal ...
```

Two samples are taken with a small delay in between:

```
idle_all  = idle + iowait
total     = user + nice + system + idle + iowait + irq + softirq + steal

CPU Usage (%) = (Δtotal − Δidle_all) / Δtotal × 100
```

### Memory Usage Calculation

```
Used Memory (KB)   = MemTotal − MemAvailable
Memory Usage (%)   = (Used Memory / MemTotal) × 100
```

### Process States

| Code | Meaning |
|------|---------|
| `R` | Running |
| `S` | Sleeping (interruptible) |
| `D` | Uninterruptible sleep (usually I/O) |
| `T` | Stopped |
| `Z` | Zombie |
| `I` | Idle kernel thread |

---

## 🛠️ Requirements

- Linux (Ubuntu recommended) — the `/proc` filesystem is required
- GCC compiler
- `make` (optional)
- POSIX threads library (included with glibc)

Install the tools on Ubuntu/Debian:

```bash
sudo apt update
sudo apt install build-essential
```

---

##  Build and Run

### 1. Clone the repository

```bash
git clone https://github.com/janumpallyhansika/Kernal-Krew-_OSSP.git
cd Kernal-Krew-_OSSP
```

### 2. Compile

```bash
gcc -Wall -Wextra -o kernel_monitor *.c -pthread
```

or, if a Makefile is present:

```bash
make
```

### 3. Run

```bash
./kernel_monitor
```

> No `sudo` is needed. Regular users can read the `/proc` files used here.

---

##  Usage

When the program starts, a menu is shown:

```
========================================
          LINUX KERNEL MONITOR
========================================
 1. CPU Information & Usage
 2. Memory Usage
 3. System Uptime
 4. Load Average
 5. Running Processes
 6. Kernel Information
 7. Live Monitoring
 0. Exit
========================================
Enter your choice:
```

Enter the option number to view that statistic. In **Live Monitoring** mode, the dashboard refreshes at regular intervals; press `Ctrl + C` (or the exit key shown on screen) to stop.

### Sample Output *(illustrative)*

```
---------------- SYSTEM SUMMARY ----------------
CPU Usage        : 12.4 %
Total Memory     : 7982 MB
Used Memory      : 3120 MB
Available Memory : 4862 MB
Memory Usage     : 39.1 %
Uptime           : 2 h 14 m 08 s
Load Average     : 0.42  0.51  0.47

  PID   NAME              STATE
  1     systemd           S (sleeping)
  842   sshd              S (sleeping)
  1523  bash              S (sleeping)
  2210  kernel_monitor    R (running)
------------------------------------------------
```

---

## Project Structure

> Adjust this section to match the files in your repository.

```
Kernal-Krew-_OSSP/
├── main.c          # Menu, user interface, integration
├── cpu.c / cpu.h   # CPU usage, CPU info, kernel info, uptime, load average
├── memory.c / .h   # Memory monitoring
├── process.c / .h  # Process listing (PID, name, state)
├── live.c / .h     # Live monitoring using pthreads
├── Makefile        # Build script (optional)
└── README.md
```

---

##  Testing and Validation

The output of Kernel Monitor was compared with standard Linux tools under different system loads (idle, and while running CPU/memory-heavy programs):

| Kernel Monitor | Compared With |
|----------------|---------------|
| CPU usage | `top`, `mpstat` |
| Memory usage | `free -m` |
| Uptime | `uptime` |
| Load average | `uptime`, `cat /proc/loadavg` |
| Process list | `ps aux`, `top` |

---

## Future Scope

- Sort processes by CPU or memory usage (top-N view)
- Per-process CPU and memory usage
- Disk and network statistics (`/proc/diskstats`, `/proc/net/dev`)
- Ability to kill a process from the menu using `kill()`
- Color-coded output and an `ncurses` based interface
- Logging statistics to a file for later analysis

---

## References

- `man 5 proc` — Linux `/proc` filesystem documentation
- `man 2 open`, `man 2 read`, `man 2 close`
- `man 7 pthreads`
- Linux kernel documentation: https://docs.kernel.org/filesystems/proc.html

---

##  Team

**Kernel Krew** — Section 8, Team 13
D. Venya Sri • K. Vansika Reddy • N. Rishika Chowdary • J. Hansika

---

*This project was developed as part of the Operating Systems and Systems Programming course (25CS2104E) at KL University.*
