# 🛡️ 10 — Ownership and ACLs

> **Linux Knowledge Base | Beginner → Advanced Administration → Security → SOC/Blue Team**

Traditional Linux permissions are powerful, but real systems often need more precise access control. Ownership, Access Control Lists (ACLs), default ACLs, masks, and related mechanisms allow administrators to answer a more detailed question:

> **Exactly which identity should be allowed to access this resource, and with which permissions?**

This module builds on Module 09 and takes permissions into practical enterprise and security territory.

---

## 🎯 Learning Objectives

By the end of this module you should be able to:

- Understand ownership in depth
- Distinguish owner, owning group and ACL entries
- Read and modify ACLs with `getfacl` and `setfacl`
- Understand named-user and named-group ACLs
- Understand the ACL mask
- Use default ACLs for directory inheritance
- Troubleshoot unexpected access
- Understand how ACLs interact with traditional permissions
- Audit files for hidden or excessive access
- Understand Linux capabilities at a practical introductory level
- Investigate permission-related security issues without destroying evidence

---

# 1. Ownership Is Part of the Security Model

Every normal filesystem object has an owner and group identity.

```text
                 FILE
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
      Owner                Group
        │                   │
        └─────────┬─────────┘
                  ▼
             Permissions
                  │
                  ▼
             Access decision
```

Example:

```bash
ls -l report.txt
```

Possible output:

```text
-rw-r----- 1 alice security 1200 Sep 14 12:00 report.txt
```

Here:

- Owner = `alice`
- Group = `security`
- Mode = `640`

---

# 2. Change Ownership with `chown`

Change owner:

```bash
sudo chown alice report.txt
```

Change owner and group:

```bash
sudo chown alice:security report.txt
```

Change only group using `chgrp`:

```bash
sudo chgrp security report.txt
```

Verify:

```bash
ls -l report.txt
```

or:

```bash
stat report.txt
```

---

# 3. Recursive Ownership Changes

You can recursively change ownership:

```bash
sudo chown -R alice:security /path/to/directory
```

### ⚠️ Why `-R` is dangerous

A recursive ownership change can affect hundreds or thousands of objects.

Before using it:

```bash
find /path/to/directory -maxdepth 2 -ls
```

Understand what you are about to change.

On production systems, use approved change-management procedures.

---

# 4. Ownership Does Not Mean Access

A file can be owned by you but still have permissions that prevent the operation you want.

Example:

```text
Owner = alice
Mode  = 400
```

Alice can read the file but cannot write to it.

Therefore:

```text
Ownership ≠ Permission

Ownership identifies
WHO

Permissions determine
WHAT
```

---

# 5. Traditional Permissions vs ACLs

Traditional mode bits provide three classes:

```text
OWNER | GROUP | OTHER
```

ACLs can provide more specific entries:

```text
OWNER
NAMED USER
OWNING GROUP
NAMED GROUP
MASK
OTHER
```

Visual comparison:

```text
Traditional
───────────
user  → rwx
group → r-x
other → ---

ACL
───
owner          → rwx
alice          → r--
bob            → rw-
owning group   → r-x
security group → r--
mask           → r-x
other          → ---
```

ACLs are useful when the simple owner/group/other model is not expressive enough.

---

# 6. What Is an ACL?

**ACL = Access Control List**.

It provides additional access rules for individual users and groups.

Example requirement:

> Everyone in the `security` group can read a report, but analyst `alice` needs write access while the rest of the group should remain read-only.

Traditional permissions may not express this cleanly.

An ACL can.

```text
report.txt
   │
   ├── owner: utsav → rw-
   ├── group: security → r--
   ├── user: alice → rw-
   ├── group: auditors → r--
   ├── mask → rw-
   └── other → ---
```

---

# 7. Check Whether ACLs Exist

Use:

```bash
getfacl file.txt
```

Example:

```text
# file: file.txt
# owner: utsav
# group: security
user::rw-
group::r--
other::---
```

This represents the basic access entries.

If named ACL entries exist, they will also appear.

---

