# 💻 05 — Linux Command Line

> Learn the Linux terminal from absolute zero and gradually become comfortable using the command line for administration, troubleshooting and cybersecurity.

---

## 🎯 Learning Objectives

By the end of this module, you should be able to:

- Explain what a terminal, shell and command are
- Understand Linux command syntax
- Navigate the filesystem confidently
- Read command help and manual pages
- Use command options and arguments
- Use absolute and relative paths
- Create useful command-line workflows
- Use pipes and redirection
- Search for files and text
- Inspect processes and system information
- Use command history and tab completion
- Combine commands safely
- Understand exit status
- Use the command line for basic security investigation

---

# 🧠 1. What Is the Command Line?

The **command line** is an interface where you interact with a computer by typing commands instead of primarily using graphical buttons and menus.

Example:

```bash
ls
```

This asks Linux to list directory contents.

A simple mental model:

```text
👤 You
 │
 │ type command
 ▼
⌨️ Terminal
 │
 ▼
🐚 Shell
 │
 ▼
🐧 Linux / Kernel + Programs
 │
 ▼
💻 System
```

The command line is extremely important in Linux because servers frequently operate without a graphical desktop, and administrators, developers and security professionals often manage them remotely.

---

# 🖥️ 2. Terminal vs Shell vs Command

These three terms are often confused.

### Terminal

The terminal is the interface/application through which you interact with a shell.

Examples:

- GNOME Terminal
- Konsole
- Windows Terminal
- A terminal provided by a cloud or SSH session

### Shell

The shell interprets commands and interacts with programs and the operating system environment.

Common shells include:

```text
bash
zsh
fish
sh
```

### Command

A command is an instruction you give to the shell, often referring to an executable program.

Example:

```bash
ls
```

Mental model:

```text
Terminal
   ↓
Shell
   ↓
Command
   ↓
Program
   ↓
Result
```

---

# 🐚 3. What Is Bash?

**Bash** stands for **Bourne Again SHell**.

It is one of the most widely used Unix/Linux shells and is especially important for shell scripting and administration.

Check your current shell:

```bash
echo $SHELL
```

Check the shell process more directly:

```bash
ps -p $$ -o pid,comm,args
```

`$$` represents the current shell's process ID in Bash and many compatible shells.

---

# 📍 4. Understanding the Prompt

You may see something similar to:

```text
utsav@linux:~$
```

Break it down:

```text
utsav  → username
@      → separator
linux  → hostname
:      → separator
~      → current directory
$      → normal user shell
```

A root shell commonly uses `#` instead of `$`.

```text
utsav@linux:~$    normal user
root@linux:~#     root user
```

> ⚠️ The exact prompt can be customized, so do not rely on its appearance alone to determine the current user.

Verify the user with:

```bash
whoami
```

---

# 🧩 5. Command Anatomy

A command commonly looks like:

```text
command [options] [arguments]
```

Example:

```bash
ls -lah /var/log
```

Breakdown:

```text
ls        → command
-lah      → options
/var/log  → argument
```

Not every command follows exactly the same option style, but this model is useful for learning.

---

# 🏠 6. Your Current Location — pwd

`pwd` means **print working directory**.

```bash
pwd
```

Example:

```text
/home/utsav
```

This answers:

> "Where am I in the filesystem?"

A beginner should develop the habit of checking `pwd` whenever the current location is unclear.

---

# 📂 7. Listing Files — ls

Basic:

```bash
ls
```

Detailed listing:

```bash
ls -l
```

Show hidden files:

```bash
ls -a
```

Detailed + hidden:

```bash
ls -la
```

Human-readable sizes:

```bash
ls -lh
```

Combine options:

```bash
ls -lah
```

List another directory without moving there:

```bash
ls -lah /var/log
```

---

# 🚶 8. Moving Around — cd

Change directory:

```bash
cd /var/log
```

Go to your home directory:

```bash
cd ~
```

You can usually also use:

```bash
cd
```

Go to the parent directory:

```bash
cd ..
```

Go to the previous directory:

```bash
cd -
```

Visualize:

