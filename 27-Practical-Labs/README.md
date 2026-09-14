# 🧪 27 — Practical Labs

These labs turn theory into operational skill. Use disposable VMs and authorized systems.

## Lab Environment
Recommended: one Ubuntu/Debian VM and optionally one RHEL/Fedora VM. Take snapshots before experiments.

## Lab 01 — Linux Recon
Collect:
```bash
hostnamectl
uname -a
uptime
whoami
id
free -h
df -h
lsblk
ip addr
ip route
```
**Deliverable:** one-page host profile.

## Lab 02 — Users & Permissions
Create test users/groups, practice ownership and permissions, then verify using `id`, `ls -l`, `stat` and `getfacl` where available.

## Lab 03 — Processes
Use `ps`, `top`, `/proc`, `pgrep` and `ss`. Map a listening service to its process.

## Lab 04 — Services
Inspect service state, dependencies and journal output. Intentionally stop a harmless lab service and recover it.

## Lab 05 — Storage
Use `lsblk`, `df`, `du`, `findmnt` and a loop-backed lab filesystem. Do not experiment with unknown disks.

## Lab 06 — Networking
Map interfaces, routes, neighbors, DNS and listening ports. Explain each hop in a connectivity test.

## Lab 07 — SSH
Use two VMs or localhost. Configure key authentication in a lab, inspect `known_hosts` and `authorized_keys`, then review authentication logs.

## Lab 08 — Firewall
Create controlled allow/deny rules for a lab service. Test reachability from a separate host and document the result.

## Lab 09 — Logging
Generate harmless `logger` events, filter journal entries and build a timeline.

## Lab 10 — Scheduling
Create a harmless cron job and systemd timer; identify both from the host.

## Lab 11 — Hardening Audit
Audit users, services, SSH, packages, firewall, mounts, scheduled tasks and logging.

## Lab 12 — SOC Investigation
Use a supplied lab dataset or deliberately planted artifacts. Determine what happened, when, which account/process was involved, scope the activity and document evidence.

## Standard Lab Template
1. Objective
2. Environment
3. Starting state
4. Tasks
5. Commands
6. Expected output
7. Verification
8. Troubleshooting
9. Security relevance
10. Deliverable
11. Lessons learned

## Evidence Rule
During investigation labs, observe and collect before changing artifacts. Keep snapshots and copies of evidence where appropriate.
