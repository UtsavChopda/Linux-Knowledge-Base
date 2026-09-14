# 📦 12 — Package Management

> **Goal:** Learn how Linux installs, updates, verifies, removes and troubleshoots software — and understand why package management is an important administration and cybersecurity skill.

---

## 🎯 Learning Objectives

By the end of this module, you should be able to:

- Explain what a Linux package is
- Understand repositories and package managers
- Understand package metadata and dependencies
- Use APT on Debian/Ubuntu systems
- Understand DNF/RPM on Fedora/RHEL-family systems
- Search for packages
- Install, update and remove software
- Inspect installed packages
- Verify package ownership of files
- Understand package caches and repository metadata
- Troubleshoot dependency and repository problems
- Understand package signatures and trust
- Audit software installed on a Linux host
- Recognize package-management activity as a useful SOC signal

---

# 1. What Is Software on Linux?

Linux applications are usually distributed as **packages** rather than as one isolated executable.

A package can contain:

```text
PACKAGE
  │
  ├── Executable files
  ├── Libraries
  ├── Configuration files
  ├── Documentation
  ├── Metadata
  ├── Dependencies information
  └── Installation scripts/hooks
```

A package manager handles the work of installing and maintaining these components.

---

# 2. What Is a Package?

A package is a structured software bundle prepared for a Linux distribution's package-management system.

Examples of package formats:

```text
Debian / Ubuntu → .deb
Fedora / RHEL   → .rpm
```

A package normally contains metadata describing things such as:

- package name
- version
- architecture
- dependencies
- files included
- maintainer information
- package description

---

# 3. Why Use a Package Manager?

Without a package manager, installing software manually can become difficult.

You would have to:

```text
Find software
    ↓
Download it
    ↓
Find dependencies
    ↓
Install dependencies
    ↓
Copy files
    ↓
Configure software
    ↓
Track installed files
    ↓
Update manually
    ↓
Remove manually
```

A package manager automates much of this.

```text
                PACKAGE MANAGER
                       │
       ┌───────────────┼───────────────┐
       ↓               ↓               ↓
    Install         Update          Remove
       │               │               │
       └───────────────┼───────────────┘
                       ↓
                  Dependencies
                       ↓
                  Package DB
```

---

# 4. Package Manager vs Repository

These terms are related but different.

### Package manager

The tool that manages software packages.

Examples:

```text
apt
apt-get
dnf
rpm
```

### Repository

A source containing packages and metadata that the package manager can use.

Conceptually:

```text
Repository
   │
   ├── package A
   ├── package B
   ├── package C
   └── metadata
          ↓
     Package Manager
          ↓
        System
```

---

# 5. Why Repositories Matter for Security

A repository is part of the software supply chain.

When installing software, you should care about:

```text
Where did the package come from?
        ↓
Was the source trusted?
        ↓
Is the package authentic?
        ↓
Has it been tampered with?
        ↓
Is the version maintained?
```

For production systems, prefer trusted distribution/vendor repositories and approved organizational sources.

Avoid blindly installing packages from random websites.

---

# 6. APT — Debian and Ubuntu

On Debian-based systems, **APT** is the high-level package-management interface.

Check availability:

```bash
apt --version
```

Useful commands:

```bash
apt update
apt upgrade
apt install
apt remove
apt purge
apt search
apt show
```

Most administrative package operations require root privileges, so you will commonly use `sudo`.

---

# 7. `apt update`

Run:

```bash
sudo apt update
```

This refreshes the local package metadata from configured repositories.

Important distinction:

```text
apt update
     ↓
Refresh package information
```

It does **not** normally upgrade every installed package.

Think:

```text
UPDATE REPOSITORY INFORMATION
```

not:

```text
UPDATE INSTALLED SOFTWARE
```

---

# 8. `apt upgrade`

To upgrade installed packages where the normal upgrade rules allow it:

```bash
sudo apt upgrade
```

Typical workflow:

```bash
sudo apt update
sudo apt upgrade
```

Conceptually:

```text
Repository metadata
       ↓
Find newer versions
       ↓
Resolve dependencies
       ↓
Upgrade packages
```

Review what will change before confirming on important systems.

---

# 9. Installing a Package

