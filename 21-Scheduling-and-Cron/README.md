# ⏰ 21 — Scheduling and Cron

## Why Scheduling Matters
Linux runs maintenance, backups, updates, monitoring and other recurring work automatically.

## Cron
User crontabs:
```bash
crontab -l
crontab -e
```
System schedules may exist under `/etc/crontab`, `/etc/cron.d/`, and periodic directories.

Format:
```text
MIN HOUR DOM MON DOW COMMAND
```
Example:
```text
0 2 * * * /path/to/script.sh
```
Always use absolute paths and test the command manually.

## Cron Environment
Cron has a limited environment. Use explicit `PATH`, absolute command paths where useful, and redirect output deliberately.

## systemd Timers
On systemd systems:
```bash
systemctl list-timers
systemctl list-timers --all
systemctl status name.timer
```
Timers can provide calendar schedules, monotonic delays, service dependencies and journal integration.

## Security Investigation
Scheduled execution is a persistence and administration mechanism. Review:
```bash
crontab -l
sudo ls -la /etc/cron.d /etc/cron.daily /etc/cron.hourly
systemctl list-timers --all
```
Then inspect referenced scripts, ownership, timestamps, permissions, parent service and logs. An unfamiliar scheduled job is a finding to validate, not automatic proof of compromise.

## 🧪 Labs
1. Create a harmless cron job that writes a timestamp to a lab file.
2. Inspect system cron locations.
3. List systemd timers and map a timer to its service.
4. Build a scheduled system inventory report.
5. Investigate a deliberately planted suspicious lab cron entry.

## 🛠️ Project — Scheduled Task Auditor
Build a read-only tool that inventories user crontabs, system cron directories and systemd timers, then flags unusual ownership, writable script paths and recently changed entries.

## Interview Questions
- Cron format?
- User vs system cron?
- Why can cron be a persistence mechanism?
- Cron vs systemd timer?
- Why use absolute paths?
- What is a limited cron environment?
