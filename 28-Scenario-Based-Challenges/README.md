# 🎯 28 — Scenario-Based Challenges

## How to Solve
Never jump to the answer. Build an evidence chain.

```text
SYMPTOM → HYPOTHESES → COMMANDS → EVIDENCE → CORRELATION → CONCLUSION
```

## Challenge 01 — SSH Brute Force
You receive an alert for repeated SSH failures.
**Find:** source IPs, usernames, time window, successful logins and affected account.
**Tools:** `journalctl`, auth logs, `last`, `ss`.

## Challenge 02 — Suspicious Process
A server shows an unexpected high-CPU process.
**Find:** PID, PPID, user, executable, arguments, open files, network connections and service relationship.

## Challenge 03 — Unexpected Listener
A new TCP listener appears.
**Find:** bind address, port, PID/process, package/service ownership and whether exposure is expected.

## Challenge 04 — New Account
A privileged account appears.
**Find:** UID/GID, groups, shell, home, creation evidence and related authentication events.

## Challenge 05 — Persistence
A suspicious job runs periodically.
**Find:** cron/timer source, referenced script, ownership, modification metadata, execution evidence and parent context.

## Challenge 06 — Disk Full
A server reaches 100% usage.
Determine whether blocks, inodes or deleted-open files caused the issue.

## Challenge 07 — Service Failure
A web service repeatedly crashes. Correlate service state, journal, dependencies, resource pressure and recent package/configuration changes.

## Challenge 08 — Firewall Mystery
A service listens locally but cannot be reached remotely. Determine whether the cause is routing, firewall policy, bind address or application behavior.

## Challenge 09 — Log Gap
SIEM ingestion stops. Determine whether the host stopped generating events, the agent stopped, transport failed, parsing broke or filtering changed.

## Challenge 10 — Full Incident Timeline
Combine authentication, sudo, process, network, service and file evidence into one normalized timeline.

## Investigation Report Template
```text
Case ID:
Host:
Time zone:
Alert:
Initial hypothesis:
Evidence:
Timeline:
Scope:
Alternative explanations:
Confidence:
Recommended next steps:
```

## Rule
All challenges should use systems and datasets you are authorized to investigate. Do not destroy or alter evidence just to make a finding easier.