Example:

```bash
sudo apt install curl
```

APT may install additional dependencies.

```text
curl
 │
 ├── dependency A
 ├── dependency B
 └── dependency C
```

The package manager resolves these relationships according to repository metadata and package constraints.

---

# 10. Searching for a Package

```bash
apt search nginx
```

This searches available package metadata.

You can inspect package information:

```bash
apt show nginx
```

Useful information may include:

- version
- architecture
- dependencies
- description
- package source

---

# 11. Removing a Package

Remove the package:

```bash
sudo apt remove PACKAGE
```

`remove` generally removes the package while leaving some configuration files behind.

### Purge

```bash
sudo apt purge PACKAGE
```

This also removes package-managed configuration files where applicable.

Do not assume purge removes every file the software ever created; application data may be stored elsewhere.

---

# 12. Find Installed Packages

On Debian/Ubuntu:

```bash
apt list --installed
```

Search installed packages:

```bash
apt list --installed 2>/dev/null | grep nginx
```

You can also query the dpkg database:

```bash
dpkg -l
```

---

# 13. `dpkg` vs `apt`

This distinction is important.

```text
APT
 │
 ├── Repository handling
 ├── Dependency resolution
 └── High-level package management

DPKG
 │
 └── Low-level Debian package database / package operations
```

Inspect an installed package:

```bash
dpkg -s PACKAGE
```

List files installed by a package:

```bash
dpkg -L PACKAGE
```

Find which package owns a file:

```bash
dpkg -S /path/to/file
```

---

# 14. Why File Ownership Matters

Suppose you find an unexpected executable:

```bash
/usr/bin/example
```

Ask:

> Which package installed this file?

On Debian/Ubuntu:

```bash
dpkg -S /usr/bin/example
```

This can help distinguish:

```text
Distribution-managed file
        vs
Unexpected/unmanaged file
```

This is a useful defensive investigation technique.

---

# 15. Package Verification on Debian Systems

You can inspect package status with:

```bash
dpkg -s PACKAGE
```

For deeper verification, Debian-family tooling can compare installed file metadata against package metadata.

For example:

```bash
dpkg -V PACKAGE
```

If a file differs, investigate why.

A difference does **not automatically prove compromise**. Configuration changes, legitimate updates and application behavior can all explain differences.

---

# 16. APT Package Cache

Downloaded `.deb` files may be stored in:

```text
/var/cache/apt/archives/
```

Inspect:

```bash
ls -lh /var/cache/apt/archives/
```

Clean unused package cache when appropriate:

```bash
sudo apt clean
```

Be aware that cleaning cache changes local evidence, so during a security investigation do not casually remove artifacts before following your incident-response process.

---

# 17. APT Sources

Repository configuration is commonly found under:

```text
/etc/apt/sources.list
/etc/apt/sources.list.d/
```

Inspect configured sources:

```bash
grep -Rhv '^#' /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null
```

Look for:

- trusted distribution repositories
- approved organizational repositories
- unexpected third-party sources
- obsolete distributions/releases
- unusual repository URLs

Repository configuration is part of the host's software supply chain.

---

# 18. DNF — Fedora and RHEL Family

Modern Fedora and RHEL-family systems commonly use **DNF** as the high-level package manager.

Check:

```bash
dnf --version
```

Common commands:

```bash
sudo dnf check-update
sudo dnf install PACKAGE
sudo dnf update
sudo dnf remove PACKAGE
sudo dnf search PACKAGE
sudo dnf info PACKAGE
```

Exact commands and repository configuration can vary by distribution/version.

---

# 19. `dnf install`

Example:

```bash
sudo dnf install nginx
```

DNF resolves dependencies and installs the required packages from configured repositories.

---

# 20. Updating with DNF

```bash
sudo dnf update
```

Review proposed changes carefully, particularly on servers and production systems.

Check for available updates:

```bash
dnf check-update
```

A non-zero exit status from `dnf check-update` can indicate that updates are available; don't treat every non-zero status as a failure without checking the command's documented behavior.

---

# 21. Searching with DNF

```bash
dnf search nginx
```

Inspect package information:

```bash
dnf info nginx
```

List installed packages:

```bash
dnf list installed
```

