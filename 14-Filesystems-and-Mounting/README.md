# 🗂️ 14 — Filesystems and Mounting

> **Linux Knowledge Base — Beginner → Advanced → SOC / Blue Team**

## 🎯 Module Objectives

By the end of this module, you should be able to:

- Explain what a filesystem is and why Linux needs one.
- Understand filesystems, partitions, block devices, and mount points.
- Describe high-level ext4 concepts such as superblocks, inodes, data blocks, and journaling.
- Identify filesystem type, UUID, label, mount point, and mount options.
- Mount and unmount filesystems safely in an isolated lab.
- Understand `/etc/fstab` and why UUIDs are commonly used.
- Work with read-only mounts, bind mounts, loop devices, and `tmpfs`.
- Understand `/proc`, `/sys`, and `/dev` as special kernel/device-related filesystems.
- Recognize common filesystem failures and read-only remounts.
- Perform basic filesystem-health troubleshooting.
- Investigate unexpected mounts from a defensive/SOC perspective.

---

# 1. What Is a Filesystem?

A **filesystem** is the set of structures and rules Linux uses to organize data into files, directories, and metadata on a storage device or other storage-like resource.

A disk gives you **storage space**. A filesystem gives that space a structure that the operating system can understand.

```text
Physical Storage
      │
      ▼
┌─────────────────────┐
│ Block Device        │
│ /dev/sda            │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Partition           │
│ /dev/sda2           │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Filesystem          │
│ ext4 / XFS / ...    │
└──────────┬──────────┘
           │ mount
           ▼
┌─────────────────────┐
│ Mount Point         │
│ /data               │
└──────────┬──────────┘
           │
           ▼
     files & directories
```

### Simple analogy

Think of a filesystem like a library organization system:

- **Disk** = building
- **Filesystem** = library organization rules
- **Directories** = shelves/sections
- **Files** = books
- **Inodes/metadata** = catalog records
- **Blocks** = physical storage areas

---

# 2. Why Do Filesystems Exist?

Without filesystem structures, an operating system would have no convenient standard way to answer questions such as:

- Where is this file stored?
- Who owns it?
- What permissions does it have?
- How large is it?
- When was its metadata changed?
- Which blocks contain its data?
- Which directory contains its name?

A filesystem manages these relationships.

```text
                 FILESYSTEM
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
   Metadata        Names         Data
       │             │             │
       ▼             ▼             ▼
  permissions    directories    data blocks
  owner/group    filenames      file contents
  timestamps
```

---

# 3. Disk vs Partition vs Filesystem vs Mount Point

These terms are often confused.

| Concept | Meaning | Example |
|---|---|---|
| Disk | Physical/virtual storage device | `/dev/sda` |
| Partition | Region of a disk | `/dev/sda2` |
| Filesystem | Structure placed on storage | `ext4` |
| Mount point | Directory where filesystem becomes accessible | `/data` |
| Mount | Operation connecting filesystem to directory tree | `mount /dev/sdb1 /mnt/data` |

A partition does **not** automatically mean a filesystem exists on it.

```text
/dev/sdb
   │
   ├── /dev/sdb1  → ext4 → /data
   │
   └── /dev/sdb2  → XFS  → /backup
```

Some systems also use filesystems directly on whole devices, logical volumes, RAID devices, loop devices, network resources, or other storage abstractions.

---

# 4. Linux's Single Directory Tree

Linux presents mounted filesystems through one directory tree.

```text
/
├── boot/
├── etc/
├── home/
├── var/
├── usr/
├── tmp/
├── dev/
├── proc/
├── sys/
├── run/
└── data/          ← another filesystem could be mounted here
```

If `/dev/sdb1` is mounted on `/data`, files stored on that filesystem appear under `/data`.

```text
/dev/sdb1 (ext4)
       │
       │ mount
       ▼
      /data
       │
       ├── reports/
       ├── logs/
       └── backup/
```

