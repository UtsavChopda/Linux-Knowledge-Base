# 🚀 04 — Linux Boot Process and systemd

> Understand what happens from pressing the power button to getting a usable Linux system, and learn how `systemd` manages services and the system after boot.

---

## 🎯 Learning Objectives

By the end of this module, you should be able to:

- Explain the Linux boot process in simple terms
- Understand BIOS/UEFI, bootloader, kernel and initramfs
- Explain what GRUB does
- Understand the role of the Linux kernel during boot
- Understand `initramfs`
- Explain PID 1
- Understand `systemd`
- Start, stop, restart and inspect services
- Enable and disable services at boot
- Read service status and logs
- Understand targets
- Troubleshoot common boot and service problems
- Connect systemd and boot logs to Linux security investigations

---

# 🧠 1. What Happens When a Linux Computer Starts?

Think about booting a Linux computer like starting a company:

```text
🖥️ Power ON
     │
     ▼
⚡ Firmware (BIOS / UEFI)
     │
     ▼
📦 Bootloader (GRUB)
     │
     ▼
🐧 Linux Kernel
     │
     ▼
🧰 initramfs
     │
     ▼
👑 PID 1 (systemd)
     │
     ▼
⚙️ Services / Targets
     │
     ▼
🖥️ Login / Shell / Desktop
```

The exact implementation varies by distribution and configuration, but this is the mental model you should remember.

---

# 🔌 2. Stage 1 — Power On

When the machine receives power, the CPU starts executing firmware instructions.

Modern systems commonly use **UEFI**. Older systems may use legacy **BIOS**.

The firmware performs early hardware initialization and determines how to continue booting.

### Simple idea

```text
Firmware
   ↓
Find a bootable device
   ↓
Load the bootloader
```

---

# 🧩 3. BIOS vs UEFI

| BIOS | UEFI |
|---|---|
| Older firmware interface | Modern firmware interface |
| Common on older systems | Standard on modern systems |
| Traditionally associated with MBR booting | Commonly used with GPT |
| More limited environment | More capable firmware environment |

### Important

Do not treat BIOS and UEFI as operating systems. They are **firmware interfaces** responsible for initializing hardware and starting the boot process.

---

# 📦 4. Stage 2 — Bootloader

A bootloader loads the operating system kernel.

On many Linux installations, the bootloader is **GRUB (GRand Unified Bootloader)**.

GRUB can:

- Display boot entries
- Select an operating system/kernel
- Pass parameters to the kernel
- Load the Linux kernel
- Load the initial RAM filesystem

Visual model:

```text
UEFI / BIOS
     │
     ▼
   GRUB
     │
     ├── Linux Kernel
     └── initramfs
```

---

# 🐧 5. Stage 3 — Linux Kernel

The kernel is the core of the Linux operating system.

During boot it begins initializing the system and managing hardware/resources.

The kernel is responsible for areas such as:

- CPU scheduling
- Memory management
- Device management
- Networking
- Filesystems
- Security mechanisms
- Process management

The kernel is **not** the same thing as the complete Linux distribution.

```text
Linux Distribution
├── Kernel
├── User-space programs
├── Libraries
├── Package manager
├── Services
└── Applications
```

---

# 🧰 6. What Is initramfs?

`initramfs` means **initial RAM filesystem**.

It provides a temporary filesystem and tools needed during the early boot stage, before the real root filesystem is ready.

For example, a system may need:

- Storage drivers
- Filesystem support
- RAID/LVM support
- Disk unlocking components
- Scripts required to locate the root filesystem

Mental model:

```text
Kernel starts
    │
    ▼
initramfs
    │
    ├── Load required drivers
    ├── Prepare storage
    ├── Find root filesystem
    └── Prepare transition
            │
            ▼
      Real root filesystem
```

---

# 👑 7. PID 1

After the kernel initializes enough of the system, the first user-space process is started.

This process receives **PID 1**.

On most modern Linux distributions using systemd:

```text
PID 1 = systemd
```

Check it:

```bash
ps -p 1 -o pid,comm,args
```

Another useful command:

```bash
systemctl status
```

### Why PID 1 matters

PID 1 is responsible for coordinating important user-space initialization and service management.

If PID 1 is unavailable or fails in a normally running system, the system can become seriously impaired.

---

# ⚙️ 8. What Is systemd?

`systemd` is a system and service manager used by many modern Linux distributions.

It can manage:

- Services
- Startup processes
- Targets
- Mounts
- Timers
- Sockets
- Dependencies
- Logging through the systemd journal

A useful mental model:

```text
                 systemd
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
   Services      Targets       Timers
       │            │
       ▼            ▼
   sshd          multi-user
   cron          graphical
   firewall
```