# 8. Understanding `getfacl` Output

Consider:

```text
user::rw-
user:alice:r--
group::r--
group:auditors:r-x
mask::r-x
other::---
```

Meaning:

| Entry | Meaning |
|---|---|
| `user::rw-` | File owner permissions |
| `user:alice:r--` | Named user `alice` |
| `group::r--` | Owning group |
| `group:auditors:r-x` | Named group `auditors` |
| `mask::r-x` | Maximum effective permissions for relevant named/group entries |
| `other::---` | Everyone else |

The ACL mask is one of the most important concepts in this module.

---

# 9. The ACL Mask

The mask can be understood as a ceiling on the effective permissions of:

- Named users other than the owner
- Named groups
- The owning group entry

Example:

```text
user:alice:rwx
mask::r-x
```

The named ACL requests `rwx`, but the mask limits the effective permissions to `r-x`.

You may therefore see:

```text
user:alice:rwx        # requested ACL entry
                         │
                         ▼
mask::r-x             # effective ceiling
                         │
                         ▼
user:alice:r-x        # effective access
```

When `getfacl` displays an effective permission comment, pay attention to it.

---

# 10. Create a Named User ACL

Create a lab file:

```bash
mkdir -p ~/linux-labs/acl
cd ~/linux-labs/acl

touch report.txt
chmod 640 report.txt
```

Give `alice` read access:

```bash
setfacl -m u:alice:r-- report.txt
```

Inspect:

```bash
getfacl report.txt
```

You should now see a named-user ACL entry.

---

# 11. Give a Named User Write Access

```bash
setfacl -m u:alice:rw- report.txt
```

Check:

```bash
getfacl report.txt
```

The ACL lets you grant access to `alice` without changing the file's traditional owner.

---

# 12. Remove a Named User ACL

```bash
setfacl -x u:alice report.txt
```

Verify:

```bash
getfacl report.txt
```

The named-user entry should be removed.

---

# 13. Named Group ACL

Create a lab group:

```bash
sudo groupadd aclteam
```

Grant the group read access:

```bash
setfacl -m g:aclteam:r-- report.txt
```

Inspect:

```bash
getfacl report.txt
```

This allows a specific group to have an additional access rule.

---

# 14. Remove a Named Group ACL

```bash
setfacl -x g:aclteam report.txt
```

Verify:

```bash
getfacl report.txt
```

---

# 15. Setting the ACL Mask

Explicitly set the mask:

```bash
setfacl -m m:r-- report.txt
```

Inspect:

```bash
getfacl report.txt
```

Now compare a named entry with the mask.

For example:

```text
user:alice:rwx
mask::r--
```

The effective permission for `alice` is constrained by the mask.

### Important

The ACL mask is **not** the same thing as `other` permissions.

---

# 16. `setfacl -m` Syntax

General form:

```bash
setfacl -m entry file
```

Examples:

```bash
setfacl -m u:alice:r file
setfacl -m u:alice:rw file
setfacl -m g:security:r-x file
setfacl -m m:r-x file
```

Multiple entries can be specified:

```bash
setfacl -m u:alice:rw-,g:auditors:r-- file
```

Always verify afterward with:

```bash
getfacl file
```

---

# 17. Default ACLs

A normal ACL applies to an existing filesystem object.

A **default ACL** on a directory provides inheritance rules for newly created objects inside that directory.

Conceptually:

```text
                  Directory
                default ACL
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       new file   new file   new directory
          │          │          │
          └──────────┼──────────┘
                     ▼
              inherited ACL
```

This is useful for shared project directories.

---

# 18. Create a Default ACL

Create a lab directory:

```bash
mkdir -p ~/linux-labs/acl/shared
```

Grant the `aclteam` group inherited read/write access:

```bash
setfacl -m d:g:aclteam:rwX ~/linux-labs/acl/shared
```

Inspect:

```bash
getfacl ~/linux-labs/acl/shared
```

The `d:` indicates a default ACL entry.

---

# 19. Test Default ACL Inheritance

Create a file:

```bash
touch ~/linux-labs/acl/shared/example.txt
```

