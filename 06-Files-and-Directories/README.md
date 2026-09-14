# 📂 06 — Linux Files and Directories

> Build a strong understanding of the Linux filesystem, file types, paths, links, metadata and everyday file-management operations.

---

## 🎯 Learning Objectives

By the end of this module, you should be able to:

- Understand the Linux filesystem hierarchy
- Explain `/`, `/home`, `/etc`, `/var`, `/tmp`, `/usr`, `/opt`, `/dev`, `/proc` and `/sys`
- Understand Linux files and directories
- Identify common Linux file types
- Create, copy, move and remove files/directories
- Understand hidden files
- Read file metadata
- Understand inodes at a practical level
- Understand hard links and symbolic links
- Use `stat`, `file`, `ls`, `cp`, `mv`, `rm`, `ln` and related tools
- Find files efficiently
- Recognize filesystem artifacts useful in cybersecurity investigations

---

# 🧠 1. The Most Important Linux Filesystem Idea

Linux organizes files and directories into **one hierarchical filesystem tree**.

At the top is:

```text
/
```

called the **root directory**.

A simplified view:

```text
/
├── bin
├── boot
├── dev
├── etc
├── home
│   └── utsav
├── lib
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── srv
├── sys
├── tmp
├── usr
└── var
```

Think of `/` as the top of a huge tree:

```text
                 /
                 │
       ┌─────────┼──────────┐
       │         │          │
      home      etc        var
       │         │          │
     utsav      ...        log
```

---

# 🌳 2. Linux Filesystem Hierarchy

## `/` — Root

The top-level directory of the filesystem namespace.

Do not confuse:

```text
/       → filesystem root
/root   → home directory of the root user
```

These are different things.

---

## `/home` — Normal Users

Usually contains users' home directories.

Example:

```text
/home/utsav
/home/alice
/home/bob
```

A user's personal files, configuration and working data commonly live here.

---

## `/root` — Root User's Home

The home directory of the `root` account.

```text
/root
```

It is not the same as `/`.

---

## `/etc` — System Configuration

Contains system-wide configuration files.

Examples may include:

```text
/etc/hosts
/etc/passwd
/etc/group
/etc/ssh/
/etc/systemd/
```

Security relevance:

> Configuration files can reveal system settings, accounts, services and security controls. Access them only according to your authorization.

---

## `/var` — Changing Data

`/var` commonly stores data that changes during system operation.

Examples:

```text
/var/log
/var/cache
/var/lib
/var/spool
```

Logs are especially important for administrators and SOC analysts.

---

## `/tmp` — Temporary Files

Temporary working data is commonly stored here.

Security note:

> Temporary directories can be useful during investigations because suspicious programs may drop files there. They also contain legitimate temporary data, so location alone is never proof of maliciousness.

---

## `/usr` — User-Space Programs and Data

Contains a large portion of installed user-space software, libraries and shared data on many Linux systems.

Common paths include:

```text
/usr/bin
/usr/sbin
/usr/lib
/usr/share
```

---

## `/bin` and `/sbin`

On many modern distributions these may be symbolic links into `/usr` because of the **usr-merge** approach.

Historically:

```text
/bin  → essential user commands
/sbin → essential system-administration commands
```

Do not assume every modern distribution has an independent physical directory for these paths.

---

## `/boot` — Boot Files

Contains files needed for the boot process, such as kernels and bootloader-related data depending on system configuration.

---

## `/dev` — Devices

Linux represents many devices through special filesystem objects under `/dev`.

Examples can include:

```text
/dev/null
/dev/zero
/dev/random
```

This is one reason Linux is often described with the idea:

> "Everything is a file" — but remember this is a useful simplification, not a statement that every kernel object is literally an ordinary disk file.

---

## `/proc` — Process and Kernel Information

`/proc` is a virtual filesystem exposing information about processes and kernel/system state.

Example:

```text
/proc/1
/proc/cpuinfo
/proc/meminfo
/proc/net
```

A process with PID `1234` may have information under:

```text
/proc/1234/
```

Security relevance:

```text
Process
   ↓
/proc/PID
   ↓
Command line
Environment
File descriptors
Status
   ↓
Investigation
```

---

## `/sys` — Kernel/Device Information

`/sys` is another virtual filesystem exposing kernel and device information.

It is commonly used to inspect hardware and kernel-managed device relationships.

---

## `/run` — Runtime Data

