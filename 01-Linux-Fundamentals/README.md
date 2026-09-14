# 🧱 01 — Linux Fundamentals

> **Build the mental model before memorizing commands.**

This module explains the main building blocks of a Linux system and how they fit together. You should understand these ideas before going deep into command-line work.

---

## 🎯 Objectives

By the end of this module, you should be able to explain:

* User space vs kernel space
* What the kernel does
* What a shell does
* What a terminal does
* What a process is
* What a service is
* What a daemon is
* What a system call is
* What a file is in Linux
* What an environment variable is
* What happens when you run a command

---

# 🧠 1. The Linux Mental Model

Think of Linux as layers working together.

```text
+----------------------------------+
|          Applications            |
| Browser | Editor | Tools | Apps  |
+----------------------------------+
                 │
                 ▼
+----------------------------------+
|          User Space              |
| Shell | Libraries | Utilities    |
+----------------------------------+
                 │
                 ▼
+----------------------------------+
|         System Calls             |
+----------------------------------+
                 │
                 ▼
+----------------------------------+
|            Kernel                |
| Process | Memory | Files | Net    |
+----------------------------------+
                 │
                 ▼
+----------------------------------+
|            Hardware              |
| CPU | RAM | Disk | NIC | Devices |
+----------------------------------+
```

The important idea is that applications do not normally control hardware directly. They use operating-system interfaces provided through the kernel.

---

# ⚙️ 2. User Space vs Kernel Space

Linux separates normal applications from the core operating-system code.

## User Space

This is where most applications and utilities run.

Examples:

* Shell
* Text editor
* Web server
* Python program
* SSH client
* Many administration tools

## Kernel Space

This is where the kernel runs and manages privileged operations such as:

* CPU scheduling
* Memory management
* Device access
* Networking
* Filesystem operations

Visual:

```text
USER SPACE
────────────────────────
Applications
Shells
Utilities
Libraries
────────────────────────
        ↓
   System Calls
        ↓
────────────────────────
KERNEL SPACE

Linux Kernel
────────────────────────
        ↓
────────────────────────
HARDWARE
```

### Simple analogy

Think of a restaurant:

```text
Customer
   ↓
Waiter
   ↓
Kitchen
   ↓
Equipment
```

The customer does not directly operate the kitchen equipment.

Similarly:

```text
Application
   ↓
System Call
   ↓
Kernel
   ↓
Hardware
```

---

# 🖥️ 3. What Is a Terminal?

A **terminal** is an interface through which you can interact with a command-line shell.

On a graphical desktop, a terminal emulator gives you a window in which you can type commands.

Examples include terminal applications provided by Linux desktop environments.

Important:

> **Terminal ≠ Shell**

The terminal provides the interface. The shell interprets the commands.

---

# 🐚 4. What Is a Shell?

A **shell** is a program that reads commands, interprets them and starts programs or performs shell operations.

Common shells include:

* Bash
* Zsh
* Fish
* Dash

Bash is especially common in Linux administration and scripting.

Flow:

```text
You type:

ls -la
   │
   ▼
Shell reads command
   │
   ▼
Shell interprets options
   │
   ▼
Shell starts the required program
   │
   ▼
Program interacts with Linux
```

---

# ⌨️ 5. What Is the CLI?

CLI means **Command-Line Interface**.

Instead of selecting buttons, you interact with a system by typing commands.

Example:

```bash
pwd
ls
cd /var/log
```

CLI is especially important in Linux because servers are often administered remotely and many powerful administration and security workflows are command based.

---

# 🧩 6. What Is a Process?

A **process** is a running instance of a program.

The distinction is useful:

```text
Program
= code stored on disk

Process
= running instance of that program
```

Example:

```text
/usr/bin/python3
      ↓
Program on disk
      ↓
You start it
      ↓
Python process created
      ↓
Gets PID
```

Each process has information such as:

* Process ID (PID)
* Parent process ID (PPID)
* Owner/user
* State
* Resources
* Open files
* Environment

We will investigate all of these later.

---

# 🆔 7. What Is a PID?

