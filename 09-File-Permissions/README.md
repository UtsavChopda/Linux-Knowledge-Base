# 🔐 09 — File Permissions

> **Linux Knowledge Base | Beginner → Administration → Security → SOC/Blue Team**

Linux permissions answer a fundamental security question:

> **Who is allowed to do what to this resource?**

A strong understanding of permissions is essential for Linux administration, troubleshooting, hardening, incident response, and understanding privilege-escalation risk.

---

## 🎯 Learning Objectives

By the end of this module you should be able to:

- Read Linux permission strings
- Understand owner, group and other
- Understand `r`, `w`, and `x` for files and directories
- Use symbolic and numeric `chmod`
- Change ownership with `chown` and `chgrp`
- Understand `umask`
- Understand SUID, SGID and sticky bit
- Troubleshoot access-denied errors systematically
- Recognize dangerous permission configurations
- Audit permissions from a defensive/SOC perspective
- Understand why permissions matter to privilege escalation

---

# 1. Why Permissions Exist

Imagine a Linux server with three users:

```text
Alice ──┐
Bob   ──┼──► Linux Server ──► /data/company.txt
Carol ──┘
```

Should all three be able to:

- Read the file?
- Modify it?
- Delete it?

Linux permissions provide a basic access-control mechanism.

```text
             RESOURCE
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
     OWNER     GROUP    OTHER
       │        │        │
       ▼        ▼        ▼
     rwx       rwx      rwx
```

Permissions are evaluated together with ownership, identity, directory traversal permissions, ACLs, security modules and other controls.

---

# 2. The Three Permission Classes

Traditional Unix permissions divide access into:

| Class | Meaning |
|---|---|
| User/Owner | The file's owning user |
| Group | The file's owning group |
| Other | Everyone else |

Example:

```text
-rwxr-x---
```

Break it apart:

```text
-   rwx   r-x   ---
│    │     │     │
│    │     │     └── Other
│    │     └──────── Group
│    └────────────── Owner
└─────────────────── File type
```

---

# 3. Reading `ls -l`

Run:

```bash
ls -l file.txt
```

Example:

```text
-rw-r----- 1 alice security 1200 Sep 14 12:00 file.txt
```

Important fields:

```text
-rw-r-----
│  │ │ │
│  │ │ └── permissions
│  │ └──── group
│  └────── owner
└───────── file type + permissions
```

A more complete mental model:

```text
TYPE | OWNER | GROUP | OTHER | LINK COUNT | OWNER | GROUP | SIZE | TIME | NAME
```

---

# 4. File Type Character

The first character of a permission string identifies the file type.

Common examples:

| Character | Type |
|---|---|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device |
| `b` | Block device |
| `p` | Named pipe |
| `s` | Socket |

Example:

```text
drwxr-xr-x
```

The `d` means directory.

---

# 5. `r`, `w`, `x`

For a **regular file**:

| Permission | Meaning |
|---|---|
| `r` | Read file contents |
| `w` | Modify file contents |
| `x` | Execute file as a program/script when otherwise permitted |

For a **directory**, the meaning changes:

| Permission | Typical meaning |
|---|---|
| `r` | List directory entries |
| `w` | Create/delete/rename entries, subject to other controls |
| `x` | Traverse/access objects inside the directory when allowed |

This distinction is extremely important.

---

# 6. Directory `x` Permission

Suppose:

```text
/data/reports/report.txt
```

To access the file, the user generally needs appropriate execute/traverse permission on the directories in the path.

```text
/
 │
 └── data
      │
      └── reports
             │
             └── report.txt
```

A user might have read permission on `report.txt` but still fail to access it because they cannot traverse a parent directory.

### Key lesson

> File permissions alone do not tell the complete access story.

---

# 7. Permission Values

Each basic permission has a numeric value:

| Permission | Value |
|---|---:|
| `r` | 4 |
| `w` | 2 |
| `x` | 1 |
| `-` | 0 |

Add them together.

```text
rwx = 4 + 2 + 1 = 7
rw- = 4 + 2     = 6
r-x = 4     + 1 = 5
r-- = 4         = 4
-wx =     2 + 1 = 3
-w- =     2     = 2
--x =         1 = 1
--- = 0
```

---

# 8. Numeric Permission Notation

Example:

```text
rwxr-xr--
```

Convert each triplet:

```text
rwx = 7
r-x = 5
r-- = 4
```

Therefore:

```text
754
```

Visual:

```text
        OWNER   GROUP   OTHER
          7       5       4
         rwx     r-x     r--
```

---

# 9. Common Permission Modes

| Mode | Owner | Group | Other | Typical interpretation |
|---:|---|---|---|---|
| `600` | `rw-` | `---` | `---` | Owner only read/write |
| `640` | `rw-` | `r--` | `---` | Owner read/write, group read |
| `644` | `rw-` | `r--` | `r--` | Common non-executable file |
| `700` | `rwx` | `---` | `---` | Owner only |
| `750` | `rwx` | `r-x` | `---` | Owner full, group read/execute |
| `755` | `rwx` | `r-x` | `r-x` | Common executable/directory mode |
| `770` | `rwx` | `rwx` | `---` | Owner/group full access |
| `777` | `rwx` | `rwx` | `rwx` | Everyone full access — usually risky |

These are examples, not universal requirements. The correct mode depends on the purpose of the resource.

---

# 10. `chmod` — Change Permissions

Syntax:

```bash
chmod MODE FILE
```

Numeric example:

```bash
chmod 640 report.txt
```

Check:

```bash
ls -l report.txt
```

---

# 11. Symbolic `chmod`

Instead of numbers, use:

```text
u = user/owner
 g = group
 o = other
 a = all
```

Add execute permission for owner:

```bash
chmod u+x script.sh
```

Remove write permission from group:

```bash
chmod g-w file.txt
```

Add read permission for everyone:

```bash
chmod a+r file.txt
```

Set exact owner permissions:

```bash
chmod u=rw file.txt
```

Multiple changes:

```bash
chmod u=rwx,g=rx,o= file.txt
```

---

# 12. Recursive `chmod`

```bash
chmod -R 755 directory
```

### ⚠️ Dangerous habit

Do not blindly run:

```bash
chmod -R 777 /some/directory
```

Recursive permission changes can expose sensitive files, break application behavior, or create security weaknesses.

For mixed file/directory trees, files and directories often need different permissions. Inspect before changing.

---

# 13. `chown` — Change Owner

Syntax:

```bash
chown user file
```

Example:

```bash
sudo chown alice report.txt
```

Change owner and group:

```bash
sudo chown alice:security report.txt
```

Recursive ownership change:

```bash
sudo chown -R alice:security /path/to/directory
```

Again, verify the target carefully before using `-R`.

---

# 14. `chgrp` — Change Group

```bash
sudo chgrp security report.txt
```

Verify:

```bash
ls -l report.txt
```

Ownership and permissions work together:

```text
OWNER + GROUP + MODE
        │
        ▼
   ACCESS DECISION
```

---

# 15. `stat` — Detailed Metadata

```bash
stat file.txt
```

Useful information includes:

- File type
- Size
- Permissions
- UID
- GID
- Inode
- Timestamps

Example permission-related query:

```bash
stat -c '%A %a %U %G %n' file.txt
```

Possible output:

```text
-rw-r----- 640 alice security file.txt
```

---

# 16. Access Testing with `namei`

For a path such as:

```text
/var/www/site/index.html
```

`namei` can help break the path into components:

```bash
namei -l /var/www/site/index.html
```

This is extremely useful when diagnosing:

```text
Permission denied
```

because it lets you inspect permissions along the directory path.

---

# 17. Permission Decision — Simplified Model

A useful mental model is:

```text
Who am I?
    ↓
What groups do I have?
    ↓
Who owns the resource?
    ↓
What group owns it?
    ↓
What permissions apply?
    ↓
Can I traverse the path?
    ↓
Are ACLs/security controls involved?
    ↓
ACCESS / DENIED
```

Real Linux authorization can involve more mechanisms, including ACLs, capabilities, SELinux/AppArmor policies, mount options and application-level controls.

---

# 18. Permission Troubleshooting Example

Suppose:

```bash
cat /shared/report.txt
```

returns:

```text
Permission denied
```

Do not immediately use `chmod 777`.

Investigate:

```bash
whoami
id
ls -l /shared/report.txt
namei -l /shared/report.txt
```

Then inspect ACLs if relevant:

```bash
getfacl /shared/report.txt
```

This approach identifies the actual access-control problem instead of weakening the system.

---

