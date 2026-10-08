# 00 — Lab Environment

## Objective

Create a clean, repeatable Ubuntu Server virtual machine that will become the foundation for every Linux administration lab in this project.

## Lab Design

Windows Host -> VirtualBox / VMware -> Ubuntu Server VM

The VM will be used for Linux administration, Bash scripting, networking, services, storage, backup, and troubleshooting.

## Recommended VM Specification

| Resource | Baseline |
|---|---|
| OS | Ubuntu Server LTS |
| CPU | 2 vCPU |
| RAM | 4 GB |
| Disk | 40 GB+ |
| Network | NAT initially |
| Hostname | linux-admin-lab |

## Initial Build Rules

1. Use a fresh VM for the project.
2. Use a non-root administrative account.
3. Never commit passwords, private keys, tokens, or other secrets.
4. Take a VM snapshot after the clean baseline is verified.
5. Record actual results instead of copying expected output.
6. Keep the lab isolated from sensitive personal or production systems.

## Baseline Commands

Run these after installation:

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

Then update the system:

    sudo apt update
    sudo apt upgrade

## Verification Checklist

- [ ] Ubuntu Server installed
- [ ] Hostname configured
- [ ] Non-root admin user created
- [ ] Network working
- [ ] Internet connectivity verified
- [ ] System updated
- [ ] Storage verified
- [ ] Memory and CPU verified
- [ ] Failed services checked
- [ ] Clean VM snapshot created

## Evidence

Capture screenshots of the VM configuration, successful login, hostname, networking, storage, memory, package update, and clean snapshot.

Do not include passwords, private keys, API tokens, or other secrets in screenshots.

## Status

**Phase 0 — Ready for local VM setup.**
