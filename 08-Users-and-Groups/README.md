# 👤 08 — Users and Groups

> **Linux Knowledge Base | Beginner → Administration → Security → SOC/Blue Team**

Linux is a multi-user operating system. Understanding who can access a system, which groups they belong to, what identity a process runs as, and how privileges are obtained is fundamental to Linux administration and cybersecurity.

---

## 🎯 Learning Objectives

By the end of this module you should be able to:

- Explain Linux users, groups, UIDs and GIDs
- Understand `/etc/passwd`, `/etc/shadow` and `/etc/group`
- Create, modify, lock and remove user accounts
- Understand primary and supplementary groups
- Use `id`, `who`, `w`, `last` and related commands
- Understand `su` and `sudo`
- Recognize service/system accounts
- Audit account configuration from a defensive perspective
- Investigate suspicious users and privilege changes
- Build a practical user-audit workflow

---

# 1. Why Linux Has Users

Linux is designed to support multiple users and processes safely.

```text
                     Linux System
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
      Alice              Bob             Service
       UID 1000         UID 1001          UID 998
        │                 │                 │
        ▼                 ▼                 ▼
      Files            Files            Processes
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ▼
                    Kernel enforces
                 identity + permissions
```

A user's identity affects:

- Which files they can access
- Which processes they can control
- Which commands they can run through `sudo`
- Which groups and resources they can use
- Which files they own

This is why account management is also a security topic.

---

# 2. User vs Group

### User

A **user account** represents an identity on the system.

Example:

```text
utsav
```

### Group

A **group** is a collection of users used to manage access and permissions.

Example:

```text
developers
security
sudo
```

Instead of granting access individually to 50 users, an administrator can grant access to a group.

```text
                 /shared/security
                         │
                         ▼
                  group: security
                   /      |      \
                  /       |       \
              Alice      Bob      Carol
```

---

# 3. UID and GID

Linux represents users and groups internally using numeric identifiers.

- **UID** = User ID
- **GID** = Group ID

Check your identity:

```bash
id
```

Example:

```text
uid=1000(utsav) gid=1000(utsav) groups=1000(utsav),27(sudo)
```

Conceptually:

```text
Username ──► UID
Group    ──► GID
```

The kernel ultimately works with these numeric identities when enforcing permissions.

---

# 4. Common UID Ranges

Exact ranges vary by distribution and configuration, so do not treat these numbers as universal rules.

Typical Linux systems contain:

```text
root                  UID 0
System/service users  lower-numbered UIDs
Regular users         commonly start around 1000
```

The important security fact is:

> **UID 0 represents root-level identity.**

Do not assume every non-root UID is a human user. Service accounts can have non-zero UIDs too.

---

# 5. The Root User

`root` is the traditional superuser account.

Check:

```bash
id root
```

Typical result includes:

```text
uid=0(root)
```

Root can perform operations that ordinary users cannot, subject to mechanisms such as Linux capabilities, security modules, filesystem restrictions, and application behavior.

### Security principle

> Use the least privilege necessary.

Avoid performing everyday work as root when it is not required.

---

# 6. `/etc/passwd`

One of the most important account databases is:

```text
/etc/passwd
```

View it:

```bash
cat /etc/passwd
```

A typical entry looks like:

```text
utsav:x:1000:1000:Utsav:/home/utsav:/bin/bash
```

The fields are separated by `:`.

```text
username
   ↓
password placeholder
   ↓
UID
   ↓
GID
   ↓
GECOS/comment
   ↓
home directory
   ↓
login shell
```

Field layout:

```text
username:password:UID:GID:GECOS:home:shell
```

### Important

The `x` commonly indicates that the password hash is stored in `/etc/shadow`, not directly in `/etc/passwd`.

Do not interpret `/etc/passwd` as a file containing plaintext passwords.

---

# 7. `/etc/shadow`

Password-related account aging information is commonly stored in:

```text
/etc/shadow
```

It is normally readable only by privileged users or processes with appropriate access.

Inspect permissions:

```bash
ls -l /etc/shadow
```

You may see a restricted mode such as:

```text
-rw-r-----
```

The exact owner/group and permissions can vary by distribution.

### Why it matters for security

