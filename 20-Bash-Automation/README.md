# ⚙️ 20 — Bash Automation

Automation means reducing repeated manual work while keeping verification and safety.

## Automation Pattern
```text
INPUT → VALIDATE → COLLECT → PROCESS → VERIFY → REPORT
```

## Useful Building Blocks
```bash
uname -a
hostnamectl
free -h
df -h
lsblk
ip addr
ip route
ss -tulpen
systemctl --type=service --state=running
```

## Functions and Modular Design
Keep collection functions separate from reporting functions. Prefer small functions that can be tested independently.

## CSV/Report Output
```bash
printf 'metric,value\n'
printf 'kernel,%s\n' "$(uname -r)"
printf 'uptime,%s\n' "$(uptime -p)"
```
Quote and escape data correctly when generating CSV.

## Scheduling Automation
Use cron or systemd timers for recurring jobs. Never schedule a destructive script until it has been tested manually.

## Idempotency
An idempotent script can run repeatedly without causing unintended repeated changes. Example: creating a directory with `mkdir -p` is generally safer than assuming it does not exist.

## Defensive Automation
For security tools, default to **read-only collection**. Separate collection from remediation.

## 🧪 Labs
1. Automated system inventory.
2. Disk threshold reporter.
3. Service health checker.
4. Failed-login summary.
5. Daily security report generator.

## 🛠️ Project — Linux Daily Security Report
Create a script that produces a timestamped report containing host identity, uptime, users, UID 0 accounts, disk/inode usage, running services, listening sockets, recent authentication events and recent kernel warnings.

## Security Principles
- Least privilege.
- Validate all input.
- Quote paths.
- Do not store passwords/tokens in scripts.
- Log script execution appropriately.
- Test in a lab before production.
- Preserve evidence during investigations.