# 19. `umask` — Default Permission Mask

`umask` influences the permissions removed from newly created files and directories.

Check:

```bash
umask
```

Example:

```text
0022
```

Conceptually:

```text
Base creation permissions
          │
          ▼
       umask
          │
          ▼
Final default permissions
```

For regular files, the maximum initial permission base is commonly `666`; for directories, commonly `777`. The actual result also depends on the program creating the object.

Do not think of `umask` as simply “the permissions of every new file.” It is a mask applied by the creating process.

---

# 20. Testing `umask`

Create a temporary lab directory:

```bash
mkdir -p ~/linux-labs/permissions
cd ~/linux-labs/permissions
```

Check current mask:

```bash
umask
```

Create a file and directory:

```bash
touch test.txt
mkdir testdir
```

Inspect:

```bash
ls -ld test.txt testdir
```

This lets you observe how the current process's `umask` influences newly created objects.

---

# 21. SUID — Set User ID

SUID is a special permission bit primarily relevant to executable files.

A SUID executable can run with the effective user identity of the file owner, rather than simply the identity of the user launching it.

Conceptually:

```text
User launches program
        │
        ▼
SUID executable
        │
        ▼
Effective identity may become
file owner's identity
```

Historically, this mechanism is used by some legitimate system programs that need narrowly scoped elevated operations.

### Security significance

Unexpected or vulnerable SUID executables can create privilege-escalation risk.

---

# 22. Finding SUID Files

On an authorized Linux system:

```bash
find / -type f -perm -4000 2>/dev/null
```

Inspect results carefully.

You can also use:

```bash
find / -type f -perm -4000 -ls 2>/dev/null
```

### Analyst questions

For an unusual SUID binary:

- Is it expected?
- What package installed it?
- Who owns it?
- Has it recently changed?
- Is the software version trusted?
- Does it have a legitimate reason to require SUID?

Finding a SUID file is **not itself proof of compromise**.

---

# 23. SGID — Set Group ID

SGID has different behavior depending on the object type.

For an executable, SGID can cause the process to use the file's group identity as its effective group in relevant contexts.

For a directory, SGID commonly causes newly created files and directories to inherit the directory's group ownership.

Example:

```text
/shared/security
        │
        ├── report1.txt → group security
        ├── report2.txt → group security
        └── report3.txt → group security
```

This is very useful for collaborative directories.

---

# 24. Setting SGID on a Directory

Example lab:

```bash
mkdir ~/linux-labs/shared
chmod 2770 ~/linux-labs/shared
```

The leading `2` represents SGID in numeric notation.

Inspect:

```bash
ls -ld ~/linux-labs/shared
```

You may see an `s` in the group execute position:

```text
drwxrws---
```

---

# 25. Sticky Bit

The sticky bit on a directory commonly means that users cannot arbitrarily delete or rename files owned by other users within that directory, subject to the system's access controls.

The classic example is:

```text
/tmp
```

Inspect:

```bash
ls -ld /tmp
```

You may see:

```text
drwxrwxrwt
```

The final `t` indicates the sticky bit.

---

# 26. Special Permission Numeric Values

| Special bit | Numeric value |
|---|---:|
| SUID | 4 |
| SGID | 2 |
| Sticky | 1 |

Example:

```text
4755
```

means:

```text
4    755
│     │
SUID  normal permissions
```

Example:

```text
2770
```

means:

```text
2    770
│     │
SGID  normal permissions
```

Example:

```text
1777
```

means:

```text
1    777
│     │
sticky normal permissions
```

---

# 27. Symbolic Special Permissions

Set SUID:

```bash
chmod u+s program
```

Set SGID:

```bash
chmod g+s directory
```

Set sticky bit:

```bash
chmod +t directory
```

Remove them:

```bash
chmod u-s program
chmod g-s directory
chmod -t directory
```

Always verify afterward:

```bash
ls -l
```

---

# 28. Understanding `s`, `S`, `t`, and `T`

You may see:

```text
-rwsr-xr-x
```

Lowercase `s` means the special bit is set and the corresponding execute bit is also set.

You may also encounter uppercase `S` when the special bit is set but the corresponding execute bit is not set.

Likewise:

```text
drwxrwxrwt
```

shows sticky bit plus execute permission.

Uppercase `T` indicates sticky bit set without the corresponding execute bit.

---