Contains runtime state created after boot and during system operation.

It commonly contains PID files, sockets and other runtime information.

---

## `/opt` — Optional Software

Often used for add-on or third-party application software.

---

## `/mnt` — Temporary Mount Point

Traditionally used as a mount point for temporarily mounted filesystems.

---

## `/media` — Removable Media

Commonly used for automatically mounted removable devices in desktop environments.

---

## 📊 3. Filesystem Quick Reference

| Directory | Simple meaning | Security relevance |
|---|---|---|
| `/` | Filesystem root | Understand entire system |
| `/home` | User homes | User activity/data |
| `/root` | Root user's home | Privileged user data |
| `/etc` | Configuration | Accounts/services/config |
| `/var` | Changing data | Logs/application state |
| `/tmp` | Temporary data | Potential artifacts |
| `/usr` | User-space software/data | Executables/libraries |
| `/boot` | Boot files | Boot investigation |
| `/dev` | Device objects | Device access |
| `/proc` | Process/kernel view | Live process investigation |
| `/sys` | Kernel/device view | Hardware/kernel investigation |
| `/run` | Runtime state | Services/process artifacts |
| `/opt` | Optional software | Third-party applications |

---

# 📄 4. What Is a File?

A file is a named object used to store or represent information.

Common examples:

```text
notes.txt
config.conf
script.sh
photo.jpg
access.log
```

But Linux supports more than ordinary files.

---

# 🧩 5. Linux File Types

Use:

```bash
ls -l
```

The first character in the permissions display indicates the file type.

Common types:

```text
-  regular file
d  directory
l  symbolic link
c  character device
b  block device
s  socket
p  named pipe (FIFO)
```

Example:

```text
-rw-r--r--  regular file
 drwxr-xr-x  directory
 lrwxrwxrwx  symbolic link
```

The exact permissions after the first character are a separate topic covered deeply in **09 — File Permissions**.

---

# 🔎 6. The `file` Command

Use `file` to identify the type/content characteristics of a file:

```bash
file notes.txt
```

For an executable:

```bash
file /usr/bin/ls
```

This is useful because a filename extension does not reliably determine file type on Linux.

For example:

```text
malware.jpg
```

could technically contain something other than a JPEG image.

Do not trust filenames alone.

---

# 👻 7. Hidden Files

Linux commonly treats names beginning with `.` as hidden from ordinary `ls` output.

Example:

```text
.bashrc
.profile
.ssh/
```

Show them:

```bash
ls -la
```

Important:

> Hidden does not mean secure, encrypted or inaccessible.

It primarily affects normal directory listing behavior.

---

# ✏️ 8. Creating Files

Create an empty file:

```bash
touch notes.txt
```

Create multiple files:

```bash
touch one.txt two.txt three.txt
```

Create a file with content:

```bash
echo "Hello Linux" > hello.txt
```

Append:

```bash
echo "Second line" >> hello.txt
```

---

# 📁 9. Creating Directories

```bash
mkdir lab
```

Create nested directories:

```bash
mkdir -p ~/linux-lab/files/logs
```

The `-p` option creates missing parent directories as needed.

---

# 📋 10. Copying Files

Copy one file:

```bash
cp source.txt destination.txt
```

Copy into a directory:

```bash
cp source.txt ~/linux-lab/
```

Copy a directory recursively:

```bash
cp -r source-dir destination-dir
```

For larger or more advanced copy operations, tools such as `rsync` become important later.

---

# 🚚 11. Moving and Renaming

Linux commonly uses `mv` for both moving and renaming.

Rename:

```bash
mv old.txt new.txt
```

Move:

```bash
mv new.txt ~/linux-lab/
```

Visual:

```text
old.txt
   │
   │ mv
   ▼
new.txt
```

---

# 🗑️ 12. Deleting Files

```bash
rm file.txt
```

Remove an empty directory:

```bash
rmdir empty-directory
```

Remove a directory and its contents recursively:

```bash
rm -r directory
```

### Safety rule

Before a destructive command, inspect:

```bash
pwd
ls -lah
```

Be especially cautious with:

```bash
rm -r
rm -rf
sudo rm
```

Never execute destructive commands against systems or data without authorization.

---

# 🔗 13. Symbolic Links

A symbolic link is a filesystem object that points to another path.

Create one:

```bash
ln -s original.txt shortcut.txt
```

Visual:

```text
shortcut.txt
      │
      │ points to
      ▼
 original.txt
```

Check:

```bash
ls -l shortcut.txt
```

You may see something like:

```text
shortcut.txt -> original.txt
```

---

# 🧱 14. Hard Links

A hard link is another directory entry referring to the same underlying inode/data object on a filesystem that supports hard links.

Create one:

```bash
ln original.txt hardlink.txt
```

Simplified mental model:

```text
original.txt ──┐
               ├──> inode ──> file data
hardlink.txt ──┘
```

Deleting one directory entry does not necessarily delete the underlying data if another hard link still references it.

Important limitations and filesystem rules apply; for example, hard links generally cannot cross filesystems, and directories normally cannot be hard-linked by ordinary users.

---

# 🔍 15. Symbolic Link vs Hard Link

| Feature | Symbolic Link | Hard Link |
|---|---|---|
| Points to | Path | Same inode |
| Can cross filesystem boundaries | Usually yes | No |
| Can point to directory | Yes, with caveats | Normally no for users |
| Can become broken | Yes | Not in the same way |
| Identified by `ls -l` | Shows `->` | Looks like another file name |

The practical distinction is:

```text
symlink → path reference
hard link → additional name for same inode
```

---

# 🧬 16. What Is an Inode?

An **inode** stores filesystem metadata about an object, such as information associated with ownership, permissions, timestamps and links to the file's data blocks, depending on filesystem implementation.

The filename is associated with a directory entry that references an inode.

Simplified:

```text
Directory entry
      │
      ▼
    inode
      │
      ├── metadata
      ├── permissions
      ├── ownership
      └── data block references
```

A useful mental model:

```text
filename ≠ file's complete identity
```

---

# 🔎 17. Inspecting Metadata with stat

Use:

```bash
stat file.txt
```

You can inspect:

- File type
- Size
- Inode
- Permissions
- Owner
- Group
- Access time
- Modification time
- Change time
- Link count

Example concepts:

```text
Access     → last access time
Modify     → content modification time
Change     → metadata/inode change time
```

Do not confuse these timestamps.

---

# ⏰ 18. Linux File Timestamps

Linux filesystems commonly expose timestamps such as:

### atime
Access time.

### mtime
Modification time of file contents.

### ctime
Change time for filesystem metadata/inode state.

### Birth time
Some filesystems/tools can expose file creation/birth time, but availability and semantics vary.

Check:

```bash
stat file.txt
```

Security relevance:

> Timestamps can help build a timeline, but they should not be treated as unquestionable proof. Attackers, administrators and normal system processes can modify files and metadata, and filesystem behavior varies.

---

# 🔐 19. Filesystem Permissions Preview

You will study permissions deeply later, but learn to read this basic structure:

```text
-rwxr-xr--
│││ │││ │││
│││ │││ ││└── others
│││ │││ └───── others
│││ │└└─────── group
│││ └───────── group
│└└─────────── owner
└───────────── file type
```

The nine permission positions represent:

```text
owner | group | others
 rwx   | rwx   | rwx
```

We will break this down completely in **09 — File Permissions**.

---

# 📊 20. Directory Size vs File Size

A common beginner mistake is assuming:

```bash
ls -lh directory
```

shows the total size of everything inside the directory.

It does not provide a recursive total in the way beginners often expect.

For directory disk usage:

```bash
du -sh directory
```

For filesystem free/used space:

```bash
df -h
```

Mental model:

```text
ls → directory entries

du → disk usage

df → filesystem capacity
```

---

# 🔎 21. Finding Files

Basic name search:

```bash
find ~/linux-lab -name "*.txt"
```

Find regular files:

```bash
find ~/linux-lab -type f
```

Find directories:

```bash
find ~/linux-lab -type d
```

Find recently modified files:

```bash
find ~/linux-lab -type f -mtime -1
```

Find by size:

```bash
find ~/linux-lab -type f -size +1M
```

Find and display metadata:

```bash
find ~/linux-lab -type f -exec stat {} \;
```

Use `find` carefully on very large trees.

---

# 🧭 22. Useful File-Management Commands

| Command | Purpose |
|---|---|
| `pwd` | Current directory |
| `ls` | List entries |
| `cd` | Change directory |
| `mkdir` | Create directory |
| `rmdir` | Remove empty directory |
| `touch` | Create/update file timestamp |
| `cp` | Copy |
| `mv` | Move/rename |
| `rm` | Remove |
| `ln` | Create links |
| `file` | Identify file type |
| `stat` | Show metadata |
| `find` | Search filesystem |
| `du` | Disk usage |
| `df` | Filesystem free space |
| `readlink` | Inspect symlink target |

