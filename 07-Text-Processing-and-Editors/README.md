# 📝 07 — Text Processing and Editors

> **Linux Knowledge Base | Beginner → Advanced → SOC/Blue Team**

Text is everywhere in Linux: configuration files, logs, command output, process information, user databases, scripts, and security evidence. Learning to inspect, filter, transform, and safely edit text is one of the most important Linux skills for a system administrator and SOC analyst.

---

## 🎯 Learning Objectives

By the end of this module you should be able to:

- Read text files efficiently from the terminal
- Search large files with `grep`
- Understand basic regular expressions
- Build pipelines using `grep`, `sort`, `uniq`, `cut`, `tr`, `sed`, and `awk`
- Follow live log activity with `tail -f`
- Save terminal output with `tee`
- Understand when to use `xargs`
- Edit files with `nano` and `vim`
- Analyze Linux logs from a defensive/SOC perspective
- Avoid accidentally modifying evidence or configuration files

---

# 1. Why Text Processing Matters

Linux exposes a huge amount of useful information as text.

```text
                 Linux System
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
   Config files      Logs        Command output
       │              │              │
       └──────────────┼──────────────┘
                      ▼
              Text Processing
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
     grep           awk            sed
       │              │              │
       └──────────────┼──────────────┘
                      ▼
             Useful Information
```

A SOC analyst may need to answer questions such as:

- Which users failed SSH authentication?
- Which IP addresses appear repeatedly?
- Which process is listening on a port?
- Which configuration line enables a setting?
- How many times did an event occur?
- What changed between two text files?

You should learn to answer these questions **without opening a graphical application**.

---

# 2. Standard Streams: The Foundation

Linux commands normally work with three standard streams:

| Stream | Number | Purpose |
|---|---:|---|
| stdin | 0 | Input |
| stdout | 1 | Normal output |
| stderr | 2 | Error output |

Visual model:

```text
Keyboard/file ──► stdin ──► COMMAND ──► stdout ──► Terminal
                                  │
                                  └────► stderr ──► Terminal
```

These streams make command pipelines possible.

```bash
cat /etc/passwd | grep bash
```

The output of `cat` becomes the input of `grep`.

---

# 3. `cat` — Display File Content

```bash
cat file.txt
```

Useful options:

```bash
cat -n file.txt
```

Shows line numbers.

```bash
cat -A file.txt
```

Can reveal hidden characters such as tabs and line endings.

### ⚠️ Important

Do not use `cat` as your default tool for very large files. A multi-gigabyte log can flood your terminal.

Use `less` instead.

---

# 4. `less` — Safely Read Large Files

```bash
less /var/log/syslog
```

Common controls:

| Key | Action |
|---|---|
| `Space` | Page down |
| `b` | Page up |
| `/word` | Search |
| `n` | Next match |
| `N` | Previous match |
| `g` | Beginning |
| `G` | End |
| `q` | Quit |

`less` is especially useful during investigations because it lets you inspect large files without editing them.

---

# 5. `head` — Beginning of a File

```bash
head file.txt
```

Default: first 10 lines.

```bash
head -n 20 file.txt
```

First 20 lines.

Example:

```bash
head -n 5 /etc/passwd
```

---

# 6. `tail` — End of a File

```bash
tail file.txt
```

Last 10 lines.

```bash
tail -n 50 /var/log/syslog
```

Last 50 lines.

## `tail -f`

Follow a file as new lines are written:

```bash
tail -f application.log
```

This is extremely useful for live monitoring.

```text
Application
    │
    ▼
application.log
    │
    ▼
tail -f
    │
    ▼
Terminal
    │
    ▼
Analyst observes new events
```

Stop with `Ctrl+C`.

---

# 7. `wc` — Count Lines, Words and Bytes

```bash
wc file.txt
```

Typical output contains:

```text
lines words bytes filename
```

Useful forms:

```bash
wc -l file.txt
wc -w file.txt
wc -c file.txt
```

Example:

```bash
wc -l /etc/passwd
```

This tells you how many lines are present.

---

# 8. `grep` — Search Text

`grep` is one of the most important Linux commands for administration and security work.

Basic syntax:

```bash
grep "pattern" file
```

Example:

```bash
grep "root" /etc/passwd
```

### Common options

```bash
grep -i "error" app.log
```

Case-insensitive search.

```bash
grep -n "error" app.log
```

Show line numbers.

```bash
grep -v "DEBUG" app.log
```