# 29. `777` — Why It Is Usually a Bad Fix

When a user sees:

```text
Permission denied
```

one of the worst quick fixes is:

```bash
chmod 777 file
```

Why?

```text
Owner  → read/write/execute
Group  → read/write/execute
Other  → read/write/execute
```

This may give every local user broad access.

Better approach:

```text
Identify required access
        ↓
Identify correct owner/group
        ↓
Set minimum permissions
        ↓
Test
        ↓
Verify
```

---

# 30. Permissions and Security Boundaries

Permissions help enforce boundaries between:

```text
User A
  │
  ├── personal files
  │
  └── allowed shared resources

User B
  │
  ├── personal files
  │
  └── allowed shared resources
```

A compromised low-privilege account is much less dangerous when it cannot write to sensitive system files or execute privileged resources.

---

# 31. World-Writable Files

A world-writable file can be modified by users outside the owner/group classes.

Find regular files writable by everyone in an authorized lab:

```bash
find / -type f -perm -0002 -ls 2>/dev/null
```

Do not automatically treat every result as malicious. Some applications legitimately require shared writable locations.

Investigate:

```text
Why is it writable?
Who needs write access?
Who owns it?
What program uses it?
Can modifying it influence a privileged process?
```

---

# 32. World-Writable Directories

Find directories writable by everyone:

```bash
find / -type d -perm -0002 -ls 2>/dev/null
```

A world-writable directory can be legitimate, especially for temporary/shared data, but it deserves careful review when it is part of a privileged application's path.

The sticky bit can reduce arbitrary deletion/rename behavior in shared directories.

---

# 33. Files Owned by Unexpected Users

Ownership anomalies can be useful investigation clues.

For example:

```bash
find /var/www -user unexpecteduser -ls
```

The correct command depends on the account and path being investigated.

Never assume unusual ownership equals compromise. Validate against deployment processes, package installation, service accounts and administrative changes.

---

# 34. Permission Investigation Workflow

When investigating a suspicious file:

```text
                 Suspicious file
                       │
                       ▼
                    stat
                       │
              ┌────────┴────────┐
              ▼                 ▼
          Ownership          Permissions
              │                 │
              └────────┬────────┘
                       ▼
                    getfacl
                       │
                       ▼
                 Parent paths
                       │
                       ▼
                  Processes
                       │
                       ▼
                     Logs
                       │
                       ▼
                   Correlate
```

This is much stronger than looking only at the permission string.

---

# 35. ACL Preview

Traditional permissions are not always enough.

Access Control Lists can provide additional per-user/per-group rules.

Inspect an ACL:

```bash
getfacl file.txt
```

You may see entries such as:

```text
user::rw-
user:alice:r--
group::r--
mask::r--
other::---
```

The ACL **mask** can limit effective permissions for named users/groups and the owning group.

Detailed ACL management is covered further in **Module 10 — Ownership and ACLs**.

---

# 36. Capabilities Preview

Modern Linux can grant specific privileges through **capabilities** instead of giving a process complete root authority.

Inspect capabilities on an authorized system with:

```bash
getcap -r /usr/bin /usr/sbin 2>/dev/null
```

Capabilities are more granular than traditional UID 0 access, but unexpected capabilities can still create security risk.

Detailed capabilities will be revisited in the security/hardening modules.

---

# 37. SOC Example — Suspicious Script Permissions

Imagine an analyst finds:

```text
/opt/app/maintenance.sh
```

with:

```text
-rwxrwxrwx
```

Do not immediately change it.

Investigate:

```bash
ls -l /opt/app/maintenance.sh
stat /opt/app/maintenance.sh
getfacl /opt/app/maintenance.sh
```

Then determine:

- Who owns it?
- What group owns it?
- Who can modify it?
- Which process executes it?
- Is it launched by a privileged service or scheduled job?
- Was it recently changed?
- Is it part of an approved application?

### Why this matters

A writable script executed by a privileged process can create a serious privilege boundary problem.

The investigation question is not simply:

> “Is this file writable?”

It is:

> **“Who can modify this file, and what security-sensitive action will consume it afterward?”**

---

# 38. SOC Example — SUID Audit

Run in an authorized lab:

```bash
find / -type f -perm -4000 -ls 2>/dev/null | tee suid-audit.txt
```

Review:

```bash
less suid-audit.txt
```