---

# 🧪 23. Hands-On Lab — Build a Linux Filesystem Lab

## 🎯 Objective

Practice creating, moving, copying and inspecting files.

Create the lab:

```bash
mkdir -p ~/linux-lab/files/{documents,logs,backup}
cd ~/linux-lab/files
```

Create files:

```bash
touch documents/report.txt
touch logs/application.log
```

Add content:

```bash
echo "Linux filesystem practice" > documents/report.txt
echo "INFO: application started" > logs/application.log
```

Inspect:

```bash
find . -type f -print
```

Copy:

```bash
cp documents/report.txt backup/report.txt
```

Rename:

```bash
mv backup/report.txt backup/report-v1.txt
```

Inspect metadata:

```bash
stat backup/report-v1.txt
```

---

# 🧪 24. Hands-On Lab — Links

Create a test file:

```bash
cd ~/linux-lab/files
echo "link laboratory" > original.txt
```

Create a symbolic link:

```bash
ln -s original.txt symbolic.txt
```

Create a hard link:

```bash
ln original.txt hard.txt
```

Inspect:

```bash
ls -li original.txt symbolic.txt hard.txt
```

### Questions

- Which entries share an inode number?
- Which entry shows `->`?
- What happens if you edit `original.txt`?
- What happens if you remove the original filename?
- Why does the symbolic link behave differently from the hard link?

---

# 🧪 25. Hands-On Lab — Filesystem Investigation

Use your own lab directory.

Find all files:

```bash
find ~/linux-lab -type f
```

Find files modified within the last day:

```bash
find ~/linux-lab -type f -mtime -1 -ls
```

Inspect metadata:

```bash
stat ~/linux-lab/files/documents/report.txt
```

Identify file types:

```bash
file ~/linux-lab/files/documents/report.txt
```

### Investigation questions

```text
What files exist?
       ↓
Which are recent?
       ↓
Which users own them?
       ↓
What permissions do they have?
       ↓
What are their timestamps?
       ↓
Are any symbolic links present?
```

---

# 🔐 26. Cybersecurity Perspective

Filesystem knowledge is fundamental to host-based security investigations.

When investigating a Linux system, defenders may need to understand:

```text
Users
 ↓
/home
 ↓
Processes
 ↓
/proc
 ↓
Services
 ↓
/etc
 ↓
Logs
 ↓
/var/log
 ↓
Temporary artifacts
 ↓
/tmp / /var/tmp
```

Examples of useful defensive questions:

- Which files changed recently?
- Which configuration files changed?
- Which executable is being launched by a suspicious service?
- What user owns the file?
- What permissions does it have?
- Is there an unexpected symbolic link?
- What process currently has the file open?

Later tools such as `lsof`, `ps`, `ss`, `journalctl` and auditing tools will extend this investigation workflow.

---

# 🧠 27. Why `/proc` Matters to SOC Analysts

`/proc` gives a live view into processes and kernel information.

For a process:

```text
/proc/<PID>/
```

Useful areas include:

```text
/proc/<PID>/cmdline
/proc/<PID>/status
/proc/<PID>/fd/
/proc/<PID>/environ
```

Access to some information may be restricted by permissions or system security settings.

A useful mental model:

```text
Suspicious PID
     │
     ▼
/proc/PID
     │
     ├── What command?
     ├── What user?
     ├── What state?
     ├── What file descriptors?
     └── What environment?
```

Use only on systems you are authorized to inspect.

---

# 🚨 28. Troubleshooting Scenario

### Problem

You ran:

```bash
cat report.txt
```

and Linux says:

```text
No such file or directory
```

Do not immediately recreate the file.

Investigate:

```bash
pwd
ls -lah
find . -name "report.txt"
```

Then ask:

```text
Am I in the correct directory?
       ↓
Does the file exist?
       ↓
Did I type the name correctly?
       ↓
Is the path relative or absolute?
```

This is a basic but important troubleshooting habit.

---

# ⚠️ 29. Common Mistakes

### 1. Confusing `/` with `/root`

They are completely different paths.

### 2. Assuming extensions define file type

Linux does not require a file extension to determine what a file is.