---

# 22. RPM — Low-Level Package Tool

RPM is the package format and low-level package-management tooling used across the RPM ecosystem.

Check:

```bash
rpm --version
```

Query an installed package:

```bash
rpm -q PACKAGE
```

Show package information:

```bash
rpm -qi PACKAGE
```

List package files:

```bash
rpm -ql PACKAGE
```

Find the package owning a file:

```bash
rpm -qf /path/to/file
```

---

# 23. DNF vs RPM

Similar to APT vs DPKG:

```text
DNF
 │
 ├── Repositories
 ├── Dependency resolution
 └── High-level management

RPM
 │
 └── Package database + low-level package operations
```

For normal software installation on modern Fedora/RHEL-family systems, use the distribution's recommended high-level tool such as DNF.

---

# 24. Package Dependencies

Applications depend on other software components.

Example:

```text
Application
    │
    ├── Library A
    ├── Library B
    │      └── Library C
    └── Runtime
```

Package managers maintain dependency relationships.

This prevents you from manually hunting for every library in many common situations.

---

# 25. Dependency Problems

You may encounter errors such as:

```text
unmet dependencies
held packages
conflicting packages
package not found
broken packages
```

A good troubleshooting approach is:

```text
Read exact error
      ↓
Check repository configuration
      ↓
Refresh metadata
      ↓
Inspect package versions
      ↓
Check dependencies
      ↓
Check held/pinned packages
      ↓
Repair according to distribution guidance
```

Do not randomly remove packages to make an error disappear.

---

# 26. Package Versioning

A package version tells you which release/build is installed.

Check on Debian/Ubuntu:

```bash
dpkg -s PACKAGE | grep '^Version:'
```

Or:

```bash
apt policy PACKAGE
```

On RPM systems:

```bash
rpm -q PACKAGE
```

Version information matters for:

- compatibility
- bug fixes
- security patches
- vulnerability assessment
- incident investigation

---

# 27. Security Updates

Software vulnerabilities are often fixed by package updates.

Conceptually:

```text
Vulnerability discovered
        ↓
Vendor develops fix
        ↓
Package released
        ↓
Repository publishes update
        ↓
Administrator updates system
        ↓
Vulnerable version replaced
```

Therefore patch management is an important security control.

---

# 28. Package Management and CVEs

Suppose a SOC analyst receives an alert for a vulnerable version of a package.

Useful questions:

```text
Is the package installed?
        ↓
Which version?
        ↓
Is it actually vulnerable?
        ↓
Is a fixed version available?
        ↓
Can it be updated safely?
```

Package inventory is therefore useful for vulnerability management.

A CVE match alone does not always mean the host is exploitable; applicability, configuration, architecture and vendor backports matter.

---

# 29. Vendor Backports

A distribution may apply a security fix to an older upstream version while keeping the distribution's versioning scheme.

Therefore:

> **Do not judge vulnerability only by comparing the upstream version string.**

Use the distribution's security advisories/package metadata when assessing whether a package is patched.

This is an important real-world Linux security concept.

---

# 30. Package Signatures and Trust

Packages may be cryptographically signed by a distribution/vendor signing key.

Conceptually:

```text
Package
   │
   ↓
Cryptographic signature
   │
   ↓
Trusted key
   │
   ↓
Package manager verifies authenticity/integrity
```

This helps protect the software supply chain from unauthorized modification.

The exact trust mechanism varies by distribution and repository.

---

# 31. Why `curl | bash` Deserves Caution

You may see installation instructions like:

```bash
curl https://example.com/install.sh | bash
```

This executes downloaded content directly through a shell.

That can be dangerous if the source is untrusted or compromised.

A safer learning approach is to inspect the script first:

```bash
curl -O https://example.com/install.sh
less install.sh
```

Then verify the source, integrity and instructions before executing anything.

For production environments, follow approved software-deployment processes.

---

# 32. Package Installation as a SOC Signal

Package-management activity can be important during an investigation.

For example:

```text
Unexpected package installation
          ↓
What package?
          ↓
Which user performed it?
          ↓
When?
          ↓
From which repository?
          ↓
What files were installed?
          ↓
What service/process appeared afterward?
```

A package installation is not automatically malicious.