Mounting does not move the filesystem's contents into the directory. It makes the filesystem available at that location in the directory hierarchy.

---

# 5. Common Linux Filesystems

## ext4

`ext4` is a widely used general-purpose Linux filesystem.

Important concepts include:

- superblocks
- block groups
- inodes
- data blocks
- directory structures
- journaling

It is a strong filesystem to understand first because many Linux environments use it.

## XFS

XFS is a high-performance filesystem commonly encountered in enterprise Linux environments.

It is designed for large filesystems and workloads and has its own administration and repair tools.

## Btrfs

Btrfs is a modern copy-on-write filesystem with features such as:

- checksums
- subvolumes
- snapshots
- compression
- advanced storage management features

Feature availability and administration differ from ext4/XFS.

### Other filesystems you may encounter

- FAT32
- exFAT
- NTFS
- NFS
- CIFS/SMB
- tmpfs
- overlayfs
- squashfs

---

# 6. Filesystem Internal Structure

A filesystem contains more than file contents.

A simplified model:

```text
Filesystem
│
├── Superblock
│     └── filesystem-wide information
│
├── Metadata / Inodes
│     └── file metadata + references to data
│
├── Directories
│     └── names → filesystem objects
│
├── Data Blocks
│     └── file contents
│
└── Journal (filesystem-dependent)
      └── helps recover from interrupted metadata operations
```

The exact implementation differs between filesystems.

---

# 7. Superblock

The **superblock** contains important filesystem-wide information.

Depending on the filesystem, it can contain information such as:

- filesystem type
- size
- block information
- filesystem state
- feature flags
- UUID

If critical filesystem metadata is damaged, the filesystem may become difficult or impossible to mount normally.

---

# 8. Inodes

An **inode** stores metadata about a filesystem object.

It can contain information such as:

- file type
- permissions
- owner
- group
- timestamps
- size
- references to data blocks

The filename itself is associated with the directory structure rather than being the core identity stored in the inode.

```text
Directory
   │
   ├── report.txt ─────► inode 1204
   │                       │
   │                       ├── permissions
   │                       ├── owner
   │                       ├── timestamps
   │                       ├── size
   │                       └── data block references
```

This is why Linux can have hard links: multiple directory entries can refer to the same inode.

---

# 9. File Timestamps

Linux files commonly expose three important timestamps:

| Timestamp | Meaning |
|---|---|
| `atime` | Last access time, subject to filesystem/mount behavior |
| `mtime` | Last modification of file contents |
| `ctime` | Last change to inode/file metadata |

### Important

`ctime` does **not** universally mean creation time.

Some filesystems support a birth/creation time, but support and visibility vary.

Check timestamps with:

```bash
stat file.txt
```

Example:

```bash
stat /etc/hosts
```

---

# 10. Journaling

Many Linux filesystems use a journal to improve recovery after an interrupted operation or crash.

Conceptually:

```text
Operation
   │
   ▼
Journal records intended metadata changes
   │
   ▼
Filesystem updates metadata
   │
   ▼
Operation completes
```

Journaling is **not a backup**.

It helps filesystem consistency/recovery; it does not protect you from accidental deletion, ransomware, or every type of corruption.

---

# 11. Identify Filesystem Information

## `lsblk`

```bash
lsblk
```

Useful detailed form:

```bash
lsblk -f
```

This can show:

- device
- filesystem type
- label
- UUID
- mount point

Example conceptual output:

```text
NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
sda
├─sda1 ext4         1111-2222                            /
└─sda2 ext4 DATA   3333-4444                            /data
```

## `blkid`

```bash
sudo blkid
```

Useful for identifying filesystem signatures, UUIDs, and labels.

## `findmnt`

```bash
findmnt
```

Specific mount point:

```bash
findmnt /data
```

Filesystem type only:

```bash
findmnt -t ext4
```

---

