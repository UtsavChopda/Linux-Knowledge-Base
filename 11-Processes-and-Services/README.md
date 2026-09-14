# ⚙️ 11 — Processes and Services

> **Goal:** Understand what is actually running on a Linux system, how processes use CPU/memory/resources, how services are managed, and how a SOC analyst can investigate suspicious processes safely.

---

## 🎯 Learning Objectives

By the end of this module, you should be able to:

- Explain the difference between a program and a process
- Understand PID, PPID and process trees
- Read common Linux process states
- Inspect processes with `ps`, `top`, `pgrep` and `pidof`
- Understand foreground/background jobs
- Use signals safely with `kill`
- Investigate `/proc/<PID>`
- Understand CPU, memory and load average
- Use `nice` and `renice`
- Understand services and daemons
- Manage services with `systemctl`
- Read service logs with `journalctl`
- Identify listening services and their owning processes
- Investigate suspicious processes from a defensive/SOC perspective
- Troubleshoot services that fail to start

---

# 1. What Is a Process?

A **process** is a running instance of a program.

Think of it like this:

```text
PROGRAM
  │
  │ executed
  ▼
PROCESS
  │
  ├── PID
  ├── Memory
  ├── CPU time
  ├── Open files
  ├── Environment
  ├── Network connections
  └── Security identity
```

A program is usually stored on disk. A process exists while that program is running.

### Example

When you have `/usr/bin/python3` on disk, that is a program.

When you run:

```bash
python3 script.py
```

Linux creates a process to execute it.

You can have multiple processes running the same program:

```text
/usr/bin/python3
      │
      ├── PID 2101 → script A
      ├── PID 2135 → script B
      └── PID 2190 → script C
```

---

# 2. Why Do Processes Matter?

Almost everything your Linux system does involves processes.

Examples:

| Activity | Example process |
|---|---|
| Web server | nginx / apache |
| SSH server | sshd |
| Database | postgres / mysqld |
| Shell | bash / zsh |
| Desktop | graphical processes |
| Logging | journald / rsyslog |
| Security agent | endpoint/security daemon |

For a Linux administrator, processes answer:

> **What is running right now?**

For a SOC analyst, processes answer:

> **What is running, who started it, where did it come from, and is it expected?**

---

# 3. PID — Process ID

Every running process has a **PID (Process ID)**.

Example:

```bash
ps
```

Possible output:

```text
    PID TTY          TIME CMD
   4210 pts/0    00:00:00 bash
   4382 pts/0    00:00:00 ps
```

Here:

```text
4210 = bash PID
4382 = ps PID
```

PIDs are useful because most process-management commands operate on them.

---

# 4. PID 1

Linux starts an initial userspace process with **PID 1**.

On many modern mainstream distributions, PID 1 is `systemd`.

Check it:

```bash
ps -p 1 -f
```

Or:

```bash
ps -p 1 -o pid,ppid,comm,args
```

You may see:

```text
PID  PPID COMMAND  COMMAND
1    0    systemd  /sbin/init
```

Not every Linux environment uses systemd, so always inspect the actual system rather than assuming.

---

# 5. PPID — Parent Process ID

A process can have a **parent process**.

Its parent is identified by the **PPID**.

```text
Parent Process
      │
      ├── Child Process A
      ├── Child Process B
      └── Child Process C
```

Inspect PID and PPID:

```bash
ps -eo pid,ppid,user,comm
```

Example:

```text
PID   PPID USER     COMMAND
1000  900  utsav    bash
1050  1000 utsav    python3
```

This means:

```text
bash (1000)
   │
   └── python3 (1050)
```

### Why PPID matters in security

A suspicious process may look harmless by itself.

Its **parent process** can reveal how it was launched.

For example:

```text
systemd
   └── expected-service
```

may look normal, while an unexpected chain such as:

```text
interactive shell
   └── unusual executable
       └── network connection
```

may deserve investigation.

Parent-child relationships are clues, not proof of malicious activity.

---

# 6. Process Tree

A process tree shows process relationships.

Useful command:

```bash
pstree -p
```

Example:

```text
systemd(1)
 ├─sshd(700)
 │  └─sshd(2100)
 │     └─bash(2110)
 │        └─python3(2150)
 └─cron(500)
```