It becomes more interesting when combined with other indicators.

---

# 33. Investigating an Unexpected Package

Suppose you find an unfamiliar program.

### Step 1 — Identify executable

```bash
command -v PROGRAM
```

### Step 2 — Identify package ownership

Debian/Ubuntu:

```bash
dpkg -S /path/to/program
```

RPM family:

```bash
rpm -qf /path/to/program
```

### Step 3 — Inspect package

```bash
dpkg -s PACKAGE
```

or:

```bash
rpm -qi PACKAGE
```

### Step 4 — Check version

```bash
apt policy PACKAGE
```

or:

```bash
rpm -q PACKAGE
```

### Step 5 — Check repository/source

Review configured repositories and package metadata.

### Step 6 — Correlate with processes/services

```bash
ps aux | grep PROGRAM
systemctl status SERVICE
```

### Step 7 — Review logs

```bash
journalctl --since "2 hours ago"
```

---

# 34. Package Files and Configuration

A package may install files into locations such as:

```text
/usr/bin/
/usr/sbin/
/usr/lib/
/etc/
/usr/share/
/var/
```

The exact locations depend on the package and distribution.

Use the package manager to discover what a package owns rather than assuming a path.

---

# 35. Package Database

The package manager maintains local metadata about installed software.

Conceptually:

```text
Installed Software
       │
       ↓
Package Database
       │
       ├── Name
       ├── Version
       ├── Files
       ├── Dependencies
       └── Status
```

This database is extremely useful for inventory and forensic triage.

---

# 36. Package Inventory

A basic Linux software inventory should capture:

```text
Package name
Version
Architecture
Installation source
Repository
Installation/update time where available
```

A vulnerability-management system can then compare inventory against known security advisories.

---

# 37. Useful Debian Commands

```bash
apt update
apt upgrade
apt search PACKAGE
apt show PACKAGE
apt policy PACKAGE
apt list --installed
dpkg -l
dpkg -s PACKAGE
dpkg -L PACKAGE
dpkg -S /path/to/file
dpkg -V PACKAGE
```

---

# 38. Useful RPM/DNF Commands

```bash
dnf check-update
dnf update
dnf search PACKAGE
dnf info PACKAGE
dnf list installed
rpm -q PACKAGE
rpm -qi PACKAGE
rpm -ql PACKAGE
rpm -qf /path/to/file
```

---

# 39. Troubleshooting: `Package Not Found`

Example:

```text
E: Unable to locate package example
```

Check:

```text
1. Is the package name correct?
2. Is the repository configured?
3. Has metadata been refreshed?
4. Is the package available for this release?
5. Is the package architecture supported?
```

On Debian/Ubuntu, start by checking:

```bash
sudo apt update
apt search example
```

Do not immediately download a random `.deb` from the internet.

---

# 40. Troubleshooting: Broken Dependencies

If dependencies are broken, first inspect the exact package-manager output.

On Debian-family systems, package-management tools provide mechanisms for repairing package state; use the appropriate documented repair operation for the distribution/version.

Useful diagnostic commands include:

```bash
dpkg --audit
apt policy PACKAGE
apt-cache policy PACKAGE
```

The goal is to understand **which dependency is broken and why** before changing the system.

---

# 41. Troubleshooting: Repository Errors

Common causes:

- network failure
- DNS failure
- repository unavailable
- incorrect repository URL
- expired/invalid metadata
- unsupported distribution release
- proxy configuration
- signature/trust problems

Investigate systematically:

```text
Network
  ↓
DNS
  ↓
Repository URL
  ↓
Repository availability
  ↓
Trust/signature
  ↓
Package metadata
```

---

# 42. Troubleshooting: Disk Full During Update

Package operations can fail when storage is exhausted.

Check:

```bash
df -h
```

Find large directories carefully:

```bash
du -sh /var/* 2>/dev/null | sort -h
```

Check inode exhaustion too:

```bash
df -i
```

Remember:

```text
Disk space full ≠ inode exhaustion
```

Both can prevent software installation.

---

# 43. Practical SOC Workflow — Software Inventory