Inspect:

```bash
getfacl ~/linux-labs/acl/shared/example.txt
```

The inherited ACL demonstrates how directory defaults influence newly created objects.

### Important

Actual resulting permissions also depend on the creating process and its `umask`.

---

# 20. Remove a Default ACL

Remove a specific default entry:

```bash
setfacl -x d:g:aclteam ~/linux-labs/acl/shared
```

Remove the entire default ACL:

```bash
setfacl -k ~/linux-labs/acl/shared
```

Verify:

```bash
getfacl ~/linux-labs/acl/shared
```

---

# 21. Copying ACLs

ACLs can be backed up and restored.

Save ACL information:

```bash
getfacl -R project/ > project.acl
```

Restore:

```bash
setfacl --restore=project.acl
```

### Important

Treat ACL backup files as configuration/security data. Store them appropriately and verify the target before restoration.

---

# 22. Recursive ACL Changes

You can recursively apply ACLs:

```bash
setfacl -R -m g:security:r-X project/
```

`X` is useful because it adds execute permission only where appropriate, such as directories or files that already have an execute bit.

### ⚠️ Caution

As with `chmod -R`, recursive ACL operations can affect many objects.

Inspect first and verify afterward.

---

# 23. ACL vs `chmod`

`chmod` changes the traditional mode bits and, on ACL-enabled filesystems, can also affect the ACL's relevant entries/mask.

Example:

```bash
chmod 640 report.txt
```

Then:

```bash
getfacl report.txt
```

If extended ACL entries exist, do not assume a later `chmod` will leave every effective permission unchanged.

### Best practice

After significant permission changes:

```bash
ls -l file
getfacl file
```

Verify the actual access model.

---

# 24. ACL and `ls -l` — The `+` Sign

You may see:

```text
-rw-r-----+ 1 alice security 1200 report.txt
```

The `+` after the permission string commonly indicates that extended ACL entries exist.

Compare:

```text
-rw-r-----
```

with:

```text
-rw-r-----+
```

The second should prompt you to inspect:

```bash
getfacl report.txt
```

---

# 25. ACL Troubleshooting Example

Suppose a user says:

> “The file says group permissions are `r--`, but I can still write it.”

Do not assume the user is wrong.

Inspect:

```bash
ls -l file.txt
getfacl file.txt
id username
```

There may be:

```text
user:username:rw-
```

in the ACL.

The `+` on `ls -l` is an important clue.

---

# 26. ACL Troubleshooting — Mask Example

Suppose:

```text
user:alice:rwx
mask::r-x
```

Alice may not actually have write permission because the mask limits effective permissions.

Use:

```bash
getfacl file.txt
```

and read the `effective:` annotations when displayed.

### Key lesson

> **Never diagnose ACL access from only the `ls -l` permission string.**

---

# 27. Ownership + ACL Mental Model

Think of a file as having several layers:

```text
                    FILE
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      Owner        Group        Other
        │            │            │
        └────────────┼────────────┘
                     ▼
               Extended ACLs
                     │
                     ▼
                 ACL Mask
                     │
                     ▼
              Other controls
                     │
                     ▼
             Final access result
```

Other security controls may include:

- SELinux
- AppArmor
- Linux capabilities
- Mount options
- Application-level authorization

---

# 28. Linux Capabilities — Why They Matter

Root historically represents broad privilege, but Linux capabilities can divide some privileged operations into more granular units.

Examples include capabilities related to:

- Network administration
- Binding to privileged ports
- Changing file ownership
- Debugging/inspecting processes

The exact effect depends on the capability and process context.

This allows a program to receive a specific privilege without necessarily running with full UID 0 authority.

---

# 29. Inspect File Capabilities

On systems with the relevant tools:

```bash
getcap /path/to/program
```

Search selected system directories:

```bash
getcap -r /usr/bin /usr/sbin 2>/dev/null
```

A result may look conceptually like:

```text
/path/to/program cap_net_bind_service=ep
```

### Security relevance