Show lines that do **not** match.

```bash
grep -r "PermitRootLogin" /etc/ssh
```

Recursively search a directory.

```bash
grep -E "error|failed|denied" app.log
```

Use extended regular expressions.

### Combine options

```bash
grep -inE "failed|denied" app.log
```

---

# 9. `grep` with Pipes

You rarely use Linux commands alone.

```bash
ps aux | grep ssh
```

Another example:

```bash
ss -tulpn | grep ':22'
```

This asks: **Does the output contain port 22?**

### A common mistake

```bash
ps aux | grep ssh
```

May also show the `grep ssh` command itself.

A more precise approach can be:

```bash
ps aux | grep '[s]sh'
```

or use a more specific command such as `pgrep` when appropriate.

---

# 10. Basic Regular Expressions

Regular expressions let you describe patterns rather than exact text.

| Pattern | Meaning |
|---|---|
| `.` | Any single character |
| `^` | Beginning of line |
| `$` | End of line |
| `*` | Zero or more of previous item |
| `+` | One or more of previous item with extended regex |
| `?` | Zero or one with extended regex |
| `[abc]` | a, b, or c |
| `[0-9]` | Digit |
| `[^0-9]` | Not a digit |
| `|` | OR with extended regex |

Examples:

```bash
grep '^root' /etc/passwd
```

Lines beginning with `root`.

```bash
grep 'bash$' /etc/passwd
```

Lines ending with `bash`.

```bash
grep -E 'error|failed|denied' app.log
```

Matches any of the three words.

### Regex warning

Regex can become complex quickly. Start with simple patterns and verify your result before using a pattern in automation.

---

# 11. `sort` — Sort Lines

```bash
sort names.txt
```

Reverse:

```bash
sort -r names.txt
```

Numeric sorting:

```bash
sort -n numbers.txt
```

Sort by a field:

```bash
sort -k2 data.txt
```

---

# 12. `uniq` — Find Repeated Lines

`uniq` normally compares **adjacent** lines, so sorting is often used first.

```bash
sort names.txt | uniq
```

Count occurrences:

```bash
sort names.txt | uniq -c
```

Most frequent first:

```bash
sort names.txt | uniq -c | sort -nr
```

### SOC example

If a log-derived list contains one IP address per line:

```bash
sort ips.txt | uniq -c | sort -nr
```

You can quickly identify repeated IPs.

> Frequency alone does not prove malicious activity. Validate the surrounding events and context.

---

# 13. `cut` — Extract Columns

Suppose `/etc/passwd` contains colon-separated fields.

```bash
cut -d: -f1 /etc/passwd
```

Meaning:

- `-d:` → delimiter is `:`
- `-f1` → select field 1

Get usernames and shells:

```bash
cut -d: -f1,7 /etc/passwd
```

Visual:

```text
root:x:0:0:root:/root:/bin/bash
 │                        │
field 1                  field 7
```

---

# 14. `tr` — Translate or Remove Characters

Convert lowercase to uppercase:

```bash
echo "linux" | tr 'a-z' 'A-Z'
```

Replace spaces with underscores:

```bash
echo "linux security" | tr ' ' '_'
```

Delete characters:

```bash
echo "123-456-789" | tr -d '-'
```

`tr` is useful when normalizing command output before further processing.

---

# 15. `tee` — Display and Save Output

```bash
command | tee output.txt
```

The output is both:

```text
                 ┌──► Terminal
COMMAND ─► tee ──┤
                 └──► output.txt
```

Append instead of overwrite:

```bash
command | tee -a output.txt
```

### Investigation use

When collecting authorized diagnostic output, `tee` can let you see the result while preserving a copy for later review.

---

# 16. `xargs` — Build Commands from Input

`xargs` takes input and uses it as arguments to another command.

Simple example:

```bash
printf '%s\n' one two three | xargs echo
```

Conceptually:

```text
Input lines
    │
    ▼
  xargs
    │
    ▼
command arguments
```

A common use is processing results from `find`.

```bash
find . -name '*.log' -print0 | xargs -0 wc -l
```

`-print0` and `-0` safely handle filenames containing spaces or unusual characters.

### Safety rule

Before running a destructive command through `xargs`, inspect the generated list first. Do not blindly pipe untrusted filenames into commands that modify or delete data.

---

# 17. `sed` — Stream Editor

`sed` processes text line by line.

Print selected lines:

```bash
sed -n '1,10p' file.txt
```

Search and replace in output:

```bash
sed 's/old/new/g' file.txt
```

Delete matching lines from output:

```bash
sed '/DEBUG/d' application.log
```

### Important: `sed` vs `sed -i`

This:

```bash
sed 's/old/new/g' file.txt
```

displays transformed output but does not modify the original file.

This:

```bash
sed -i 's/old/new/g' file.txt
```

modifies the file in place.

For configuration changes, make a backup or use version control/change-management procedures first.

---

# 18. `awk` — Powerful Field Processing

`awk` is especially useful when text has columns or structured fields.

Basic structure:

```bash
awk 'pattern { action }' file
```

Print the first field:

```bash
awk '{print $1}' file.txt
```

Print the first and third fields:

```bash
awk '{print $1, $3}' file.txt
```

With `/etc/passwd`:

```bash
awk -F: '{print $1, $7}' /etc/passwd
```

Here `-F:` means the field separator is `:`.

### Example: count lines with a condition

```bash
awk '$3 > 100 {print $0}' data.txt
```

This prints records where field 3 is greater than 100.

### Useful built-in variables

| Variable | Meaning |
|---|---|
| `$0` | Entire current line |
| `$1` | Field 1 |
| `$2` | Field 2 |
| `NF` | Number of fields |
| `NR` | Current record/line number |
| `FS` | Input field separator |

---

# 19. A Practical Text-Processing Pipeline

Linux becomes powerful when small commands are chained together.

Example:

```bash
cat access.log | grep '404' | cut -d' ' -f1 | sort | uniq -c | sort -nr
```

Conceptually:

```text
access.log
   │
   ▼
 grep 404
   │
   ▼
 extract IP
   │
   ▼
  sort
   │
   ▼
 count duplicates
   │
   ▼
 sort by frequency
   │
   ▼
 Most frequent IPs
```

A shorter equivalent may be possible depending on the file format:

```bash
grep '404' access.log | awk '{print $1}' | sort | uniq -c | sort -nr
```

The goal is not to memorize one pipeline. The goal is to understand how each stage transforms the data.

---

# 20. Inspect Before You Transform

A professional workflow is:

```text
Observe
  ↓
Understand format
  ↓
Test small sample
  ↓
Transform
  ↓
Verify result
  ↓
Automate if needed
```

For example, before extracting field 7 from a file, first inspect a few lines and identify its delimiter and structure.

This habit prevents many administrative mistakes.

---

# 21. `nano` — Beginner-Friendly Editor

Open a file:

```bash
nano notes.txt
```

Important shortcuts:

| Shortcut | Action |
|---|---|
| `Ctrl+O` | Save |
| `Ctrl+X` | Exit |
| `Ctrl+W` | Search |
| `Ctrl+K` | Cut line |
| `Ctrl+U` | Paste line |
| `Ctrl+G` | Help |

Nano is excellent when you are learning Linux or making a small controlled edit.

---

# 22. `vim` — Powerful Terminal Editor

Open:

```bash
vim notes.txt
```

Vim is **modal**.

```text
          ┌──────────────┐
          │ Normal Mode  │
          └──────┬───────┘
                 │ i
                 ▼
          ┌──────────────┐
          │ Insert Mode  │
          └──────┬───────┘
                 │ Esc
                 ▼
          ┌──────────────┐
          │ Normal Mode  │
          └──────┬───────┘
                 │ :
                 ▼
          ┌──────────────┐
          │ Command Mode │
          └──────────────┘
```

### Essential Vim commands

Enter editing:

```text
i
```

Return to normal mode:

```text
Esc
```

Save and quit:

```text
:wq
```

Quit without saving:

```text
:q!
```

Save:

```text
:w
```

Delete current line:

```text
dd
```

Copy current line:

```text
yy
```

Paste:

```text
p
```

Search:

```text
/word
```

Next match:

```text
n
```

### Beginner survival rule

If you accidentally enter Vim and do not know what happened:

1. Press `Esc`
2. Type `:q!`
3. Press Enter

---

# 23. Text Processing for Linux Logs

Linux logging differs by distribution and configuration.

Common traditional log locations include:

```text
/var/log/
```

On Debian/Ubuntu systems, authentication events may commonly appear in:

```text
/var/log/auth.log
```

On RHEL/Fedora-family systems, authentication events may commonly appear in:

```text
/var/log/secure
```

Many modern Linux systems also use `systemd-journald`, which can be queried with `journalctl`.

Never assume a log path exists on every distribution.