# 12. Mounting a Filesystem

A filesystem must be attached to the Linux directory tree before normal path-based access is available.

Basic syntax:

```bash
sudo mount DEVICE MOUNT_POINT
```

Example — **lab device only**:

```bash
sudo mount /dev/sdb1 /mnt/data
```

Verify:

```bash
findmnt /mnt/data
```

or:

```bash
df -h /mnt/data
```

### ⚠️ Safety

Never blindly mount or format an unknown device on a production system. Always identify the device first with commands such as `lsblk -f`.

---

# 13. Creating a Mount Point

A mount point is normally an existing directory.

Example:

```bash
sudo mkdir -p /mnt/lab
```

Then, in an authorized disposable lab:

```bash
sudo mount /dev/sdb1 /mnt/lab
```

Before mounting, understand what is already inside the directory. Once another filesystem is mounted there, the underlying directory contents are hidden until it is unmounted.

---

# 14. Unmounting

```bash
sudo umount /mnt/lab
```

You can also specify the device:

```bash
sudo umount /dev/sdb1
```

Verify:

```bash
findmnt /mnt/lab
```

No output generally means that mount point is not currently mounted.

---

# 15. Why `umount` Says "Target Is Busy"

A filesystem may be busy because a process has:

- an open file there
- its current working directory there
- a shell positioned inside the directory
- an open executable or library there

Check:

```bash
sudo fuser -vm /mnt/lab
```

Another useful tool:

```bash
sudo lsof /mnt/lab
```

Then leave the directory if your shell is inside it:

```bash
cd ~
```

Re-check before unmounting.

### Investigation principle

Do not immediately terminate processes just to force an unmount. Identify why the mount is busy first.

---

# 16. Read-Only Mounts

A filesystem can be mounted read-only:

```bash
sudo mount -o ro /dev/sdb1 /mnt/lab
```

This is useful when you need to reduce accidental modification during inspection.

Check options:

```bash
findmnt /mnt/lab
```

Look for `ro` in the mount options.

### SOC / Forensics relevance

When examining potentially compromised removable media or a suspicious disk image, a read-only workflow can help reduce accidental changes. Follow your organization's evidence-handling procedure and use appropriate forensic tooling where required.

---

# 17. Remounting

A mounted filesystem can sometimes have its options changed without fully unmounting it.

Example pattern:

```bash
sudo mount -o remount,ro /mnt/lab
```

Exact behavior depends on the filesystem, mount setup, and options.

Always verify afterward:

```bash
findmnt /mnt/lab
```

---

# 18. Mount Options and Security

Mount options can influence how content on a filesystem behaves.

Common options include:

| Option | Purpose |
|---|---|
| `ro` | Read-only |
| `rw` | Read-write |
| `nosuid` | Prevent set-user-ID/set-group-ID bits and file capabilities from taking effect on that mount |
| `nodev` | Do not allow device nodes on that mount to be used as devices |
| `noexec` | Prevent direct execution of binaries from that mount; not a complete application-control boundary |
| `noatime` | Reduce access-time updates where supported |
| `defaults` | Common default option set |
| `nofail` | Do not make boot fail solely because this mount is unavailable |

### Security example

A removable-data filesystem might be mounted with restrictive options depending on the organization's requirements.

```text
Removable Media
      │
      ▼
  /media/usb
      │
      ├── nosuid
      ├── nodev
      └── possibly noexec
```

### Important caveat

`noexec` should not be treated as a complete security boundary. An interpreter may be able to read a script from the filesystem, and applications can behave differently depending on how they load code.

---

# 19. `/etc/fstab`

`/etc/fstab` describes filesystems that should normally be mounted automatically.

View it:

```bash
cat /etc/fstab
```

Typical conceptual entry:

```text
UUID=3333-4444  /data  ext4  defaults,nofail  0  2
```

Fields:

```text
┌──────────┬───────────┬────────┬────────────────┬─────┬─────┐
│ source   │ mountpoint│ type   │ options        │ dump│ fsck│
└──────────┴───────────┴────────┴────────────────┴─────┴─────┘
```

### Why UUID?

Device names such as `/dev/sdb1` can change depending on hardware discovery order.

UUIDs provide a more stable identifier for a filesystem.

Find UUID:

```bash
lsblk -f
```

or:

```bash
sudo blkid
```

---

# 20. Understanding `nofail`

Example:

```text
UUID=... /backup ext4 defaults,nofail 0 2
```

`nofail` tells the boot process not to fail solely because that filesystem cannot be mounted.

It does **not** repair the filesystem or guarantee that applications can access the mount.

This can be useful for optional storage such as removable or secondary disks, but should be used intentionally.

---

# 21. Safely Testing `/etc/fstab`

After changing `/etc/fstab`, you should validate the configuration carefully.

A common command is:

```bash
sudo mount -a
```

This attempts to mount entries from `fstab` that are not excluded by options such as `noauto`.

### ⚠️ Important

A typo in `/etc/fstab` can cause boot or mount problems. Always:

1. Keep a backup/copy of the original file.
2. Verify UUIDs.
3. Verify filesystem types.
4. Verify mount points exist.
5. Test with `mount -a`.
6. Check `findmnt` and logs.

---

# 22. Bind Mounts

A bind mount makes an existing directory or file available at another location in the directory tree.

Example in an authorized lab:

```bash
sudo mount --bind /home/user/lab /mnt/lab-copy
```

Conceptually:

```text
/home/user/lab
      │
      │ bind mount
      ▼
/mnt/lab-copy
```

A bind mount does **not** create another physical copy of the data.

It provides another path to the same underlying filesystem objects.

---

# 23. Loop Devices

A **loop device** allows a regular file containing a filesystem image to be presented to Linux like a block device.

This is extremely useful for labs.

```text
filesystem.img
      │
      ▼
  loop device
      │
      ▼
   filesystem
      │
      ▼
    /mnt/lab
```

Example:

```bash
sudo mount -o loop filesystem.img /mnt/lab
```

This avoids needing a spare physical disk for many filesystem experiments.

Check loop devices:

```bash
losetup -a
```

---

# 24. `tmpfs`

`tmpfs` is a temporary filesystem that uses memory and related kernel-managed resources.

Examples include directories used for runtime state on many Linux systems, such as `/run`.

Check:

```bash
findmnt -t tmpfs
```

Important properties:

- fast for many workloads
- temporary
- contents can disappear after reboot
- size is managed differently from ordinary disk filesystems

Do not assume every Linux distribution uses exactly the same tmpfs layout.

---

# 25. `/proc`, `/sys`, and `/dev`

Linux has special filesystems and kernel interfaces that do not behave like ordinary disk filesystems.

## `/proc`

Provides process and kernel-related information.

```bash
mount | grep ' on /proc '
```

Examples:

```bash
ls /proc
cat /proc/meminfo
cat /proc/uptime
```

Process-specific information appears under paths such as:

```text
/proc/1/
/proc/1234/
```

## `/sys`

Exposes kernel/device model information and interfaces.

```bash
ls /sys
```

## `/dev`

Contains device nodes and related device interfaces managed by the Linux device system.

```bash
ls -l /dev
```

### Important model

The phrase **"everything is a file"** is a useful Unix/Linux mental model, but it is a simplification. Regular files, device nodes, sockets, pipes, and virtual kernel interfaces have different semantics.

---

# 26. Filesystem Hierarchy and Mount Relationships

Use:

```bash
findmnt
```

to understand the current mount tree.

A conceptual result:

```text
/
├─ /boot
├─ /home
├─ /proc
├─ /sys
├─ /dev
├─ /run
└─ /data
      └─ ext4 on /dev/sdb1
```