Unexpected permissions on `/etc/shadow` can expose password hashes and increase risk.

As an analyst, you should inspect permissions rather than changing them blindly.

---

# 8. `/etc/group`

Group definitions are commonly stored in:

```text
/etc/group
```

View:

```bash
cat /etc/group
```

Typical format:

```text
group_name:password:GID:members
```

Example:

```text
security:x:1002:alice,bob
```

This means the group has GID `1002` and lists supplementary members.

---

# 9. `/etc/gshadow`

Some Linux systems also use:

```text
/etc/gshadow
```

It contains protected group-related information.

Inspect safely:

```bash
ls -l /etc/gshadow
```

As with `/etc/shadow`, do not expose or alter sensitive account databases unnecessarily.

---

# 10. Useful Account Commands

## `whoami`

Shows the current effective username:

```bash
whoami
```

## `id`

Shows UID, GID and groups:

```bash
id
```

For another account:

```bash
id alice
```

## `groups`

```bash
groups
```

or:

```bash
groups alice
```

## `getent`

`getent` queries configured system databases and can work with sources beyond local files depending on system configuration.

```bash
getent passwd alice
getent group security
```

This is often better than assuming all account information comes directly from `/etc/passwd` or `/etc/group`.

---

# 11. Current Login Sessions

## `who`

```bash
who
```

Shows logged-in sessions.

## `w`

```bash
w
```

Provides login/session information plus current activity and system load.

## `last`

```bash
last
```

Displays login history from the system's login accounting database where available.

### Security relevance

These commands can help answer:

- Who is currently logged in?
- When did a user log in?
- From where did a session originate?
- Is a session unexpected?

Treat local login records as one source of evidence and correlate with authentication logs/journald when investigating.

---

# 12. Creating a User

On systems using the standard Linux account tools:

```bash
sudo useradd alice
```

Create a home directory as well:

```bash
sudo useradd -m alice
```

Specify a login shell:

```bash
sudo useradd -m -s /bin/bash alice
```

Set a password using the appropriate administrative procedure:

```bash
sudo passwd alice
```

Verify:

```bash
id alice
```

Check the home directory:

```bash
ls -ld /home/alice
```

---

# 13. `useradd` vs `adduser`

You may encounter both:

```bash
useradd
```

and:

```bash
adduser
```

`useradd` is a lower-level account utility commonly available across Linux systems.

`adduser` may be a friendlier distribution-provided interface, especially on Debian-family systems.

Do not assume their behavior or options are identical on every distribution.

---

# 14. Modifying Users with `usermod`

Add a user to a supplementary group:

```bash
sudo usermod -aG security alice
```

### Critical option: `-a`

Use:

```bash
-aG
```

rather than simply:

```bash
-G
```

because `-G` can replace the existing supplementary group list rather than append to it.

Verify:

```bash
id alice
```

### Change shell

```bash
sudo usermod -s /bin/bash alice
```

### Change home directory

```bash
sudo usermod -d /home/newalice alice
```

Changing account properties can have consequences. Understand the current state before modifying production accounts.

---

# 15. Locking and Unlocking Accounts

An account can be locked through standard account-management tools.

Lock:

```bash
sudo usermod -L alice
```

Unlock:

```bash
sudo usermod -U alice
```

Another commonly used command is:

```bash
sudo passwd -l alice
```

and:

```bash
sudo passwd -u alice
```

### Important

Account locking behavior depends on the authentication stack and account configuration. Verify the actual result rather than assuming one command completely prevents every possible form of access.

---

# 16. Expiring an Account

Account expiration can be configured with tools such as `chage`.

Inspect account aging:

```bash
sudo chage -l alice
```

This can show password-aging and account-expiration information.

This is particularly useful for checking whether temporary accounts are still valid.

---

# 17. Removing a User

```bash
sudo userdel alice
```

Remove the account and its home directory:

```bash
sudo userdel -r alice
```

### ⚠️ Caution

Deleting an account can remove its home directory and affect files, ownership, services, scheduled jobs, and evidence.

In an investigation, **do not delete a suspicious account simply because it looks suspicious**. Preserve and investigate first according to your incident-response procedure.

---

# 18. Primary vs Supplementary Groups