---

# 24. SOC Example — Find Failed SSH Authentication

On a system where authentication events are stored in `/var/log/auth.log`:

```bash
grep -i 'failed password' /var/log/auth.log
```

Show line numbers:

```bash
grep -in 'failed password' /var/log/auth.log
```

Extract likely source IPs from the common OpenSSH message format:

```bash
grep 'Failed password' /var/log/auth.log | awk '{print $(NF-3)}'
```

> Log formats can differ. Always inspect sample lines and verify which field contains the IP before relying on a parser.

Count repeated values after validation:

```bash
grep 'Failed password' /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr
```

This can help identify repeated authentication failures.

### Defensive interpretation

A high count may indicate:

- Password guessing
- A misconfigured service
- A legitimate user repeatedly entering the wrong password
- A scanner or other automated activity

The command produces **evidence to investigate**, not a final verdict.

---

# 25. SOC Example — Search Multiple Security Indicators

```bash
grep -Ei 'failed|denied|invalid|sudo|authentication' /var/log/auth.log
```

Save results while viewing them:

```bash
grep -Ei 'failed|denied|invalid|sudo|authentication' /var/log/auth.log | tee auth-review.txt
```

Review:

```bash
less auth-review.txt
```

This is a simple example of a repeatable defensive workflow.

---

# 26. Journald + Text Processing

If the system uses systemd-journald:

```bash
journalctl
```

Recent entries:

```bash
journalctl -n 50
```

Follow new entries:

```bash
journalctl -f
```

Search within output:

```bash
journalctl | grep -i 'failed'
```

Or use journalctl's own filtering where appropriate, then pipe the result into text-processing tools.

---

# 27. Safe Investigation Workflow

When analyzing a potentially compromised system:

```text
             Authorized System
                    │
                    ▼
                Observe
                    │
                    ▼
              Preserve data
                    │
                    ▼
            Analyze read-only
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
        Logs              Processes
          │                   │
          └─────────┬─────────┘
                    ▼
                Correlate
                    │
                    ▼
             Form hypothesis
                    │
                    ▼
              Verify evidence
```

Avoid changing configuration, deleting files, clearing logs, or otherwise destroying evidence while you are still investigating unless that action is part of an approved response procedure.

---

# 28. Hands-On Lab 1 — Build a Text Pipeline

## Objective

Practice combining commands through a pipeline.

## Create sample data

```bash
mkdir -p ~/linux-labs/text
cd ~/linux-labs/text

cat > users.txt <<'EOF'
alice
bob
alice
charlie
bob
bob
david
alice
EOF
```

## Tasks

### Task 1 — Count lines

```bash
wc -l users.txt
```

### Task 2 — Sort users

```bash
sort users.txt
```

### Task 3 — Count duplicates

```bash
sort users.txt | uniq -c
```

### Task 4 — Find the most common user

```bash
sort users.txt | uniq -c | sort -nr
```

### Task 5 — Search for `alice`

```bash
grep -n 'alice' users.txt
```

## Expected result

You should discover which username appears most frequently.

## What you learned

- `wc`
- `sort`
- `uniq`
- `grep`
- Pipelines

---

# 29. Hands-On Lab 2 — Log Analysis

## Create a simulated authentication log

```bash
cat > auth-demo.log <<'EOF'
Sep 14 10:01 server sshd[1001]: Failed password for invalid user admin from 10.10.10.20 port 41221 ssh2
Sep 14 10:02 server sshd[1002]: Accepted password for utsav from 10.10.10.5 port 41222 ssh2
Sep 14 10:03 server sshd[1003]: Failed password for root from 10.10.10.20 port 41223 ssh2
Sep 14 10:04 server sshd[1004]: Failed password for invalid user test from 10.10.10.20 port 41224 ssh2
Sep 14 10:05 server sudo[1005]: utsav : TTY=pts/0 ; PWD=/home/utsav ; COMMAND=/usr/bin/id
EOF
```

## Tasks

Find failed logins:

```bash
grep -i 'failed password' auth-demo.log
```

Extract source IPs:

```bash
grep 'Failed password' auth-demo.log | awk '{print $(NF-3)}'
```

Count source IPs:

```bash
grep 'Failed password' auth-demo.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr
```

Find sudo activity:

```bash
grep -i 'sudo' auth-demo.log
```

## Challenge

Modify the pipeline so it shows only the most frequent source IP.

---

# 30. Hands-On Lab 3 — `sed` and `awk`