Unexpected file capabilities can create privilege-boundary risks and deserve investigation.

Do not remove capabilities simply because you do not recognize them; determine why the software needs them first.

---

# 30. `setcap` — Capability Assignment

Capabilities can be assigned with tools such as:

```bash
setcap
```

Example syntax:

```bash
sudo setcap cap_net_bind_service=+ep ./program
```

This should be performed only in a controlled lab or under approved administration because it changes the program's privilege model.

Remove a file capability in a lab:

```bash
sudo setcap -r ./program
```

Verify:

```bash
getcap ./program
```

---

# 31. Why Capabilities Are Security-Relevant

Compare:

```text
Traditional
───────────
UID 0
  ↓
Broad privilege

Capabilities
────────────
Specific capability
  ↓
More granular privilege
```

Granular privilege can reduce blast radius, but a dangerous capability attached to an unexpected executable may still cross an important security boundary.

---

# 32. Find Files with Extended ACLs

A simple approach is to inspect candidate directories and look for the `+` indicator:

```bash
ls -la /path/to/directory
```

For detailed auditing:

```bash
getfacl /path/to/file
```

For larger environments, use purpose-built audit tooling or carefully designed scripts rather than assuming one command will discover every access-control mechanism.

---

# 33. Security Investigation — Hidden Access

Scenario:

```text
ls -l sensitive.txt
```

shows:

```text
-rw-------+ ... sensitive.txt
```

The `+` is a clue.

Investigate:

```bash
getfacl sensitive.txt
```

Then ask:

- Are there named users?
- Are there named groups?
- What is the ACL mask?
- Are default ACLs involved?
- Is the access authorized?

This is a classic example of why a simple `ls -l` review can miss important access rules.

---

# 34. Security Investigation — Unexpected Group Access

Suppose:

```text
/etc/company/secrets.conf
```

appears to be:

```text
-rw-r-----
```

but an unexpected user can read it.

Investigate:

```bash
getfacl /etc/company/secrets.conf
```

Then:

```bash
id suspicious-user
```

The cause could be:

```text
Named ACL
      OR
Group membership
      OR
Another security control
```

Do not immediately alter the file.

---

# 35. SOC Investigation — Writable Application Directory

A privileged service uses:

```text
/opt/company/app/
```

An analyst notices that an ordinary user can write there.

Inspect:

```bash
namei -l /opt/company/app/
```

```bash
ls -la /opt/company/app/
```

For suspicious files:

```bash
getfacl /opt/company/app/file
```

Then determine:

```text
Who can write?
       ↓
What can they change?
       ↓
What process consumes the changed object?
       ↓
With which identity does that process run?
       ↓
Could the change cross a privilege boundary?
```

This is a defensive way to reason about privilege-escalation risk without performing exploitation.

---

# 36. Evidence Preservation Principle

During security investigation:

```text
Observe
  ↓
Record
  ↓
Collect
  ↓
Analyze
  ↓
Correlate
  ↓
Remediate
```

Avoid casually running commands that modify:

- Ownership
- ACLs
- Permissions
- Timestamps
- Files
- Logs

For example, do not “fix” a suspicious ACL before recording what it was.

---

# 37. Hands-On Lab 1 — Basic ACL

> Use a disposable Linux VM and test accounts. Do not modify sensitive system files.

Create:

```bash
mkdir -p ~/linux-labs/acl/basic
cd ~/linux-labs/acl/basic

touch report.txt
chmod 640 report.txt
```

Inspect:

```bash
getfacl report.txt
```

If a lab user `alice` exists, grant read access:

```bash
setfacl -m u:alice:r-- report.txt
```

Inspect:

```bash
getfacl report.txt
```

Remove:

```bash
setfacl -x u:alice report.txt
```

Verify again.

### Goal

Understand the lifecycle:

```text
Traditional permissions
        ↓
Add named ACL
        ↓
Inspect
        ↓
Remove ACL
        ↓
Verify
```

---

# 38. Hands-On Lab 2 — ACL Mask

Create:

```bash
touch mask-test.txt
chmod 600 mask-test.txt
```

