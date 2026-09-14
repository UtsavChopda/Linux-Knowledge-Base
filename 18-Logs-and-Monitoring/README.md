# 📊 18 — Logs and Monitoring

## 🎯 Objectives
- Understand Linux logs, journald, syslog and auditd.
- Search, filter, follow and correlate events.
- Build incident timelines for SOC investigations.
- Understand retention, rotation, time synchronization and centralized logging.

## 1. Log vs Monitoring vs Alerting
A **log** records an event. **Monitoring** observes systems and metrics. **Detection/alerting** applies rules to identify events that deserve attention.

```text
EVENT → LOG → COLLECT → PARSE → CORRELATE → DETECT → ALERT → INVESTIGATE
```

## 2. journald
Many modern distributions use systemd. `systemd-journald` collects structured journal records.

```bash
journalctl
journalctl -b
journalctl -b -1
journalctl -u ssh
journalctl -p warning
journalctl --since "1 hour ago"
journalctl --until "2026-01-01 12:00"
journalctl -k
journalctl -f
journalctl -n 100
journalctl -o short-iso
journalctl _COMM=sshd
journalctl _PID=1234
journalctl SYSLOG_IDENTIFIER=sshd
journalctl --list-boots
```

Service names differ by distribution (`ssh` vs `sshd`). Previous-boot logs may require persistent journal storage.

### Persistent journal
Common locations:
- `/run/log/journal` — volatile
- `/var/log/journal` — persistent when configured

Useful administration commands:
```bash
journalctl --disk-usage
journalctl --vacuum-time=30d
journalctl --vacuum-size=500M
```
Retention changes can destroy evidence, so use them deliberately and according to policy.

## 3. Traditional `/var/log`
Common files vary by distribution:

| Purpose | Debian/Ubuntu examples | RHEL/Fedora examples |
|---|---|---|
| Authentication | `/var/log/auth.log` | `/var/log/secure` |
| General system | `/var/log/syslog` | `/var/log/messages` |
| Audit | `/var/log/audit/audit.log` | `/var/log/audit/audit.log` |

Applications may maintain their own files under `/var/log/` or elsewhere.

## 4. Audit Framework
Linux Audit can record security-relevant events such as selected syscalls, account changes and policy activity. It is **rule/configuration dependent**; not every command is automatically captured in full.

```bash
sudo systemctl status auditd
sudo ausearch -m USER_LOGIN
sudo aureport --auth
sudo aureport --summary
```

Common audit log: `/var/log/audit/audit.log`.

## 5. Log Rotation
`logrotate` prevents logs from growing indefinitely.

```text
current.log → rotate → compress → retain → expire
```

Configuration commonly lives in `/etc/logrotate.conf` and `/etc/logrotate.d/`.

## 6. Time Matters
Incident timelines fail when clocks disagree.

```bash
date
timedatectl
```

Time synchronization may use chrony, systemd-timesyncd or another service depending on the distro. Normalize incident timelines to UTC and record the source timezone.

## 7. Command-Line Log Analysis
```bash
grep -i "failed password" /var/log/auth.log
grep -E "error|failed|denied" application.log
tail -F /var/log/auth.log
journalctl -f
cat file.log | sort | uniq -c | sort -nr
cut -d' ' -f1-4 file.log
tr '[:upper:]' '[:lower:]' < file.log
awk '{print $1,$2,$3}' file.log
```

Log field positions are format-dependent; never assume `$9` means the same thing in every log.

Generate a lab event with:
```bash
logger "LINUX-KB test event"
```

## 8. SSH Investigation
Look for authentication failures, successful sessions, unusual users and source IPs.

```bash
journalctl -u ssh --since "2 hours ago"
grep -i "failed password" /var/log/auth.log
grep -i "accepted" /var/log/auth.log
last
lastlog
who
w
```

A burst of failures is a signal, not proof of compromise. Correlate with successful authentication, account activity, source IP, process activity and network connections.

