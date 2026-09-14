# 🧯 22 — Linux Troubleshooting

## Troubleshooting Mindset
Do not randomly change settings. Use:
```text
SYMPTOM → SCOPE → EVIDENCE → HYPOTHESIS → TEST → FIX → VERIFY → DOCUMENT
```

## First 10 Commands
```bash
whoami
hostname
uptime
uname -a
free -h
df -h
ip addr
ip route
ss -tulpen
journalctl -p warning -b
```

## Service Failure
```bash
systemctl status SERVICE
journalctl -u SERVICE -b
systemctl cat SERVICE
systemctl show SERVICE
```
Check dependencies, ports, configuration, permissions, disk space and recent changes.

## CPU / Memory
```bash
top
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
free -h
vmstat 1 5
```
Load average is not simply CPU percentage; interpret it with CPU count and I/O state.

## Disk Full
```bash
df -h
df -i
du -xhd1 /
sudo lsof +L1
```
A deleted file can still consume space while a process keeps it open.

## Network Failure
```bash
ip addr
ip route
ip route get 8.8.8.8
ip neigh
getent hosts example.com
ss -tulpen
ping -c 4 gateway
curl -v https://example.com
```
Test layer by layer: interface → address → route → DNS → transport → application.

## Boot Problems
Inspect boot target, failed units and previous boot logs:
```bash
systemctl --failed
journalctl -b
journalctl -b -1
```
Recovery may involve console/rescue mode; preserve evidence if the problem may be security-related.

## SSH Failure Matrix
| Symptom | Likely area |
|---|---|
| timeout | route/firewall/network/listener |
| refused | no listener/firewall reject/service |
| permission denied | authentication/authorization |
| host key warning | changed host key, rebuild, or security issue |

## Password Recovery
Password recovery is system- and bootloader-dependent. Use authorized console/recovery procedures. Understand that resetting a password does not by itself explain how the original access occurred. In an incident, preserve evidence before making changes when possible.

## Security Troubleshooting
Correlate:
```text
LOG → USER → PROCESS → FILE → NETWORK → SERVICE → PERSISTENCE
```
Never treat a single alert as proof. Establish baseline, collect evidence and document changes.

## 🧪 Labs
1. Diagnose a failed service in a disposable VM.
2. Find why a lab filesystem is full.
3. Diagnose a broken DNS configuration.
4. Diagnose an unreachable SSH service.
5. Build a troubleshooting decision tree.

## Project — Linux Troubleshooting Assistant
Build a read-only script that gathers health indicators and produces likely problem areas without automatically changing configuration.

## Interview Questions
- How do you troubleshoot a failed service?
- Difference between disk full and inode full?
- How do you troubleshoot DNS?
- What does connection refused indicate?
- How do you investigate high load?
- Why should you avoid random fixes?