A user normally has a primary group and may belong to many supplementary groups.

Example:

```bash
id alice
```

Conceptually:

```text
alice
 │
 ├── Primary group: alice
 │
 ├── Supplementary: developers
 │
 ├── Supplementary: docker
 │
 └── Supplementary: security
```

Groups influence access to files and resources.

---

# 19. Creating Groups

Create a group:

```bash
sudo groupadd developers
```

Verify:

```bash
getent group developers
```

Add a user:

```bash
sudo usermod -aG developers alice
```

Verify:

```bash
id alice
```

---

# 20. Removing a User from a Group

On systems providing `gpasswd`:

```bash
sudo gpasswd -d alice developers
```

Verify:

```bash
id alice
```

A user may need to start a new login session before group membership changes are reflected in all existing processes.

---

# 21. `su` — Switch User

Switch to another account:

```bash
su - alice
```

The `-` requests a login-style environment.

Switch to root where permitted:

```bash
su -
```

Return to the previous shell:

```bash
exit
```

### Security perspective

`su` changes the user identity of the new shell, but how authentication and authorization work depends on the system configuration.

---

# 22. `sudo` — Controlled Privilege Elevation

Instead of logging in as root for routine administration, users can be granted permission to execute specific commands with elevated privileges.

Example:

```bash
sudo systemctl status ssh
```

Check sudo privileges:

```bash
sudo -l
```

### Visual model

```text
Normal user
    │
    │ sudo command
    ▼
Authorization policy
    │
    ├── allowed ──► elevated command
    │
    └── denied ───► error
```

`sudo` is an important security boundary and should be configured according to least privilege.

---

# 23. `/etc/sudoers` and Sudo Configuration

The main sudo policy is commonly associated with:

```text
/etc/sudoers
```

Additional configuration may be stored under:

```text
/etc/sudoers.d/
```

Inspect safely:

```bash
sudo -l
```

If you need to edit sudoers, use:

```bash
sudo visudo
```

`visudo` validates syntax before installing the edited policy, helping prevent accidental syntax errors that could break administrative access.

### Security warning

Do not casually grant:

```text
ALL=(ALL) ALL
```

to users. Broad administrative privileges dramatically increase the impact of account compromise.

---

# 24. Dangerous Sudo Configurations

From a defensive perspective, look for:

- Unexpected users with sudo access
- Unexpected groups with administrative privileges
- Commands allowed without authentication where not justified
- Wildcard permissions that permit unintended execution
- Custom files under `/etc/sudoers.d/` that appeared unexpectedly

Example inspection:

```bash
sudo -l -U alice
```

Use this only where you have administrative authorization.

---

# 25. Service and System Accounts

Not every account represents a human.

Examples may include accounts associated with services such as:

```text
www-data
nginx
postgres
sshd
```

Names vary by distribution and installed software.

These accounts often exist so services do not need to run as root.

```text
Service
   │
   ▼
Dedicated account
   │
   ▼
Limited permissions
   │
   ▼
Reduced blast radius
```

This is a security design principle called **least privilege**.

---

# 26. Login Shells and Non-Login Accounts

Look at the shell field:

```bash
getent passwd
```

Common shells include:

```text
/bin/bash
/bin/sh
/bin/zsh
```

Some service accounts may have a shell such as:

```text
/usr/sbin/nologin
```

or:

```text
/bin/false
```

These can be used to prevent normal interactive login through the shell.

### Security note

A suspicious shell is a useful investigation clue, not proof of compromise. Check the account's purpose and related configuration.

---

# 27. Finding Accounts with Login Shells

A simple inspection approach:

```bash
awk -F: '$7 !~ /(nologin|false)$/ {print $1, $7}' /etc/passwd
```

This can produce accounts whose shell does not end in `nologin` or `false`.

Remember:

- `/etc/passwd` may not contain every identity source
- Shell availability does not alone prove interactive access is possible
- Authentication configuration matters

---

# 28. Find UID 0 Accounts

Root has UID 0.

Find all accounts with UID 0:

```bash
awk -F: '$3 == 0 {print $1}' /etc/passwd
```

### Security significance

Normally you expect the primary administrative identity to be `root`, but additional UID 0 accounts may be legitimate or may deserve investigation.