```text
/home/utsav/projects/linux
              ↑
              │
             cd ..
              │
              ▼
/home/utsav/projects
```

---

# 🧭 9. Absolute vs Relative Paths

This is one of the most important Linux fundamentals.

## Absolute path

Starts from `/`.

```bash
/home/utsav/Documents/file.txt
```

It identifies a location from the root of the filesystem.

## Relative path

Starts from your current location.

If you are in:

```text
/home/utsav
```

then:

```bash
Documents/file.txt
```

is a relative path.

Mental model:

```text
/                    ← filesystem root
└── home
    └── utsav
        └── Documents
            └── file.txt
```

---

# ⭐ 10. Special Path Symbols

| Symbol | Meaning |
|---|---|
| `/` | Root directory |
| `.` | Current directory |
| `..` | Parent directory |
| `~` | Current user's home directory |
| `-` | Previous directory for `cd -` |

Examples:

```bash
cd .
cd ..
cd ~
cd /var/log
```

---

# ❓ 11. How Do I Learn a Command?

Do not memorize commands blindly.

Learn how to discover them.

## `--help`

Many commands provide a short help screen:

```bash
ls --help
```

## `man`

Manual pages provide detailed documentation:

```bash
man ls
```

Search inside a man page with `/` followed by a term.

Quit with:

```text
q
```

## `type`

Find out what a command refers to:

```bash
type ls
```

## `command -v`

Locate the executable or command resolution:

```bash
command -v ls
```

This is a powerful habit:

```text
Don't know command
      ↓
--help
      ↓
man
      ↓
type / command -v
      ↓
Experiment safely
```

---

# 🔍 12. Which Command Is Actually Running?

Linux shells can use aliases, functions, builtins and external programs.

Try:

```bash
type cd
type ls
type echo
```

You may discover that not every command is a standalone executable.

This matters when troubleshooting unexpected command behavior.

---

# 🕵️ 13. Command History

View previous commands:

```bash
history
```

Search history interactively in Bash using:

```text
Ctrl + R
```

Example:

```text
(reverse-i-search)`ssh': ssh user@server
```

History is useful for productivity, but remember that command history can contain sensitive information if secrets are typed directly into commands.

> ⚠️ Never treat shell history as a secure password vault.

---

# ⚡ 14. Tab Completion

Press:

```text
TAB
```

to complete commands, paths and other shell-supported values.

Example:

```bash
cd /var/lo<TAB>
```

may complete to:

```bash
cd /var/log
```

Tab completion:

- Saves time
- Reduces typing mistakes
- Helps discover available paths
- Makes command-line work much faster

---

# 🧹 15. Clearing the Terminal

```bash
clear
```

Keyboard shortcut:

```text
Ctrl + L
```

These normally clear the visible terminal display without deleting your command history.

---

# 📝 16. Creating Files and Directories

Create an empty file:

```bash
touch notes.txt
```

Create a directory:

```bash
mkdir lab
```

Create nested directories:

```bash
mkdir -p lab/linux/commands
```

Inspect:

```bash
ls -lah lab
```

> Detailed file operations will be covered in **06 — Files and Directories**. Here we are learning them as command-line building blocks.

---

# 🗑️ 17. Removing Files Carefully

Remove a file:

```bash
rm notes.txt
```

Remove an empty directory:

```bash
rmdir lab
```

Recursive removal exists, but should be treated carefully:

```bash
rm -r directory
```

> ⚠️ `rm` generally does not provide a normal recycle-bin experience. Verify paths before destructive operations. Never blindly paste destructive commands from the internet.

---

# 📖 18. Reading Text Quickly

Display a small text file:

```bash
cat file.txt
```

View one screen/page at a time:

```bash
less file.txt
```

Show the beginning:

```bash
head file.txt
```

Show the end:

```bash
tail file.txt
```

Follow a changing file:

```bash
tail -f application.log
```

This becomes extremely useful later for **log monitoring**.

---

# 🔢 19. Counting and Inspecting Text

Count lines, words and bytes:

```bash
wc file.txt
```

Count only lines:

```bash
wc -l file.txt
```

Search text:

```bash
grep "error" application.log
```

Case-insensitive search:

```bash
grep -i "error" application.log
```

Show line numbers:

```bash
grep -n "error" application.log
```

These commands will become important for Linux administration and SOC investigations.

---

# 🔗 20. Pipes — Connecting Commands

A pipe `|` sends the standard output of one command into the standard input of another command.

Example:

```bash
ls -lah | less
```

Another example:

```bash
ps aux | grep ssh
```

Visual model:

```text
Command A
   │
   │ stdout
   ▼
   |
   │
   ▼
