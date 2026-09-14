# 💾 13 — Disk and Storage Management

> **Goal:** Understand how Linux sees disks and storage, how partitions and block devices work, how to measure capacity, how swap works, and how to troubleshoot storage problems from both an administrator and SOC/Blue Team perspective.

---

## 🎯 Learning Objectives

By the end of this module, you should be able to:

- Explain disks, partitions and block devices
- Understand device names such as `/dev/sda` and `/dev/nvme0n1`
- Inspect storage with `lsblk`
- Understand partition tables and partitions
- Use `fdisk` and understand `parted`
- Measure filesystem capacity with `df`
- Find large directories/files with `du`
- Understand inodes and inode exhaustion
- Understand swap and virtual memory support
- Understand mount points at a high level
- Identify storage-related performance problems
- Safely reason about partitioning and destructive commands
- Investigate unexpected storage usage
- Connect disk activity and storage artifacts to Linux security investigations

---

# 1. Why Storage Matters

Linux needs storage for much more than personal files.

```text
STORAGE
  │
  ├── Operating system
  ├── Applications
  ├── Configuration
  ├── User data
  ├── Logs
  ├── Temporary files
  ├── Package cache
  └── Security evidence
```

If storage fills completely, services may stop working.

Examples:

```text
Disk full
   ↓
Application cannot write
   ↓
Logs stop being written
   ↓
Database may fail
   ↓
Service becomes unstable
```

Storage is therefore both an administration and security concern.

---

# 2. Storage Mental Model

Think about Linux storage in layers:

```text
PHYSICAL / VIRTUAL DISK
        ↓
BLOCK DEVICE
        ↓
PARTITION
        ↓
FILESYSTEM
        ↓
MOUNT POINT
        ↓
FILES & DIRECTORIES
```

For example:

```text
/dev/nvme0n1
      ↓
/dev/nvme0n1p3
      ↓
ext4 filesystem
      ↓
/
      ↓
/etc /var /home /usr ...
```

Not every Linux system uses this exact layout.

---

# 3. What Is a Disk?

A disk is persistent storage used to retain data across reboots.

Examples include:

- HDD
- SATA SSD
- NVMe SSD
- Virtual disks provided by cloud/hypervisors
- USB storage

Linux exposes storage devices through the device model, commonly under `/dev`.

---

# 4. What Is a Block Device?

A **block device** provides access to storage in blocks.

Examples:

```text
/dev/sda
/dev/sdb
/dev/nvme0n1
```

Block devices can represent whole disks or partitions.

List block devices:

```bash
lsblk
```

---

# 5. Device Names

Traditional SCSI/SATA-style disks often appear as:

```text
/dev/sda
/dev/sdb
```

Partitions may appear as:

```text
/dev/sda1
/dev/sda2
```

NVMe devices commonly appear as:

```text
/dev/nvme0n1
```

Partitions:

```text
/dev/nvme0n1p1
/dev/nvme0n1p2
```

Do not assume `/dev/sda` is always the boot disk. Device naming depends on hardware, kernel discovery and virtualization.

---

# 6. `lsblk` — Your First Storage Command

Run:

```bash
lsblk
```

Example:

```text
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   100G  0 disk
├─sda1        8:1    0     1G  0 part /boot
└─sda2        8:2    0    99G  0 part /
```

This quickly shows:

- device names
- sizes
- partitions
- device type
- mount points

---

# 7. Better `lsblk` Output

Use:

```bash
lsblk -f
```

This adds filesystem information.

Useful fields include:

```text
NAME
FSTYPE
FSVER
LABEL
UUID
MOUNTPOINTS
```

