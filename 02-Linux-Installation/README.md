# 02 — Linux Installation

> Build a safe Linux practice environment before learning administration, networking, and security.

## 🎯 Objectives

By the end of this module you should be able to:

- Choose an appropriate Linux distribution for a learning task.
- Understand bare-metal, virtual machine, WSL, live USB, and cloud installations.
- Install a Linux system safely in a virtual machine.
- Understand partitions, filesystems, swap, users, hostname, timezone, and networking during installation.
- Perform the first boot and verify that the system is healthy.
- Take a VM snapshot before experiments.
- Build a repeatable lab environment for later Linux and SOC exercises.

---

## 🧠 What Does "Installing Linux" Actually Mean?

Installing Linux means preparing a computer with several layers working together:

```text
Physical / Virtual Hardware
          ↓
       Bootloader
          ↓
        Kernel
          ↓
     Init / systemd
          ↓
   Services + User Space
          ↓
      Login / Shell
```

The Linux kernel alone is not the complete desktop/server operating environment. A distribution combines the kernel with user-space programs, package management, libraries, configuration, and other components.

---

# 🐧 Choosing a Distribution

A distribution is a packaged Linux operating system built around the Linux kernel.

| Distribution family | Good for | What to learn |
|---|---|---|
| Ubuntu / Debian | Beginner learning, servers, general administration | `apt`, `dpkg` |
| Fedora / RHEL family | Enterprise concepts | `dnf`, RPM |
| Kali Linux | Security testing and cybersecurity labs | security tooling + Debian base |
| Arch Linux | Deep hands-on Linux learning | manual configuration, `pacman` |
| Rocky / AlmaLinux | RHEL-compatible server practice | enterprise administration |

### ⭐ Recommended learning approach

Use **Ubuntu Server or Ubuntu Desktop** as the main learning machine. Add **Kali Linux** later for security-specific labs.

Do not start by using Kali for every Linux lesson. The goal is to learn Linux itself, not just security tools.

---

# 🧪 Installation Methods

## 1. Virtual Machine — ⭐ Recommended

```text
Your Windows PC
      │
      ▼
 Virtualization Software
      │
      ├── Linux VM 1
      ├── Linux VM 2
      └── Security Lab VM
```

Advantages:

- Safe experimentation
- Snapshots
- Multiple machines on one PC
- Easy networking labs
- Easy rollback

## 2. WSL

Windows Subsystem for Linux provides a Linux environment integrated with Windows.

Useful for learning commands and scripting, but it does not reproduce every aspect of a traditional Linux machine exactly.

## 3. Bare Metal

Linux is installed directly on physical hardware.

```text
Hardware
   ↓
Linux
```

Useful when you want Linux to be the primary operating system, but it requires more care with partitions, boot configuration, drivers, and backups.

## 4. Live USB

Linux boots from removable media without necessarily installing it to the internal disk.

Useful for recovery, testing hardware, and temporary environments.

## 5. Cloud VM

A remote virtual machine supplied by a cloud provider.

```text
Your PC
  │
 Internet
  │
  ▼
Cloud VM
  │
  └── Linux Server
```

Later this becomes important for server administration and SOC work.

---

# 🏗️ Recommended Personal Lab

For this knowledge base, create a dedicated Linux lab rather than changing your main computer unnecessarily.

```text
                 🧪 LINUX LAB
                     │
        ┌────────────┴────────────┐
        │                         │
   Ubuntu VM                 Kali VM
   Administration             Security
        │                         │
        └──────────┬──────────────┘
                   │
             Isolated Lab Network
```

Start with **one Ubuntu VM**. Add additional systems only when a later lab requires them.

---

# 💻 VM Planning

Before installation decide:

| Resource | Learning VM starting point |
|---|---|
| CPU | 2 virtual cores |
| RAM | 2–4 GB depending on desktop/server choice |
| Disk | 25 GB+ |
| Network | NAT initially |
| Snapshot | Take one after a clean installation |

These are practical starting points, not universal requirements. Adjust them to the host computer's available resources.

---

# 💿 Installation Concepts

## ISO Image

An ISO is a disk-image file containing installation media.

```text
Linux ISO
   ↓
Virtual CD/DVD or USB
   ↓
Boot installer
```

## Bootloader

The bootloader starts the operating system kernel.

## Partition

A partition is a defined region of a storage device.

## Filesystem

A filesystem organizes data inside a storage area.

Common Linux filesystems include:

- ext4
- XFS
- Btrfs

## Swap

Swap provides disk-backed memory space that the operating system can use under certain conditions. It is not simply a replacement for RAM.

---

# 👤 Installation User

During installation you normally create a user account.

Important distinction:

```text
Normal User
    │
    ├── Own files
    ├── Limited privileges
    └── Uses sudo when authorized

root
    │
    └── Full administrative privileges
```

### Security principle

Use a normal account for daily work and elevate privileges only when required.

---

# 🌐 Network Configuration

During installation you may encounter:

- DHCP
- Static IP
- Hostname
- DNS
- Gateway
- Network interface

Basic relationship:

```text
Linux Host
   │
   ├── IP Address
   ├── Subnet
   ├── Default Gateway
   └── DNS Server
```

Networking will be studied in depth in `15-Linux-Networking`.

---

# 🚀 First Boot Checklist

After installation, do not immediately start changing random configuration.

First verify:

```bash
whoami
hostname
pwd
uname -a
cat /etc/os-release
ip addr
ip route
```

Then test connectivity:

```bash
ping -c 4 127.0.0.1
ping -c 4 8.8.8.8
```

If the second test works but DNS names do not, test DNS separately:

```bash
ping -c 4 example.com
```

Do not assume that every failed `ping` means the network is broken; some hosts and networks block ICMP.

---

# 🔄 Update the New System

On a Debian/Ubuntu-based system:

```bash
sudo apt update
sudo apt upgrade
```

Understand the difference:

```text
apt update
    ↓
Refresh package information

apt upgrade
    ↓
Install available package updates
```

Do not blindly copy package commands between Linux distributions. Package managers differ.

---

# 📸 Snapshot Strategy

A snapshot is extremely useful for learning.

Recommended checkpoints:

```text
Fresh VM
   ↓
Install updates
   ↓
Configure basic tools
   ↓
📸 Snapshot: clean-lab
   ↓
Experiment
   ↓
Break something 😄
   ↓
Rollback
```

Snapshots are a lab convenience, not a substitute for backups.

---

# 🔐 Installation Security Checklist

- Use a strong password.
- Avoid using `root` for normal work.
- Keep the system updated.
- Do not expose services to the internet unnecessarily.
- Use NAT or an isolated lab network while learning.
- Do not install unknown scripts just because a tutorial tells you to.
- Keep VM snapshots before risky configuration changes.
- Never put real credentials, API keys, or secrets into a public repository.

---

# 🧪 Hands-On Lab 01 — Install Ubuntu in a VM

### Objective

Create your first repeatable Linux learning environment.

### Tasks

1. Create a Linux VM.
2. Attach the official Linux ISO.
3. Install the operating system.
4. Create a normal user.
5. Set the hostname.
6. Complete the first boot.
7. Update the system.
8. Verify CPU, memory, disk, network, and OS information.
9. Create a clean snapshot.

### Verification

```bash
whoami
hostnamectl
uname -r
cat /etc/os-release
free -h
df -h
ip addr
ip route
```

### Expected Result

You should have a working Linux VM that you can safely break and rebuild while following this repository.

---

# 🧪 Hands-On Lab 02 — Break and Recover

After creating your clean snapshot:

1. Change a harmless configuration.
2. Record what you changed.
3. Verify the change.
4. Roll back to the snapshot.
5. Verify the original state.

### Learning outcome

You learn that administration includes both **making changes** and **recovering from changes**.

---

# 🎯 Challenge

Without looking at notes, answer:

1. What is an ISO?
2. What is a partition?
3. What is a filesystem?
4. What is swap?
5. Why should you avoid daily work as root?
6. What is a VM snapshot?
7. What is the difference between `apt update` and `apt upgrade`?
8. Why is NAT useful for a beginner lab?
9. What information does `/etc/os-release` provide?
10. What is the purpose of a hostname?

---

# 🔐 Cybersecurity Connection

A security analyst often works with Linux systems that are:

- Servers
- Cloud workloads
- Web servers
- SIEM infrastructure
- Security tools
- Containers
- Appliances
- Investigation environments

Therefore, installation knowledge matters because security work depends on understanding the system underneath the security tools.

---

# 🎤 Interview Questions

### Beginner

**Q: What is Linux?**  
A: Linux commonly refers to operating systems built around the Linux kernel together with user-space software and distribution tooling.

**Q: What is a Linux distribution?**  
A: A distribution packages the Linux kernel with system software, libraries, package management, configuration, and other components into a usable operating system.

**Q: What is a VM?**  
A: A virtual machine is an isolated software-defined computer running on a physical host through virtualization.

### Intermediate

**Q: Why are VMs useful for Linux learning?**  
A: They allow isolated experimentation, snapshots, multiple systems, and repeatable labs without modifying the host operating system.

**Q: Why is Kali not the best choice for learning every Linux fundamental?**  
A: Kali is designed primarily for penetration testing and security work. A general-purpose distribution makes it easier to focus on core Linux administration without confusing Linux fundamentals with specialized security tooling.

---

# ⚡ Quick Revision

```text
ISO        → Installation image
VM         → Virtual computer
Partition  → Storage region
Filesystem → Organizes data
Swap       → Disk-backed memory area
Hostname   → System's network identity/name
DHCP       → Automatic network configuration
root       → Superuser account
sudo       → Controlled privilege elevation
Snapshot   → VM state checkpoint
```

---

# ✅ Module Checklist

- [ ] Understand Linux installation concepts
- [ ] Understand distributions
- [ ] Understand VM vs WSL vs bare metal
- [ ] Create a Linux VM
- [ ] Create a normal user
- [ ] Verify the first boot
- [ ] Update the system
- [ ] Understand basic networking during installation
- [ ] Create a clean snapshot
- [ ] Complete the recovery lab
