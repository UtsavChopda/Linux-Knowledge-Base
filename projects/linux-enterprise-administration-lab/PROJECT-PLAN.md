# Linux Enterprise Administration & Automation Lab

## Purpose

This project is the hands-on companion to the Linux Knowledge Base. The Knowledge Base explains concepts; this lab demonstrates practical Linux administration, troubleshooting, scripting, automation, and operational discipline.

## Scope

- Linux server setup and baseline configuration
- Filesystem and directory management
- Users and groups
- Permissions and ACLs
- Package management
- Processes and resource management
- systemd services
- Linux networking
- SSH administration
- Storage and LVM
- Backup and recovery
- cron and task scheduling
- Bash scripting
- System health checks
- Final Linux Administration Toolkit

## What this project is NOT

Detailed attack detection, SIEM correlation, threat hunting, and incident response will be handled in the dedicated SOC/SIEM portfolio project.

## Target Architecture

Windows host
|
+-- Ubuntu Server VM
    +-- Administration
    +-- Services
    +-- Networking
    +-- Storage
    +-- Automation
    +-- Backups

## Build Phases

1. Lab environment and baseline
2. Core administration
3. Users, groups, permissions
4. Packages, processes, services
5. Networking and SSH
6. Storage and backups
7. Bash automation
8. Scheduling and health checks
9. Final administration toolkit
10. Evidence, troubleshooting notes, and final documentation

## Evidence Standard

Each module should contain:
- Objective
- Environment
- Commands
- Configuration
- Test procedure
- Expected result
- Actual result
- Troubleshooting
- Screenshot/evidence
- Key takeaways

## Final Deliverable

A reusable linux-admin.sh toolkit with a menu for common administrative tasks, backed by documented scripts and tested against the Ubuntu Server VM.