---

# 🔧 9. systemctl

`systemctl` is the primary command-line interface for interacting with systemd.

## Check system state

```bash
systemctl status
```

## Check a service

```bash
systemctl status ssh
```

On some distributions the service may be named `sshd` instead:

```bash
systemctl status sshd
```

## Start a service

```bash
sudo systemctl start ssh
```

## Stop a service

```bash
sudo systemctl stop ssh
```

## Restart a service

```bash
sudo systemctl restart ssh
```

## Reload configuration

```bash
sudo systemctl reload ssh
```

A reload is different from a restart. When supported, reload asks the service to reread its configuration without completely stopping the service.

---

# 🔁 10. Enable vs Start

This is a very important beginner concept.

### Start

Starts the service **now**.

```bash
sudo systemctl start ssh
```

### Enable

Configures the service to start automatically during the appropriate boot process.

```bash
sudo systemctl enable ssh
```

### Both

```bash
sudo systemctl enable --now ssh
```

Mental model:

```text
start  → NOW
stop   → NOW
restart → NOW

enable  → FUTURE BOOTS

disable → FUTURE BOOTS
```

---

# 🔎 11. Inspecting Services

Useful commands:

```bash
systemctl status ssh
systemctl is-active ssh
systemctl is-enabled ssh
systemctl is-failed ssh
```

List running services:

```bash
systemctl --type=service --state=running
```

List failed units:

```bash
systemctl --failed
```

Show service configuration:

```bash
systemctl cat ssh
```

Show dependencies:

```bash
systemctl list-dependencies ssh
```

---

# 🎯 12. systemd Targets

A **target** groups units together to represent a particular system state.

Common examples include:

```text
multi-user.target
   → Multi-user system without requiring a graphical desktop

 graphical.target
   → Graphical desktop environment
```

Check the current default target:

```bash
systemctl get-default
```

Change the default target carefully:

```bash
sudo systemctl set-default multi-user.target
```

> ⚠️ Changing the default target can change how your machine boots. Perform this only in a lab when learning.

---

# 📝 13. Linux Boot Sequence — Full Picture

```text
┌──────────────────────┐
│     Power ON         │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│     BIOS / UEFI      │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│        GRUB          │
│      Bootloader      │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│   Linux Kernel       │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│      initramfs       │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│   Root Filesystem    │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│    systemd / PID 1   │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Targets + Services   │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│   Login / Shell / UI │
└──────────────────────┘
```

---

# 📋 14. Important Commands Cheat Sheet

| Purpose | Command |
|---|---|
| Systemd status | `systemctl status` |
| Service status | `systemctl status SERVICE` |
| Start | `systemctl start SERVICE` |
| Stop | `systemctl stop SERVICE` |
| Restart | `systemctl restart SERVICE` |
| Reload | `systemctl reload SERVICE` |
| Enable at boot | `systemctl enable SERVICE` |
| Disable at boot | `systemctl disable SERVICE` |
| Enable + start | `systemctl enable --now SERVICE` |
| Running services | `systemctl --type=service --state=running` |
| Failed units | `systemctl --failed` |
| Current target | `systemctl get-default` |
| Current system state | `systemctl is-system-running` |
| Service dependencies | `systemctl list-dependencies SERVICE` |
| Unit definition | `systemctl cat SERVICE` |
| Boot time analysis | `systemd-analyze` |
| Critical startup chain | `systemd-analyze critical-chain` |
| List booted units | `systemctl list-units` |

---

# ⏱️ 15. Boot Performance

Check how long the system took to boot:

```bash
systemd-analyze
```

View the critical startup path:

```bash
systemd-analyze critical-chain
```

This can help identify services or dependencies contributing to slow startup.

---

# 📜 16. Boot and Service Logs

systemd commonly works with the **journal**, which can be queried using `journalctl`.

View recent logs:

```bash
journalctl -n 50
```

View logs for a service:

```bash
journalctl -u ssh
```

Follow logs live:

```bash
journalctl -u ssh -f
```

View logs from the current boot:

```bash
journalctl -b
```

View kernel messages available through the journal:

```bash
journalctl -k
```

Security relevance:

```text
Event
 ↓
Journal
 ↓
journalctl
 ↓
Investigation
 ↓
Timeline / Detection / Response
```

---

# 🔐 17. Cybersecurity Perspective

Boot and systemd knowledge is important for defenders because attackers may attempt persistence through services, timers, startup configuration or other mechanisms.

When investigating a suspicious Linux host, ask:

```text
What services are running?
        ↓
Which services start automatically?
        ↓
Were any units recently changed?
        ↓
What processes do they launch?
        ↓
What user runs them?
        ↓
What network connections do they create?
        ↓
What logs support the timeline?
```