Never delete an unexpected UID 0 account during an investigation without understanding its origin and purpose.

---

# 29. Find Users with a Specific Group

Using `getent`:

```bash
getent group sudo
```

On systems where the administrative group is named `wheel`:

```bash
getent group wheel
```

The group name is distribution-dependent.

### Security investigation

Unexpected membership in a privileged group can be an important indicator of unauthorized privilege changes.

---

# 30. Account Enumeration Workflow

A useful defensive workflow:

```text
List accounts
     ↓
Check UID/GID
     ↓
Check shells
     ↓
Check groups
     ↓
Check sudo privileges
     ↓
Check password/account aging
     ↓
Check login history
     ↓
Check recent changes
     ↓
Correlate with logs/processes
```

This is much stronger than simply looking for a strange username.

---

# 31. SOC Investigation — Suspicious Account

Suppose an authorized lab server contains a new account named `backup-admin`.

Do not immediately delete it.

Investigate:

```bash
id backup-admin
```

```bash
getent passwd backup-admin
```

```bash
getent group
```

```bash
sudo -l -U backup-admin
```

```bash
sudo chage -l backup-admin
```

Then investigate authentication records using the appropriate log source or journal.

Questions:

- When was the account created?
- Who created it?
- Is it documented?
- Does it have a valid business purpose?
- Is it in privileged groups?
- Does it have an interactive shell?
- Has it logged in?
- From where?
- What commands or processes are associated with it?

---

# 32. Account Creation Does Not Equal Compromise

A newly created account can be:

- A legitimate employee
- A deployment account
- A temporary administrator
- A service account
- A test account
- An attacker-created persistence mechanism

The correct approach is:

```text
Observation ≠ Conclusion

Observation
    ↓
Evidence
    ↓
Context
    ↓
Correlation
    ↓
Hypothesis
    ↓
Verification
    ↓
Conclusion
```

This mindset is essential for SOC work.

---

# 33. Hands-On Lab 1 — Create and Audit a User

> Perform this lab inside a disposable Linux VM where you have administrative authorization.

## Create user

```bash
sudo useradd -m labuser
sudo passwd labuser
```

## Verify

```bash
id labuser
```

```bash
getent passwd labuser
```

```bash
ls -ld /home/labuser
```

## Inspect aging

```bash
sudo chage -l labuser
```

## Challenge

Answer:

1. What is the user's UID?
2. What is the primary GID?
3. What is the home directory?
4. What is the login shell?
5. Is the account expired?

---

# 34. Hands-On Lab 2 — Groups and Privilege

Create:

```bash
sudo groupadd labsecurity
```

Add the user:

```bash
sudo usermod -aG labsecurity labuser
```

Verify:

```bash
id labuser
```

```bash
getent group labsecurity
```

Create a test directory:

```bash
sudo mkdir -p /opt/labsecurity
sudo chown root:labsecurity /opt/labsecurity
sudo chmod 770 /opt/labsecurity
```

Test group-based access using a new login session for `labuser`.

### What to observe

```text
User
 ↓
Supplementary group
 ↓
Directory group ownership
 ↓
Group permission bits
 ↓
Access decision
```

---

# 35. Hands-On Lab 3 — Account Audit

Run:

```bash
getent passwd
```

Find UID 0:

```bash
awk -F: '$3 == 0 {print $1}' /etc/passwd
```

Find accounts with interactive-looking shells:

```bash
awk -F: '$7 !~ /(nologin|false)$/ {print $1, $7}' /etc/passwd
```

Inspect privileged groups:

```bash
getent group sudo 2>/dev/null
getent group wheel 2>/dev/null
```

Inspect current sessions:

```bash
who
w
```

Inspect recent login history:

```bash
last | head -n 20
```

### Challenge

Create a report with:

```text
Total accounts:
UID 0 accounts:
Interactive-shell accounts:
Privileged-group members:
Currently logged-in users:
Recent login observations:
```

---

# 36. Hands-On Lab 4 — Defensive Suspicious Account Scenario

Create a simulated account:

```bash
sudo useradd -m -s /bin/bash lab-admin
```