For each unusual result, record:

```text
Path:
Owner:
Group:
Permissions:
Package/source:
Expected? 
Recent change?
Security impact:
```

This becomes useful later in Linux hardening and privilege-escalation detection labs.

---

# 39. Hands-On Lab 1 — Basic Permissions

> Use a disposable Linux VM or lab account. Do not perform these experiments on important system files.

Create:

```bash
mkdir -p ~/linux-labs/permissions
cd ~/linux-labs/permissions

touch report.txt
mkdir reports
```

Inspect:

```bash
ls -l
```

Set:

```bash
chmod 640 report.txt
chmod 750 reports
```

Verify:

```bash
ls -ld report.txt reports
```

### Challenge

Convert these to symbolic permissions:

```text
600
644
700
750
755
770
```

Then convert:

```text
rwxr-x---
rw-r-----
r-xr-xr-x
```

to numeric notation.

---

# 40. Hands-On Lab 2 — Owner and Group

Create a test group:

```bash
sudo groupadd labfiles
```

Create a test user if needed:

```bash
sudo useradd -m permuser
```

Create a file:

```bash
touch shared.txt
```

Change group:

```bash
sudo chgrp labfiles shared.txt
```

Set permissions:

```bash
chmod 640 shared.txt
```

Inspect:

```bash
ls -l shared.txt
```

### Questions

1. Who owns the file?
2. Which group owns it?
3. What can the owner do?
4. What can the group do?
5. What can other users do?

Clean up later according to your lab procedure.

---

# 41. Hands-On Lab 3 — Directory Permissions

Create:

```bash
mkdir -p ~/linux-labs/dir-test
cd ~/linux-labs/dir-test

touch secret.txt
```

Test these modes one at a time:

```bash
chmod 700 .
chmod 750 .
chmod 755 .
```

Use another authorized lab account to observe the difference.

Pay particular attention to the directory's `x` permission.

### Goal

Understand why:

```text
Directory read ≠ directory access
```

and why:

```text
Directory execute/traverse
```

is critical for accessing objects inside a directory.

---

# 42. Hands-On Lab 4 — Special Permissions

Create:

```bash
mkdir -p ~/linux-labs/special
cd ~/linux-labs/special
```

Create a shared directory:

```bash
mkdir shared
chmod 2770 shared
```

Inspect:

```bash
ls -ld shared
```

Create a temporary shared directory for sticky-bit testing:

```bash
mkdir sticky
chmod 1777 sticky
```

Inspect:

```bash
ls -ld sticky
```

You should observe:

```text
SGID   → s in group position
Sticky → t in other execute position
```

Do not experiment with SUID on important system executables.

---

# 43. Hands-On Lab 5 — Permission Troubleshooting

Create:

```bash
mkdir -p ~/linux-labs/access/a/b
printf 'secret\n' > ~/linux-labs/access/a/b/secret.txt
chmod 600 ~/linux-labs/access/a/b/secret.txt
```

Inspect:

```bash
ls -l ~/linux-labs/access/a/b/secret.txt
namei -l ~/linux-labs/access/a/b/secret.txt
```

Change a parent directory permission in the lab and observe the resulting access behavior.

### Investigation checklist

```bash
whoami
id
ls -l file
namei -l path
getfacl file
```

This teaches a repeatable troubleshooting process.

---

# 44. Scenario Challenge — Writable Privileged Script

## Scenario

You are investigating an authorized Linux server. A service runs a maintenance script as a privileged account:

```text
/opt/maintenance/cleanup.sh
```

You discover that a normal user can modify the script.

## Your task

Determine the security significance without changing the system.

### Investigate

```bash
ls -l /opt/maintenance/cleanup.sh
stat /opt/maintenance/cleanup.sh
getfacl /opt/maintenance/cleanup.sh
```

Identify the parent directory permissions:

```bash
namei -l /opt/maintenance/cleanup.sh
```

Then determine how the script is executed using authorized system-inspection methods.

### Questions

- Who owns the script?
- Who can modify it?
- Who executes it?
- Is execution privileged?
- Is the configuration legitimate?
- Is there an approved change record?

### Important

Do **not** replace the script, add commands, or attempt privilege escalation. The objective is defensive analysis, not exploitation.

---

# 45. Scenario Challenge — Unexpected SUID Binary

You discover:

```text
/usr/local/bin/custom-tool
```

with SUID enabled.

Investigate:

```bash
ls -l /usr/local/bin/custom-tool
stat /usr/local/bin/custom-tool
file /usr/local/bin/custom-tool
```

Search package ownership where supported by your distribution's package manager.

Questions:

```text
Who installed it?
Why is it SUID?
Is it documented?
When did it appear?
Is it trusted software?
Is the permission still required?
```

Document evidence before taking remediation action.

---

# 46. Common Mistakes

### Mistake 1 — Memorizing numbers without understanding `rwx`

Always understand:

```text
4 = read
2 = write
1 = execute
```

### Mistake 2 — Using `chmod 777` to fix everything

Fix the ownership/access model instead.

### Mistake 3 — Forgetting directory `x`

Directory execute permission means traversal/access, not execution of a directory as a program.

### Mistake 4 — Using `chmod -R` carelessly

Recursive changes can damage an application's security model.

### Mistake 5 — Changing ownership during an investigation

It may destroy useful evidence or alter behavior.

### Mistake 6 — Assuming permissions are the only access-control layer

ACLs, SELinux/AppArmor, capabilities, mount options and applications can also affect access.

### Mistake 7 — Treating every SUID file as malicious

Many legitimate system binaries use SUID.

---

# 47. Troubleshooting Permission Denied

Use this order:

### Step 1 — Identify yourself

```bash
whoami
id
```

### Step 2 — Inspect target

```bash
ls -l target
stat target
```

### Step 3 — Inspect path

```bash
namei -l /path/to/target
```

### Step 4 — Check ACLs

```bash
getfacl target
```

### Step 5 — Check special/security controls if relevant

Examples include:

```text
SELinux
AppArmor
Capabilities
Mount options
Application-level authorization
```

### Step 6 — Determine the smallest safe correction

Do not start with:

```bash
chmod 777
```

---

# 48. Cybersecurity Relevance

Permissions are a major part of Linux attack surface and defense.

### Defenders care about:

```text
Weak permissions
      ↓
Unauthorized modification
      ↓
Persistence / tampering
      ↓
Privilege boundary failure
      ↓
Potential compromise
```

Important audit targets include:

- World-writable files
- World-writable directories
- Unexpected SUID files
- Unexpected SGID files
- Writable scripts executed by privileged services
- Sensitive files with excessive permissions
- Unexpected ownership
- Suspicious changes under `/etc`, `/usr/local`, `/opt`, service directories and application paths
- Privileged users able to modify sensitive resources

---

# 49. Permission Audit Commands

Find SUID files:

```bash
find / -type f -perm -4000 -ls 2>/dev/null
```

Find SGID files:

```bash
find / -type f -perm -2000 -ls 2>/dev/null
```

Find world-writable files:

```bash
find / -type f -perm -0002 -ls 2>/dev/null
```

Find world-writable directories:

```bash
find / -type d -perm -0002 -ls 2>/dev/null
```

Inspect a file:

```bash
stat file
getfacl file
```

Inspect a path:

```bash
namei -l /path/to/file
```

> These commands can produce large outputs. Run them deliberately and understand the scope of the filesystem being scanned.

---

# 50. Mini Project — Linux Permission Auditor

Build:

```text
linux-permission-audit.sh
```

The tool should report:

```text
====================================
       LINUX PERMISSION AUDIT
====================================

SUID files:
SGID files:
World-writable files:
World-writable directories:
Current umask:

====================================
```

### Recommended improvements

- Save results to a timestamped report
- Use `tee` so results are visible and recorded
- Exclude known virtual/pseudo filesystems when appropriate
- Clearly label that findings require validation
- Avoid changing permissions automatically
- Add a summary count for each category

### Stretch goal

Add checks for:

```text
UID 0 accounts
Privileged groups
Suspicious writable service directories
Unexpected executable files in writable locations
```

This project can later evolve into the repository's **Linux Security Audit Toolkit**.

---

# 51. Interview Questions

### Beginner

**1. What are the three Linux permission classes?**  
Owner, group and other.

**2. What do `r`, `w` and `x` represent?**  
Read, write and execute for files; for directories they have directory-specific meanings, including listing and traversal.

**3. What does `chmod` do?**  
Changes file or directory permission bits.

**4. What does `chown` do?**  
Changes ownership, and can also change group ownership.

