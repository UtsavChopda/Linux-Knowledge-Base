# 🔎 26 — Linux Security Monitoring

## Monitoring Architecture
```text
HOST → COLLECTORS → NORMALIZE → STORE/SIEM → DETECTION → ALERT → RESPONSE
```

## Monitor These Areas
| Area | Examples |
|---|---|
| Authentication | SSH failures/successes, sudo |
| Identity | new users, groups, UID 0 |
| Processes | unusual executable/parent |
| Network | new listeners/outbound peers |
| Files | sensitive configuration changes |
| Services | new/disabled/enabled services |
| Scheduling | cron/timers |
| Packages | unexpected installs/removals |
| Logs | gaps, flooding, errors |

## Baselines
A baseline describes expected behavior. Compare current state to known-good state rather than declaring every unusual event malicious.

## File Integrity
Conceptually:
```text
KNOWN HASH → CURRENT HASH → COMPARE → INVESTIGATE
```
Hashing helps detect content changes but does not prove who changed a file or whether a change is malicious.

## Process Monitoring
Capture PID, PPID, user, executable, arguments where appropriate and network associations. Process IDs are reused, so timestamps and context matter.

## Network Monitoring
Track listening sockets and established connections, then map connections to processes. Consider DNS, IP reputation and expected application behavior in a broader SOC system.

## Log Health
Monitor whether important logs are arriving. A missing log stream can itself be a security signal, but also has benign causes such as agent failure or configuration changes.

## Alert Quality
Good detections minimize noise while preserving important signals. Include context in alerts so analysts can act quickly.

## 🧪 Labs
1. Establish a clean-host baseline.
2. Change one harmless configuration and detect it.
3. Create a lab process and map its socket.
4. Generate authentication events and build a detection.
5. Simulate a missing log source and troubleshoot collection.

## Project — Linux Security Monitor
Build a periodic read-only collector that compares system state against a baseline and outputs changes with severity and evidence. Store baseline securely and document expected changes.

## Interview Questions
- What is a baseline?
- How do you detect file changes?
- Why can PID alone be insufficient?
- How do you detect log-source failure?
- What makes a good security alert?