Add an ACL:

```bash
setfacl -m u:alice:rwx mask-test.txt
```

Set a restrictive mask:

```bash
setfacl -m m:r-- mask-test.txt
```

Inspect:

```bash
getfacl mask-test.txt
```

### Questions

1. What permission does the ACL request for `alice`?
2. What is the mask?
3. What is the effective permission?
4. Why can the effective permission be lower than the named ACL entry?

---

# 39. Hands-On Lab 3 — Default ACL

Create:

```bash
mkdir -p ~/linux-labs/acl/project
```

Create a group if needed:

```bash
sudo groupadd aclproject
```

Set a default ACL:

```bash
setfacl -m d:g:aclproject:rwX ~/linux-labs/acl/project
```

Inspect:

```bash
getfacl ~/linux-labs/acl/project
```

Create:

```bash
touch ~/linux-labs/acl/project/newfile.txt
mkdir ~/linux-labs/acl/project/newdir
```

Inspect both:

```bash
getfacl ~/linux-labs/acl/project/newfile.txt
getfacl ~/linux-labs/acl/project/newdir
```

### Goal

Observe ACL inheritance.

---

# 40. Hands-On Lab 4 — Ownership + ACL Troubleshooting

Create:

```bash
mkdir -p ~/linux-labs/acl/troubleshoot
cd ~/linux-labs/acl/troubleshoot
printf 'classified\n' > secret.txt
chmod 640 secret.txt
```

Add a named user ACL in your lab:

```bash
setfacl -m u:alice:r-- secret.txt
```

Now inspect only with:

```bash
ls -l secret.txt
```

Then:

```bash
getfacl secret.txt
```

### Challenge

Explain why `ls -l` alone does not provide the complete access picture.

---

# 41. Hands-On Lab 5 — Capability Audit

On an authorized lab machine:

```bash
getcap -r /usr/bin /usr/sbin 2>/dev/null | tee capability-audit.txt
```

Review:

```bash
less capability-audit.txt
```

For each entry, research the purpose of the capability using local documentation or trusted documentation for the installed software.

### Do not

- Remove capabilities randomly
- Modify system binaries
- Test privilege escalation against systems you do not own or administer

The goal is understanding and auditing.

---

# 42. Scenario Challenge — ACL Hidden Access

## Scenario

A sensitive file appears restricted:

```text
-rw-r-----+ 1 root security ... secrets.conf
```

A user outside the expected security group reports being able to read it.

## Investigation

Start with:

```bash
ls -l secrets.conf
```

Then:

```bash
getfacl secrets.conf
```

Check the user's identity and groups:

```bash
id username
```

Record:

```text
Owner:
Owning group:
Traditional mode:
Named users:
Named groups:
ACL mask:
Other:
```

### Questions

- Is there a named-user ACL?
- Is there a named-group ACL?
- Is the ACL mask limiting access?
- Is the access authorized?
- Was the ACL change documented?

Do not remove the ACL during the investigation unless remediation is explicitly authorized.

---

# 43. Scenario Challenge — Default ACL Misconfiguration

## Scenario

A shared project directory should only be accessible to the `developers` group. New files unexpectedly become readable by another group.

Investigate:

```bash
getfacl /path/to/project
```

Then inspect a newly created file:

```bash
getfacl /path/to/project/example.txt
```

Compare:

```text
Directory default ACL
          ↓
New object ACL
```

Also check:

```bash
umask
```

Determine whether the behavior comes from:

- Default ACL
- Traditional permissions
- Group membership
- `umask`
- Application-specific file creation behavior

---

# 44. Scenario Challenge — Capability Review

A newly installed executable has a file capability that was not present in the previous baseline.

Investigate:

```bash
getcap /path/to/program
```

Inspect:

```bash
ls -l /path/to/program
stat /path/to/program
file /path/to/program
```

Determine:

- What capability is assigned?
- Why does the program need it?
- Is the software trusted?
- Was the change approved?
- Did package installation or an administrator make the change?

Document before remediation.

---

# 45. Common Mistakes