Another useful option:

```bash
pstree -p -a
```

If `pstree` is unavailable, use:

```bash
ps -ef --forest
```

---

# 7. Process States

Linux processes can be in different states.

Common `ps` state codes include:

| State | Meaning |
|---|---|
| `R` | Running or runnable |
| `S` | Interruptible sleep |
| `D` | Uninterruptible sleep, often I/O-related |
| `T` | Stopped or traced |
| `Z` | Zombie |

Check states:

```bash
ps -eo pid,ppid,user,stat,comm
```

Example:

```text
PID   PPID STAT COMMAND
1001  900  S    sshd
1200  900  R    python3
1400  900  D    worker
1500  900  Z    oldchild
```

`STAT` may contain additional modifiers beyond the main state letter.

---

# 8. Zombie Processes 👻

A zombie is a process that has already exited, but its parent has not yet collected its exit status.

Conceptually:

```text
Child
  │
  ├── exits
  │
  ▼
Zombie
  │
  └── waiting for parent to collect status
```

Find zombies:

```bash
ps -eo pid,ppid,stat,comm | awk '$3 ~ /^Z/ {print}'
```

A zombie is **not a normal running process** and generally does not consume the same CPU/memory resources as an active process.

A large or persistent number of zombies can indicate a problem in a parent application.

---

# 9. Orphan Processes

If a parent process exits while a child is still running, the child becomes an **orphan** and is reparented by the operating system.

Do not automatically treat orphan processes as suspicious.

Process relationships must be interpreted in context.

---

# 10. Viewing Processes with `ps`

`ps` is one of the most important process-inspection commands.

### Current shell processes

```bash
ps
```

### All processes in BSD-style format

```bash
ps aux
```

### Full process listing

```bash
ps -ef
```

### Custom columns

```bash
ps -eo pid,ppid,user,stat,%cpu,%mem,etime,cmd
```

### Sort by CPU

```bash
ps -eo pid,ppid,user,%cpu,%mem,etime,cmd --sort=-%cpu
```

### Sort by memory

```bash
ps -eo pid,ppid,user,%cpu,%mem,etime,cmd --sort=-%mem
```

---

# 11. Understanding `ps aux`

Example:

```text
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
```

Important fields:

| Field | Meaning |
|---|---|
| USER | Process owner |
| PID | Process ID |
| %CPU | CPU usage |
| %MEM | Memory usage |
| VSZ | Virtual memory size |
| RSS | Resident memory in RAM |
| TTY | Controlling terminal |
| STAT | Process state |
| START | Start time |
| TIME | CPU time consumed |
| COMMAND | Command/executable |

---

# 12. Finding a Specific Process

## `pgrep`

```bash
pgrep ssh
```

Show PID and command:

```bash
pgrep -a ssh
```

Search for a user:

```bash
pgrep -u utsav
```

## `pidof`

```bash
pidof sshd
```

These are useful when you already know what process you are looking for.

---

# 13. Real-Time Monitoring with `top`

Run:

```bash
top
```

`top` provides continuously updated information about:

- CPU usage
- memory usage
- load average
- processes
- process IDs
- process states

Useful interactive keys include:

```text
P → sort by CPU
M → sort by memory
k → request a signal for a process
q → quit
```

Always verify the selected PID before sending a signal.

---

# 14. `htop`

`htop` provides a more user-friendly interactive process viewer on systems where it is installed.

Run:

```bash
htop
```

If it is not installed, use `top` or install it through your distribution's package manager.

For a knowledge base, learn both:

```text
top  → commonly available
htop → easier interactive experience
```

---

# 15. Foreground and Background Processes

Normally, a command runs in the foreground.

```bash
sleep 60
```

Your shell waits for it to finish.

You can start it in the background:

```bash
sleep 60 &
```

The shell returns control to you.

---

# 16. Shell Job Control

Job control operates on jobs started by your current shell.

### List jobs

```bash
jobs
```

### Suspend a foreground job

Press:

```text
Ctrl + Z
```

### Continue in background

```bash
bg
```

### Bring back to foreground

```bash
fg
```

Example:

```bash
sleep 300
```

Press `Ctrl + Z`, then:

```bash
bg
```

Check:

```bash
jobs
```

---

# 17. `nohup`

`nohup` helps a command continue after the shell receives a hangup.

Example:

```bash
nohup ./backup.sh > backup.log 2>&1 &
```

This is useful for some long-running shell tasks.

However, `nohup` is **not a replacement for proper service management**. Long-running production services are generally better managed by a service manager such as systemd where available.

---

# 18. Signals

A signal is a notification sent to a process.

Common signals:

| Signal | Number | Typical meaning |
|---|---:|---|
| `SIGTERM` | 15 | Request graceful termination |
| `SIGKILL` | 9 | Force termination |
| `SIGINT` | 2 | Interrupt, often from `Ctrl+C` |
| `SIGHUP` | 1 | Hangup; often used to request reload depending on application |
| `SIGSTOP` | 19 | Stop process; cannot be caught |
| `SIGCONT` | 18 | Continue stopped process |

List signals:

```bash
kill -l
```

---

# 19. `kill` Does Not Mean “Immediately Kill”

By default:

```bash
kill PID
```

sends `SIGTERM`.

That means:

> Please terminate gracefully.

A program can handle `SIGTERM` and perform cleanup.

### Force termination

```bash
kill -9 PID
```

This sends `SIGKILL`.

`SIGKILL` cannot be caught or handled by the target process, so it should generally be a **last resort**.

It can prevent graceful cleanup and may make troubleshooting or evidence preservation harder.

---

# 20. Other Process-Signaling Commands

## `pkill`

Can signal processes based on a matching name or other criteria.

Example:

```bash
pkill -TERM -u utsav -x example-process
```

## `killall`

Can signal processes by name.

```bash
killall example-process
```

⚠️ Be careful: these commands can affect **multiple processes**.

Before using them:

```bash
pgrep -a example-process
```

Verify exactly what will be affected.

---

# 21. Process Ownership

Processes run with a user identity.

Check:

```bash
ps -eo pid,user,group,comm
```

Example:

```text
PID   USER     GROUP    COMMAND
1200  root     root     sshd
2400  utsav    utsav    bash
2500  utsav    utsav    python3
```

### Security importance

A process running as `root` has substantially greater access than a normal user process.

Therefore ask:

```text
Who owns it?
What does it execute?
Why does it need that privilege?
```

---

# 22. `/proc` — The Process Window

Linux provides process information through the `/proc` pseudo-filesystem.

A process often has a directory like:

```text
/proc/1234/
```

Useful entries include:

```text
/proc/1234/cmdline
/proc/1234/status
/proc/1234/exe
/proc/1234/cwd
/proc/1234/root
/proc/1234/fd/
/proc/1234/environ
```

The exact information available can depend on permissions and system configuration.

---

# 23. Investigating a PID

Suppose PID is `1234`.

### Command line

```bash
tr '\0' ' ' < /proc/1234/cmdline
```

### Process status

```bash
cat /proc/1234/status
```

### Executable path

```bash
readlink -f /proc/1234/exe
```

### Current working directory

```bash
readlink -f /proc/1234/cwd
```

### Open file descriptors

```bash
ls -l /proc/1234/fd
```

### Environment

```bash
tr '\0' '\n' < /proc/1234/environ
```

⚠️ Environment variables can contain secrets or sensitive values. Do not unnecessarily copy or expose them.

---

# 24. A Useful `/proc` Investigation Flow

```text
PID
 │
 ├── /proc/PID/status
 │       ↓
 │   owner/state
 │
 ├── /proc/PID/exe
 │       ↓
 │   executable path
 │
 ├── /proc/PID/cmdline
 │       ↓
 │   launch arguments
 │
 ├── /proc/PID/cwd
 │       ↓
 │   working directory
 │
 └── /proc/PID/fd
         ↓
     open resources
```

This is extremely useful for Linux troubleshooting and SOC investigations.

---

# 25. CPU Usage

A process may consume CPU because it is performing work.

Find high CPU processes:

```bash
ps -eo pid,ppid,user,%cpu,%mem,cmd --sort=-%cpu | head
```

Or:

```bash
top
```

High CPU is **not automatically malicious**.

Examples of legitimate high CPU workloads:

- compilation
- video processing
- encryption
- scientific computation
- backups
- data processing

Always investigate context.

---

# 26. Memory Usage

Find high-memory processes:

```bash
ps -eo pid,ppid,user,%cpu,%mem,cmd --sort=-%mem | head
```

Important concepts:

```text
Virtual Memory
      ↓
Resident Memory (RSS)
      ↓
Actual pages currently resident in RAM
```

Do not interpret `%MEM` alone as proof of a memory leak.

---

# 27. Load Average

You may see something like:

```text
load average: 1.20, 0.90, 0.70
```

These represent approximately:

```text
1 minute   5 minutes   15 minutes
```

Load average is **not simply CPU percentage**.

It reflects runnable and certain uninterruptible tasks, so it must be interpreted together with:

- number of CPU cores
- CPU utilization
- I/O activity
- process states
- workload type

Example:

```text
4 CPU cores
load average: 1.0
```

is very different from:

```text
1 CPU core
load average: 1.0
```

---

# 28. Nice Values

Linux can assign a **nice value** that influences CPU scheduling priority.

Common range:

```text
-20  → higher scheduling priority
  0  → default
+19  → lower scheduling priority
```

Check:

```bash
ps -eo pid,ni,pri,comm
```

Start with a nice value:

```bash
nice -n 10 command
```

Change an existing process:

```bash
renice 10 -p PID
```

Permissions and capabilities affect which priority changes a user can make.

---

# 29. What Is a Service?

A **service** is a program managed to provide a persistent system function.

Examples:

```text
SSH server
Web server
Database
Logging service
Network service
Security agent
```

A **daemon** is a background process that performs a service or system function.

The terms overlap in everyday Linux administration, but conceptually:

```text
Service = managed system function
Daemon  = background process implementing a function
```

---

# 30. Service vs Process

A service is not exactly the same thing as a process.

```text
SERVICE
  │
  ├── configuration
  ├── startup policy
  ├── dependencies
  ├── logs
  └── one or more processes
```

A service manager can start, stop, restart and monitor the processes associated with a service.

---

# 31. systemd

Many modern Linux distributions use **systemd** as their init and service-management system.

Conceptually:

```text
systemd (PID 1)
      │
      ├── ssh.service
      ├── cron.service
      ├── network service
      ├── logging service
      └── other units
```

Systemd manages more than services; it works with units such as services, sockets, timers, mounts and targets.

---

# 32. `systemctl`

`systemctl` is the main command-line interface for systemd management.

Check service status:

```bash
systemctl status ssh
```

On some distributions the service may be named differently, for example `sshd`.

Start:

```bash
sudo systemctl start ssh
```

Stop:

```bash
sudo systemctl stop ssh
```

Restart:

```bash
sudo systemctl restart ssh
```

Reload configuration when supported by the service:

```bash
sudo systemctl reload ssh
```

---

# 33. Start vs Enable

This distinction is extremely important.

### Start

```bash
sudo systemctl start ssh
```

Starts the service **now**.

### Enable

```bash
sudo systemctl enable ssh
```

Configures the service to start automatically according to its boot configuration.

Therefore:

```text
start  → now
 enable → future boots
```

You can combine them:

```bash
sudo systemctl enable --now ssh
```

---

# 34. Disable a Service

```bash
sudo systemctl disable ssh
```

This changes the automatic-start configuration; it does not necessarily stop a currently running service.

To stop it too:

```bash
sudo systemctl disable --now ssh
```

Use this carefully on production or remote systems.

---

# 35. Masking a Service

Masking is stronger than disabling.

```bash
sudo systemctl mask example.service
```

A masked unit is normally linked to `/dev/null`, preventing normal activation.

Reverse it with:

```bash
sudo systemctl unmask example.service
```

Think:

```text
disable → don't start automatically
mask    → prevent normal activation
```

---

# 36. Check Service State

```bash
systemctl is-active ssh
```

Check whether it is enabled:

```bash
systemctl is-enabled ssh
```

List running services:

```bash
systemctl list-units --type=service --state=running
```

List installed/enabled unit files:

```bash
systemctl list-unit-files --type=service
```

---

# 37. Service Logs with `journalctl`

