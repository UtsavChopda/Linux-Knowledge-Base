# Linux Enterprise Administration & Automation Lab — Complete Documentation

## 1. Project Purpose
This project demonstrates practical Linux system administration from initial server build through filesystem administration, identity management, permissions, packages, processes, services, networking, SSH, storage, backup/recovery, Bash, automation, health checks, troubleshooting and defensive hardening.

The project is intentionally separate from the later CEH and SOC/SIEM work. It proves that the administrator can operate and troubleshoot a Linux server before investigating security events.

## 2. Learning Outcomes
By the end of the project, the administrator should be able to:
- Build and baseline an Ubuntu Server VM.
- Navigate and administer the Linux filesystem.
- Manage users, groups and sudo privileges.
- Apply Unix permissions and POSIX ACLs.
- Install, update and remove software safely.
- Inspect processes and resource usage.
- Operate and troubleshoot systemd services.
- Configure and troubleshoot Linux networking.
- Perform secure SSH administration.
- Manage filesystems, mounts and LVM.
- Create and restore verified backups.
- Write reliable Bash administration scripts.
- Schedule recurring jobs.
- Produce system health reports.
- Troubleshoot common Linux failures systematically.
- Apply defensive Linux hardening and verify the result.
- Build a reusable administration toolkit.

## 3. Environment
Recommended baseline:
- Ubuntu Server LTS
- 2 vCPU
- 4 GB RAM
- 40 GB+ disk
- NAT networking initially
- Hostname: linux-admin-lab
- Non-root administrator with sudo
- OpenSSH Server when remote administration is needed
- Snapshot after clean baseline

Baseline commands:
    hostnamectl
    uname -a
    cat /etc/os-release
    whoami
    id
    uptime
    ip addr
    ip route
    df -h
    lsblk
    free -h
    systemctl --failed
    sudo apt update
    sudo apt upgrade

## 4. Module 01 — Linux Fundamentals
Learn kernel vs user space, shell, terminal, commands, PATH, environment variables, exit codes, stdin/stdout/stderr, pipes, redirection and command chaining.

Core commands:
    pwd
    ls -lah
    cd
    man
    type
    command -v
    printenv
    history
    echo
    date

Administration rule: understand what a command will do before using sudo or a destructive operation.

Lab: create a system report, redirect output, pipe data through filters, inspect exit status and document actual output.

## 5. Module 02 — Filesystem
Learn /, /etc, /home, /var, /tmp, /usr, /opt, /proc, /dev and /run; absolute/relative paths; files/directories; metadata; hard/symbolic links.

Core commands:
    mkdir -p
    touch
    cp -a
    mv
    rm -i
    ln
    ln -s
    file
    stat
    find
    du
    df

Lab: build /srv/company with department directories, create test files, inspect metadata and links, then clean up safely.

## 6. Module 03 — Users & Groups
Learn UID/GID, root, service accounts, primary/supplementary groups, password management and sudo.

Core commands:
    getent passwd
    getent group
    id
    useradd
    usermod
    userdel
    groupadd
    passwd
    chage
    sudo
    visudo

Important files: /etc/passwd, /etc/shadow, /etc/group, /etc/sudoers and /etc/sudoers.d/.

Lab: create engineering, operations and finance groups; create test users; apply least privilege; verify membership and sudo rights.

## 7. Module 04 — Permissions & ACL
Learn owner/group/other, r/w/x, numeric modes, directory traversal, chmod, chown, chgrp, umask and POSIX ACLs.

Core commands:
    chmod 640 file
    chmod 750 directory
    chown user:group file
    umask
    getfacl file
    setfacl -m u:user:rw file
    namei -l /path

Never use 777 as a generic fix. Verify effective access as the relevant identity.

Lab: implement a shared department directory with group access and a named-user ACL; test both permitted and denied access.

## 8. Module 05 — Package Management
Learn repositories, package metadata, dependencies, apt and dpkg.

Core commands:
    sudo apt update
    apt search package
    apt show package
    sudo apt install package
    sudo apt remove package
    sudo apt purge package
    sudo apt autoremove
    apt list --installed
    dpkg -l
    dpkg -S /path

Lab: install a harmless utility, verify it, remove it and confirm cleanup.

Security: use trusted repositories, patch regularly and avoid random binaries.