This is especially useful during troubleshooting because it tells you **which filesystem actually contains a path**.

---

# 27. Disk Full vs Filesystem Full vs Inodes Full

A filesystem can fail because different resources are exhausted.

Check block usage:

```bash
df -h
```

Check inode usage:

```bash
df -i
```

Example:

```text
Disk blocks: 95% used
Inodes:      42% used
```

or:

```text
Disk blocks: 60% used
Inodes:      100% used
```

In the second case, creating new files can fail even though plenty of byte capacity remains.

---

# 28. Filesystem Health Checks

Filesystem repair tools are filesystem-specific.

For ext-family filesystems, tools include:

```bash
fsck
```

and:

```bash
e2fsck
```

For XFS, administration uses tools such as:

```bash
xfs_repair
```

### Critical rule

Filesystem checks/repairs often need the filesystem to be **unmounted or otherwise offline**, depending on the tool and operation.

Do not casually run repair commands against a mounted root filesystem.

A repair tool is not a replacement for backups.

---

# 29. When Linux Remounts a Filesystem Read-Only

A filesystem may become read-only after serious errors as a protective measure, depending on filesystem/kernel behavior.

Symptoms may include:

```text
Read-only file system
```

Investigate kernel messages:

```bash
journalctl -k
```

or:

```bash
dmesg | tail -n 50
```

Look for signs such as:

- filesystem errors
- I/O errors
- storage timeouts
- device resets
- corruption warnings

Do not treat a read-only remount as merely a permissions problem.

---

# 30. Filesystem Troubleshooting Workflow

Use this sequence:

```text
Problem
  │
  ▼
Is path on expected filesystem?
  │
  ▼
findmnt
  │
  ▼
Filesystem type / device / options?
  │
  ▼
lsblk -f
  │
  ▼
Space or inode exhaustion?
  │
  ├── df -h
  └── df -i
  │
  ▼
Kernel/filesystem errors?
  │
  ├── journalctl -k
  └── dmesg
  │
  ▼
Is filesystem mounted read-only?
  │
  ▼
Plan safe recovery / offline check
```

---

# 31. Mount Permissions vs File Permissions

A mount option and a file's Unix permissions solve different problems.

For example:

```text
Mount policy
   │
   ├── ro
   ├── nosuid
   ├── nodev
   └── noexec

Filesystem object policy
   │
   ├── owner
   ├── group
   ├── mode bits
   └── ACLs
```

A file can have permissive mode bits while the mounted filesystem imposes additional restrictions.

Conversely, mounting a filesystem read-write does not automatically grant every user permission to modify every file.

---

# 32. Removable Media Security

USB drives and other removable media can introduce security risk.

Potential concerns:

- unknown executables
- malicious documents/scripts
- unexpected filesystem types
- unauthorized data transfer
- suspicious mounted images
- persistence mechanisms
- sensitive-data copying

Defensive controls may include:

- controlled mounting
- restrictive mount options
- endpoint monitoring
- device control policies
- malware scanning
- audit logging
- least privilege

Always follow organizational policy.

---

# 33. SOC Investigation — Unexpected Mount

### Scenario

An analyst notices an unfamiliar mount point:

```text
/mnt/.cache-data
```

Do not immediately delete it.

### Step 1 — Identify the mount

```bash
findmnt /mnt/.cache-data
```

### Step 2 — Identify the source and filesystem

```bash
lsblk -f
```

### Step 3 — Inspect mount options

```bash
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS /mnt/.cache-data
```

### Step 4 — Check filesystem contents carefully

```bash
sudo ls -la /mnt/.cache-data
```

### Step 5 — Check recent kernel/system logs

```bash
journalctl -k --since "1 hour ago"
```

### Step 6 — Determine whether it is expected

Ask:

- Is this a legitimate application mount?
- Is it a container/runtime mount?
- Is it removable media?
- Is it a loop-mounted image?
- Is it a network filesystem?
- Was it recently created?
- Is there a corresponding service or job?