Another useful view:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,UUID,MOUNTPOINTS
```

---

# 8. UUIDs

A filesystem can have a UUID (Universally Unique Identifier).

Example:

```text
UUID=7f3a... 
```

View UUIDs with:

```bash
lsblk -f
```

or:

```bash
blkid
```

UUIDs are commonly used to identify filesystems consistently, especially in mount configuration.

---

# 9. Labels

Filesystems can also have labels.

Example:

```text
BACKUP
DATA
HOME
```

View labels:

```bash
lsblk -f
```

Labels are human-friendly, while UUIDs are generally more unique.

---

# 10. Partitions

A partition is a defined region of a storage device.

Conceptually:

```text
100 GB DISK
┌────────────────────────────────────────┐
│ Partition 1 │ Partition 2 │ Partition 3│
│    1 GB     │    20 GB     │    79 GB   │
└────────────────────────────────────────┘
```

Partitions help organize storage and can separate system roles or filesystems.

---

# 11. Partition Tables

A disk needs partitioning information describing its partitions.

Two important partition-table schemes are:

```text
MBR
GPT
```

### MBR

Older partitioning scheme with historical limitations.

### GPT

Modern partitioning scheme commonly used with UEFI systems and large disks.

For modern installations, GPT is generally preferred when appropriate.

---

# 12. Inspecting Partition Tables

Use:

```bash
sudo fdisk -l
```

This displays disks, partitions and partition information.

Another option:

```bash
sudo parted -l
```

⚠️ These commands are primarily for inspection here. Partition editing can destroy data if used incorrectly.

---

# 13. `fdisk`

`fdisk` is a partitioning utility.

Interactive usage typically looks like:

```bash
sudo fdisk /dev/sdX
```

Inside the interactive tool, commands can inspect or modify partition tables.

### Important safety rule

Never substitute a real disk for `sdX` unless you have positively identified it.

A partitioning mistake can cause:

```text
DATA LOSS
BOOT FAILURE
FILESYSTEM DAMAGE
```

Practice partitioning only on disposable lab disks/VMs or snapshots.

---

# 14. `parted`

`parted` is another partition-management tool.

Inspect disks:

```bash
sudo parted -l
```

It supports modern partitioning workflows and is especially useful in automation and systems where GPT is used.

For this module, focus first on **understanding and inspecting** partitions before performing destructive operations.

---

# 15. Filesystem vs Partition

These are not the same thing.

```text
Partition
   ↓
Container/region on disk

Filesystem
   ↓
Structure that organizes files within storage
```

A common arrangement is:

```text
Disk
 ↓
Partition
 ↓
ext4 filesystem
 ↓
Mounted directory
```

But Linux can also use filesystems on other block-device layers such as LVM or RAID.

---

# 16. Common Linux Filesystems

You may encounter:

| Filesystem | Common context |
|---|---|
| ext4 | Common Linux filesystem |
| XFS | Common on enterprise Linux |
| Btrfs | Advanced features, snapshots/subvolumes |
| FAT32/exFAT | Removable/cross-platform storage |
| NTFS | Windows interoperability |

Filesystem choice depends on workload, distribution and operational requirements.

---

# 17. `df` — Filesystem Capacity

Use:

```bash
df -h
```

Example:

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        99G   71G   23G  76% /
```

`df` tells you about filesystem-level space usage.

Important:

```text
df → filesystem capacity
```

---

# 18. `df -hT`

For filesystem type too:

```bash
df -hT
```

Example concept:

```text
Filesystem  Type  Size  Used Avail Use% Mounted on
/dev/sda2   ext4   99G   71G   23G  76% /
```

This is an excellent first command when a system reports:

> Disk space is running out.

---

# 19. `df` vs `du`

This distinction is critical.

```text
df → How much filesystem space is used?

du → Which directories/files are consuming space?
```

Example:

```bash
df -h
```

then:

```bash
du -sh /var/* 2>/dev/null | sort -h
```

Think:

```text
df = filesystem view
 du = directory/file view
```

---

# 20. `du` — Directory Usage

Check current directory:

```bash
du -sh .
```

Check a directory:

```bash
du -sh /var
```

Break it down:

```bash
du -sh /var/* 2>/dev/null | sort -h
```

Find the largest entries at the bottom of the sorted output.

---

# 21. Finding Large Files

A simple approach:

```bash
find /var -type f -size +500M -ls 2>/dev/null
```

This can help identify large files.

However, use care on production systems because broad `find` operations can consume resources on very large filesystems.

---

# 22. Disk Full Troubleshooting

When you see:

```text
No space left on device
```

Do not immediately delete random files.

Use:

```bash
df -h
```

Then identify the affected mount.

Next:

```bash
du -sh /var/* 2>/dev/null | sort -h
```

Investigate likely sources:

```text
Logs
Package cache
Backups
Temporary files
Application data
User files
Container images
Database files
```

---