View logs for a service:

```bash
sudo journalctl -u ssh
```

Recent logs:

```bash
sudo journalctl -u ssh --since "1 hour ago"
```

Follow logs live:

```bash
sudo journalctl -u ssh -f
```

Boot-specific logs can also be queried:

```bash
sudo journalctl -b
```

The exact logs available depend on system configuration and retention settings.

---

# 38. `daemon-reload` vs Restart

If you modify a systemd unit file, use:

```bash
sudo systemctl daemon-reload
```

This tells systemd to reload unit-file definitions.

It does **not** restart the service.

Then, if appropriate:

```bash
sudo systemctl restart example.service
```

Remember:

```text
daemon-reload → reread unit definitions
restart       → restart the service process
```

---

# 39. Service Dependencies

Services may depend on other units.

Inspect dependencies:

```bash
systemctl list-dependencies ssh.service
```

This helps explain why a service may fail when another component is unavailable.

---

# 40. Find Listening Services

One of the most important commands for Linux networking and security:

```bash
ss -lntup
```

Common flags:

```text
-l → listening
-n → numeric addresses/ports
-t → TCP
-u → UDP
-p → process information when permitted
```

Example conceptually:

```text
Local Address       Port   Process
0.0.0.0             22     sshd
0.0.0.0             80     nginx
127.0.0.1           5432   postgres
```

A listening port represents a potential network exposure and should be compared with the expected system baseline.

---

# 41. Process ↔ Port Investigation

When a port is unexpected:

```text
Unexpected Port
      ↓
ss -lntup
      ↓
Identify PID/process
      ↓
ps -fp PID
      ↓
Identify owner
      ↓
/proc/PID/exe
/proc/PID/cmdline
      ↓
Identify origin/service
      ↓
Review logs + baseline
```

This workflow is highly relevant to SOC investigations.

---

# 42. Process Investigation — Defensive Workflow

If you find an unfamiliar process on an authorized system:

### Step 1 — Identify it

```bash
ps -fp PID
```

### Step 2 — Identify owner

```bash
ps -o pid,ppid,user,group,stat,lstart,cmd -p PID
```

### Step 3 — Identify executable

```bash
readlink -f /proc/PID/exe
```

### Step 4 — Inspect command line

```bash
tr '\0' ' ' < /proc/PID/cmdline
```

### Step 5 — Inspect parent

```bash
ps -fp $(ps -o ppid= -p PID)
```

### Step 6 — Inspect open resources

```bash
ls -l /proc/PID/fd
```

### Step 7 — Check network exposure

```bash
ss -lntup
```

### Step 8 — Check service origin

```bash
systemctl status example.service
```

### Step 9 — Review logs

```bash
journalctl --since "1 hour ago"
```

### Step 10 — Compare against baseline

Ask:

```text
Is it expected?
Is the path normal?
Is the owner appropriate?
Is the parent expected?
Is the command line normal?
Is the network activity expected?
```

---

# 43. Do NOT Immediately Kill a Suspicious Process

During a real security investigation, immediately running:

```bash
kill -9 PID
```

may destroy useful volatile evidence and change the system state.

A safer defensive principle is:

```text
OBSERVE
  ↓
IDENTIFY
  ↓
DOCUMENT
  ↓
COLLECT ACCORDING TO IR PROCEDURE
  ↓
CONTAIN / REMEDIATE
```

The correct response depends on the incident-response plan, system criticality and authorization.

In a training lab, you can safely practice process termination on processes you created yourself.

---

# 44. Service Troubleshooting Workflow

When a service is not working:

```text
Service failure
      ↓
1. systemctl status
      ↓
2. journalctl -u
      ↓
3. Check configuration
      ↓
4. Check dependencies
      ↓
5. Check permissions
      ↓
6. Check resources
      ↓
7. Check listening ports
      ↓
8. Restart/reload if appropriate
      ↓
9. Verify
```

Do not jump directly to reinstalling the package.

The error message usually gives you the next clue.

---

# 45. Common Service Failure Causes

| Cause | Example clue |
|---|---|
| Bad configuration | syntax/config error |
| Port already in use | address already in use |
| Permission problem | permission denied |
| Missing dependency | dependency failed |
| Missing file | no such file |
| Resource exhaustion | out of memory |
| Wrong path | executable not found |
| Certificate issue | TLS/certificate error |
| User/account issue | authentication/identity failure |