Command B
```

This is one of the ideas that makes the Linux command line extremely powerful.

---

# 📤 21. Output Redirection

## `>` — overwrite

```bash
echo "hello" > file.txt
```

If the file exists, its contents are replaced.

## `>>` — append

```bash
echo "second line" >> file.txt
```

This adds output to the end.

Mental model:

```text
command
  │
  ▼
stdout
  │
  ├── >  file     overwrite
  └── >> file     append
```

---

# 📥 22. Standard Input and Output

Linux programs commonly work with three standard streams:

```text
stdin   → 0 → input
stdout  → 1 → normal output
stderr  → 2 → error output
```

Visual:

```text
             Program
           /    |    \
          /     |     \
       stdin  stdout  stderr
         0       1       2
```

Redirect standard error:

```bash
command 2> errors.txt
```

Redirect both stdout and stderr in Bash:

```bash
command > output.txt 2>&1
```

A common shorthand is:

```bash
command &> output.txt
```

Use redirection carefully because it changes where information goes.

---

# 🔗 23. Command Chaining

Run the next command regardless of the first command's result:

```bash
command1 ; command2
```

Run the second command only if the first succeeds:

```bash
command1 && command2
```

Run the second command only if the first fails:

```bash
command1 || command2
```

Example:

```bash
mkdir test && cd test
```

Mental model:

```text
A succeeds ──→ && ──→ B
A fails    ──→ || ──→ B
```

---

# 🚦 24. Exit Status

Commands return an exit status.

A common convention is:

```text
0       → success
non-zero → failure/error condition
```

Check the previous command's exit status in Bash:

```bash
echo $?
```

Example:

```bash
true
echo $?
```

Then:

```bash
false
echo $?
```

This concept is fundamental to scripting, automation and troubleshooting.

---

# 🔎 25. Finding Files

`find` is one of the most useful Linux tools.

Find a file by name:

```bash
find /tmp -name "test.txt"
```

Find directories:

```bash
find /tmp -type d
```

Find files:

```bash
find /tmp -type f
```

Find files modified recently:

```bash
find /var/log -type f -mtime -1
```

> Be careful when searching large parts of the filesystem because some searches can generate significant system activity.

---

# 🔤 26. Which Program Provides a Command?

Useful tools:

```bash
which ls
command -v ls
type ls
```

On systems with `whereis`:

```bash
whereis ls
```

These help answer:

> "Where is this command coming from?"

That becomes useful when different versions of a program exist.

---

# 🌍 27. Environment Variables

Environment variables provide information to processes and shells.

Show a variable:

```bash
echo $HOME
```

Show your `PATH`:

```bash
echo $PATH
```

View the environment:

```bash
env
```

A simplified `PATH` model:

```text
$PATH
  │
  ├── /usr/local/bin
  ├── /usr/bin
  ├── /bin
  └── ...
```

When you type a command such as `ls`, the shell uses its command-resolution rules, including searching directories in `PATH` for external commands.

Security relevance:

> If an attacker can manipulate command resolution or a privileged process uses an unsafe `PATH`, the wrong executable may be executed. Understanding command resolution is therefore useful in defensive security.

---

# 🔐 28. sudo

`sudo` allows an authorized user to run a command with elevated privileges according to the system's configuration.

Example:

```bash
sudo systemctl status ssh
```

Check your identity:

```bash
whoami
```

Check whether sudo is available:

```bash
sudo -l
```

> `sudo -l` can reveal what commands your account is permitted to run. On systems you do not own or administer, do not use privileged commands without authorization.

We will study privilege management deeply in the user, group and permissions modules.

---

# 🧠 29. Useful Beginner Command Set

Start becoming comfortable with these:

```text
pwd
ls
cd
mkdir
touch
cp
mv
rm
cat
less
head
tail
grep
find
wc
echo
whoami
id
hostname
uname
history
man
```

Do not try to memorize everything in one sitting.

The goal is to understand:

```text
What does the command do?
        ↓