# 23. The “Deleted but Still Using Space” Problem

Sometimes:

```text
df → filesystem almost full

du → does not explain the usage
```

One possible reason is a deleted file still held open by a running process.

Conceptually:

```text
Process
  │
  └── open file
        │
        └── file deleted
              │
              └── storage remains allocated
                  while process holds descriptor
```

Investigate with:

```bash
sudo lsof +L1
```

Not every system has `lsof` installed.

This is an important real-world troubleshooting technique.

---

# 24. Inodes

Filesystems track more than just data blocks.

They also track file metadata through **inodes** on filesystems such as ext4.

A filesystem can theoretically run out of inodes even when some storage space remains.

Check:

```bash
df -i
```

Example:

```text
Filesystem     Inodes  IUsed   IFree IUse% Mounted on
/dev/sda2     6500000 6490000 10000   99% /
```

---

# 25. Disk Space vs Inodes

```text
CASE A
Disk blocks exhausted
→ df -h shows high usage

CASE B
Inodes exhausted
→ df -i shows high usage
```

Large numbers of tiny files can exhaust inodes.

Common causes:

- mail queues
- temporary files
- application caches
- log fragments
- container layers
- badly behaving applications

---

# 26. Mount Points

Linux makes filesystems accessible through directories called **mount points**.

Example:

```text
/dev/sda2
    ↓
    /
```

Another disk could be mounted at:

```text
/data
```

Then:

```text
/data
  ├── backups
  ├── reports
  └── applications
```

Mounting makes the filesystem accessible within the Linux directory tree.

---

# 27. View Mounted Filesystems

Use:

```bash
findmnt
```

Or:

```bash
mount
```

For a specific path:

```bash
findmnt /
```

For filesystem usage:

```bash
df -hT
```

---

# 28. `/etc/fstab`

Persistent filesystem mount configuration is commonly stored in:

```text
/etc/fstab
```

Inspect:

```bash
cat /etc/fstab
```

A typical conceptual entry contains:

```text
filesystem   mountpoint   type   options   dump   pass
```

Example concept:

```text
UUID=xxxx   /data   ext4   defaults   0   2
```

Exact configuration depends on the system.

---

# 29. Why `/etc/fstab` Matters

A bad `/etc/fstab` entry can cause boot or mount problems.

Before changing it:

```text
BACK UP CONFIGURATION
        ↓
CHECK DEVICE/UUID
        ↓
CHECK FILESYSTEM TYPE
        ↓
CHECK MOUNT POINT
        ↓
VALIDATE CAREFULLY
```

A typo in storage configuration can make a system difficult to boot.

---

# 30. Swap

Swap provides disk-backed space that the kernel can use for memory-management purposes when appropriate.

It is not simply “extra RAM.”

Conceptually:

```text
RAM
 │
 ├── Active memory
 │
 └── Kernel memory management
          │
          ↓
        SWAP
      (disk-backed)
```

Swap can be a partition or a file.

---

# 31. Check Swap

```bash
swapon --show
```

Also:

```bash
free -h
```

Example concept:

```text
               total   used   free
Mem:            8GiB    6GiB   500MiB
Swap:           2GiB    200MiB 1.8GiB
```

Swap usage alone is not proof of a problem. Interpret it together with memory pressure and system behavior.

---

# 32. `free`

Use:

```bash
free -h
```

This helps inspect:

- total memory
- used memory
- available memory
- swap

Linux uses otherwise-unused RAM for useful caching, so “used memory” should not automatically be interpreted as memory exhaustion.

---

# 33. Storage Performance

A disk can have plenty of free space and still be slow.

Possible causes:

```text
High I/O workload
Slow storage
Filesystem pressure
Application behavior
Network storage latency
Virtualization contention
```

Useful commands include:

```bash
iostat
vmstat
```

These may require the `sysstat` package depending on the distribution.

---

# 34. `iostat`

If available:

```bash
iostat -xz 1
```

It can help analyze CPU and device I/O statistics.

Useful concepts include:

- I/O operations
- throughput
- utilization
- latency-related indicators

Do not interpret one metric in isolation.

---

# 35. `vmstat`

Run:

```bash
vmstat 1
```

This provides a repeated view of system activity including:

- processes
- memory
- swap activity
- I/O
- system activity
- CPU

