# Linux Enterprise Administration & Automation Lab

A hands-on, portfolio-grade Linux administration project built on an Ubuntu Server VM. It turns the Linux Knowledge Base into practical system administration, troubleshooting, Bash scripting, automation, storage, networking, backup, recovery, and hardening skills.

## Project Goal

Build, configure, administer, troubleshoot, automate, back up, and recover a Linux server in a safe virtual lab.

**Boundary:** this project focuses on Linux administration. Detailed offensive testing, SIEM detection, threat hunting, and incident response belong to the later CEH and SOC/SIEM projects.

## A-to-Z Roadmap

```text
00  Lab Environment & Baseline
01  Linux Fundamentals
02  Filesystem & Core Administration
03  Users & Groups
04  Permissions, Ownership & ACLs
05  Package Management
06  Processes & Resource Management
07  systemd & Services
08  Linux Networking
09  SSH & Remote Administration
10  Storage & LVM
11  Backup & Recovery
12  Bash Scripting
13  Cron & Automation
14  System Health Checks
15  Linux Troubleshooting
16  Linux Administration Security / Hardening
17  Final Linux Administration Toolkit
18  Interview Preparation
```

## Repository Structure

```text
linux-enterprise-administration-lab/
+-- README.md
+-- PROJECT-PLAN.md
+-- CHANGELOG.md
+-- lab-template.md
+-- docs/
+-- labs/
+-- scripts/
+-- config/
+-- backups/
+-- evidence/
+-- troubleshooting/
+-- capstone/
+-- interview/
```

## Evidence Standard

Every completed lab records:

1. Objective
2. Environment
3. Commands/configuration
4. Test procedure
5. Expected result
6. Actual result
7. Verification
8. Troubleshooting
9. Evidence
10. Key takeaways
11. Interview questions

## Final Capstone

Build a reusable `linux-admin.sh` menu-driven toolkit covering system information, users, storage, processes, services, networking, backup/restore, and health checks.

## Security Boundary

Security is included where it belongs to responsible Linux administration: least privilege, permissions, SSH hardening, patching, service minimization, firewall basics, backups, and secure configuration. SOC detection and incident response are intentionally separated.

## Current Status

**Project architecture: complete. Lab execution: pending.**

The repository will only mark practical work complete after the corresponding VM lab is actually performed and verified.
