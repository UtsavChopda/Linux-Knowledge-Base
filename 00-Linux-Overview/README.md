# 🐧 00 — Linux Overview

> Start here if you are completely new to Linux.

This module builds the mental foundation required for everything that comes later. Do not worry about memorizing commands yet. First understand **what Linux is, why it exists, where it is used, and what the important words mean**.

---

## 🎯 Objectives

By the end of this module, you should be able to explain:

* What an operating system is
* What Linux is
* What the Linux kernel does
* The difference between Linux and a Linux distribution
* What open source means
* What a terminal, shell and CLI are
* Why Linux is heavily used on servers
* Why Linux matters in cybersecurity and SOC work
* What common Linux distributions are used for

---

## 🧠 1. What Is an Operating System?

An **Operating System (OS)** is the main software layer that helps applications use computer hardware.

Think of it as a manager between applications and hardware.

```text
+---------------------------+
|      Applications         |
| Browser | Editor | Tools  |
+---------------------------+
             ↓
+---------------------------+
|    Operating System       |
|  Process | Memory | I/O   |
|  Files   | Network | etc. |
+---------------------------+
             ↓
+---------------------------+
|         Hardware          |
| CPU | RAM | Disk | NIC    |
+---------------------------+
```

Without an operating system, normal applications would need to communicate with hardware directly, which would be far more difficult.

---

## 🐧 2. What Is Linux?

Linux is a **family of operating systems built around the Linux kernel**.

The word **Linux** is often used casually to describe a complete operating system such as Ubuntu or Debian. Technically, the Linux kernel is the core component, while a distribution combines the kernel with other software needed to create a usable system.

```text
Linux Distribution
        │
        ├── Linux Kernel
        ├── System Libraries
        ├── Shells
        ├── Utilities
        ├── Package Manager
        ├── Applications
        └── Configuration
```

### Simple idea

> **Kernel = core**
>
> **Distribution = complete packaged system built around the kernel**

---

## ⚙️ 3. What Is the Linux Kernel?

The **kernel** is the core part of the operating system that controls access to hardware and provides fundamental services to programs.

At a beginner level, think of it as the layer responsible for things such as:

* CPU scheduling
* Memory management
* Device interaction
* Filesystem support
* Networking
* Process management
* Security controls at the kernel level

```text
Application
    ↓
System Call
    ↓
Linux Kernel
    ↓
Hardware
```

When we later study processes, networking, permissions, filesystems and booting, we will repeatedly come back to the kernel.

---

## 🧩 4. Linux Kernel vs Linux Distribution

This distinction is important.

| Term | Simple meaning |
|---|---|
| Linux kernel | Core that manages hardware and system resources |
| Distribution | Complete operating-system package built around the kernel |
| Ubuntu | A Linux distribution |
| Debian | A Linux distribution |
| Fedora | A Linux distribution |
| Kali Linux | A Linux distribution focused on security and penetration-testing use cases |

A distribution normally includes more than just the kernel.

---

## 🌍 5. Where Is Linux Used?

Linux is not only a desktop operating system.

It is widely used across technical environments such as:

```text
Servers
  ↓
Cloud
  ↓
Containers
  ↓
Networking
  ↓
Embedded systems
  ↓
Cybersecurity
  ↓
DevOps
  ↓
High-performance computing
```

### Examples of practical environments

* Web servers
* Database servers
* Cloud infrastructure
* Containers
* Network appliances
* Development environments
* Security tools
* SOC and incident-response systems

---

## 🔐 6. Why Linux Matters for Cybersecurity

Linux is extremely important for defensive security work because security teams frequently investigate systems through:

* Processes
* Users
* File permissions
* Services
* Network sockets
* Authentication events
* System logs
* Scheduled jobs
* Configuration files

That creates a direct connection:

```text
Linux Knowledge
      ↓
System Understanding
      ↓
Security Visibility
      ↓
Investigation
      ↓
SOC / Blue Team
```

Later modules will turn this into hands-on investigation skills.

---

## 🆚 7. Linux vs Windows — Beginner View

Do not think of this as one being universally better. They have different ecosystems and common use cases.

| Area | Linux | Windows |
|---|---|---|
| Interface | GUI + CLI | GUI + CLI |
| Customization | Very high | More controlled |
| Command-line use | Very important | Important, especially PowerShell |
| Server usage | Very common | Very common |
| Open-source core | Linux kernel is open source | Windows is proprietary |
| Package management | Strong package-manager culture | Different application distribution models |
| Cybersecurity labs | Very common | Also important |

For a cybersecurity learner, learning Linux deeply is a major advantage.

---

## 🔓 8. What Does Open Source Mean?

Open-source software is software whose source code is made available under a license that permits defined rights such as inspection, modification and redistribution.

The exact rights depend on the license.