It can help correlate storage pressure with memory and CPU behavior.

---

# 36. Disk Health

Physical disk health can be investigated with technologies such as S.M.A.R.T. on supported devices.

A common utility is:

```bash
smartctl
```

Usually supplied by a package such as `smartmontools`.

Example inspection command:

```bash
sudo smartctl -a /dev/sdX
```

⚠️ Hardware and virtual disks vary. Some cloud/virtual environments do not expose physical health information directly.

---

# 37. Storage Layers Can Be More Complex

Real enterprise Linux systems may use:

```text
Physical Disk
    ↓
RAID
    ↓
LVM
    ↓
Logical Volume
    ↓
Filesystem
    ↓
Mount Point
```

Or:

```text
Cloud Disk
    ↓
Virtual Block Device
    ↓
Partition
    ↓
Filesystem
```

Later modules will go deeper into filesystems and mounting.

---

# 38. LVM Preview

**LVM (Logical Volume Manager)** provides flexible storage management.

Basic conceptual layers:

```text
Physical Volume (PV)
        ↓
Volume Group (VG)
        ↓
Logical Volume (LV)
        ↓
Filesystem
```

Example names:

```text
/dev/mapper/vg0-root
```

Do not confuse a logical volume with a physical disk.

Detailed LVM work belongs in advanced storage administration.

---

# 39. RAID Preview

RAID combines multiple storage devices for goals such as:

- redundancy
- performance
- capacity

Common levels include:

```text
RAID 0 → striping, no redundancy
RAID 1 → mirroring
RAID 5 → distributed parity
RAID 6 → dual parity
RAID 10 → mirrored + striped
```

RAID is **not a backup**.

A RAID array can protect against some disk failures while still losing data through deletion, corruption, malware or operational mistakes.

---

# 40. Storage and Security

Storage is a major source of security evidence.

Examples:

```text
/var/log/
User home directories
Application logs
Authentication records
Shell history
Temporary files
Downloaded files
Malware artifacts
Configuration files
```

A security analyst should understand where relevant artifacts can reside.

---

# 41. Unexpected Storage Usage

Suppose `/var` suddenly grows by 40 GB.

A defensive investigation might be:

```text
df -h
   ↓
Identify affected filesystem
   ↓
du -sh /var/*
   ↓
Identify large subdirectory
   ↓
du -sh /var/SUSPECT/*
   ↓
Identify large files
   ↓
Check timestamps/ownership
   ↓
Correlate with process/service activity
   ↓
Review logs and change history
```

Possible legitimate explanations include logs, backups, package caches and application data.

---

# 42. Security Scenario — Log Explosion

### Scenario

A server suddenly has almost no free disk space.

You discover:

```text
/var/log/application.log → 45 GB
```

Do not immediately delete the file.

Ask:

```text
Why did logging increase?
When did it begin?
Which application owns it?
Was there an error loop?
Was there unusual activity?
Is log rotation working?
```

Large logs can be caused by both operational failures and security events.

---

# 43. Security Scenario — Hidden Storage Consumption

If:

```text
df -h → 95% used

du -sh /var/* → doesn't explain usage
```

Investigate open deleted files:

```bash
sudo lsof +L1
```

Then correlate the owning process:

```bash
ps -fp PID
```

This is a powerful troubleshooting pattern.

---

# 44. Storage Triage Workflow

```text
Storage Alert
     ↓
   df -hT
     ↓
Which filesystem?
     ↓
   du -sh
     ↓
Which directory?
     ↓
Find large files
     ↓
Deleted-but-open files?
     ↓
Inodes exhausted?
     ↓
Process/service responsible?
     ↓
Expected or suspicious?
     ↓
Remediate safely
```

---

# ⚠️ 45. Dangerous Storage Commands

Commands that modify partition tables, filesystems or disks can cause irreversible data loss.

Examples include tools/options involving:

```text
fdisk write operations
parted modifications
mkfs
wipefs
dd
```

Never experiment with these against a disk containing important data.

For labs:

```text
Use disposable VM disks
Use snapshots
Verify device names
Document the target device
```

A professional Linux administrator verifies the target multiple times before destructive operations.

---

# 🧪 Lab 1 — Storage Discovery

## Objective

Build a complete picture of your lab machine's storage.