Create:

```bash
cat > services.txt <<'EOF'
ssh 22 running
http 80 running
https 443 running
dns 53 stopped
EOF
```

Print only service names:

```bash
awk '{print $1}' services.txt
```

Print services that are running:

```bash
awk '$3 == "running" {print $1, $2}' services.txt
```

Replace `running` with `ACTIVE` in displayed output:

```bash
sed 's/running/ACTIVE/g' services.txt
```

Count lines:

```bash
wc -l services.txt
```

---

# 31. Hands-On Lab 4 — Editor Practice

Create a file:

```bash
nano ~/linux-labs/editor-notes.txt
```

Write:

```text
Linux
Networking
SOC
SIEM
Incident Response
```

Save and exit.

Then open it with Vim:

```bash
vim ~/linux-labs/editor-notes.txt
```

Practice:

1. Search for `SOC`
2. Add one line
3. Delete a line
4. Save and quit
5. Reopen the file with `less`

Verification:

```bash
less ~/linux-labs/editor-notes.txt
```

---

# 32. Scenario Challenge — Suspicious SSH Activity

## Scenario

You are investigating an authorized Linux lab server. Users report repeated failed SSH authentication attempts.

Your job is to determine whether the log contains a repeated source address.

## Rules

- Do not delete logs.
- Do not disable SSH.
- Do not block an IP based only on this exercise.
- Do not modify the original log.
- Work from a copy if you need to transform data.

## Tasks

1. Locate authentication logs.
2. Identify failed SSH authentication messages.
3. Extract source IP addresses.
4. Count occurrences.
5. Identify the most frequent source.
6. Inspect the original lines for context.
7. Record your findings.

## Investigation chain

```text
Authentication log
       ↓
grep failed events
       ↓
extract IP
       ↓
sort
       ↓
uniq -c
       ↓
sort -nr
       ↓
inspect original events
       ↓
form hypothesis
```

## Analyst mindset

Do not conclude “attacker” just because an IP appears multiple times. Correlate timing, username, source, successful logins, system ownership, and other available telemetry.

---

# 33. Common Mistakes

### Mistake 1 — Using `cat` for huge logs

Use:

```bash
less large.log
```

### Mistake 2 — Forgetting that `uniq` needs adjacent duplicates

Usually:

```bash
sort file | uniq -c
```

### Mistake 3 — Editing a production file accidentally

Prefer inspection first:

```bash
less file
```

Be careful with:

```bash
sed -i ...
```

### Mistake 4 — Trusting a regex without testing it

Test against a small sample first.

### Mistake 5 — Assuming every log has the same format

Log formats vary by distribution, daemon, version, and configuration.

### Mistake 6 — Parsing fields without checking delimiters

Before using `cut -d` or `awk -F`, inspect the data structure.

### Mistake 7 — Forgetting spaces and special characters in filenames

Use correct quoting and prefer null-delimited workflows when processing filenames programmatically.

---

# 34. Troubleshooting

## `grep` returns nothing

Check:

```bash
cat file.txt
```

Try case-insensitive search:

```bash
grep -i 'word' file.txt
```

Check whether the expected file actually exists:

```bash
ls -l file.txt
```

## `Permission denied`

Check:

```bash
ls -l file.txt
```

For a protected system log, use appropriate authorized privileges rather than changing permissions just to read it.

## `awk` gives unexpected fields

Inspect the exact line format first:

```bash
head file.txt
```

Then verify the delimiter and field numbering.

## `sed` changed nothing

Check the exact pattern and whether it is case-sensitive.

## Vim seems stuck

Press:

```text
Esc
```

Then:

```text
:q!
```

---

# 35. Cybersecurity Relevance

Text processing is a core Linux security skill.

### Blue Team uses

- Authentication-log analysis
- Failed-login detection
- Sudo activity review
- IOC searching
- Configuration auditing
- Malware triage
- Process-output filtering
- Incident-response data collection
- Log normalization
- Quick command-line investigations

### SOC workflow

```text
Raw telemetry
     ↓
Filter noise
     ↓
Extract fields
     ↓
Normalize
     ↓
Count / correlate
     ↓
Investigate
     ↓
Document
```

This same logic later appears in SIEM queries, detection engineering, and log pipelines.

---

# 36. Mini Project — Linux Log Analyzer

Build a Bash tool that accepts a log file and reports:

```text
===== Linux Log Analyzer =====

File:
Total lines:
Error count:
Failed authentication count:
Unique IP count:
Top repeated IPs:

==============================
```

