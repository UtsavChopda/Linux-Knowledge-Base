# Linux Enterprise Administration & Automation Lab

A hands-on Linux administration project built on an Ubuntu Server VM. This is the practical companion to the Linux Knowledge Base.

## Goal

Build, configure, administer, troubleshoot, automate, back up, and recover a Linux server in a safe virtual lab.

This project is Linux-administration focused. Detailed SIEM detection, threat hunting, and incident response will live in the dedicated SOC/SIEM projects.

## Architecture

```text
Windows Host
    |
    +-- Ubuntu Server VM
         +-- Users & Groups
         +-- Files & Permissions
         +-- Packages
         +-- Processes
         +-- systemd Services
         +-- Networking
         +-- SSH
         +-- Storage / LVM
         +-- Backups
         +-- Bash Automation
```

## Roadmap

- [ ] 00 — Lab environment and baseline
- [ ] 01 — Linux fundamentals and filesystem
- [ ] 02 — Users and groups
- [ ] 03 — Permissions and ACLs
- [ ] 04 — Package management
- [ ] 05 — Processes and resource management
- [ ] 06 — systemd and services
- [ ] 07 — Networking
- [ ] 08 — SSH administration
- [ ] 09 — Storage and LVM
- [ ] 10 — Backup and recovery
- [ ] 11 — Bash scripting
- [ ] 12 — cron and automation
- [ ] 13 — System health checks
- [ ] 14 — Troubleshooting scenarios
- [ ] 15 — Final Linux Administration Toolkit

## Structure

```text
linux-enterprise-administration-lab/
+-- README.md
+-- PROJECT-PLAN.md
+-- CHANGELOG.md
+-- docs/
+-- labs/
+-- scripts/
+-- config/
+-- backups/
+-- evidence/
+-- troubleshooting/
```

## Evidence standard

Each completed lab records the objective, environment, commands/configuration, tests, expected and actual results, verification, troubleshooting, screenshots, and key takeaways.

## Final deliverable

A reusable `linux-admin.sh` toolkit with a menu for common administrative tasks, backed by tested Bash scripts and documented procedures.

## Security boundary

The project includes security practices that belong to good Linux administration: least privilege, SSH hardening, permissions, patching, backups, firewall basics, and secure service configuration. Offensive attack simulation and SOC investigation are intentionally kept for later projects.

See [PROJECT-PLAN.md](./PROJECT-PLAN.md) for the complete plan.