Useful defensive commands include:

```bash
systemctl --type=service --state=running
systemctl --failed
systemctl list-unit-files
journalctl -b
journalctl --since "1 hour ago"
ps aux
ss -tulpn
```

> Use these commands for systems you own or are authorized to investigate.

---

# 🧪 18. Hands-On Lab — Service Management

## 🎯 Objective

Practice starting, stopping, restarting and inspecting a Linux service.

### Tasks

1. Identify whether SSH is installed.
2. Check its service status.
3. Determine whether it is active.
4. Determine whether it is enabled at boot.
5. Restart the service in your lab.
6. View its recent journal entries.
7. Find its listening port.

### Suggested commands

```bash
systemctl status ssh
systemctl is-active ssh
systemctl is-enabled ssh
journalctl -u ssh -n 30
ss -lntp
```

If your distribution uses `sshd`, substitute `sshd` for `ssh`.

---

# 🧪 19. Hands-On Lab — Boot Investigation

Run:

```bash
systemd-analyze
systemd-analyze critical-chain
systemctl --failed
journalctl -b -p warning
```

### Questions

- How long did your system take to boot?
- Which component appears on the critical chain?
- Are any units failed?
- Are there warnings from the current boot?
- Can you explain each finding in simple language?

---

# 🚨 20. Troubleshooting Scenario

### Problem

A web server is unreachable after reboot.

Do not immediately restart everything.

Follow this process:

```text
Is the machine running?
        ↓
Is the network working?
        ↓
Is the service running?
        ↓
Is the service enabled?
        ↓
Is the expected port listening?
        ↓
Is the firewall allowing it?
        ↓
What do the service logs say?
```

Useful tools:

```bash
systemctl status SERVICE
systemctl is-enabled SERVICE
ss -lntp
journalctl -u SERVICE
ip addr
ip route
```

---

# ⚠️ Common Beginner Mistakes

### Mistake 1: Confusing `start` and `enable`

Starting a service does not necessarily make it start after the next reboot.

### Mistake 2: Restarting without investigating

A restart may temporarily hide the symptom and remove useful context from an investigation.

### Mistake 3: Assuming every Linux system uses systemd

Many modern mainstream distributions use systemd, but Linux systems can use other init/service-management systems.

### Mistake 4: Assuming the SSH service is always named `ssh`

Depending on the distribution, the service may be called `ssh` or `sshd`.

### Mistake 5: Treating the kernel as the entire operating system

The kernel is the core component; a Linux distribution contains the kernel plus user-space components and applications.

---

# 🎤 21. Interview Questions

## 🟢 Beginner

1. What happens when a Linux system boots?
2. What is BIOS?
3. What is UEFI?
4. What is GRUB?
5. What is the Linux kernel?
6. What is initramfs?
7. What is PID 1?
8. What is systemd?

## 🟡 Intermediate

1. What is the difference between `systemctl start` and `systemctl enable`?
2. What is a systemd target?
3. How do you check whether a service is running?
4. How do you find failed systemd units?
5. How do you inspect service logs?
6. How would you troubleshoot a service that fails during boot?

## 🔴 Advanced

1. Explain the complete Linux boot sequence.
2. Why is PID 1 important?
3. What is initramfs used for?
4. How would you investigate a suspicious systemd service?
5. How can systemd logs help establish an incident timeline?

---

# ⚡ Quick Revision

```text
BIOS/UEFI
   ↓
GRUB
   ↓
Kernel
   ↓
initramfs
   ↓
Root filesystem
   ↓
systemd (PID 1)
   ↓
Targets
   ↓
Services
   ↓
Login
```

Remember:

```text
start   = start now
stop    = stop now
restart = restart now
enable  = start automatically at boot
disable = don't start automatically at boot
```

Core commands:

```bash
systemctl status SERVICE
systemctl start SERVICE
systemctl stop SERVICE
systemctl restart SERVICE
systemctl enable SERVICE
systemctl disable SERVICE
systemctl --failed
journalctl -u SERVICE
journalctl -b
systemd-analyze
```

---

# ✅ Module Checklist

- [ ] I can explain BIOS/UEFI
- [ ] I understand what GRUB does
- [ ] I can explain the kernel's role
- [ ] I understand initramfs
- [ ] I know what PID 1 means
- [ ] I understand systemd
- [ ] I can manage a service with systemctl
- [ ] I understand start vs enable
- [ ] I can inspect service logs
- [ ] I can investigate failed units
- [ ] I can explain the boot sequence
- [ ] I understand why boot/service management matters to cybersecurity

---

## ➡️ Next Module

**05 — Linux Command Line**

We will start working heavily with the terminal and build command-line skills from absolute zero.