### Mistake 1 — Looking only at `ls -l`

The `+` indicator means you should investigate extended ACLs.

### Mistake 2 — Forgetting the ACL mask

A named ACL entry does not necessarily equal its effective permission.

### Mistake 3 — Assuming default ACLs affect existing files

Default ACLs primarily define inherited access for newly created objects.

### Mistake 4 — Using recursive ACL changes casually

`setfacl -R` can alter a large tree.

### Mistake 5 — Confusing ACLs with ownership

ACL entries provide additional access rules; they do not change who owns the file.

### Mistake 6 — Removing suspicious ACLs before recording them

This can destroy useful evidence.

### Mistake 7 — Treating capabilities as automatically safe

Capabilities are granular privileges, but unexpected assignments can still be dangerous.

---

# 46. Troubleshooting

## `setfacl: command not found`

The ACL utilities may not be installed. Install the appropriate package for your distribution using its normal package-management process.

## `Operation not supported`

Possible causes include filesystem or mount limitations. Check:

```bash
findmnt -T file.txt
```

and verify that the filesystem supports the requested ACL functionality.

## ACL exists but access is denied

Check:

```bash
id username
getfacl file
namei -l /path/to/file
```

Then consider security modules or application-specific authorization.

## ACL seems to grant access but user still cannot use the file

Remember:

```text
ACL permission
      ≠
Complete system authorization
```

SELinux/AppArmor, mount options, directory traversal and application behavior can still matter.

---

# 47. Cybersecurity Relevance

Ownership and ACLs are important because attackers and defenders both care about access boundaries.

### Defensive audit targets

```text
Unexpected owner
Unexpected group
Unexpected ACL
Unexpected default ACL
Unexpected SUID/SGID
Unexpected capability
World-writable resource
Privileged process + writable dependency
```

### Attack-path thinking

```text
Low-privileged identity
        ↓
Can modify resource?
        ↓
Who consumes resource?
        ↓
Does consumer have higher privilege?
        ↓
Security boundary crossed?
```

This reasoning is useful for understanding privilege-escalation risk without performing exploitation.

---

# 48. Linux Access-Control Audit Workflow

Use this workflow when a resource behaves unexpectedly:

```text
1. Identify user
       ↓
2. Identify groups
       ↓
3. Inspect owner/group
       ↓
4. Inspect mode bits
       ↓
5. Inspect ACL
       ↓
6. Inspect parent path
       ↓
7. Inspect capabilities if relevant
       ↓
8. Check security modules if relevant
       ↓
9. Check application/service context
       ↓
10. Correlate logs/change records
       ↓
11. Document
       ↓
12. Remediate with authorization
```

---

# 49. Mini Project — Linux Access Control Auditor

Build:

```text
linux-access-audit.sh
```

The tool should accept a path and report:

```text
====================================
       LINUX ACCESS AUDITOR
====================================

Path:
Owner:
Group:
Mode:
ACL present:
Named users:
Named groups:
ACL mask:
Default ACL:
SUID/SGID:
Capabilities:

====================================
```

Suggested building blocks:

```bash
stat
ls -ld
getfacl
namei
getcap
```

### Recommended behavior

- Validate the path exists
- Never modify the target
- Clearly distinguish observations from conclusions
- Handle directories and files
- Report errors without hiding them
- Save output with `tee` when requested

### Stretch goal

Add a risk summary such as:

```text
[!] World writable
[!] Unexpected named ACL
[!] SUID present
[!] File capability present
[OK] No extended ACL
```

The tool should **flag for review**, not automatically change permissions.

---

# 50. Interview Questions

### Beginner

**1. What is an ACL?**  
An Access Control List provides additional, more specific access rules beyond the basic owner/group/other mode bits.

**2. Which command displays ACLs?**  
`getfacl`.

**3. Which command modifies ACLs?**  
`setfacl`.

**4. What does the `+` in `ls -l` often indicate?**  
That extended ACL entries exist.

**5. What is a default ACL?**  
An ACL attached to a directory that provides inheritance rules for newly created objects inside it.