---

# 🧪 Lab 1 — Process Basics

## Objective

Learn to identify processes, PIDs and parents.

### Tasks

1. List your current shell processes.
2. Display all processes.
3. Find the PID of your shell.
4. Find its PPID.
5. Display the process tree.

### Commands

```bash
ps
ps -ef
ps -p $$ -f
ps -o pid,ppid,user,stat,cmd -p $$
pstree -p
```

### Verification

You should be able to explain:

```text
Your shell PID
      ↓
Parent PID
      ↓
Parent process name
```

---

# 🧪 Lab 2 — Signals Safely

## Objective

Understand `SIGTERM`, `SIGINT` and `SIGKILL` using a process you created.

Start:

```bash
sleep 300 &
```

Find it:

```bash
pgrep -a sleep
```

Send SIGTERM:

```bash
kill PID
```

Verify:

```bash
pgrep -a sleep
```

### Challenge

Repeat with:

```bash
sleep 300 &
kill -STOP PID
kill -CONT PID
```

Observe the process state with:

```bash
ps -o pid,stat,cmd -p PID
```

---

# 🧪 Lab 3 — Resource Investigation

## Objective

Find the highest CPU and memory consumers.

### Tasks

```bash
ps -eo pid,ppid,user,%cpu,%mem,etime,cmd --sort=-%cpu | head -10
```

Then:

```bash
ps -eo pid,ppid,user,%cpu,%mem,etime,cmd --sort=-%mem | head -10
```

Run:

```bash
top
```

### Questions

- Which process uses the most CPU?
- Which uses the most memory?
- Is that behavior expected?
- How long has the process been running?

---

# 🧪 Lab 4 — Service Investigation

Choose a service available in your Linux lab VM.

Examples:

```bash
ssh
cron
systemd-journald
```

Check:

```bash
systemctl status SERVICE
systemctl is-active SERVICE
systemctl is-enabled SERVICE
```

Then:

```bash
sudo journalctl -u SERVICE --since "30 minutes ago"
```

### Goal

Explain:

```text
Service name
Current state
Enabled/disabled
Recent log activity
Main process if applicable
```

---

# 🧪 Lab 5 — SOC Scenario: Unknown Process

> **Scenario:** Your authorized Linux lab machine shows an unfamiliar process consuming CPU.

Do not immediately terminate it.

### Investigation checklist

```bash
ps -fp PID
ps -o pid,ppid,user,group,stat,lstart,cmd -p PID
readlink -f /proc/PID/exe
tr '\0' ' ' < /proc/PID/cmdline
ps -fp $(ps -o ppid= -p PID)
ls -l /proc/PID/fd
ss -lntup
```

### Questions

1. Who owns the process?
2. What executable is running?
3. What arguments were used?
4. Who is its parent?
5. Is it associated with a known service?
6. Does it have network exposure?
7. Is its behavior expected on this machine?

### Expected learning

You are practicing **process triage**, not simply process termination.

---

# 🛠️ Mini Project — Linux Process Monitor

Build a small Bash script that reports important process information.

Suggested output:

```text
====================================
       LINUX PROCESS MONITOR
====================================
Hostname: lab-linux
Date: ...

Top CPU Processes
-----------------
PID   USER   CPU   COMMAND
...

Top Memory Processes
--------------------
PID   USER   MEM   COMMAND
...

Running Services
----------------
...

Listening Ports
---------------
...
```

Useful commands:

```bash
hostname
ps
systemctl
ss
uptime
```

### Project extensions

- Add timestamped output
- Save reports to a log file
- Add thresholds
- Highlight processes above a CPU threshold
- Compare against a baseline
- Report unexpected listening ports
- Generate a simple incident-triage report

This project is directly useful for Linux administration and SOC portfolio work.

---

# 🔐 Linux + SOC Cheat Sheet

### Process discovery

```bash
ps aux
ps -ef
pgrep -a NAME
pidof NAME
pstree -p
```

### Resource investigation

