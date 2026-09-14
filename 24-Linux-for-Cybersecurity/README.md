# 🛡️ 24 — Linux for Cybersecurity

Linux is both a common server platform and a powerful security-analysis environment.

## Core Analyst Workflow
```text
IDENTITY → PROCESS → FILE → NETWORK → LOG → PERSISTENCE → TIMELINE
```

## Evidence Collection
Useful read-only commands:
```bash
date
whoami
hostname
uptime
ps aux
ss -tulpen
ip addr
ip route
findmnt
lsblk
systemctl --failed
journalctl -b
```
Record command output and timestamps. Do not modify suspicious files merely to inspect them.

## Processes
Investigate PID, PPID, executable path, user, arguments, open files and network connections. `/proc/<PID>/` provides runtime information.

## Files
Useful indicators include unexpected executables, recently changed files, unusual ownership, writable directories and suspicious startup scripts. File timestamps require careful interpretation: `ctime` is inode metadata change time, not universally creation time.

## Network
Map connections to processes:
```bash
ss -tunap
```
Then inspect the associated PID/process and expected service role.

## Logs
Use `journalctl`, authentication logs and audit logs. Correlate timestamps rather than relying on one source.

## Persistence Areas
Review authorized lab systems for:
- cron/systemd timers;
- enabled services;
- SSH authorized keys;
- shell startup files;
- application/service configuration;
- package changes.

## Malware Triage Concepts
Ask:
1. Is the process expected?
2. What launched it?
3. What user owns it?
4. What executable is running?
5. What files did it access?
6. What network endpoints did it contact?
7. What persistence mechanism exists?
8. What logs support the timeline?

## 🧪 Labs
1. Process triage on a clean VM.
2. Investigate a suspicious listening socket planted in a lab.
3. Analyze authentication events.
4. Find a deliberately planted persistence artifact.
5. Build an incident timeline.

## Project — Linux Incident Triage Toolkit
Create a read-only collector that gathers host identity, processes, sockets, routes, users, services, timers, SSH keys metadata, mounts and recent logs into a timestamped evidence directory.

## Ethics
Use investigation techniques only on systems you own or are explicitly authorized to analyze. Preserve evidence and follow incident-response procedures.