## 9. SOC Correlation
```text
SSH EVENT
   ↓
USER + SOURCE IP
   ↓
PID / PROCESS
   ↓
COMMAND / EXECUTABLE
   ↓
NETWORK CONNECTIONS
   ↓
SUDO / ACCOUNT CHANGES
   ↓
TIMELINE
```

Important signals include unexpected sudo use, new accounts, changed SSH keys, service restarts, unusual outbound connections and repeated authentication failures.

## 10. Centralized Logging
```text
HOST → AGENT/COLLECTOR → TRANSPORT → SIEM → PARSER → CORRELATION → ALERT
```

Syslog commonly uses facilities and severities such as `emerg`, `alert`, `crit`, `err`, `warning`, `notice`, `info`, `debug`. Structured JSON logs can make parsing more reliable.

Local logs are not inherently trustworthy after root compromise. Remote, access-controlled and integrity-protected logs provide stronger evidence. Log flooding can also exhaust storage or hide important events.

## 🧪 Labs
### Lab 1 — Journal Exploration
1. Run `journalctl --list-boots`.
2. Inspect current boot with `journalctl -b`.
3. Filter warnings with `journalctl -p warning`.
4. Follow events with `journalctl -f`.

### Lab 2 — SSH Investigation
Find failed and successful authentication events. Extract timestamp, username and source address. Explain whether the activity is expected.

### Lab 3 — Timeline Builder
Choose a 15-minute window. Collect authentication, sudo, service and kernel events. Normalize timestamps and build a chronological table.

### Lab 4 — Audit Investigation
If `auditd` is installed, use `ausearch` and `aureport` to inspect authentication/account events. Document which rules produced the evidence.

### Lab 5 — Live Monitoring
Terminal A:
```bash
journalctl -f
```
Terminal B:
```bash
logger "Linux KB monitoring lab"
```
Observe and explain the resulting event.

## 🧩 SOC Scenarios
- Burst of failed SSH logins.
- Unexpected successful login after failures.
- New user created outside maintenance window.
- Service repeatedly crashes and restarts.
- Logs suddenly stop arriving.
- Log disk fills rapidly.
- Host clock drifts.

## 🛠️ Project — Linux Log Analyzer
Build a read-only Python/Bash tool that:
- accepts a log/journal export;
- extracts timestamps, users, IPs and event types;
- counts failed/successful authentication;
- highlights sudo/account/service events;
- creates a chronological timeline;
- outputs findings without modifying evidence.

## 🧯 Troubleshooting
| Problem | Checks |
|---|---|
| No journal | `systemctl status systemd-journald`, permissions |
| Previous boot unavailable | persistent journal configuration |
| Auth log missing | distro, journald, rsyslog configuration |
| Logs too large | rotation and retention policy |
| Events have wrong times | `timedatectl`, synchronization service |
| SIEM missing events | agent, transport, parser and filtering |

## 🎤 Interview Questions
1. journald vs syslog?
2. What does `journalctl -b -1` do?
3. Where are authentication logs stored?
4. What is auditd?
5. Why is time synchronization important?
6. What is log rotation?
7. Why are centralized logs valuable?
8. How would you investigate failed SSH attempts?
9. Why is local log evidence not always trustworthy?
10. What is the difference between logging, monitoring and alerting?

## ⚡ Cheat Sheet
```bash
journalctl -b
journalctl -u ssh
journalctl -p warning
journalctl --since "1 hour ago"
journalctl -f
journalctl -k
aureport --auth
ausearch -m USER_LOGIN
grep -i failed /var/log/auth.log
tail -F /var/log/auth.log
timedatectl
```

## ✅ Checklist
- [ ] Explain journald.
- [ ] Filter journal by unit/time/priority.
- [ ] Locate authentication logs.
- [ ] Use grep/awk/sort safely.
- [ ] Understand auditd and logrotate.
- [ ] Build a timeline.
- [ ] Correlate user, process, network and log evidence.
- [ ] Explain centralized logging and SIEM.