Suggested building blocks:

```bash
wc -l
```

```bash
grep -ci 'error'
```

```bash
grep -ci 'failed'
```

```bash
awk
sort
uniq
```

### Project goals

- Validate that the file exists
- Avoid modifying the source log
- Handle missing arguments
- Produce readable output
- Explain what each metric means
- Test with a simulated log before using real logs

This project will later become part of the **SOC Linux toolkit** section of this repository.

---

# 37. Interview Questions

### Beginner

**1. What is grep?**  
A command-line utility for searching text using patterns.

**2. What does `|` do?**  
It sends stdout from one command to another command's stdin.

**3. What is the difference between `head` and `tail`?**  
`head` reads from the beginning; `tail` reads from the end.

**4. What does `tail -f` do?**  
It follows a file and displays new appended lines.

**5. What is `less` used for?**  
Interactive, efficient viewing of text, especially large files.

### Intermediate

**6. Why use `sort | uniq -c`?**  
To count repeated values after making identical values adjacent.

**7. What is the difference between `cut` and `awk`?**  
`cut` is simple field extraction; `awk` supports patterns, calculations, conditions, and more complex field processing.

**8. What does `sed` do?**  
It is a stream editor used to filter and transform text.

**9. What does `tee` do?**  
It writes input to stdout and one or more files.

**10. What is a regular expression?**  
A pattern language used to match text.

### SOC-focused

**11. How would you find failed SSH logins?**  
Identify the system's authentication log or journal and search for the relevant failed-authentication messages.

**12. How would you find repeated IP addresses?**  
Extract the IP field, then use `sort | uniq -c | sort -nr`.

**13. Why should you avoid modifying logs during investigation?**  
Because modification can destroy evidence, change timestamps/content, or complicate forensic analysis.

**14. Why can an IP with many failed logins not automatically be called malicious?**  
It may represent a legitimate user, monitoring system, misconfiguration, or other benign activity. Context is required.

**15. Why is text processing useful in a SOC if a SIEM exists?**  
Endpoint investigation often starts directly on a host, and command-line tools can rapidly inspect local evidence when centralized telemetry is incomplete or unavailable.

---

# 38. Quick Revision Cheat Sheet

## Reading

```bash
cat file
less file
head file
tail file
tail -f file
```

## Searching

```bash
grep 'word' file
grep -i 'word' file
grep -n 'word' file
grep -r 'word' directory
grep -E 'a|b' file
```

## Counting

```bash
wc -l file
sort file | uniq -c
```

## Fields

```bash
cut -d: -f1 file
awk '{print $1}' file
awk -F: '{print $1}' file
```

## Transforming

```bash
tr 'a-z' 'A-Z'
sed 's/old/new/g' file
```

## Output

```bash
command | tee output.txt
```

## Pipelines

```bash
command1 | command2 | command3
```

## Editors

```bash
nano file
vim file
```

---

# 39. Module Checklist

- [ ] Understand stdin, stdout, stderr
- [ ] Use `cat` appropriately
- [ ] Navigate large files with `less`
- [ ] Use `head` and `tail`
- [ ] Follow live output with `tail -f`
- [ ] Count data with `wc`
- [ ] Search using `grep`
- [ ] Understand basic regex
- [ ] Sort and count values
- [ ] Extract fields using `cut`
- [ ] Transform characters with `tr`
- [ ] Save/display output with `tee`
- [ ] Understand `xargs`
- [ ] Perform safe transformations with `sed`
- [ ] Extract and filter structured text with `awk`
- [ ] Build multi-stage pipelines
- [ ] Edit files with `nano`
- [ ] Understand basic Vim modes
- [ ] Analyze simulated authentication logs
- [ ] Apply a defensive investigation workflow
- [ ] Complete the Linux Log Analyzer mini-project

---

# 🧠 Final Takeaway

> **Linux mastery is not only knowing commands — it is knowing how to turn raw system information into useful evidence.**

The most important pattern from this module is:

```text
READ
 ↓
SEARCH
 ↓
FILTER
 ↓
EXTRACT
 ↓
SORT / COUNT
 ↓
CORRELATE
 ↓
VERIFY
 ↓
ACT
```

For a future SOC analyst, commands such as `grep`, `awk`, `sed`, `sort`, `uniq`, `cut`, `tail`, and `less` become a compact investigation toolkit.

**Next module → `08-Users-and-Groups`**