PID means **Process ID**.

Linux assigns a numeric identifier to each process.

Example:

```text
PID     Process
----------------------
1       systemd
842     sshd
1542    bash
1680    python3
```

PID becomes extremely important for troubleshooting and security investigations.

For example:

```text
Suspicious process
       ↓
Find PID
       ↓
Identify owner
       ↓
Inspect executable
       ↓
Inspect network connections
       ↓
Investigate logs
```

---

# ⚙️ 8. What Is a Service?

A **service** is a background function provided by software, usually intended to remain available for other applications or users.

Examples:

* SSH server
* Web server
* Database server
* Logging service
* Network service

On systems using systemd, services are commonly managed through `systemctl`.

```text
Service
   ↓
Starts program/processes
   ↓
Runs in background
   ↓
Provides functionality
```

A service and a process are related but not identical concepts.

---

# 👻 9. What Is a Daemon?

A **daemon** is a background process that performs a task or provides a service, often without direct user interaction.

Many Linux daemon names historically end in `d`.

Examples include names such as `sshd`.

Do not memorize the naming rule as a requirement — focus on the idea:

> A daemon is a background process designed to perform ongoing work or provide a service.

---

# 📞 10. What Is a System Call?

Applications frequently need privileged operations such as reading a file, opening a network socket or creating a process.

They request these services from the kernel through **system calls**.

Simplified flow:

```text
Application
    │
    │ “Please open this file”
    ▼
System Call
    │
    ▼
Kernel
    │
    ▼
Filesystem / Storage
```

You do not need to memorize system-call internals yet. Just understand the relationship:

> **Application requests → kernel performs or mediates → result returns to application**

---

# 📁 11. Everything Is a File? 

You may hear the phrase:

> “In Linux, everything is a file.”

This is a useful teaching idea, but it is not literally true for every internal object.

Linux exposes many resources through file-like interfaces, especially through paths under directories such as `/dev` and `/proc`.

For example:

```text
/dev/...
   ↓
Device interfaces

/proc/...
   ↓
Process / kernel information interfaces
```

The important beginner lesson is:

> Linux provides a consistent way to interact with many resources through filesystem-like interfaces.

---

# 🌱 12. Environment Variables

An **environment variable** is a named value made available to processes through their environment.

Examples you will commonly encounter:

```bash
PATH
HOME
USER
SHELL
PWD
```

For example:

```bash
echo $HOME
echo $PATH
echo $USER
```

Think of environment variables as information passed into programs about their execution environment.

---

# 🔗 13. PATH — A Very Important Variable

When you type:

```bash
ls
```

you usually do not type the full path to the executable.

The shell searches directories listed in `PATH`.

Visual:

```text
You type:

ls
 │
 ▼
Shell
 │
 ▼
Search PATH directories
 │
 ├── /usr/local/bin
 ├── /usr/bin
 ├── /bin
 └── ...
 │
 ▼
Find executable
 │
 ▼
Run it
```

This becomes very important later when learning shell scripting and security.

---

# 🔄 14. What Happens When You Run `ls`?

Let's build the complete beginner mental model.

You type:

```bash
ls
```

A simplified sequence is:

```text
1. Terminal accepts your keyboard input
              ↓
2. Shell reads the command
              ↓
3. Shell determines what “ls” refers to
              ↓
4. Shell starts the program
              ↓
5. Program requests needed resources
              ↓
6. Kernel handles protected operations
              ↓
7. Program receives results
              ↓
8. Output is sent back to the terminal
```

You will learn the details gradually. For now, remember the chain.

---

# 🚦 15. Exit Status

Many Linux commands return an **exit status** when they finish.

A common convention is:

```text
0     → success
non-0 → some kind of failure / special condition
```

You can inspect the previous command's exit status with:

```bash
echo $?
```

Example:

```bash
pwd
echo $?
```

This concept becomes extremely important in shell scripting and automation.

---

# 🔀 16. Pipes

A pipe sends the output of one command to another command.

Example:

```bash
ps aux | grep ssh
```

Think:

```text
Command A
   │
   │ output
   ▼
  PIPE
   │
   ▼
Command B
```

Pipes are one of the reasons the Linux command line is so powerful.

---

# 📤 17. Redirection

Linux shells can redirect input and output.

Example:

```bash
ls > files.txt
```

Conceptually:

```text
ls
 │
 │ output
 ▼
files.txt
```

Another useful example:

```bash
echo "hello" >> notes.txt
```

Here `>>` appends rather than replacing the existing content.

We will study redirection deeply in the command-line module.

---

# 🔐 18. Why These Concepts Matter for Cybersecurity

A SOC analyst investigating Linux must understand the relationships between:

```text
User
 ↓
Shell
 ↓
Process
 ↓
File
 ↓
Network Connection
 ↓
Log Entry
```

Example investigation:

```text
Suspicious login
      ↓
Identify user
      ↓
Find shell/processes
      ↓
Find commands/process IDs
      ↓
Inspect files
      ↓
Inspect network connections
      ↓
Check logs
```

Linux fundamentals become security fundamentals because they explain **what the system is doing**.

---

# 🌍 19. Real-World Example

Suppose a Linux web server is running a website.

```text
Internet
    ↓
Network Interface
    ↓
Linux Kernel
    ↓
Web Server Service
    ↓
Web Server Process
    ↓
Website Files
```

An administrator might ask:

> Is the service running?

A SOC analyst might ask:

> Is the process legitimate and listening on the expected port?

Both questions depend on the same Linux fundamentals.

---

# 🎯 20. Practice Tasks

Complete these in your Linux lab:

### Task 1
Find the current shell.

```bash
echo $SHELL
```

### Task 2
Display your username.

```bash
whoami
```

### Task 3
Display your home directory.

```bash
echo $HOME
```

### Task 4
Display your PATH.

```bash
echo $PATH
```

### Task 5
Run a command and inspect its exit status.

```bash
pwd
echo $?
```

### Task 6
Create a simple pipeline.

```bash
ps aux | grep ssh
```

Do not worry if every field is unfamiliar. The goal is to become comfortable seeing the command line.

---

# 🎤 21. Interview Questions

### Q1. What is the difference between a terminal and a shell?

**Answer:** A terminal provides the interface for command-line interaction, while a shell interprets commands and launches programs.

### Q2. What is a process?

**Answer:** A process is a running instance of a program.

### Q3. What is a PID?

**Answer:** PID is the numeric Process ID assigned to a running process.

### Q4. What is user space?

**Answer:** User space is the environment in which most applications and non-kernel programs run.

### Q5. What is kernel space?

**Answer:** Kernel space is the privileged environment where the Linux kernel performs core system operations.

### Q6. What is a system call?

**Answer:** It is a controlled interface through which a user-space program requests services from the kernel.

### Q7. What is an environment variable?

**Answer:** It is named information made available to a process as part of its execution environment.

---

# ⚡ Quick Revision

```text
Terminal
→ Interface for command-line interaction

Shell
→ Interprets commands

CLI
→ Command-Line Interface

Process
→ Running instance of a program

PID
→ Process identifier

Service
→ Background functionality provided by software

Daemon
→ Background process providing ongoing work/service

System Call
→ Application ↔ Kernel interface

User Space
→ Normal applications and utilities

Kernel Space
→ Privileged kernel operations

PATH
→ Directories searched for executable commands

Exit Status
→ Result code from a command/process
```

---

# ✅ Module Completion Checklist

- [ ] I understand user space and kernel space.
- [ ] I can explain terminal vs shell.
- [ ] I know what CLI means.
- [ ] I understand program vs process.
- [ ] I know what a PID is.
- [ ] I understand service vs process at a basic level.
- [ ] I know what a daemon is.
- [ ] I understand the purpose of system calls.
- [ ] I know what environment variables are.
- [ ] I understand `PATH`.
- [ ] I can explain what happens when I run a command.
- [ ] I understand pipes and basic redirection.

---

## 🔜 Next

➡️ [02 — Linux Installation](../02-Linux-Installation/README.md)
