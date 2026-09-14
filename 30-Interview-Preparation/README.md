# 🎤 30 — Linux Interview Preparation

## Fundamentals
1. What is Linux?
2. Kernel vs shell?
3. Process vs thread?
4. What is an inode?
5. Absolute vs relative path?
6. Hard link vs symbolic link?
7. Permission bits and ownership?

## Administration
8. How do you find disk usage?
9. How do you identify a high-CPU process?
10. How do you troubleshoot a failed service?
11. What is systemd?
12. Cron vs systemd timer?
13. How do packages and repositories work?

## Networking
14. What is a default gateway?
15. How do you inspect routes?
16. TCP vs UDP?
17. What does `0.0.0.0:22` mean?
18. How do you map a port to a process?
19. How do you troubleshoot DNS?

## Security
20. How do you harden SSH?
21. Why is root login risky?
22. What is least privilege?
23. Firewall vs listening service?
24. What are SELinux/AppArmor?
25. How can cron be persistence?

## SOC Questions
26. How would you investigate failed SSH logins?
27. How would you investigate a suspicious process?
28. How would you investigate an unknown listening port?
29. How would you investigate a new privileged account?
30. How do you build a Linux incident timeline?
31. Why can local logs be unreliable after compromise?
32. What evidence would you collect first?

## Scenario Answer Framework
```text
1. Clarify alert/symptom
2. Define time window and host
3. Establish baseline
4. Collect read-only evidence
5. Correlate identity/process/network/logs
6. Test alternative explanations
7. Scope impact
8. State confidence
9. Recommend response
```

## Command Round
Be able to explain, not merely memorize:
```bash
ps aux
ss -tulpen
ip addr
ip route
lsblk
df -h
df -i
findmnt
systemctl status ssh
journalctl -u ssh
id
sudo -l
find
stat
getent passwd
```

## Behavioral Interview
Explain projects using **Problem → Approach → Technical choices → Result → Lesson**. Be honest about what you built yourself and what remains experimental.

## Final Goal
You should be able to take an unfamiliar Linux server and explain its users, processes, services, storage, network, logs, security controls and likely troubleshooting path.