### SOC principle

**Observe → identify → correlate → preserve evidence → contain according to procedure.**

Do not destroy potentially useful evidence before understanding the event.

---

# 34. Suspicious Loop Device Investigation

Unexpected loop devices can be legitimate. They are commonly used by packaging systems, desktop applications, VM/container workflows, and filesystem images.

Check:

```bash
losetup -a
```

Then:

```bash
lsblk -f
```

And:

```bash
findmnt
```

Correlate the result with:

- installed software
- running services
- recent system changes
- user activity
- system logs

**Do not label a loop device as malicious just because it is unfamiliar.**

---

# 35. SOC Evidence Preservation

Filesystem investigations can change evidence if you:

- edit files
- execute unknown binaries
- change timestamps
- mount read-write
- modify configuration
- run repair tools
- delete suspicious files

For real incident response:

1. Follow the incident-response plan.
2. Record what you observed.
3. Minimize unnecessary changes.
4. Preserve evidence according to policy.
5. Use approved forensic acquisition/mounting procedures.

For learning, reproduce suspicious situations inside disposable VMs/labs.

---

# 🧪 LAB 1 — Filesystem Discovery

## Objective

Learn to identify devices, filesystems, UUIDs, labels, and mount points.

## Commands

```bash
lsblk
lsblk -f
sudo blkid
findmnt
findmnt -t ext4
```

## Tasks

1. Identify your root filesystem.
2. Record its filesystem type.
3. Record its UUID.
4. Identify `/boot`, if separately mounted.
5. Find all `tmpfs` mounts.
6. Identify `/proc`, `/sys`, and `/dev` filesystem types.

## Verification

You should be able to explain:

```text
Device → Filesystem → Mount Point → Options
```

---

# 🧪 LAB 2 — Loopback Filesystem Lab

> Use a disposable VM or lab machine. Do not use a production disk.

## Objective

Create a small filesystem image and mount it through a loop device.

## Create an image

```bash
dd if=/dev/zero of=filesystem.img bs=1M count=100 status=progress
```

> `dd` can destroy data if used with the wrong target. Here the target is intentionally a new regular file named `filesystem.img`.

## Format the image

```bash
mkfs.ext4 filesystem.img
```

⚠️ `mkfs` creates a new filesystem and destroys an existing filesystem structure on its target. Confirm the target carefully.

## Mount

```bash
sudo mkdir -p /mnt/fs-lab
sudo mount -o loop filesystem.img /mnt/fs-lab
```

## Test

```bash
findmnt /mnt/fs-lab
sudo touch /mnt/fs-lab/test.txt
ls -la /mnt/fs-lab
```

## Unmount

```bash
sudo umount /mnt/fs-lab
```

### What you learned

```text
Regular file
    ↓
loop device
    ↓
ext4 filesystem
    ↓
mount point
```

---

# 🧪 LAB 3 — Read-Only Investigation

Using the filesystem image from Lab 2:

```bash
sudo mount -o loop,ro filesystem.img /mnt/fs-lab
```

Verify:

```bash
findmnt /mnt/fs-lab
```

Try creating a file:

```bash
sudo touch /mnt/fs-lab/should-fail.txt
```

The operation should fail because the filesystem is mounted read-only.

Unmount:

```bash
sudo umount /mnt/fs-lab
```

---

# 🧪 LAB 4 — `/etc/fstab` in a Disposable VM

> Perform this only in a disposable VM or controlled lab.

## Objective

Understand how UUID-based entries provide persistent mounting.

Create a test filesystem image, identify its UUID:

```bash
sudo blkid filesystem.img
```

Create a mount point:

```bash
sudo mkdir -p /data-lab
```

Create an appropriate `fstab` entry using the UUID and a suitable filesystem type.

Before trusting the configuration:

```bash
sudo mount -a
```

Then:

```bash
findmnt /data-lab
```

### Challenge

Add `nofail` and explain why it may be useful for optional storage.

---

# 🧪 LAB 5 — Filesystem Health Investigation

## Objective

Practice distinguishing capacity, inode, mount, and kernel-error problems.

Run:

```bash
df -h
```

Then:

```bash
df -i
```

Then:

```bash
findmnt
```

Then:

```bash
journalctl -k --since "2 hours ago"
```

Answer:

1. Which filesystem contains `/`?
2. Is it read-write or read-only?
3. How much space is used?
4. How many inodes are used?
5. Are there recent storage/filesystem errors?

---

# 🛠️ Mini Project — Linux Mount & Filesystem Auditor

Create a script that reports:

- hostname
- current date/time
- block devices
- filesystem types
- UUIDs
- labels
- mount points
- mount options
- disk utilization
- inode utilization
- read-only mounts
- loop devices
- tmpfs mounts

Suggested commands/tools:

```bash
hostname
lsblk -f
findmnt
df -h
df -i
losetup -a
```

Example report structure:

```text
============================
 LINUX FILESYSTEM AUDIT
============================
Hostname: lab-vm

FILESYSTEMS
-----------
...

MOUNTS
------
...

DISK USAGE
----------
...

INODE USAGE
-----------
...

READ-ONLY MOUNTS
----------------
...

LOOP DEVICES
------------
...
```

### Security enhancement

Flag:

```text
WARNING: filesystem is 90%+ full
WARNING: inode usage is 90%+ full
WARNING: unexpected read-only mount
WARNING: suspicious/unexpected mount source
WARNING: executable-capable removable mount (review policy)
```

Do not automatically label something malicious; produce findings for analyst review.

---

# 🔥 Troubleshooting Scenarios

## Scenario 1 — "No space left on device"

Check:

```bash
df -h
```

Then:

```bash
df -i
```

Then identify large directories/files using tools such as:

```bash
du -xhd1 /
```

Interpret the output before deleting anything.

---

## Scenario 2 — Mount Fails

Check:

```bash
lsblk -f
sudo blkid
findmnt
```

Questions:

- Does the device exist?
- Does it contain the expected filesystem?
- Is the filesystem already mounted?
- Is the mount point valid?
- Are there relevant kernel errors?

---

## Scenario 3 — `umount`: Target Is Busy

```bash
sudo fuser -vm /mnt/lab
```

Find the process/user holding the mount, then determine whether it is safe to release it.

---

## Scenario 4 — System Says "Read-only file system"

Check:

```bash
findmnt
```

Then:

```bash
journalctl -k --since "1 hour ago"
```

Look for storage or filesystem errors before attempting a remount.

---

## Scenario 5 — Boot Problem After `fstab` Change

Think:

```text
fstab
 ↓
UUID
 ↓
filesystem type
 ↓
mount point
 ↓
options
```

A typo or unavailable device can prevent an expected mount and, depending on configuration, affect boot behavior.

Use console/recovery procedures appropriate to your VM and distribution to correct the configuration.

---

# 🧠 Important Commands Cheat Sheet

| Goal | Command |
|---|---|
| List block devices | `lsblk` |
| Filesystem details | `lsblk -f` |
| Show UUID/type | `blkid` |
| Show mounts | `findmnt` |
| Disk usage | `df -h` |
| Inode usage | `df -i` |
| Directory usage | `du -sh DIR` |
| Mount | `mount DEVICE DIR` |
| Unmount | `umount DIR` |
| Read-only mount | `mount -o ro ...` |
| Find mount users | `fuser -vm DIR` |
| Open files | `lsof DIR` |
| Loop devices | `losetup -a` |
| Show fstab | `cat /etc/fstab` |
| Test fstab mounts | `mount -a` |
| Kernel logs | `journalctl -k` |
| ext filesystem check | `fsck` / `e2fsck` |
| XFS repair tool | `xfs_repair` |

