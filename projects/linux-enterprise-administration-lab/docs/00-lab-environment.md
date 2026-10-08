# 00 — Lab Environment

## Goal

Create a repeatable Ubuntu Server virtual machine for every Linux administration lab.

## Planned baseline

- Ubuntu Server LTS
- 2 vCPU
- 4 GB RAM
- 40+ GB virtual disk
- NAT networking initially
- Hostname: `linux-admin-lab`
- Clean snapshot after baseline setup

## Baseline checks

```bash
hostnamectl
uname -a
cat /etc/os-release
ip addr
ip route
df -h
free -h
lsblk
whoami
id
uptime
```

## Status

Not started. Real outputs and evidence will be added after the VM is tested locally.
