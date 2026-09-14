# 03 — Linux Architecture

> Understand what is happening underneath the commands you type.

## 🎯 Objectives

By the end of this module you should understand:

- Linux architecture at a high level.
- Hardware, kernel, system libraries, user space, shell, applications, and services.
- Kernel space vs user space.
- System calls.
- Processes and process IDs.
- File descriptors.
- `/proc` and `/sys` at a practical level.
- Why Linux security depends on understanding these layers.

---

# 🧠 The Big Picture

```text
┌─────────────────────────────────────────┐
│              USER SPACE                 │
│                                         │
│  Applications  Shells  Services  Tools  │
│        │          │        │            │
│        └──────────┼────────┘            │
│                   ▼                     │
│             System Calls                │
├─────────────────────────────────────────┤
│              KERNEL SPACE               │
│                                         │
│ Process Mgmt │ Memory │ Networking      │
│ Filesystems  │ Security │ Drivers       │
├─────────────────────────────────────────┤
│               HARDWARE                  │
│ CPU │ RAM │ Disk │ NIC │ Devices        │
└─────────────────────────────────────────┘
```

The key idea:

> Applications normally do not directly control hardware. They request operating-system services through the kernel.

---

# 🧱 Main Linux Layers

## 1. Hardware

The physical resources available to the operating system:

- CPU
- RAM
- Storage
- Network interface
- Keyboard
- Display
- USB devices
- Other peripherals

## 2. Kernel

The kernel is the core software responsible for managing system resources and providing controlled interfaces between applications and hardware.

Major responsibilities include:

```text
Kernel
 ├── Process management
 ├── Memory management
 ├── Filesystems
 ├── Networking
 ├── Device drivers
 ├── Scheduling
 └── Security mechanisms
```

## 3. System Libraries

Libraries provide reusable functionality that applications can use to interact with the operating system and other software components.

## 4. Shell

The shell is a command interpreter.

Examples:

- Bash
- Zsh
- Fish

The shell accepts commands and starts programs or performs shell operations.

## 5. Applications

Applications are programs running in user space, such as editors, browsers, monitoring tools, and security tools.

---

# 🔐 User Space vs Kernel Space

A simplified mental model:

```text
USER SPACE
───────────
Your commands
Applications
Shell
Services

      │
      │ system calls
      ▼
KERNEL SPACE
────────────
Kernel
Drivers
Memory management
Networking
Filesystem operations

      │
      ▼
HARDWARE
```

### Why separate them?

The separation helps protect the operating system and other processes from arbitrary direct access to privileged resources.

A normal application should not simply overwrite kernel memory or directly program every hardware device without operating-system controls.

---

# ☎️ System Calls

A system call is a controlled request from user-space software to the kernel.

Conceptual example:

```text
Program
  │
  │ "I need to read this file"
  ▼
system call
  │
  ▼
Kernel
  │
  ▼
Filesystem / Storage
  │
  ▼
Result
```

Common system-call concepts include:

- `open`
- `read`
- `write`
- `close`
- `fork`
- `execve`
- `socket`

You do not need to memorize system calls immediately. First understand the boundary they represent.

---

# ⚙️ Processes

A process is a running instance of a program.

```text
Program on disk
      │
      │ execute
      ▼
   PROCESS
      │
      ├── PID
      ├── Memory
      ├── Open files
      ├── Environment
      └── Permissions
```

View processes:

```bash
ps
ps aux
```

Interactive view:

```bash
top
```

If available, `htop` provides a more convenient interactive view.

---

# 🆔 PID

PID means **Process ID**.

Example:

```text
PID     USER      COMMAND
1234    utsav     bash
2345    utsav     python3
```

The PID lets you refer to a particular running process.

Useful commands:

```bash
ps -ef
pgrep bash
```

Later we will learn how PIDs connect to signals, parent/child processes, services, and incident investigation.

---

# 👨‍👦 Parent and Child Processes

Processes can create other processes.

```text
systemd
   │
   ├── sshd
   │     └── sshd
   │          └── bash
   │
   └── other-service
```

Useful command:

```bash
pstree
```

If `pstree` is not installed, use:

```bash
ps -ef --forest
```

This relationship becomes very useful when investigating suspicious processes.

---

# 📁 Everything Is a File — But Understand the Meaning

Linux exposes many resources through file-like interfaces.

Examples:

```text
Regular file     → document.txt
Directory        → /home/utsav
Device           → /dev/sda
Process info     → /proc/1234/
```

This does **not** mean every resource is literally stored on disk as an ordinary file. It means Linux provides a consistent file-oriented interface for many resources.

---

# 🔎 `/proc`

`/proc` is a virtual filesystem that exposes kernel and process information.

Explore:

```bash
ls /proc
ls /proc/1
cat /proc/cpuinfo
cat /proc/meminfo
```

For your current shell:

```bash
echo $$
cat /proc/$$/status
```

`$$` expands to the PID of the current shell in Bash.

---

# 🔧 `/sys`

`/sys` exposes information and interfaces associated with devices, drivers, and kernel subsystems.

