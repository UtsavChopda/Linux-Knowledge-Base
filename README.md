# 🐧 Linux Knowledge Base

> **A professional, beginner-friendly and practical Linux learning resource covering Linux fundamentals, administration, networking, security, troubleshooting, automation, cybersecurity, SOC operations and hands-on labs — from absolute beginner to advanced.**

![Linux](https://img.shields.io/badge/Linux-Learning-FCC624?logo=linux&logoColor=black)
![Bash](https://img.shields.io/badge/Bash-Scripting-4EAA25?logo=gnubash&logoColor=white)
![Cybersecurity](https://img.shields.io/badge/Cybersecurity-Blue%20Team-0A66C2)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 🎯 About This Repository

This repository is designed to be a **complete Linux knowledge base**, built to teach Linux from the ground up and then connect that knowledge to system administration, networking, cybersecurity and SOC/Blue Team work.

It is **not** intended to be a collection of short command notes.

The learning approach combines:

* 📚 Detailed concepts
* 🧠 Simple explanations
* 👀 Visual diagrams and mental models
* 🌍 Real-world examples
* 💻 Commands with syntax and examples
* 🧪 Hands-on labs
* 🔎 Verification techniques
* 🚨 Troubleshooting workflows
* 🔐 Security perspective
* 🎯 Scenario-based challenges
* 🛠️ Practical projects
* 🎤 Interview preparation
* ⚡ Quick revision notes
* 📋 Command cheat sheets

The goal is simple:

```text
“I don't know Linux.”
        ↓
“I can use Linux.”
        ↓
“I understand Linux.”
        ↓
“I can administer Linux.”
        ↓
“I can troubleshoot Linux.”
        ↓
“I can secure Linux.”
        ↓
“I can investigate Linux.”
        ↓
“I can use Linux effectively in a SOC / Blue Team role.”
```

---

# 🗺️ Linux Learning Roadmap

```text
                         🐧 LINUX
                            │
                            ▼
                  00 — Linux Overview
                            │
                            ▼
                01 — Linux Fundamentals
                            │
                            ▼
                 02 — Linux Installation
                            │
                            ▼
                03 — Linux Architecture
                            │
                            ▼
             04 — Boot Process & Systemd
                            │
                            ▼
               05 — Linux Command Line
                            │
                            ▼
             06 — Files & Directories
                            │
                            ▼
          07 — Text Processing & Editors
                            │
                            ▼
                08 — Users & Groups
                            │
                            ▼
             09 — File Permissions
                            │
                            ▼
              10 — Ownership & ACLs
                            │
                            ▼
            11 — Processes & Services
                            │
                            ▼
             12 — Package Management
                            │
                            ▼
              13 — Disk & Storage
                            │
                            ▼
          14 — Filesystems & Mounting
                            │
                            ▼
               15 — Linux Networking
                            │
                            ▼
               16 — SSH & Remote Access
                            │
                            ▼
          17 — Firewall & Linux Security
                            │
                            ▼
               18 — Logs & Monitoring
                            │
                            ▼
               19 — Shell Scripting
                            │
                            ▼
               20 — Bash Automation
                            │
                            ▼
              21 — Scheduling & Cron
                            │
                            ▼
            22 — Linux Troubleshooting
                            │
                            ▼
               23 — Linux Hardening
                            │
                            ▼
          24 — Linux for Cybersecurity
                            │
                            ▼
                25 — Linux for SOC
                            │
                            ▼
          26 — Linux Security Monitoring
                            │
                            ▼
               27 — Practical Labs
                            │
                            ▼
         28 — Scenario-Based Challenges
                            │
                            ▼
                29 — Linux Projects
                            │
                            ▼
          30 — Interview Preparation
                            │
                            ▼
               31 — Cheat Sheets
```

---

# 📚 Module Overview

## 00 — Linux Overview

Start from zero.

Topics:

* What is an Operating System?
* What is Linux?
* Why Linux exists
* Linux history at a high level
* Linux kernel vs Linux distribution
* Open source basics
* Linux vs Windows
* Why Linux is important in servers
* Linux in cloud environments
* Linux in networking
* Linux in cybersecurity
* Linux in DevOps
* Common Linux distributions
* Ubuntu, Debian, Fedora, RHEL, Rocky Linux, AlmaLinux, Kali Linux and others
* Choosing a distribution for learning
* Linux terminology beginners must know

---

## 01 — Linux Fundamentals

Build the mental model before memorizing commands.

Topics:

* Operating system basics
* Kernel
* User space and kernel space
* Shell
* Terminal
* CLI
* GUI
* Files and processes
* Users and groups
* Services
* Packages
* Environment variables
* System calls at a beginner level
* What happens when a command is executed?

Visual model:

```text
User
 │
 ▼
Terminal
 │
 ▼
Shell
 │
 ▼
Kernel
 │
 ▼
Hardware
```

---

## 02 — Linux Installation

Learn how to build a safe Linux practice environment.

Topics:

* Choosing a Linux distribution
* Virtual machines
* Bare-metal installation
* Dual boot concepts
* ISO images
* Bootable media
* VirtualBox / VMware concepts
* Disk partitioning basics
* Users during installation
* Hostname
* Networking during installation
* First login
* Updating the system
* Installing essential tools
* Building a cybersecurity lab environment

---

## 03 — Linux Architecture

Understand what is happening underneath the commands.

Topics:

* Linux kernel
* Hardware layer
* System calls
* User space
* Kernel space
* Shells
* Daemons
* Processes
* Drivers
* Modules
* `/proc`
* `/sys`
* `/dev`
* Linux directory hierarchy

Visual:

```text
+---------------------------+
|        Applications       |
+---------------------------+
|     Shell / User Space    |
+---------------------------+
|       System Calls        |
+---------------------------+
|          Kernel           |
+---------------------------+
| Drivers / Hardware        |
+---------------------------+
```

---

## 04 — Boot Process & Systemd

Understand what happens between pressing the power button and getting a login prompt.

Topics:

* BIOS / UEFI
* Bootloader
* GRUB
* Kernel loading
* initramfs
* PID 1
* systemd
* Targets
* Units
* Services
* Dependencies
* Startup sequence
* Service management
* Boot troubleshooting

Typical flow:

```text
Power On
   ↓
BIOS / UEFI
   ↓
GRUB
   ↓
Linux Kernel
   ↓
initramfs
   ↓
systemd (PID 1)
   ↓
Services / Targets
   ↓
Login
```

---

## 05 — Linux Command Line

Master the terminal from the absolute basics.

Topics:

* Terminal and shell
* Command structure
* Options and arguments
* `man`
* `help`
* `pwd`
* `ls`
* `cd`
* `echo`
* `clear`
* `history`
* `whoami`
* `id`
* `date`
* `uname`
* `hostname`
* `which`
* `type`
* Quoting
* Wildcards
* Command substitution
* Exit status
* Redirection
* Pipes
* Command chaining
* Environment variables

---

## 06 — Files & Directories

Learn how Linux organizes and manipulates data.

Topics:

* Linux filesystem hierarchy
* Absolute paths
* Relative paths
* Files
* Directories
* Hidden files
* `mkdir`
* `touch`
* `cp`
* `mv`
* `rm`
* `rmdir`
* `find`
* `locate`
* `file`
* `stat`
* Links
* Hard links
* Symbolic links
* Important directories such as `/etc`, `/var`, `/home`, `/tmp`, `/usr`, `/opt`, `/proc`, `/dev`

---

## 07 — Text Processing & Editors

Become comfortable reading and manipulating text — a critical Linux and SOC skill.

Topics:

* `cat`
* `less`
* `head`
* `tail`
* `wc`
* `sort`
* `uniq`
* `cut`
* `tr`
* `grep`
* `sed`
* `awk`
* Regular expression basics
* Nano
* Vim fundamentals
* Searching logs
* Extracting fields
* Filtering command output

---

## 08 — Users & Groups

Understand Linux identity and account management.

Topics:

* Users
* Groups
* UID
* GID
* Root user
* `/etc/passwd`
* `/etc/shadow`
* `/etc/group`
* `useradd`
* `usermod`
* `userdel`
* `passwd`
* `groupadd`
* `groupmod`
* `groupdel`
* `su`
* `sudo`
* Service accounts
* User auditing

---

## 09 — File Permissions

Master one of the most important Linux security concepts.

Topics:

* Read / write / execute
* User / group / others
* Permission notation
* Numeric permissions
* `chmod`
* `chown`
* `chgrp`
* Default permissions
* `umask`
* Directory permissions
* Special permission behavior
* Common permission mistakes
* Security implications

Example:

```text
-rwxr-x---
 │││ │││
 │││ ││└── Others
 │││ └└─── Group
 └└└────── Owner
```

---

## 10 — Ownership & ACLs

Go beyond basic permissions.

Topics:

* File ownership
* Group ownership
* Access Control Lists
* `getfacl`
* `setfacl`
* Default ACLs
* ACL troubleshooting
* When ACLs are useful
* Security use cases

---

## 11 — Processes & Services

Understand what is running on a Linux system and why.

Topics:

* Processes
* PID
* PPID
* Parent / child processes
* Foreground / background
* Process states
* `ps`
* `top`
* `htop`
* `pstree`
* `pgrep`
* `kill`
* Signals
* Jobs
* `systemctl`
* Services
* Daemons
* Service troubleshooting

Security perspective:

```text
Running Process
      ↓
Which user owns it?
      ↓
Which executable?
      ↓
What files does it access?
      ↓
What network connections?
      ↓
Is it legitimate?
```

---

## 12 — Package Management

Learn how software is installed, updated and removed.

Topics:

* Packages
* Repositories
* Debian packages
* RPM packages
* `apt`
* `dpkg`
* `dnf`
* `rpm`
* Package metadata
* Updates
* Dependencies
* Installing security tools
* Package troubleshooting

---

## 13 — Disk & Storage

Learn how Linux sees and manages storage.

Topics:

* Disk concepts
* Partitions
* `lsblk`
* `blkid`
* `fdisk`
* `parted`
* Disk usage
* `df`
* `du`
* Inodes
* Mount points
* Storage troubleshooting
* Disk exhaustion
* Secure handling of storage

---

## 14 — Filesystems & Mounting

Understand how storage becomes usable files and directories.

Topics:

* Filesystem concepts
* ext4
* XFS
* tmpfs
* Mounting
* Unmounting
* `/etc/fstab`
* UUID
* Mount options
* Read-only mounts
* Filesystem checks
* Recovery concepts

---

## 15 — Linux Networking

Build Linux networking knowledge on top of the CCNA foundation.

Topics:

* Network interfaces
* MAC address
* IP addresses
* IPv4 / IPv6
* Default gateway
* Routing table
* DNS configuration
* `/etc/hosts`
* `ip`
* `ss`
* `ping`
* `traceroute`
* `dig`
* `nslookup`
* `curl`
* `wget`
* NetworkManager
* `nmcli`
* Interface troubleshooting

Visual:

```text
Application
    ↓
Socket
    ↓
TCP / UDP
    ↓
IP
    ↓
Network Interface
    ↓
Network
```

---

## 16 — SSH & Remote Access

Learn secure remote Linux administration.

Topics:

* SSH basics
* Client / server model
* SSH keys
* Password authentication
* Public / private keys
* `ssh`
* `scp`
* `sftp`
* SSH configuration
* Port configuration
* Authentication troubleshooting
* SSH logs
* Secure SSH practices

---

## 17 — Firewall & Linux Security

Understand host-level traffic control and defensive security.

Topics:

* Firewall concepts
* `ufw`
* `firewalld`
* `nftables` fundamentals
* Ports
* Services
* Inbound vs outbound traffic
* Allow / deny rules
* Rule troubleshooting
* SSH protection
* Host security basics

---

## 18 — Logs & Monitoring

This is a core Linux + SOC module.

Topics:

* Why logs matter
* `/var/log`
* Authentication logs
* System logs
* Application logs
* `journalctl`
* `dmesg`
* `tail -f`
* Log filtering
* Time correlation
* Log rotation
* Monitoring processes
* Monitoring network connections
* Detecting suspicious behavior

Example workflow:

```text
Event
 ↓
Log generated
 ↓
Collect
 ↓
Filter
 ↓
Correlate
 ↓
Investigate
 ↓
Respond
```

---

## 19 — Shell Scripting

Start automating repetitive Linux tasks.

Topics:

* Bash fundamentals
* Shebang
* Variables
* Input
* Output
* Conditions
* Loops
* Functions
* Arguments
* Exit codes
* Arrays
* String handling
* Arithmetic
* Command substitution
* Error handling
* Script debugging

---

## 20 — Bash Automation

Turn scripts into practical administration and security tools.

Topics:

* System information scripts
* User audit scripts
* Disk monitoring
* Service checks
* Network checks
* Log parsing
* Failed login detection
* Backup automation
* Health checks
* Automation best practices

---

## 21 — Scheduling & Cron

Automate jobs based on time and events.

Topics:

* `cron`
* `crontab`
* Cron syntax
* `at`
* System timers
* Scheduling maintenance
* Scheduled security checks
* Troubleshooting scheduled tasks

---

## 22 — Linux Troubleshooting

Learn a repeatable diagnosis process instead of guessing.

Core workflow:

```text
Problem
   ↓
Collect Evidence
   ↓
Check Basics
   ↓
Narrow Down Cause
   ↓
Verify Hypothesis
   ↓
Fix
   ↓
Verify Again
   ↓
Document
```

Scenarios:

* Network unavailable
* DNS failure
* SSH unavailable
* Service won't start
* Disk full
* Permission denied
* High CPU
* High memory usage
* Process consuming resources
* Boot failure
* Firewall blocking traffic
* Authentication failure

---

## 23 — Linux Hardening

Learn practical host defense.

Topics:

* Secure user management
* Strong authentication
* Least privilege
* Sudo control
* SSH hardening
* Firewall configuration
* Service minimization
* Patch management
* File permissions
* Log monitoring
* Time synchronization
* Backup and recovery
* Baseline configuration
* Security verification

---

## 24 — Linux for Cybersecurity

Connect Linux administration to defensive security work.

Topics:

* Linux attack surface
* Security-relevant files
* Authentication artifacts
* Process investigation
* Network investigation
* File investigation
* Persistence concepts
* Privilege concepts
* Indicators of compromise
* Evidence collection
* Defensive commands
* Basic incident-response workflow

---

## 25 — Linux for SOC

Use Linux as a SOC Analyst.

Topics:

* Authentication investigation
* Failed SSH logins
* Successful login analysis
* Suspicious processes
* Open ports
* Network connections
* User activity
* Persistence indicators
* Cron investigation
* Shell history considerations
* Log investigation
* Timeline analysis
* Initial triage
* Containment concepts

---

## 26 — Linux Security Monitoring

Build practical security visibility.

Topics:

* Authentication monitoring
* Process monitoring
* Network connection monitoring
* File integrity concepts
* Log monitoring
* Suspicious command detection
* Detection engineering basics
* Sigma concepts
* SIEM ingestion concepts
* Linux telemetry
* Investigation workflow

---

# 🧪 27 — Practical Labs

Hands-on learning is required.

```text
27-Practical-Labs/
│
├── 01-Linux-Basics-Lab/
├── 02-File-Management-Lab/
├── 03-Users-and-Groups-Lab/
├── 04-Permissions-Lab/
├── 05-Process-Management-Lab/
├── 06-Systemd-Lab/
├── 07-Package-Management-Lab/
├── 08-Disk-Management-Lab/
├── 09-Networking-Lab/
├── 10-SSH-Lab/
├── 11-Firewall-Lab/
├── 12-Log-Monitoring-Lab/
├── 13-Shell-Scripting-Lab/
├── 14-Cron-Automation-Lab/
├── 15-Linux-Hardening-Lab/
└── 16-SOC-Investigation-Lab/
```

Every major lab follows:

```text
🎯 Objective
🧰 Requirements
🗺️ Environment
📋 Tasks
💻 Commands
🔎 Verification
🚨 Troubleshooting
⚠️ Common Mistakes
🎯 Challenge
✅ Expected Result
🧠 What You Learned
```

---

# 🎯 28 — Scenario-Based Challenges

Challenges will deliberately hide the solution so that the learner must investigate.

Example:

> A Linux server is showing unusual outbound network activity. Determine which process is responsible, identify the owning user, inspect relevant connections and logs, and explain your findings.

Scenario format:

```text
Situation
   ↓
Symptoms
   ↓
Evidence Available
   ↓
Your Investigation
   ↓
Expected Findings
   ↓
Solution
   ↓
Explanation
```

---

# 🛠️ 29 — Linux Projects

Projects will turn knowledge into portfolio evidence.

Planned projects:

* Linux System Information Tool
* Linux User Audit Tool
* Disk Monitoring Script
* Service Health Monitor
* Failed SSH Login Detector
* Linux Log Analyzer
* Process Monitoring Tool
* File Integrity Monitoring Tool
* Linux Security Audit Script
* Linux SOC Investigation Toolkit

---

# 🎤 30 — Interview Preparation

Questions will be grouped by difficulty.

### 🟢 Beginner

Definitions and foundational concepts.

### 🟡 Intermediate

How Linux components work together.

### 🔴 Advanced

Administration, security and troubleshooting.

### 🚨 Scenario-Based

Real-world investigation and failure scenarios.

The goal is to understand **why**, not memorize answers.

---

# ⚡ 31 — Cheat Sheets

Quick-reference material will include:

* Essential Linux commands
* File commands
* Text-processing commands
* User-management commands
* Permission commands
* Process commands
* systemd commands
* Package-management commands
* Disk commands
* Networking commands
* SSH commands
* Firewall commands
* Log-analysis commands
* Bash syntax
* Troubleshooting commands
* Security investigation commands

---

# 🔐 Linux + Cybersecurity Connection

Linux is not a separate topic from your cybersecurity path.

This repository intentionally connects the two:

```text
Linux Fundamentals
        │
        ▼
Linux Administration
        │
        ▼
Linux Networking
        │
        ▼
Linux Security
        │
        ▼
Logs + Processes + Network Connections
        │
        ▼
Security Monitoring
        │
        ▼
Incident Investigation
        │
        ▼
SOC / Blue Team
```

For every important Linux concept, we will ask:

> **How would a system administrator use this?**
>
> **How would a SOC analyst use this?**
>
> **What security evidence can Linux provide?**

---

# 📐 Standard Topic Template

Every important topic will use a consistent structure.

```text
# Topic Name

## 🎯 Objective

## 🧠 What Is It?

## 🤔 Why Do We Need It?

## 🔍 How Does It Work?

## 👀 Visual Explanation

## 📌 Important Concepts

## 💻 Commands / Syntax

## 🧪 Examples

## 🌍 Real-World Example

## 🔐 Cybersecurity Relevance

## 🚨 Troubleshooting

## ⚠️ Common Mistakes

## 🎯 Practice Challenge

## 🎤 Interview Questions

## ⚡ Quick Revision
```

---

# 🧭 Learning Rules

This repository follows a few rules:

### 1. Understand before memorizing

A command is useful only when you know **what problem it solves**.

### 2. Practice every major concept

Theory should become terminal experience.

### 3. Always verify

After changing something, learn how to confirm that it actually worked.

### 4. Learn troubleshooting early

Good Linux users do not only know commands — they know how to investigate failures.

### 5. Connect Linux to security

Processes, users, permissions, networking and logs are all security-relevant.

### 6. Keep explanations beginner-friendly

Complex Linux internals will be introduced gradually, with simple language first and deeper technical detail afterward.

---

# 📈 Learning Progress

```text
00 — Linux Overview                  ⬜
01 — Fundamentals                    ⬜
02 — Installation                    ⬜
03 — Architecture                    ⬜
04 — Boot & Systemd                  ⬜
05 — Command Line                    ⬜
06 — Files & Directories             ⬜
07 — Text Processing                 ⬜
08 — Users & Groups                  ⬜
09 — Permissions                     ⬜
10 — Ownership & ACLs                ⬜
11 — Processes & Services            ⬜
12 — Package Management              ⬜
13 — Disk & Storage                  ⬜
14 — Filesystems                     ⬜
15 — Linux Networking                ⬜
16 — SSH                             ⬜
17 — Firewall & Security             ⬜
18 — Logs & Monitoring               ⬜
19 — Shell Scripting                 ⬜
20 — Automation                      ⬜
21 — Scheduling                      ⬜
22 — Troubleshooting                 ⬜
23 — Hardening                       ⬜
24 — Cybersecurity                   ⬜
25 — SOC                             ⬜
26 — Security Monitoring             ⬜
27 — Practical Labs                  ⬜
28 — Challenges                      ⬜
29 — Projects                        ⬜
30 — Interviews                      ⬜
31 — Cheat Sheets                    ⬜
```

---

# 🛠️ Recommended Practice Environment

The repository is designed to work with a Linux virtual machine or other safe lab environment.

A beginner should be able to start with one Linux VM and progressively add more hosts as networking and SOC labs become more advanced.

```text
                 Your Computer
                      │
                      ▼
                Linux Lab VM
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Admin        Network     Security
        Labs         Labs         Labs
```

---

# 📚 Course Mapping

The repository also maps naturally to the Linux lessons being studied alongside it:

| Course Day | Main Repository Modules |
|---|---|
| Day 1 | 00, 01, 02 |
| Day 2 | 03, 04, 05 |
| Day 3 | 06, 07 |
| Day 4 | 15, 16 |
| Day 5 | 08, 09, 10 |
| Day 6 | 13, 14, 17 |
| Day 7 | 18, 20 |
| Day 8 | 19, 22 |

The course is the **input**; this repository becomes the **long-term knowledge base**.

---

# 📜 License

MIT License

---

# 🚧 Status

**In Progress 🚀**

The repository will grow module by module through theory, terminal practice, labs, troubleshooting, security investigations and projects.