## 9. Module 06 — Processes & Resources
Learn PID/PPID, process states, signals, jobs, foreground/background execution, CPU, memory and load.

Core commands:
    ps aux
    ps -ef
    pstree
    pgrep
    top
    free -h
    uptime
    vmstat
    kill -TERM PID
    kill -KILL PID
    jobs
    fg
    bg
    nice
    renice

Lab: create a controlled process, locate it, inspect its parent, terminate it gracefully and verify. Identify resource-heavy processes without blindly killing services.

## 10. Module 07 — systemd & Services
Learn PID 1, units, services, dependencies, targets, startup state and journal logging.

Core commands:
    systemctl status service
    systemctl start service
    systemctl stop service
    systemctl restart service
    systemctl enable service
    systemctl disable service
    systemctl is-active service
    systemctl is-enabled service
    systemctl --failed
    journalctl -u service
    journalctl -b
    systemctl cat service

Remember: start/stop controls current state; enable/disable controls boot behavior.

Lab: inspect SSH, restart it safely, verify status and inspect its journal. Use harmless services for lifecycle experiments.

## 11. Module 08 — Linux Networking
Connect Linux administration to CCNA concepts: interfaces, IPv4, subnetting, routes, DNS, sockets and application connectivity.

Core commands:
    ip addr
    ip link
    ip route
    ip neigh
    ss -tulpn
    ping -c 4 gateway
    traceroute destination
    dig example.com
    nslookup example.com
    curl -I https://example.com
    nmcli device status
    nmcli connection show

Troubleshooting sequence:
link → IP → route → gateway → DNS → port → application.

Lab: record addresses/routes, identify listening sockets, test gateway, resolve DNS and retrieve an HTTP response.

## 12. Module 09 — SSH
Learn SSH client/server operation, public/private keys, authorized_keys, host keys, scp, sftp and secure daemon administration.

Core commands:
    ssh user@server
    ssh-keygen -t ed25519
    ssh-copy-id user@server
    scp file user@server:/path
    sftp user@server
    ss -lntp
    sudo systemctl status ssh
    sudo journalctl -u ssh

Safe change procedure: keep an existing session open, validate configuration, reload/restart, test a second connection, then close the original session.

Lab: configure key-based access for a test user, verify authentication and inspect logs.

Never commit private keys.

## 13. Module 10 — Storage & LVM
Learn disks, partitions, filesystems, mounts, UUIDs, /etc/fstab and LVM.

Discovery:
    lsblk -f
    blkid
    df -hT
    df -ih
    du -xhd1 /path
    findmnt
    sudo fdisk -l

LVM model:
PV → VG → LV → filesystem → mount point.

Typical disposable-lab sequence:
    pvcreate /dev/DEVICE
    vgcreate vgdata /dev/DEVICE
    lvcreate -L 5G -n lvdata vgdata
    mkfs.ext4 /dev/vgdata/lvdata
    mount /dev/vgdata/lvdata /mnt/data

Never guess a storage device. Verify it before destructive operations.

Lab: attach a second virtual disk, create LVM-backed storage, mount it, make it persistent and verify after reboot.

## 14. Module 11 — Backup & Recovery
Learn backup scope, retention, RPO/RTO, integrity and restore testing.

Core commands:
    tar -czf backup.tar.gz /path
    tar -tzf backup.tar.gz
    tar -xzf backup.tar.gz -C /restore/path
    sha256sum file

Lab: create test data, archive it, checksum it, remove a controlled copy, restore to another directory and compare.

Rule: a backup is not considered validated until restoration has been tested.

## 15. Module 12 — Bash Scripting
Learn shebang, variables, arguments, quoting, conditions, loops, functions, arrays, command substitution, exit codes, traps and debugging.

Recommended safety baseline:
    #!/usr/bin/env bash
    set -Eeuo pipefail

Testing:
    bash -n script.sh
    bash -x script.sh

Project scripts:
- system-info.sh
- user-manager.sh
- disk-monitor.sh
- service-monitor.sh
- backup.sh
- health-check.sh
- linux-admin.sh

Scripts must validate input, fail safely, use meaningful exit codes, avoid secrets and be tested for repeated execution.

## 16. Module 13 — Cron & Automation
Learn cron syntax, crontab, systemd timers, scheduled maintenance, logging and environment differences.

Commands:
    crontab -l
    crontab -e
    systemctl list-timers

