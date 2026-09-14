# 🐚 19 — Shell Scripting

## 🎯 Goal
Turn Linux commands into repeatable, safe automation.

## Shell vs Bash
A shell interprets commands. **Bash** is one popular shell. Check your environment:
```bash
echo "$SHELL"
ps -p $$ -o comm=
bash --version
```

## Script Anatomy
```bash
#!/usr/bin/env bash
set -u

name="Linux"
echo "Hello $name"
```
The shebang selects the interpreter. Prefer `#!/usr/bin/env bash` for portability where appropriate.

## Variables
```bash
name="Utsav"
echo "$name"
echo "${name}_lab"
readonly VERSION="1.0"
```
Quote variables unless you specifically need word splitting or glob expansion.

## Input and Arguments
```bash
read -r name
echo "First argument: $1"
echo "All arguments: $@"
echo "Count: $#"
```
Use `"$@"`, not `$@`, when forwarding arguments.

## Exit Status
```bash
command
status=$?
if [ "$status" -eq 0 ]; then echo "Success"; fi
```
`0` normally means success; non-zero means failure.

## Conditions
```bash
if [[ -f /etc/passwd ]]; then
  echo "File exists"
elif [[ -d /tmp ]]; then
  echo "Directory exists"
else
  echo "Neither"
fi
```

## Loops
```bash
for file in /var/log/*.log; do
  [[ -e "$file" ]] || continue
  echo "$file"
done

while read -r line; do
  echo "$line"
done < input.txt
```

## Functions
```bash
check_service() {
  local service="$1"
  systemctl is-active --quiet "$service"
}
```

## Command Substitution
```bash
kernel="$(uname -r)"
```
Prefer `$(...)` over legacy backticks.

## Pipelines and Redirection
```bash
command > output.txt
command >> output.txt
command 2> errors.txt
command > output.txt 2>&1
command | grep -i error
```
Understand that `>` overwrites while `>>` appends.

## Safe Script Defaults
For scripts where appropriate:
```bash
set -euo pipefail
```
These improve failure visibility but do not make a script automatically safe. Understand command exceptions, pipelines and cleanup requirements.

Use temporary files carefully, validate input, quote paths and avoid destructive commands unless explicitly required and tested.

## Arrays
```bash
services=(ssh cron)
for service in "${services[@]}"; do
  echo "$service"
done
```

## Text Processing
Combine small tools:
```bash
grep -i failed auth.log | awk '{print $1,$2,$3}' | sort | uniq -c | sort -nr
```
Field layouts are log-specific.

## Debugging
```bash
bash -n script.sh
bash -x script.sh
shellcheck script.sh
```
`bash -x` can expose secrets in command traces, so do not use it carelessly in production logs.

## Security Relevance
Scripts can become persistence mechanisms, accidentally leak secrets, execute untrusted input or make large-scale changes. SOC analysts should inspect:
- unexpected scripts in startup/scheduled locations;
- unusual interpreters and encoded commands;
- scripts launched by services;
- suspicious downloads followed by execution;
- unusual ownership/permissions;
- command history only as supporting evidence, not definitive proof.

## 🧪 Labs
1. **System Info Script:** OS, kernel, uptime, memory, disk and IP information.
2. **User Audit:** list users, UID 0 accounts, shells and home directories.
3. **Service Checker:** accept service names and report active/inactive state.
4. **Log Counter:** count selected authentication events from a supplied lab log.
5. **Argument Validator:** build a script that rejects missing/invalid arguments safely.

## 🛠️ Project — Linux System Auditor
Create a read-only Bash tool with modules for identity, OS, disk, memory, processes, network, listening sockets, services and recent authentication events. Produce a timestamped report.

## 🎤 Interview Questions
- What is a shell?
- Bash vs shell?
- What is `$?`?
- `$@` vs `$*`?
- Why quote variables?
- What does `set -euo pipefail` do?
- What is command substitution?
- How do you debug a Bash script?
- Why is input validation important?
- How can scripts become security risks?

## ⚡ Cheat Sheet
```bash
#!/usr/bin/env bash
set -euo pipefail
name="value"
echo "$name"
read -r value
[[ -f file ]]
[[ -d dir ]]
for x in "${arr[@]}"; do ...; done
$(command)
$?
$1
$#
"$@"
bash -n script.sh
bash -x script.sh
```