### 3. Forgetting hidden files

Use:

```bash
ls -la
```

### 4. Using `rm -r` carelessly

Always inspect the target first.

### 5. Assuming `ls -l` gives total directory size

Use `du` for disk usage.

### 6. Confusing ctime with creation time

`ctime` is metadata change time, not universally "creation time".

### 7. Treating timestamps as absolute evidence

Timestamps need context and corroboration.

---

# 🎯 30. Scenario Challenge — Find the Artifact

You are investigating an authorized Linux lab machine.

A security alert says a suspicious script may have been dropped recently in the lab user's temporary area.

Your job is to investigate **without deleting or modifying anything**.

### Tasks

1. Identify files modified recently in the designated investigation directory.
2. Determine which files are regular files.
3. Identify their file types.
4. Inspect ownership and permissions.
5. Inspect timestamps.
6. Check for symbolic links.
7. Record your findings.

Useful commands:

```bash
find /var/tmp -type f -mtime -1 -ls
file /path/to/suspicious-file
stat /path/to/suspicious-file
ls -la /var/tmp
```

### Investigation principle

```text
Observe
  ↓
Collect
  ↓
Correlate
  ↓
Analyze
  ↓
Document
```

Do not modify evidence merely because it looks suspicious.

---

# 🎤 31. Interview Questions

## 🟢 Beginner

1. What is the Linux root directory?
2. What is `/home` used for?
3. What is `/etc`?
4. What is `/var`?
5. What is `/tmp`?
6. What is `/root`?
7. What does `ls -la` show?
8. What is a hidden file?
9. What does `cp` do?
10. What does `mv` do?

## 🟡 Intermediate

1. What is an inode?
2. What is the difference between a hard link and symbolic link?
3. What is `/proc`?
4. What is `/sys`?
5. What does `stat` show?
6. What is the difference between `du` and `df`?
7. Why doesn't a filename extension necessarily identify a Linux file type?
8. What do the first characters of `ls -l` output represent?

## 🔴 Advanced / Security

1. How could `/proc` help during an incident investigation?
2. Why can symbolic links matter in security investigations?
3. How would you identify recently modified files?
4. Why should timestamps not be treated as unquestionable evidence?
5. Which Linux directories would you examine during host-based incident response and why?

---

# ⚡ 32. Quick Revision

```text
/       → filesystem root
/home   → normal users
/root   → root user's home
/etc    → configuration
/var    → changing data/logs
/tmp    → temporary data
/usr    → user-space software/data
/boot   → boot-related files
/dev    → device objects
/proc   → process/kernel information
/sys    → kernel/device information
/run    → runtime state
/opt    → optional software
```

Core commands:

```bash
ls
cd
pwd
mkdir
touch
cp
mv
rm
ln
file
stat
find
du
df
readlink
```

Link model:

```text
Symbolic link → path
Hard link     → same inode
```

---

# 🧠 33. Mental Model

When working with Linux files, think in layers:

```text
Filesystem tree
       ↓
Directory entry / filename
       ↓
Inode / metadata
       ↓
File data
       ↓
Filesystem / storage
```

And when investigating:

```text
File
 ↓
Type
 ↓
Owner
 ↓
Permissions
 ↓
Timestamps
 ↓
Links
 ↓
Process using it
 ↓
Logs / surrounding evidence
```

This mental model will become increasingly important when we reach permissions, processes, storage and Linux security.

---

# ✅ Module Checklist

- [ ] I understand the Linux filesystem tree
- [ ] I know the purpose of major directories
- [ ] I understand `/` vs `/root`
- [ ] I understand hidden files
- [ ] I can identify common file types
- [ ] I can create files and directories
- [ ] I can copy files
- [ ] I can move/rename files
- [ ] I can remove files safely
- [ ] I understand symbolic links
- [ ] I understand hard links
- [ ] I understand the basic idea of an inode
- [ ] I can use `stat`
- [ ] I understand atime/mtime/ctime
- [ ] I understand `du` vs `df`
- [ ] I can use `find`
- [ ] I understand why filesystem knowledge matters to SOC analysts

---

## ➡️ Next Module

**07 — Text Processing and Editors**

We will learn `cat`, `less`, `head`, `tail`, `grep`, `sort`, `uniq`, `cut`, `tr`, `sed`, `awk`, `nano`, `vim`, regular expressions and practical log-analysis workflows.