Add it to a test privileged group only if your lab distribution provides one and you understand the consequences. A safer alternative is to create a custom test group such as `lab-admins`.

Then investigate:

```bash
id lab-admin
getent passwd lab-admin
sudo chage -l lab-admin
```

Ask:

- Is the shell interactive?
- Does the account have a home directory?
- What groups is it in?
- Is it expired or locked?
- Does it appear in login records?

Finally, remove the test account **after documenting your findings**:

```bash
sudo userdel -r lab-admin
```

Do not apply the deletion step to a real investigation account.

---

# 37. Mini Project — Linux User Audit Tool

Build:

```text
linux-user-audit.sh
```

The script should report:

```text
================================
       LINUX USER AUDIT
================================

Current user:
UID 0 accounts:
Total local passwd entries:
Interactive shell accounts:
Privileged group information:
Current sessions:
Recent login history:

================================
```

Suggested commands:

```bash
whoami
id
getent passwd
awk
getent group
who
last
```

### Recommended features

- Check that required commands exist
- Print section headers
- Handle missing groups such as `wheel` or `sudo`
- Do not modify accounts
- Do not expose password hashes
- Save an optional report with `tee`
- Clearly label the data source

This becomes the foundation for a later **Linux Security Audit Toolkit**.

---

# 38. Security Monitoring Ideas

For a SOC or Blue Team, account-related events worth monitoring include:

### Account creation

```text
New local user
```

### Privilege changes

```text
User added to privileged group
```

### Authentication

```text
Successful login
Failed login
Unexpected source
```

### Sudo

```text
Unexpected privileged command
```

### Account state

```text
Account unlocked
Password changed
Shell changed
Expiration changed
```

### Persistence indicators

```text
Unexpected administrative account
Unexpected UID 0 account
Unexpected SSH-accessible user
```

These events should be correlated with authorization records and change-management information.

---

# 39. Common Mistakes

### Mistake 1 — Confusing username with UID

Linux permissions ultimately involve numeric identities, not just names.

### Mistake 2 — Using `usermod -G` when you meant to append

Prefer:

```bash
usermod -aG group user
```

when adding supplementary membership.

### Mistake 3 — Assuming every UID above a certain number is a human

Service accounts can use non-zero UIDs too.

### Mistake 4 — Assuming `/etc/passwd` contains passwords

Modern systems commonly store password hashes in `/etc/shadow`.

### Mistake 5 — Deleting suspicious accounts immediately

This can destroy evidence and disrupt legitimate services.

### Mistake 6 — Assuming `nologin` means an account is completely unusable

Authentication paths and service-specific behavior vary. Investigate the actual configuration.

### Mistake 7 — Giving broad sudo access

Use least privilege and narrowly scoped administrative permissions.

---

# 40. Troubleshooting

## User creation fails

Check:

```bash
id labuser
```

The username may already exist.

Check:

```bash
getent passwd labuser
```

## User cannot access a group-controlled directory

Check:

```bash
id labuser
```

Then inspect the directory:

```bash
ls -ld /path/to/directory
```

A new group membership may require a new login session before existing processes receive the updated group list.

## `sudo` says the user is not allowed

Inspect:

```bash
sudo -l -U username
```

Also inspect the relevant sudo policy using authorized administrative access.

## `last` does not show what you expected

Login accounting availability and data sources vary by distribution and configuration. Correlate with journald/authentication logs.

## Account appears in `/etc/passwd` but cannot log in

Check:

```bash
getent passwd username
```

and:

```bash
sudo chage -l username
```

Then inspect the configured authentication service and shell.

---

# 41. Cybersecurity Relevance

Users and groups are central to **Identity and Access Management (IAM)** on Linux.

### Attackers may try to:

- Create persistence accounts
- Add users to privileged groups
- Obtain root privileges
- Modify sudo rules
- Change account shells
- Abuse service accounts
- Reuse stolen credentials

### Defenders should monitor:

```text
Accounts
  ↓
Group membership
  ↓
Privileges
  ↓
Authentication
  ↓
Processes
  ↓
Logs
```

A SOC analyst should always ask:

> **Who performed this action, under which identity, and how did that identity obtain the privilege?**

---

# 42. Interview Questions

### Beginner