**5. What is `umask`?**  
A mask used by processes when determining default permissions for newly created filesystem objects.

### Intermediate

**6. Convert `rwxr-xr--` to numeric notation.**  
`754`.

**7. What is the difference between file and directory execute permission?**  
For a file it permits execution when applicable; for a directory it permits traversal/access to objects within the directory.

**8. What is SUID?**  
A special permission that can cause an executable to run with the effective user identity of its owner.

**9. What is SGID on a directory?**  
It commonly causes newly created objects to inherit the directory's group ownership.

**10. What is the sticky bit?**  
A directory permission that restricts deletion/renaming of entries by users who do not own those entries, subject to other controls.

**11. Why is `chmod 777` dangerous?**  
It grants read/write/execute permissions to owner, group and other, often creating unnecessary exposure.

### SOC-focused

**12. Why are SUID files important during security investigations?**  
Unexpected or vulnerable SUID executables can create a path across privilege boundaries.

**13. Why is a writable script executed by root significant?**  
A lower-privileged user who can modify a script consumed by a privileged process may be able to influence privileged behavior.

**14. Is a world-writable file automatically malicious?**  
No. Some applications legitimately need shared writable locations. Context and ownership matter.

**15. What would you inspect after finding a suspicious permission?**  
Ownership, ACLs, parent directories, timestamps, package/source, executing process, scheduled/service configuration and relevant logs.

**16. Why should an analyst avoid changing suspicious permissions immediately?**  
Because changing them can alter system behavior and destroy or modify useful evidence.

---

# 52. Quick Revision Cheat Sheet

## Inspect

```bash
ls -l file
stat file
namei -l /path/to/file
getfacl file
```

## Permissions

```text
r = 4
w = 2
x = 1
```

## Change permissions

```bash
chmod 640 file
chmod u+x file
chmod g-w file
chmod o-r file
```

## Ownership

```bash
chown user file
chown user:group file
chgrp group file
```

## Default permissions

```bash
umask
```

## Special bits

```text
SUID   = 4
SGID   = 2
Sticky = 1
```

Examples:

```bash
chmod 4755 program
chmod 2770 directory
chmod 1777 directory
```

## Security audits

```bash
find / -type f -perm -4000 -ls 2>/dev/null
find / -type f -perm -2000 -ls 2>/dev/null
find / -type f -perm -0002 -ls 2>/dev/null
find / -type d -perm -0002 -ls 2>/dev/null
```

---

# 53. Module Checklist

- [ ] Understand owner/group/other
- [ ] Read `ls -l`
- [ ] Understand file type characters
- [ ] Understand file `rwx`
- [ ] Understand directory `rwx`
- [ ] Convert symbolic permissions to numeric
- [ ] Convert numeric permissions to symbolic
- [ ] Use numeric `chmod`
- [ ] Use symbolic `chmod`
- [ ] Understand recursive permission changes
- [ ] Use `chown`
- [ ] Use `chgrp`
- [ ] Use `stat`
- [ ] Use `namei -l`
- [ ] Understand `umask`
- [ ] Understand SUID
- [ ] Understand SGID
- [ ] Understand sticky bit
- [ ] Recognize `s`, `S`, `t`, and `T`
- [ ] Understand why `777` is usually unsafe
- [ ] Find SUID/SGID files
- [ ] Find world-writable files/directories
- [ ] Understand ACLs at a basic level
- [ ] Understand capabilities at a basic level
- [ ] Troubleshoot `Permission denied`
- [ ] Complete the permission labs
- [ ] Complete the privileged-script scenario
- [ ] Build the Linux Permission Auditor

---

# 🧠 Final Takeaway

> **Permissions are not just three letters. They are part of the security boundary between identities, processes and resources.**

The core model is:

```text
WHO?
 │
 ▼
USER / GROUP
 │
 ▼
WHAT?
 │
 ▼
RESOURCE
 │
 ▼
WHICH ACCESS?
 │
 ▼
READ / WRITE / EXECUTE
 │
 ▼
CAN THE PATH BE TRAVERSED?
 │
 ▼
ACL / SECURITY CONTROLS?
 │
 ▼
ACCESS DECISION
```

For SOC work, remember the most important investigative question:

> **Who can modify this resource, and what happens if they do?**

**Next module → `10-Ownership-and-ACLs`**