What input does it accept?
        ↓
What output does it produce?
        ↓
How can I combine it with another command?
```

---

# 🖥️ 30. System Information Commands

Basic kernel/system information:

```bash
uname -a
```

Hostname:

```bash
hostname
```

Current user:

```bash
whoami
```

User and group IDs:

```bash
id
```

Logged-in users:

```bash
who
```

More detailed login/session information:

```bash
w
```

These commands are useful during administration and initial incident triage.

---

# 🔥 31. Linux Command Line for SOC Analysts

The command line becomes especially powerful during security investigations.

Imagine a server has suspicious activity.

You may start with:

```bash
whoami
id
hostname
who
w
ps aux
ss -tulpn
```

Then inspect logs:

```bash
journalctl -b
journalctl --since "1 hour ago"
```

Search for suspicious strings:

```bash
grep -Ri "failed" /var/log 2>/dev/null
```

Find recently modified files in an authorized investigation area:

```bash
find /var/tmp -type f -mtime -1 -ls
```

The investigation model becomes:

```text
Host identity
     ↓
Users / sessions
     ↓
Processes
     ↓
Network connections
     ↓
Files
     ↓
Logs
     ↓
Timeline
     ↓
Hypothesis
     ↓
Evidence
```

This is why strong Linux fundamentals are valuable for SOC work.

---

# 🧪 32. Hands-On Lab — Command Line Basics

## 🎯 Objective

Become comfortable navigating and inspecting your Linux environment.

### Tasks

Run these one by one:

```bash
pwd
whoami
id
hostname
uname -a
ls
ls -lah
cd /tmp
pwd
cd ~
pwd
```

Then create a practice area:

```bash
mkdir -p ~/linux-lab/command-line
cd ~/linux-lab/command-line
```

Create files:

```bash
touch one.txt two.txt three.txt
```

Verify:

```bash
ls -lah
```

---

# 🧪 33. Hands-On Lab — Pipes and Redirection

Create test data:

```bash
printf "apple\nbanana\napple\norange\nbanana\n" > fruits.txt
```

Count lines:

```bash
wc -l fruits.txt
```

Search:

```bash
grep "apple" fruits.txt
```

Pipe into another command:

```bash
grep "apple" fruits.txt | wc -l
```

Append data:

```bash
echo "mango" >> fruits.txt
```

Read it:

```bash
cat fruits.txt
```

### Challenge

Without changing the original file, construct a command that counts how many lines contain the word `banana`.

Expected approach:

```text
search → pipe → count
```

---

# 🧪 34. Hands-On Lab — Exit Status

Run:

```bash
true
echo $?
```

Then:

```bash
false
echo $?
```

Now try:

```bash
ls /this/path/does/not/exist
echo $?
```

### Questions

- Which commands returned success?
- Which returned failure?
- Why is exit status useful for scripts?
- How does it connect to `&&` and `||`?

---

# 🚨 35. Troubleshooting Method

When a command does not behave as expected:

```text
1. Read the error
       ↓
2. Check current directory
       ↓
3. Check command syntax
       ↓
4. Check command help/man page
       ↓
5. Check file/path existence
       ↓
6. Check permissions
       ↓
7. Check dependencies/environment
       ↓
8. Check exit status
       ↓