Run:

```bash
lsblk
lsblk -f
df -hT
findmnt
swapon --show
free -h
```

### Questions

1. How many block devices exist?
2. Which partitions exist?
3. Which filesystem is mounted at `/`?
4. What is its UUID?
5. How much space is available?
6. Is swap configured?
7. Which directories are separate mount points?

### Expected result

Create a simple diagram:

```text
DISK
 ↓
PARTITION
 ↓
FILESYSTEM
 ↓
MOUNT POINT
```

---

# 🧪 Lab 2 — Find Where Storage Is Going

## Scenario

Your lab VM reports that storage usage is increasing.

### Step 1

```bash
df -hT
```

### Step 2

Identify the affected filesystem.

### Step 3

Inspect major directories:

```bash
du -sh /* 2>/dev/null | sort -h
```

### Step 4

Drill down:

```bash
du -sh /var/* 2>/dev/null | sort -h
```

### Step 5

Find large files if necessary:

```bash
find /var -type f -size +100M -ls 2>/dev/null
```

### Questions

- What is consuming the space?
- Is it expected?
- What process/application created it?
- Would deleting it be safe?

---

# 🧪 Lab 3 — Inode Investigation

## Objective

Understand the difference between disk-space exhaustion and inode exhaustion.

Run:

```bash
df -h
df -i
```

### Questions

- What is your root filesystem's `Use%`?
- What is its inode `IUse%`?
- Which would become a problem first if millions of tiny files were created?

### Challenge

In a disposable lab directory, create many small files and observe inode usage carefully.

Do not create millions of files on a production or shared filesystem.

---

# 🧪 Lab 4 — Deleted File Still Consuming Space

## Objective

Understand the `df` vs `du` mismatch caused by open deleted files.

### Safe lab concept

Create a temporary file and have a test process keep it open. Delete the file while the process still has it open.

Then investigate with:

```bash
sudo lsof +L1
```

### Observe

```text
File deleted
     ↓
Directory entry removed
     ↓
Process still has file open
     ↓
Storage remains allocated
```

### Cleanup

Terminate the test process you created and verify the space is released.

---

# 🧪 Lab 5 — SOC Storage Investigation

## Scenario

An authorized Linux server has unexpectedly consumed 30 GB of storage overnight.

### Investigation goals

Determine:

```text
Which filesystem?
Which directory?
Which files?
Which owner?
When did growth occur?
Which process/service is involved?
Is the activity expected?
```

### Useful commands

```bash
df -hT
df -i
du -sh /var/* 2>/dev/null | sort -h
find /var -type f -size +500M -ls 2>/dev/null
sudo lsof +L1
ps aux
systemctl --type=service --state=running
```

### SOC questions

- Could this be normal application growth?
- Could logging have entered an error loop?
- Could a backup have run?
- Could a security event have generated unusual logs?
- Is there evidence of unexpected downloaded/generated files?

Do not delete evidence simply to restore free space without following the appropriate incident-response or operations procedure.

---

# 🛠️ Mini Project — Linux Storage Health Auditor

Create a Bash script that generates a storage health report.

Suggested output:

```text
====================================
       LINUX STORAGE AUDITOR
====================================
Hostname: ...
Date: ...

FILESYSTEM USAGE
----------------
Mount      Size   Used   Avail   Use%
/          ...    ...    ...     ...

INODE USAGE
-----------
Mount      IUse%
...        ...

BLOCK DEVICES
-------------
...

SWAP
----
...

TOP DIRECTORIES
---------------
...

LISTENING / LOG NOTES
---------------------
...
```

### Suggested commands

```bash
hostname
date
lsblk -f
df -hT
df -i
du -sh
swapon --show
free -h
```

### Extensions

- Alert above 80%, 90% and 95%
- Detect inode pressure
- Detect mount points that are missing
- Detect large recently modified files
- Detect deleted-open files when `lsof` is available
- Save timestamped reports
- Compare current usage with a previous snapshot
- Output CSV for SOC dashboards

---

# 🔐 Storage + SOC Cheat Sheet

### Block devices

```bash
lsblk
lsblk -f
blkid
```

### Partitions

```bash
sudo fdisk -l
sudo parted -l
```

### Filesystem usage

```bash
df -h
df -hT
df -i
```