```text
Unknown software
       ↓
Locate executable
       ↓
Identify package owner
       ↓
Identify package/version
       ↓
Identify repository/source
       ↓
Check installation/update history
       ↓
Check related services/processes
       ↓
Review logs
       ↓
Compare against baseline
       ↓
Assess risk
```

This connects package management with:

- asset inventory
- vulnerability management
- incident response
- change management
- threat hunting

---

# 🧪 Lab 1 — Package Manager Basics

## Objective

Learn the package-management workflow on your Linux VM.

### Debian/Ubuntu

```bash
apt --version
sudo apt update
apt search curl
apt show curl
```

### RPM-family

```bash
dnf --version
dnf search curl
dnf info curl
```

### Questions

- Which package manager does your system use?
- Which repository provides the package?
- What version is available?
- What dependencies are listed?

---

# 🧪 Lab 2 — Install and Inspect a Safe Utility

Choose a common utility such as `tree` if it is available in your distribution repository.

### Install

Debian/Ubuntu:

```bash
sudo apt install tree
```

RPM-family:

```bash
sudo dnf install tree
```

### Locate it

```bash
command -v tree
```

### Identify package ownership

Debian/Ubuntu:

```bash
dpkg -S "$(command -v tree)"
```

RPM-family:

```bash
rpm -qf "$(command -v tree)"
```

### List package files

Debian/Ubuntu:

```bash
dpkg -L tree
```

RPM-family:

```bash
rpm -ql tree
```

### Remove it after the lab

Use the appropriate package manager:

```bash
sudo apt remove tree
```

or:

```bash
sudo dnf remove tree
```

---

# 🧪 Lab 3 — Package-to-File Investigation

## Scenario

You discover an unfamiliar executable on your authorized Linux lab machine.

### Tasks

1. Locate the executable.
2. Determine which package owns it.
3. Determine package version.
4. List other files installed by the package.
5. Determine whether the package is from an expected repository.

### Useful commands

```bash
command -v PROGRAM
dpkg -S /path/to/file
dpkg -s PACKAGE
dpkg -L PACKAGE
```

or:

```bash
rpm -qf /path/to/file
rpm -qi PACKAGE
rpm -ql PACKAGE
```

### SOC question

> What additional evidence would you collect before deciding that the software is malicious?

---

# 🧪 Lab 4 — Repository Audit

## Objective

Understand where your machine gets software.

### Debian/Ubuntu

```bash
cat /etc/apt/sources.list
ls -la /etc/apt/sources.list.d/
grep -Rhv '^#' /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null
```

### RPM-family

Inspect configured repositories with the distribution's DNF repository commands, for example:

```bash
dnf repolist
```

### Questions

- Which repositories are configured?
- Which are official/vendor sources?
- Are there third-party repositories?
- Are all repositories expected for this machine?

---

# 🧪 Lab 5 — Vulnerability Triage Scenario

## Scenario

A scanner reports that a Linux host has an outdated package.

### Investigation

```text
Scanner alert
     ↓
Is package installed?
     ↓
What exact version?
     ↓
What distribution release?
     ↓
Is a vendor-fixed version available?
     ↓
Is the reported vulnerability applicable?
     ↓
Can the package be safely updated?
```

### Evidence to collect

```bash
uname -a
cat /etc/os-release
apt policy PACKAGE
```

or:

```bash
cat /etc/os-release
rpm -q PACKAGE
```

### Goal

Do not simply answer:

> “The version looks old, therefore vulnerable.”

Determine the actual distribution/package security status.

---

# 🛠️ Mini Project — Linux Software Inventory Auditor

Create a Bash script that generates a basic software inventory report.

Suggested output:

```text
====================================
     LINUX SOFTWARE INVENTORY
====================================
Hostname: ...
OS: ...
Kernel: ...

Package Manager: ...

Installed Packages
------------------
Name      Version
...

Repository Configuration
------------------------
...

Security Review Notes
---------------------
Unexpected repositories: ...
Unknown packages: ...
```

### Project extensions

- Detect Debian vs RPM-family systems
- Export CSV
- Compare two inventory snapshots
- Highlight version changes
- Report packages installed outside approved repositories
- Integrate CVE data later through an authorized vulnerability-management source
- Produce a SOC-friendly change report

---

# 🔐 SOC Investigation Cheat Sheet