9. Test a smaller command
```

Example:

```bash
cat config.txt
```

If it fails, don't immediately try random commands.

Ask:

```bash
pwd
ls -lah
ls -l config.txt
whoami
id
```

Then diagnose.

---

# ⚠️ 36. Common Beginner Mistakes

### Mistake 1 — Not knowing where you are

Use:

```bash
pwd
```

### Mistake 2 — Confusing `/` and `~`

`/` is the filesystem root.

`~` normally refers to the current user's home directory.

### Mistake 3 — Using `rm` without checking the path

Always verify what you are deleting.

### Mistake 4 — Using `sudo` for everything

Use the minimum privileges necessary.

### Mistake 5 — Copy-pasting commands you don't understand

Especially avoid blindly executing commands involving:

```text
sudo
rm
curl | shell
wget | shell
chmod
chown
```

Understand what a command does before running it.

### Mistake 6 — Ignoring error messages

Error messages are evidence. Read them carefully.

---

# 🎯 37. Scenario Challenge — Suspicious Linux Host

You are given access to a Linux server that may have suspicious activity.

You are authorized to investigate it.

Your first objective is **information gathering only**. Do not modify or delete anything.

### Questions

1. What hostname are you investigating?
2. Which user are you running as?
3. What users are currently logged in?
4. What processes are running?
5. What network sockets are listening?
6. What recent logs are available?
7. What files in the designated temporary investigation directory were recently modified?

### Suggested toolkit

```bash
hostname
whoami
id
who
w
ps aux
ss -tulpn
journalctl --since "1 hour ago"
find /var/tmp -type f -mtime -1 -ls
```

### Rule

**Collect first. Change later.**

This is a foundational defensive-investigation habit.

---

# 🎤 38. Interview Questions

## 🟢 Beginner

1. What is a terminal?
2. What is a shell?
3. What is Bash?
4. What does `pwd` do?
5. What does `ls -la` do?
6. What is the difference between an absolute and relative path?
7. What does `cd ..` do?
8. What is `~`?
9. What is `sudo`?
10. What does `whoami` show?

## 🟡 Intermediate

1. What is the difference between `>` and `>>`?
2. What is a pipe?
3. Explain stdin, stdout and stderr.
4. What is an exit status?
5. What is the difference between `&&`, `||` and `;`?
6. How do you find a file by name?
7. How do you search for text in a file?
8. What is `$PATH`?
9. How do you determine which program a command refers to?

## 🔴 Advanced

1. What happens when you type a command into Bash?
2. How does the shell locate an external command?
3. Why is `$PATH` security-sensitive?
4. Why should investigators avoid modifying evidence unnecessarily?
5. How would you use Linux command-line tools during initial SOC triage?

---

# ⚡ 39. Quick Revision

```text
pwd       → where am I?
ls        → what is here?
cd        → move
mkdir     → create directory
touch     → create/update file timestamp
cat       → print file
less      → view file interactively
head      → beginning of file
tail      → end of file
grep      → search text
find      → search filesystem
wc        → count
whoami    → current user
id        → user/group identity
history   → previous commands
man       → detailed manual
```

Remember:

```text
|   → pipe
>   → overwrite output
>>  → append output
2>  → redirect stderr
&&  → next command if success
||  → next command if failure
;   → run next command regardless
```

---

# 🧠 Mental Model

The Linux command line becomes powerful when you stop thinking of commands as isolated tools.

Think:

```text
Small command
     +
Small command
     +
Pipe
     +
Filter
     +
Redirect
     ↓
Useful workflow
```

Example:

```bash
ps aux | grep ssh
```

One command produces information.

Another command filters it.

Together they answer a more useful question.

---

# ✅ Module Checklist

- [ ] I understand terminal vs shell
- [ ] I understand Bash
- [ ] I can read a command prompt
- [ ] I understand command syntax
- [ ] I can use `pwd`
- [ ] I can use `ls`
- [ ] I can navigate with `cd`
- [ ] I understand absolute and relative paths
- [ ] I can use `man` and `--help`
- [ ] I understand pipes
- [ ] I understand output redirection
- [ ] I understand stdin/stdout/stderr
- [ ] I understand exit status
- [ ] I can use `grep` and `find`
- [ ] I understand `$PATH`
- [ ] I can use basic system-information commands
- [ ] I can perform basic Linux security triage

---

## ➡️ Next Module

**06 — Files and Directories**

We will go much deeper into the Linux filesystem, directory structure, file types, links, inode concepts, creating/copying/moving/deleting files, permissions foundations, hidden files and practical filesystem exercises.