Explore:

```bash
ls /sys
ls /sys/class
```

Do not modify files under `/sys` casually. Some interfaces can change system behavior.

---

# 📦 File Descriptors

A process uses file descriptors to refer to open input/output resources.

The three standard descriptors are:

```text
0 → stdin
1 → stdout
2 → stderr
```

Visual:

```text
Keyboard ───────► stdin  (0)
                    │
                    ▼
                  Process
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      stdout (1)          stderr (2)
          │                   │
          ▼                   ▼
       Terminal            Terminal
```

This is the foundation of shell redirection.

Example:

```bash
echo "hello" > output.txt
```

```text
echo
 │
 └── stdout (1)
          │
          ▼
      output.txt
```

---

# 🔄 What Happens When You Run `ls`?

A simplified sequence:

```text
You type:

ls
 │
 ▼
Shell receives command
 │
 ▼
Shell locates executable
 │
 ▼
Shell starts process
 │
 ▼
Program requests filesystem information
 │
 ▼
Kernel handles required operations
 │
 ▼
Directory data is returned
 │
 ▼
ls formats the output
 │
 ▼
Terminal displays it
```

This is the mental model we will reuse throughout the entire repository.

---

# 🌐 Networking in the Architecture

Networking follows the same layered idea:

```text
Application
   │
   ▼
Socket API
   │
   ▼
Kernel networking stack
   │
   ▼
Network driver
   │
   ▼
NIC
   │
   ▼
Network
```

This is why a SOC analyst needs both **Linux knowledge and networking knowledge**.

---

# 🔐 Security Perspective

Linux architecture directly affects security investigations.

If a suspicious process appears, ask:

```text
Who owns it?
     ↓
What is its PID?
     ↓
Who is its parent?
     ↓
What executable started it?
     ↓
What files does it access?
     ↓
What network sockets does it use?
     ↓
What user permissions does it have?
     ↓
What logs describe its activity?
```

Useful tools we will learn later:

```bash
ps
pstree
ss
lsof
systemctl
journalctl
find
stat
```

---

# 🧪 Hands-On Lab

## Lab 01 — Explore Your System

Run:

```bash
uname -a
cat /etc/os-release
whoami
id
ps
ps aux
ls /proc
ls /sys
```

Answer:

1. What kernel version are you running?
2. Which distribution are you using?
3. What is your UID?
4. What is the PID of your current shell?
5. What is PID 1?
6. Which process is your shell's parent?

## Lab 02 — Explore Your Shell

Run:

```bash
echo $$
echo $SHELL
ls -l /proc/$$/fd
```

Observe the file descriptors for the current shell.

## Lab 03 — Process Investigation

Start a harmless long-running process:

```bash
sleep 300 &
```

Then:

```bash
pgrep sleep
ps -ef | grep '[s]leep'
```

Investigate its PID and parent process.

Finish it:

```bash
pkill sleep
```

Only terminate processes you intentionally started or have permission to manage.

---

# 🎯 Challenge

A process called `backup-agent` is consuming significant CPU.

You are given only its PID: `2480`.

Create an investigation plan using commands you already know or will soon learn.

Your plan should answer:

```text
PID
 ↓
Process owner
 ↓
Executable
 ↓
Parent process
 ↓
Command line
 ↓
Network connections
 ↓
Open files
 ↓
Logs
```

Do not kill the process immediately. **Investigate first.**

---

# 🎤 Interview Questions

### Beginner

**Q: What is the Linux kernel?**  
The core component responsible for managing system resources and providing operating-system services.

**Q: What is a process?**  
A running instance of a program.

**Q: What is a PID?**  
A process identifier used to identify a running process.

### Intermediate

**Q: What is the difference between user space and kernel space?**  
They are distinct privilege domains used to separate ordinary application execution from privileged kernel operations.

**Q: What is a system call?**  
A controlled interface through which user-space software requests services from the kernel.

**Q: What is `/proc`?**  
A virtual filesystem exposing process and kernel-related information.

### Security

**Q: Why are parent-child process relationships useful to a SOC analyst?**  
They can reveal how a suspicious process was launched and help establish an execution chain.

---

# ⚡ Quick Revision

```text
Hardware
   ↓
Kernel
   ↓
System Calls
   ↓
User Space
   ├── Shell
   ├── Applications
   └── Services
```

```text
PID       → Process identifier
/proc     → Process/kernel information
/sys      → Device/kernel subsystem information
stdin  0  → Standard input
stdout 1  → Standard output
stderr 2  → Standard error
```

---

# ✅ Module Checklist

- [ ] Understand Linux architecture
- [ ] Understand kernel vs user space
- [ ] Understand system calls conceptually
- [ ] Understand processes and PIDs
- [ ] Understand parent/child processes
- [ ] Explore `/proc`
- [ ] Explore `/sys`
- [ ] Understand file descriptors
- [ ] Trace what happens when a command runs
- [ ] Connect architecture to cybersecurity investigations
- [ ] Complete all labs