Lab: schedule a harmless health report, log its output and verify execution.

Automation rules: absolute paths, predictable environment, controlled permissions, logging, retention and protection against overlapping executions when necessary.

## 17. Module 14 — Health Monitoring
A health check should answer:
- Is the system reachable?
- Is CPU/load abnormal?
- Is memory or swap under pressure?
- Is disk/inode usage safe?
- Are critical services active?
- Are expected ports listening?
- Is networking configured correctly?

Commands:
    uptime
    free -h
    df -hT
    df -ih
    ps aux --sort=-%cpu
    ps aux --sort=-%mem
    systemctl --failed
    ss -tulpn
    ip addr
    ip route

Lab: record baseline, create a controlled abnormal condition, detect it, remediate it and verify recovery.

## 18. Module 15 — Troubleshooting
Use:
Symptom → Scope → Evidence → Hypothesis → Test → Fix → Verify → Document.

Toolkit:
    journalctl -b
    journalctl -u service
    systemctl --failed
    dmesg
    ss -tulpn
    ip addr
    ip route
    getent hosts example.com
    df -h
    df -ih
    free -h
    ps aux
    top
    namei -l /path

Scenarios:
- Network outage
- DNS failure
- SSH failure
- Service failure
- Disk full
- Permission denied
- High CPU
- High memory

Do not make unrelated changes simultaneously; preserve evidence and identify root cause.

## 19. Module 16 — Defensive Hardening
Apply:
- OS/package patching
- account review
- least privilege
- secure permissions
- SSH hardening
- service minimization
- restricted listening ports
- host firewall controls
- protected logs
- secure backups
- secret management

Verification is mandatory: compare before/after service, account, socket, patch and access state.

This project does not cover exploitation, SIEM detection, threat hunting or incident response.

## 20. Practical Lab Standard
Every lab must contain:
1. Objective
2. Environment
3. Preconditions
4. Commands/configuration
5. Test procedure
6. Expected result
7. Actual result
8. Verification
9. Troubleshooting
10. Security considerations
11. Evidence
12. Key takeaways
13. Interview questions
14. Rollback/cleanup

Actual output must come from the user's VM. No fabricated screenshots or results.

## 21. Evidence Standard
Evidence should prove the work, not merely repeat commands. Capture:
- relevant command output
- configuration before/after
- successful and failed controlled tests
- verification after changes
- troubleshooting evidence

Redact passwords, private keys, tokens and other sensitive data.

## 22. Final Toolkit
The final linux-admin.sh should provide a menu:
1. System information
2. User management
3. Disk/storage information
4. Process management
5. Service management
6. Network information
7. Backup and restore
8. System health check
9. Exit

Quality requirements:
- input validation
- help/usage
- safe failure
- meaningful exit codes
- no secrets
- tested on Ubuntu VM
- documented examples
- readable functions
- clear logging where appropriate

## 23. Capstone
Build and administer a small Linux server environment using the knowledge above. The capstone must demonstrate identity, permissions, packages, services, networking, SSH, storage, backup/recovery, automation, health checks, troubleshooting and hardening.

Capstone completion requires actual VM evidence and a final report.

## 24. Interview Preparation
Prepare to explain not only commands but why they are used and how failures are diagnosed.

High-value questions:
- What happens when a Linux command executes?
- What is the role of the kernel?
- How do permissions work?
- Why is directory x permission important?
- How does sudo work?
- How do you troubleshoot a failed systemd service?
- How do you troubleshoot no network?
- How does SSH key authentication work?
- What is LVM?
- How do you validate a backup?
- How do you troubleshoot a full disk?
- How do you write a safe Bash script?
- Cron vs systemd timers?
- What does least privilege mean?
- How do you harden SSH without locking yourself out?

## 25. Project Completion Definition
Documentation phase is complete when:
- all modules are documented;
- labs, evidence requirements and troubleshooting workflows are defined;
- scripts and capstone requirements are documented;
- interview preparation exists;
- security boundary is clear;
- final report structure exists.

Practical phase is complete only after the VM labs, real evidence, tested scripts, capstone and final report are completed.

## 26. Status
DOCUMENTATION PHASE: COMPLETE
PRACTICAL EXECUTION: NOT STARTED
FINAL CAPSTONE: PENDING PRACTICAL EXECUTION