### Identify OS

```bash
cat /etc/os-release
uname -a
```

### Debian/Ubuntu

```bash
apt list --installed
dpkg -l
dpkg -s PACKAGE
dpkg -L PACKAGE
dpkg -S /path/to/file
apt policy PACKAGE
```

### RPM-family

```bash
dnf list installed
rpm -q PACKAGE
rpm -qi PACKAGE
rpm -ql PACKAGE
rpm -qf /path/to/file
dnf repolist
```

### Repository audit

```bash
ls -la /etc/apt/sources.list.d/
grep -Rhv '^#' /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null
dnf repolist
```

### Storage troubleshooting

```bash
df -h
df -i
du -sh /var/* 2>/dev/null | sort -h
```

---

# 🎯 Interview Questions

## Beginner

1. What is a Linux package?
2. What is a package manager?
3. What is a repository?
4. Why are dependencies important?
5. What is the difference between `.deb` and `.rpm`?
6. What is APT?
7. What is DNF?

## Intermediate

8. Difference between `apt update` and `apt upgrade`?
9. Difference between APT and DPKG?
10. Difference between DNF and RPM?
11. How do you find which package owns a file?
12. How do you list files installed by a package?
13. How do you check an installed package version?
14. What can cause dependency errors?
15. What can cause repository errors?
16. Why should you use trusted repositories?

## Security / SOC

17. How can package inventory help vulnerability management?
18. How would you investigate an unexpected package?
19. Why is repository configuration security-relevant?
20. What is package signing?
21. Why is `curl | bash` risky?
22. Why can an old upstream version string be misleading when assessing vulnerabilities?
23. How would you determine whether a suspicious executable was installed by a package?
24. What evidence would you collect before removing suspicious software?
25. How can package-management activity become a SOC detection signal?

---

# 🧠 Key Mental Model

Remember:

```text
                 SOFTWARE
                    │
                    ↓
                 PACKAGE
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
     REPOSITORY           PACKAGE DB
          │                   │
          ↓                   ↓
     PACKAGE MANAGER     INSTALLED STATE
          │                   │
     ┌────┼────┐              │
     ↓    ↓    ↓              ↓
  Install Update Remove    File ownership
     │    │    │              │
     └────┼────┴──────────────┘
          ↓
       SECURITY
          │
    ┌─────┼──────┐
    ↓     ↓      ↓
  Patch  CVE   Supply Chain
```

---

# ⚠️ Common Mistakes

### Mistake 1 — Thinking `apt update` upgrades packages

It refreshes repository metadata.

### Mistake 2 — Installing random packages from the internet

Prefer trusted and approved repositories.

### Mistake 3 — Ignoring dependencies

Dependencies are part of the application ecosystem.

### Mistake 4 — Removing packages blindly

A package may be required by other software.

### Mistake 5 — Assuming old version = vulnerable

Distribution backports can change the security status.

### Mistake 6 — Cleaning evidence during an investigation

Do not delete package caches, logs or software before following the incident-response process.

---

# ✅ Module Checklist

- [ ] I understand packages
- [ ] I understand package managers
- [ ] I understand repositories
- [ ] I understand dependencies
- [ ] I know APT basics
- [ ] I know DPKG basics
- [ ] I know DNF basics
- [ ] I know RPM basics
- [ ] I can install software safely
- [ ] I can update software
- [ ] I can remove software
- [ ] I can inspect installed packages
- [ ] I can identify package ownership of a file
- [ ] I can inspect repository configuration
- [ ] I understand package signatures
- [ ] I understand why patching matters
- [ ] I understand vendor backports
- [ ] I can troubleshoot common package errors
- [ ] I can audit installed software
- [ ] I can connect package management to SOC investigations
- [ ] I completed the practical labs
- [ ] I understand the software supply-chain perspective

---

# 🚀 What Comes Next?

Now that you understand how Linux software is installed and maintained, the next major administration topic is **disk and storage management**.

➡️ **Next Module: `13-Disk-and-Storage`**

You will learn disks, partitions, block devices, `lsblk`, `fdisk`, `parted`, `df`, `du`, swap, storage troubleshooting, capacity investigation and the security/SOC importance of storage.