---

# 🎯 Interview Questions

### Beginner

1. What is a filesystem?
2. What is the difference between a disk and a partition?
3. What is a mount point?
4. What does `mount` do?
5. What does `umount` do?
6. Why do Linux systems use UUIDs in `/etc/fstab`?
7. What is ext4?
8. What is an inode?
9. What is a superblock?
10. What is journaling?

### Intermediate

11. What is the difference between `df` and `du`?
12. Why can a filesystem report no space when `df -h` still shows free capacity?
13. What does `findmnt` show?
14. What is a loop device?
15. What is a bind mount?
16. What is `tmpfs`?
17. What happens if a mount point already contains files?
18. Why can `umount` report that the target is busy?
19. What is the purpose of `mount -a`?
20. Why can a filesystem become read-only?

### Advanced / SOC

21. Why are `nosuid`, `nodev`, and `noexec` useful security controls?
22. Why is `noexec` not a complete application-control boundary?
23. How would you investigate an unexpected mount?
24. How would you investigate an unexpected loop device?
25. Why should filesystem repair tools be used carefully during incident response?
26. How can mount information help identify suspicious activity?
27. What evidence might be changed by mounting a disk read-write?
28. How would you distinguish a legitimate application mount from a suspicious one?
29. What is the difference between filesystem capacity exhaustion and inode exhaustion?
30. Why should you avoid immediately deleting an unfamiliar mounted filesystem during an investigation?

---

# 🔐 SOC / Blue Team Takeaways

Filesystem knowledge is essential for Linux security work.

A SOC analyst may need to understand:

```text
Unexpected mount
      │
      ▼
What device/source?
      │
      ▼
What filesystem?
      │
      ▼
What mount options?
      │
      ▼
What files are present?
      │
      ▼
What process/service created or uses it?
      │
      ▼
What logs/events correlate?
```

Key defensive skills:

- inspect mount topology
- recognize expected vs unexpected storage
- identify filesystem types
- understand read-only behavior
- investigate capacity anomalies
- investigate removable media
- recognize loop-mounted images
- preserve evidence
- correlate mounts with processes/services/logs
- avoid destructive actions before understanding the event

---

# ✅ Module 14 Checklist

- [ ] I understand what a filesystem is.
- [ ] I understand disk vs partition vs filesystem vs mount point.
- [ ] I can explain inode, superblock, data blocks, and journaling at a high level.
- [ ] I can identify filesystems with `lsblk -f`.
- [ ] I can identify UUIDs with `blkid`.
- [ ] I can inspect mounts with `findmnt`.
- [ ] I understand `mount` and `umount`.
- [ ] I understand why a mount can be busy.
- [ ] I understand read-only mounts.
- [ ] I understand basic mount security options.
- [ ] I understand `/etc/fstab`.
- [ ] I understand UUID-based fstab entries.
- [ ] I know why `mount -a` must be used carefully.
- [ ] I understand bind mounts.
- [ ] I understand loop devices.
- [ ] I understand `tmpfs`.
- [ ] I understand the roles of `/proc`, `/sys`, and `/dev`.
- [ ] I can distinguish `df -h` from `df -i`.
- [ ] I understand why filesystem repair requires caution.
- [ ] I can investigate a suspicious mount from a SOC perspective.

---

# 🚀 Next Module

## **15 — Linux Networking**

Next we move from storage to one of the most important Linux administration and SOC domains:

```text
Linux Networking
      │
      ├── Network interfaces
      ├── IP addressing
      ├── Routes
      ├── DNS
      ├── ARP / neighbor discovery
      ├── Ports and sockets
      ├── Network troubleshooting
      ├── Network namespaces
      └── SOC network investigation
```

> **Goal:** understand not only how Linux connects to a network, but how to investigate what a Linux host is communicating with.