For learning purposes, remember:

```text
Source Code Available
        ↓
People can inspect it
        ↓
Communities can improve it
        ↓
Projects can evolve openly
```

Linux has a large open-source ecosystem, with many communities and organizations contributing to different projects.

---

## 📦 9. What Is a Linux Distribution?

A **distribution (distro)** combines the Linux kernel with user-space software, package management, installers, configuration tools and other components to create a usable operating system.

Common families include:

```text
Debian Family
 ├── Debian
 └── Ubuntu

Red Hat Family
 ├── Fedora
 ├── RHEL
 ├── Rocky Linux
 └── AlmaLinux

Security-focused
 └── Kali Linux
```

Different distributions can use the same Linux kernel family while providing different defaults, tooling and support models.

---

## 🐣 10. Which Distribution Should a Beginner Use?

For this knowledge base, the exact distribution can vary by lab.

A good learning approach is:

**Ubuntu or another mainstream general-purpose distribution** for core Linux learning, then use **Kali Linux in a controlled lab** when a security-focused environment is useful.

Why?

You want to learn Linux itself rather than accidentally learning only one special-purpose toolset.

---

## 💻 11. Terminal, Shell and CLI

These words are related, but they are not identical.

### Terminal

A program that provides an interface for interacting with a shell.

### Shell

A command interpreter that reads commands and runs programs.

Examples include Bash and Zsh.

### CLI

**Command-Line Interface** — interacting with a computer primarily by typing commands rather than clicking graphical controls.

Visual:

```text
You
 ↓
Terminal
 ↓
Shell
 ↓
Commands / Programs
 ↓
Kernel
 ↓
Hardware
```

We will study this flow in depth in the next modules.

---

## 🏠 12. GUI vs CLI

### GUI

You typically use:

* Windows
* Buttons
* Menus
* Icons

### CLI

You type commands such as:

```bash
pwd
ls
cd /etc
```

For Linux administration and cybersecurity, CLI skills are especially important because many systems are accessed remotely and many powerful tools are command-line based.

---

## 🧱 13. Linux Building Blocks

Keep this simple mental model in your head:

```text
                   LINUX SYSTEM
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
      Users          Processes          Files
        │                │                │
        └────────────────┼────────────────┘
                         ▼
                    Services
                         │
                         ▼
                      Network
                         │
                         ▼
                       Kernel
                         │
                         ▼
                      Hardware
```

Almost everything we study later fits somewhere in this picture.

---

## 🌍 14. Real-World Example

Imagine a company's web server.

```text
Internet
   ↓
Network Interface
   ↓
Linux Kernel
   ↓
Web Server Process
   ↓
Website Files
   ↓
Database / Other Services
```

A Linux administrator may manage:

* Users
* Services
* Files
* Permissions
* Network settings
* Storage
* Logs

A SOC analyst may investigate:

* Suspicious processes
* Failed logins
* New users
* Unexpected services
* Strange network connections
* Modified files
* Security-relevant logs

The same Linux system is viewed from two different perspectives.

---

## 🎯 15. Beginner Practice Questions

Try answering these without looking back:

1. What is an operating system?
2. What is Linux?
3. What is the Linux kernel?
4. What is a Linux distribution?
5. Is Ubuntu a kernel or a distribution?
6. What is a terminal?
7. What is a shell?
8. What does CLI mean?
9. Why is Linux important in cybersecurity?
10. Why might a server use CLI instead of a GUI?

---

## 🎤 16. Interview Questions

### Beginner

**Q: What is Linux?**

A: Linux is a family of operating systems built around the Linux kernel. In everyday use, the term often refers to a complete Linux distribution.

**Q: What is the kernel?**

A: The kernel is the core part of the operating system responsible for managing system resources and providing controlled access to hardware.

**Q: What is a distribution?**

A: A distribution packages the Linux kernel with user-space software, package management, configuration tools and other components into a usable operating system.

---

## ⚡ Quick Revision

```text
OS
→ Manages the computer

Linux Kernel
→ Core of a Linux-based operating system

Distribution
→ Complete system built around the kernel

Terminal
→ Program that gives you access to a shell

Shell
→ Command interpreter

CLI
→ Command-Line Interface

Linux in Cybersecurity
→ Administration + investigation + monitoring + security
```

---

## ✅ Module Completion Checklist

- [ ] I can explain what an operating system is.
- [ ] I can explain Linux in simple language.
- [ ] I know what the kernel does.
- [ ] I understand kernel vs distribution.
- [ ] I know what a terminal is.
- [ ] I know what a shell is.
- [ ] I know what CLI means.
- [ ] I can name common Linux distributions.
- [ ] I understand why Linux matters for cybersecurity.

---

## 🔜 Next

➡️ [01 — Linux Fundamentals](../01-Linux-Fundamentals/README.md)