**1. What is a UID?**  
A numeric identifier assigned to a user identity.

**2. What is a GID?**  
A numeric identifier assigned to a group.

**3. What is root?**  
The traditional Linux superuser identity represented by UID 0.

**4. Where are local user definitions commonly stored?**  
`/etc/passwd`.

**5. Where are password-related hashes commonly stored?**  
`/etc/shadow`.

**6. Where are local group definitions commonly stored?**  
`/etc/group`.

### Intermediate

**7. What is the difference between a primary and supplementary group?**  
A user has a primary group identity and can additionally belong to multiple supplementary groups.

**8. What does `id` show?**  
UID, primary GID and group memberships for an identity.

**9. Why use `usermod -aG`?**  
To append a supplementary group without replacing the existing supplementary group list.

**10. What is `sudo`?**  
A mechanism for controlled execution of commands with elevated privileges according to policy.

**11. Why use `visudo`?**  
It safely edits sudo policy while validating syntax.

**12. What is a service account?**  
An account used by software/services rather than a normal human interactive user.

### SOC-focused

**13. Why is a new UID 0 account suspicious?**  
Because UID 0 provides root-level identity and an unexpected additional UID 0 account may indicate unauthorized privilege or persistence.

**14. Is a new user automatically malicious?**  
No. It must be correlated with authorization, change records, login activity and system context.

**15. What would you investigate after finding an unexpected privileged user?**  
Account creation details, groups, sudo policy, shell, authentication history, processes, files, persistence mechanisms and relevant logs.

**16. Why shouldn't you immediately delete a suspicious account?**  
It can destroy evidence and disrupt the system before the account's purpose and activity are understood.

**17. Why are service accounts useful for security?**  
They allow services to run with narrower privileges instead of using root unnecessarily.

---

# 43. Quick Revision Cheat Sheet

## Identity

```bash
whoami
id
id username
groups
```

## Account databases

```bash
cat /etc/passwd
cat /etc/group
ls -l /etc/shadow
```

Prefer:

```bash
getent passwd username
getent group groupname
```

## User management

```bash
sudo useradd -m username
sudo passwd username
sudo usermod -aG group username
sudo usermod -L username
sudo usermod -U username
sudo userdel username
```

## Groups

```bash
sudo groupadd groupname
getent group groupname
sudo gpasswd -d username groupname
```

## Account aging

```bash
sudo chage -l username
```

## Sessions

```bash
who
w
last
```

## Privilege

```bash
sudo -l
sudo -l -U username
su - username
```

## Security checks

```bash
awk -F: '$3 == 0 {print $1}' /etc/passwd
```

```bash
awk -F: '$7 !~ /(nologin|false)$/ {print $1, $7}' /etc/passwd
```

---

# 44. Module Checklist

- [ ] Understand Linux users
- [ ] Understand groups
- [ ] Understand UID and GID
- [ ] Understand root/UID 0
- [ ] Read `/etc/passwd` safely
- [ ] Understand `/etc/shadow`
- [ ] Understand `/etc/group`
- [ ] Use `id`
- [ ] Use `who`, `w` and `last`
- [ ] Create a lab user
- [ ] Modify user properties
- [ ] Add supplementary groups correctly
- [ ] Create and inspect groups
- [ ] Lock/unlock a test account
- [ ] Understand account aging
- [ ] Understand `su`
- [ ] Understand `sudo`
- [ ] Use `visudo` conceptually and safely
- [ ] Identify service accounts
- [ ] Identify UID 0 accounts
- [ ] Identify interactive-looking shells
- [ ] Perform an account audit
- [ ] Investigate a suspicious account without destroying evidence
- [ ] Build the Linux User Audit Tool

---

# 🧠 Final Takeaway

> **Every Linux action happens in a security context: an identity, a group membership, a privilege level, and a process.**

For administration:

```text
USER → GROUP → PERMISSION → RESOURCE
```

For SOC investigation:

```text
IDENTITY
   ↓
AUTHENTICATION
   ↓
PRIVILEGE
   ↓
ACTION
   ↓
LOG
   ↓
INVESTIGATION
```

Once you understand users and groups, the next major security layer becomes much easier:

**Next module → `09-File-Permissions`**