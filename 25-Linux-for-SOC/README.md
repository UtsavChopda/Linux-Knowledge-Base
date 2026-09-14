# 🖥️ 25 — Linux for SOC

## SOC Analyst Mental Model
```text
ALERT → VALIDATE → TRIAGE → SCOPE → CORRELATE → DOCUMENT → ESCALATE/RESPOND
```

## What to Know Quickly
- users and privilege;
- processes and services;
- network connections;
- authentication events;
- scheduled tasks;
- filesystem changes;
- package changes;
- firewall and SSH configuration.

## High-Value Commands
```bash
id
who
w
ps aux
ss -tulpen
ip route
systemctl --failed
systemctl list-timers --all
journalctl -b
findmnt
lsblk -f
```

## Common SOC Cases
### Brute Force
Correlate failed logins by source IP and username. Look for a successful login after the failure burst.

### Suspicious Process
Map PID → executable → parent → user → network → persistence → logs.

### Privilege Escalation
Review sudo activity, account changes, UID 0 accounts, unexpected services and writable execution paths.

### Persistence
Check cron, systemd timers/services, SSH keys and startup files, then correlate modification times with access events.

## SIEM Thinking
Raw Linux events become useful when normalized into fields such as timestamp, host, user, process, source IP, destination IP, action and result.

## Detection Examples
Conceptual detections:
- many failed SSH logins from one source;
- successful login following abnormal failures;
- new privileged account;
- new SSH authorized key;
- unusual service enabled;
- suspicious outbound connection by an unexpected process.

A detection is an investigation starting point, not proof of malicious activity.

## 🧪 Labs
1. Triage a failed-login alert.
2. Investigate a suspicious process.
3. Investigate an unexpected listening port.
4. Review a new-user scenario.
5. Create a timeline from multiple Linux sources.

## Project — SOC Linux Triage Toolkit
Create a read-only command-line collector plus Markdown/JSON report. Include evidence source, timestamp, observation, risk hypothesis and next investigative step.

## Interview Focus
Explain how you would investigate brute force, suspicious process, unexpected port, privilege escalation and persistence on a Linux host.