```bash
top
ps -eo pid,%cpu,%mem,cmd --sort=-%cpu
ps -eo pid,%cpu,%mem,cmd --sort=-%mem
uptime
```

### Process details

```bash
ps -fp PID
cat /proc/PID/status
readlink -f /proc/PID/exe
tr '\0' ' ' < /proc/PID/cmdline
ls -l /proc/PID/fd
```

### Signals

```bash
kill PID
kill -TERM PID
kill -STOP PID
kill -CONT PID
kill -9 PID
```

### Services

```bash
systemctl status SERVICE
systemctl start SERVICE
systemctl stop SERVICE
systemctl restart SERVICE
systemctl reload SERVICE
systemctl enable SERVICE
systemctl disable SERVICE
systemctl mask SERVICE
systemctl is-active SERVICE
systemctl is-enabled SERVICE
```

### Logs

```bash
journalctl -u SERVICE
journalctl -u SERVICE --since "1 hour ago"
journalctl -u SERVICE -f
```

### Network/process correlation

```bash
ss -lntup
```

---

# 🎯 Interview Questions

### Beginner

1. What is a process?
2. What is the difference between a program and a process?
3. What is a PID?
4. What is a PPID?
5. What is PID 1?
6. What is a zombie process?
7. What is an orphan process?

### Intermediate

8. What does `ps aux` show?
9. Difference between `ps aux` and `ps -ef`?
10. What is `top` used for?
11. What is the difference between foreground and background processes?
12. What does `kill` do by default?
13. Difference between SIGTERM and SIGKILL?
14. What does `nice` do?
15. What is `/proc`?

### Services

16. What is a daemon?
17. What is systemd?
18. Difference between `systemctl start` and `systemctl enable`?
19. Difference between `disable` and `mask`?
20. What does `systemctl daemon-reload` do?
21. How do you check service logs?
22. How would you troubleshoot a service that failed to start?

### SOC / Security

23. How would you investigate an unknown process?
24. Why is process ownership important?
25. Why inspect `/proc/PID/exe`?
26. How do you identify the parent process?
27. How can you correlate a process with a listening port?
28. Why should you avoid immediately killing a suspicious process during an investigation?
29. What indicators would make a process suspicious?
30. How would you distinguish an unusual process from a confirmed malicious process?

---

# 🧠 Key Mental Model

Remember this workflow:

```text
                 LINUX SYSTEM
                      │
          ┌───────────┴───────────┐
          │                       │
      PROCESSES               SERVICES
          │                       │
     PID / PPID              systemd
          │                       │
      /proc/PID             systemctl
          │                       │
     CPU / Memory              journalctl
          │                       │
          └───────────┬───────────┘
                      │
                 INVESTIGATION
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       PROCESS       OWNER      NETWORK
          │           │           │
       EXECUTABLE   PRIVILEGE   PORT/SOCKET
          │           │           │
          └───────────┴───────────┘
                      ↓
                   BASELINE
                      ↓
                 DECISION
```

---

# ✅ Module Checklist

- [ ] I understand program vs process
- [ ] I understand PID and PPID
- [ ] I can read a process tree
- [ ] I know common process states
- [ ] I understand zombie vs orphan processes
- [ ] I can use `ps`
- [ ] I can use `top`
- [ ] I can find processes with `pgrep`
- [ ] I understand foreground/background jobs
- [ ] I understand Linux signals
- [ ] I know why SIGKILL should be a last resort
- [ ] I can investigate `/proc/PID`
- [ ] I understand CPU, memory and load average
- [ ] I understand nice values
- [ ] I understand services and daemons
- [ ] I can use `systemctl`
- [ ] I understand start vs enable
- [ ] I understand disable vs mask
- [ ] I can read service logs with `journalctl`
- [ ] I can inspect listening ports with `ss`
- [ ] I can perform basic process triage
- [ ] I can troubleshoot a failed service
- [ ] I completed the process labs
- [ ] I can explain process investigation from a SOC perspective

---

# 🚀 What Comes Next?

After understanding processes and services, the next major administration skill is **software and package management**.

➡️ **Next Module: `12-Package-Management`**

You will learn how Linux installs, updates, verifies and removes software, how repositories work, how to troubleshoot package failures, and how package management connects to system security and SOC investigations.