### Intermediate

**6. What is the ACL mask?**  
A limit on the effective permissions of the owning group and named user/group ACL entries, except the file owner and other entry.

**7. What is the difference between an ACL and ownership?**  
Ownership identifies the owner/group; ACLs provide additional access rules for users and groups.

**8. What does `setfacl -x` do?**  
It removes a specified ACL entry.

**9. What does `setfacl -k` do?**  
It removes the default ACL from a directory.

**10. Why should recursive ACL operations be used carefully?**  
They can change access across many filesystem objects at once.

### Security-focused

**11. Why is an unexpected ACL important during an investigation?**  
It may provide access that is invisible from the basic permission triplets and could represent misconfiguration or unauthorized persistence.

**12. Why is a writable privileged dependency dangerous?**  
A lower-privileged identity may be able to influence something consumed by a higher-privileged process.

**13. What are Linux capabilities?**  
A mechanism for dividing certain privileged operations into more granular privileges that can be assigned to processes/files.

**14. Is an unexpected capability automatically malicious?**  
No. Determine its purpose, software source, authorization and history.

**15. What would you preserve before changing a suspicious ACL?**  
The ACL output, ownership, permissions, timestamps, relevant file metadata, system context and supporting logs/change records.

---

# 51. Quick Revision Cheat Sheet

## Ownership

```bash
ls -l file
stat file
sudo chown user file
sudo chown user:group file
sudo chgrp group file
```

## ACL inspection

```bash
getfacl file
```

## Named user ACL

```bash
setfacl -m u:alice:r-- file
setfacl -x u:alice file
```

## Named group ACL

```bash
setfacl -m g:security:r-- file
setfacl -x g:security file
```

## ACL mask

```bash
setfacl -m m:r-x file
```

## Default ACL

```bash
setfacl -m d:g:security:rwX directory
```

Remove default ACL:

```bash
setfacl -k directory
```

## ACL backup/restore

```bash
getfacl -R directory > backup.acl
setfacl --restore=backup.acl
```

## Capabilities

```bash
getcap file
getcap -r /usr/bin /usr/sbin 2>/dev/null
```

---

# 52. Module Checklist

- [ ] Understand ownership
- [ ] Use `chown`
- [ ] Use `chgrp`
- [ ] Understand why ownership is not the same as access
- [ ] Understand ACLs
- [ ] Use `getfacl`
- [ ] Use `setfacl`
- [ ] Create named-user ACLs
- [ ] Create named-group ACLs
- [ ] Remove ACL entries
- [ ] Understand the ACL mask
- [ ] Understand effective permissions
- [ ] Understand default ACLs
- [ ] Test ACL inheritance
- [ ] Remove default ACLs
- [ ] Back up and restore ACLs
- [ ] Understand the `+` in `ls -l`
- [ ] Troubleshoot ACL access
- [ ] Understand Linux capabilities at a basic level
- [ ] Inspect file capabilities
- [ ] Audit suspicious access-control changes
- [ ] Preserve evidence before remediation
- [ ] Complete the ACL labs
- [ ] Complete the hidden-access scenario
- [ ] Complete the capability-audit lab
- [ ] Build the Linux Access Control Auditor

---

# 🧠 Final Takeaway

> **If `ls -l` does not explain who can access a file, look deeper.**

The practical investigation model is:

```text
WHO?
 ↓
UID / GROUPS
 ↓
WHO OWNS IT?
 ↓
OWNER / GROUP
 ↓
WHAT DO MODE BITS SAY?
 ↓
R/W/X
 ↓
IS THERE AN ACL?
 ↓
NAMED USERS / GROUPS / MASK
 ↓
IS THERE A DEFAULT ACL?
 ↓
IS THERE A CAPABILITY?
 ↓
ARE OTHER SECURITY CONTROLS INVOLVED?
 ↓
FINAL ACCESS DECISION
```

For a Linux SOC analyst, the most important habit is:

> **Do not stop at the permission string. Investigate the complete access-control chain.**

**Next module → `11-Processes-and-Services`**