### Directory usage

```bash
du -sh .
du -sh /var/* 2>/dev/null | sort -h
```

### Mounts

```bash
findmnt
findmnt /
cat /etc/fstab
```

### Swap/memory

```bash
swapon --show
free -h
```

### Open deleted files

```bash
sudo lsof +L1
```

### Performance

```bash
iostat -xz 1
vmstat 1
```

---

# 🎯 Interview Questions

## Beginner

1. What is a disk?
2. What is a block device?
3. What is a partition?
4. What is a filesystem?
5. What is a mount point?
6. What is `/dev/sda`?
7. What is an NVMe device name?
8. What is the difference between MBR and GPT?

## Intermediate

9. What does `lsblk` show?
10. What is the difference between `df` and `du`?
11. What does `df -i` show?
12. What are inodes?
13. Why can a filesystem be full when `du` doesn't explain the usage?
14. What is `/etc/fstab`?
15. What is UUID used for?
16. What is swap?
17. Is swap the same as RAM?
18. How would you troubleshoot `No space left on device`?
19. What is LVM?
20. What is RAID?
21. Why is RAID not a backup?

## Security / SOC

22. Why is disk usage relevant to security monitoring?
23. How would you investigate sudden growth in `/var/log`?
24. How would you investigate an unexpected large file?
25. What would you do if `df` and `du` disagree?
26. Why can deleting a file during an investigation be a problem?
27. What storage artifacts might be useful during incident response?
28. How can package caches contribute to disk usage?
29. How can a malicious process indirectly cause disk exhaustion?
30. What evidence would you collect before cleaning suspicious storage?

---

# 🧠 Key Mental Model

```text
                 STORAGE
                    │
                    ↓
              BLOCK DEVICE
                    │
                    ↓
                PARTITION
                    │
                    ↓
               FILESYSTEM
                    │
                    ↓
               MOUNT POINT
                    │
                    ↓
              FILES/DIRECTORIES
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
       df -h                du -sh
          │                   │
   filesystem view      file/directory view
          │                   │
          └─────────┬─────────┘
                    ↓
               INVESTIGATION
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
       inodes     process    logs
          │         │         │
          └─────────┴─────────┘
                    ↓
               SECURITY / OPS
```

---

# ⚠️ Common Mistakes

### Mistake 1 — Assuming `/dev/sda` is always the main disk

Device names depend on the environment.

### Mistake 2 — Confusing `df` with `du`

`df` reports filesystem usage; `du` helps locate directory/file usage.

### Mistake 3 — Ignoring inodes

Millions of small files can exhaust inodes before data blocks.

### Mistake 4 — Treating swap as extra RAM

Swap is disk-backed memory-management space and is much slower than RAM.

### Mistake 5 — Running destructive commands on the wrong disk

Always verify the target device.

### Mistake 6 — Deleting suspicious files immediately

During investigations, preserve evidence according to the incident-response process.

### Mistake 7 — Thinking RAID equals backup

RAID can provide redundancy, but it does not replace backups.

---

# ✅ Module Checklist

- [ ] I understand disks and block devices
- [ ] I understand partitions
- [ ] I understand MBR and GPT
- [ ] I can use `lsblk`
- [ ] I can identify filesystem types and UUIDs
- [ ] I understand `df`
- [ ] I understand `du`
- [ ] I can troubleshoot disk-full conditions
- [ ] I understand inodes
- [ ] I can check inode usage
- [ ] I understand mount points
- [ ] I understand `/etc/fstab`
- [ ] I understand swap
- [ ] I can inspect memory/swap usage
- [ ] I understand basic storage performance concepts
- [ ] I know what LVM is
- [ ] I know what RAID is
- [ ] I understand why RAID is not a backup
- [ ] I understand storage as security evidence
- [ ] I completed the storage labs
- [ ] I can investigate unexpected storage growth
- [ ] I understand why destructive disk commands require extreme caution

---

# 🚀 What Comes Next?

The next module goes deeper into **filesystems and mounting**.

➡️ **Next Module: `14-Filesystems-and-Mounting`**

You will learn how Linux filesystems organize data, filesystem creation and checking, mounting/unmounting, persistent mounts, filesystem troubleshooting, permissions at the storage layer, and security considerations around removable and mounted storage.