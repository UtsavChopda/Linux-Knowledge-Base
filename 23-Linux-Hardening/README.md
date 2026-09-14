# 🔐 23 — Linux Hardening

## Hardening Goal
Reduce attack surface, strengthen authentication, minimize privilege and improve visibility.

```text
INVENTORY → REMOVE UNNEEDED → RESTRICT → PATCH → MONITOR → VERIFY
```

## Account Hardening
```bash
getent passwd
getent group
sudo -l
```
Review unnecessary accounts, shells, privileged users and sudo permissions. Use least privilege.

## SSH Hardening
Review `sshd_config`: root login, password authentication, allowed users/groups, authentication attempts, forwarding and listening addresses. Validate with:
```bash
sshd -t
sshd -T
```
Keep an existing session open and test a second session before applying remote SSH changes.

## File Permissions
Find dangerous writable locations and unexpected ownership. Avoid blanket permission changes without understanding application requirements.

## Services
```bash
systemctl --type=service --state=running
systemctl list-unit-files --state=enabled
```
Disable only services confirmed unnecessary.

## Firewall
Use the platform's firewall tooling and review both IPv4 and IPv6. A listening service and an externally reachable service are different questions.

## Updates
Patch through trusted repositories and understand vendor backports/CVEs. Verify packages and repository configuration.

## Mount Security
For suitable data mounts consider controls such as `nosuid`, `nodev` and, where appropriate, `noexec`. These are defense layers, not universal security boundaries.

## Logging
Enable useful authentication, audit and system logging; forward important logs centrally where appropriate. Protect log access and retention.

## MAC Controls
SELinux and AppArmor can enforce mandatory access-control policies. Learn their concepts before changing policy.

## Kernel / System Security
Keep the kernel and critical components patched. Review exposed interfaces and unnecessary kernel modules according to organizational baseline.

## Baseline
Record expected users, services, ports, packages, scheduled tasks, SSH keys and firewall rules. A baseline makes anomaly detection possible.

## 🧪 Labs
1. Create a hardening checklist for a VM.
2. Audit SSH configuration.
3. Inventory running/enabled services.
4. Review privileged accounts and sudo rules.
5. Compare a before/after security baseline.

## Project — Linux Hardening Auditor
Produce a read-only report covering accounts, SSH, services, packages, firewall, mounts, permissions, scheduled tasks and logging. Include risk, evidence and recommended remediation.

## Interview Questions
- What is attack surface?
- Least privilege?
- Authentication vs authorization?
- Why is changing SSH port insufficient?
- SELinux/AppArmor?
- Why baseline a server?
- Why hardening must be followed by monitoring?
