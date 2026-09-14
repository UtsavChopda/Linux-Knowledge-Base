# 🔐 16 — SSH and Remote Access

> **Linux Knowledge Base — Beginner → Advanced → SOC / Blue Team**

## 🎯 Module Objectives

By the end of this module, you should be able to:

- Explain what SSH is and why it is used.
- Understand SSH client/server architecture.
- Connect securely to a Linux host.
- Understand passwords, public-key authentication, host keys, and fingerprints.
- Generate and manage SSH keys safely.
- Understand `~/.ssh`, `authorized_keys`, and `known_hosts`.
- Understand `ssh-agent` and agent forwarding risks.
- Read and understand important `sshd` configuration options.
- Transfer files securely with `scp` and `sftp`.
- Use SSH port forwarding in authorized environments.
- Troubleshoot SSH connectivity and authentication problems.
- Harden an SSH server using least privilege.
- Investigate failed and suspicious SSH activity from a SOC perspective.

---

# 1. What Is SSH?

**SSH — Secure Shell** is a protocol used for secure remote administration and communication over an untrusted network.

It provides mechanisms for:

- encrypted communication
- server authentication
- client/user authentication
- secure remote command execution
- secure file transfer
- tunneling/port forwarding

Typical connection:

```text
┌─────────────────┐             ┌─────────────────┐
│ SSH Client      │   network   │ SSH Server      │
│                 │ ──────────► │                 │
│ ssh             │             │ sshd            │
└─────────────────┘             └─────────────────┘
       laptop                         Linux server
```

The SSH server process is commonly called **`sshd`**.

---

# 2. Why SSH Matters for Cybersecurity

SSH is one of the most important Linux security surfaces.

It is used for legitimate administration but is also frequently targeted by attackers because successful SSH authentication can provide remote access.

A SOC may investigate:

```text
Failed SSH login
       ↓
Source IP
       ↓
Username targeted
       ↓
Authentication method
       ↓
Successful login?
       ↓
Session activity
       ↓
Persistence / privilege changes
```

Therefore, SSH knowledge is essential for a Linux Blue Team/SOC analyst.

---

# 3. SSH Client vs SSH Server

## Client

The client initiates the connection.

Command:

```bash
ssh user@server
```

The SSH client binary is usually:

```bash
ssh
```

## Server

The server accepts SSH connections.

The daemon is commonly:

```text
sshd
```

Typical flow:

```text
Client
  │
  │ TCP connection
  ▼
Server port 22
  │
  ▼
sshd
  │
  ▼
Authentication
  │
  ▼
User session
```

Port `22` is the standard SSH port, although administrators can configure another port.

---

# 4. Basic SSH Connection

Syntax:

```bash
ssh username@hostname
```

Example in a lab:

```bash
ssh analyst@192.168.56.10
```

You can specify a different port:

```bash
ssh -p 2222 analyst@192.168.56.10
```

Verbose troubleshooting:

```bash
ssh -v analyst@192.168.56.10
```

More verbosity:

```bash
ssh -vv analyst@192.168.56.10
```

Maximum debugging verbosity:

```bash
ssh -vvv analyst@192.168.56.10
```

Use high verbosity when troubleshooting, but avoid pasting sensitive authentication material into public tickets or chats.

---

# 5. What Happens During an SSH Connection?

A simplified sequence is:

```text
TCP connection
      ↓
Protocol/version negotiation
      ↓
Key exchange
      ↓
Server host authentication
      ↓
Encrypted session established
      ↓
User authentication
      ↓
Session/channel created
```

The exact protocol details are more complex, but this model is excellent for troubleshooting.

---

# 6. SSH Encryption

SSH protects the session using cryptographic mechanisms.

The session provides confidentiality and integrity protections, and SSH authenticates the server using host keys.

Conceptually:

```text
Plaintext command
      ↓
SSH encryption/integrity
      ↓
Encrypted network traffic
      ↓
SSH server
      ↓
Decryption/verification
      ↓
Command execution
```

A network observer should not normally be able to read the SSH session's plaintext commands when a properly configured modern SSH connection is used.

---

# 7. Server Host Keys

The SSH server has one or more **host keys**.

They identify the server cryptographically to clients.

Conceptually:

```text
SSH Server
   │
   └── Host private key
          │
          ▼
       fingerprint
          │
          ▼
SSH Client remembers server identity
```

Common host-key algorithms you may encounter include modern Ed25519 keys and other supported algorithms.

---

# 8. `known_hosts`

SSH clients commonly store remembered server host keys in:

```text
~/.ssh/known_hosts
```

View it:

```bash
cat ~/.ssh/known_hosts
```

The file helps SSH detect when a server's presented host key differs from the key previously associated with that host.

### Why this matters

If you previously connected to:

```text
server.example
```

and its host key suddenly changes, SSH may warn you.

This can be legitimate, for example after rebuilding a server, but it can also indicate a serious security issue such as a man-in-the-middle scenario.

**Do not blindly remove host-key warnings without understanding why the key changed.**

---

# 9. Host Key Fingerprints

A fingerprint is a compact representation of a host key.

Administrators can verify a server's fingerprint through a trusted channel.

Example concept:

```text
Expected fingerprint
        │
        ▼
SSH server presented key
        │
        ▼
Compare
        │
   ┌────┴────┐
   │         │
 match    mismatch
   │         │
 trusted   investigate
```

This is especially important when connecting to a newly provisioned or sensitive server.

---

# 10. Password Authentication

A server may allow password-based authentication.

Example:

```bash
ssh user@server
```

The server asks for the user's password if password authentication is enabled and selected.

### Security concerns

Passwords can be:

- guessed
- reused
- phished
- exposed through poor operational practices
- targeted by automated login attempts

For many administrative environments, public-key authentication is preferred.

---

# 11. Public-Key Authentication

SSH public-key authentication uses a key pair:

```text
Private Key 🔒
      +
Public Key
```

The private key remains with the client and must be protected.

The public key can be placed on the server.

Typical flow:

```text
Client private key
       │
       │ proves possession
       ▼
SSH server
       │
       ▼
Matching public key
       │
       ▼
Authentication succeeds
```

The private key is not simply sent to the server as a password.

---

# 12. SSH Key Locations

User SSH configuration normally lives under:

```text
~/.ssh/
```

Common files include:

```text
~/.ssh/
├── authorized_keys
├── known_hosts
├── config
├── id_ed25519
└── id_ed25519.pub
```

### Important distinction

```text
authorized_keys → server-side allowed public keys
known_hosts     → client-side remembered server keys
```

Do not confuse these files.

---

# 13. Generate an SSH Key

A common modern choice is Ed25519:

```bash
ssh-keygen -t ed25519
```

The command asks where to save the key and whether to protect it with a passphrase.

Typical result:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

### Security rule

Protect the private key.

```text
id_ed25519     → PRIVATE 🔒
id_ed25519.pub → PUBLIC
```

Never publish or share the private key.

---

# 14. SSH Key Passphrases

A passphrase protects the private key if the key file is stolen.

```text
Private key file
      │
      ▼
Encrypted/protected by passphrase
      │
      ▼
Attacker obtains file
      │
      ▼
Still needs passphrase
```

A passphrase does not make an exposed private key harmless, but it adds an important protection layer.

---

# 15. Copying a Public Key to a Server

The common helper is:

```bash
ssh-copy-id user@server
```

It adds your public key to the remote user's `authorized_keys` file when supported.

Then connect:

```bash
ssh user@server
```

### Manual concept

The server needs your public key in:

```text
~/.ssh/authorized_keys
```

with appropriate ownership and permissions.

---

# 16. `authorized_keys`

On the server, public keys authorized for a user are commonly stored in:

```bash
~/.ssh/authorized_keys
```

Example conceptual entry:

```text
ssh-ed25519 AAAA... comment
```

Each authorized key can permit authentication for that account.

### SOC relevance

An unexpected key added to `authorized_keys` can be a persistence mechanism.

During an investigation, review:

```bash
cat ~/.ssh/authorized_keys
```

and compare entries against the organization's known administrative keys.

Do not delete suspicious evidence before preserving it according to incident-response procedures.

---

# 17. SSH File Permissions

SSH is sensitive to unsafe permissions.

Typical secure patterns include:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/config
chmod 644 ~/.ssh/known_hosts
```

Exact permissions can vary with configuration and platform, but private keys should not be broadly readable.

Check:

```bash
ls -la ~/.ssh
```

---

# 18. SSH Client Configuration

Per-user client configuration:

```text
~/.ssh/config
```

System-wide client configuration is commonly:

```text
/etc/ssh/ssh_config
```

Example client configuration:

```text
Host lab-server
    HostName 192.168.56.10
    User analyst
    Port 22
```

Then:

```bash
ssh lab-server
```

### Security principle

Be careful with options that automatically trust keys, identities, proxies, or forwarding.

---

# 19. SSH Server Configuration — `sshd_config`

The SSH server commonly reads:

```text
/etc/ssh/sshd_config
```

Check relevant configuration:

```bash
sudo sshd -T
```

This can be useful because it shows the effective configuration after defaults and included configuration are considered.

Before making changes, back up the configuration and understand how your distribution manages SSH.

---

# 20. Important SSH Server Settings

Common settings you should understand include:

```text
Port
ListenAddress
PermitRootLogin
PasswordAuthentication
PubkeyAuthentication
AllowUsers
AllowGroups
MaxAuthTries
X11Forwarding
AllowTcpForwarding
```

Exact defaults vary by OpenSSH version and distribution.

### Example security goals

```text
Root remote login
      ↓
Prefer disabled/restricted where appropriate

Password authentication
      ↓
Prefer key-based authentication where operationally feasible

Allowed users
      ↓
Least privilege
```

Do not blindly copy a hardening configuration from the Internet. Test changes in a lab first.

---

# 21. Root Login Over SSH

A major hardening consideration is:

```text
PermitRootLogin
```

Direct remote root login increases the impact of a successful authentication.

A common administrative model is:

```text
SSH as normal user
        ↓
Controlled privilege elevation
        ↓
Administrative action
```

The correct policy depends on the environment, recovery requirements, and organization's security standards.

---

# 22. Restricting SSH Users

You can restrict which accounts may use SSH using options such as:

```text
AllowUsers
AllowGroups
```

Example concept:

```text
SSH access
    │
    ▼
Approved administrative group
    │
    ├── admin1
    └── admin2
```

Least privilege reduces the number of accounts exposed to remote authentication attacks.

---

# 23. SSH Port Changes — Security Reality

Changing SSH from port 22 to another port can reduce low-quality automated scanning noise.

However:

```text
Port change
   ≠
Strong security
```

Attackers can discover services by scanning.

Real security comes from:

- strong authentication
- key protection
- access controls
- patching
- firewall policy
- monitoring
- least privilege
- rate limiting/automated defenses where appropriate

---

# 24. SSH Service Management

On a system using systemd:

```bash
systemctl status ssh
```

or, depending on distribution/service naming:

```bash
systemctl status sshd
```

Check whether it is listening:

```bash
sudo ss -lntp | grep ':22'
```

Start/stop/restart commands depend on your service configuration.

After configuration changes, validate the SSH configuration before restarting the service.

---

# 25. Validate SSH Server Configuration

Before restarting after editing `sshd_config`, use:

```bash
sudo sshd -t
```

If supported by the installed OpenSSH version, this checks configuration syntax.

Then inspect effective settings:

```bash
sudo sshd -T
```

### Critical remote-administration rule

If you are connected to a server over SSH, **do not make untested SSH configuration changes that could lock you out**.

A safer approach is:

```text
Existing session
      │
      ├── Keep open
      │
      ▼
Validate configuration
      │
      ▼
Open second test session
      │
      ▼
Confirm access
      │
      ▼
Then finalize change
```

---

# 26. SSH Verbose Troubleshooting

Use:

```bash
ssh -v user@server
```

More detail:

```bash
ssh -vv user@server
```

Maximum:

```bash
ssh -vvv user@server
```

The output can reveal whether the failure occurs around:

```text
DNS
 ↓
TCP connection
 ↓
SSH negotiation
 ↓
Host-key verification
 ↓
Authentication
 ↓
Session setup
```

This is much better than guessing.

---

# 27. SSH Troubleshooting — Connection Refused

Example:

```text
ssh: connect to host server port 22: Connection refused
```

Possible causes:

- SSH server not running
- service listening on another port
- local/remote firewall behavior
- server process failed
- incorrect destination

On the server, investigate:

```bash
systemctl status ssh
```

or:

```bash
systemctl status sshd
```

Then:

```bash
sudo ss -lntp
```

and:

```bash
journalctl -u ssh --since "30 minutes ago"
```

Service naming varies by distribution.

---

# 28. SSH Troubleshooting — Timeout

Example:

```text
Connection timed out
```

Investigate the network path:

```bash
ip route get SERVER_IP
ping -c 3 SERVER_IP
```

Then test the port using an appropriate authorized diagnostic method.

Possible causes include:

- routing issue
- firewall filtering
- host unavailable
- security-group/network ACL policy
- wrong IP/port

A timeout does not identify the root cause by itself.

---

# 29. SSH Troubleshooting — Permission Denied

Example:

```text
Permission denied (publickey,password)
```

Possible causes:

- wrong username
- wrong private key
- public key missing from `authorized_keys`
- incorrect permissions
- server disallows the authentication method
- account restrictions
- key rejected by policy

Client-side debugging:

```bash
ssh -vv user@server
```

Server-side logs:

```bash
journalctl -u ssh
```

or distribution-specific authentication logs.

---

# 30. SSH Authentication Logs

SSH authentication events are valuable security telemetry.

Depending on the distribution, authentication events may appear in:

```text
systemd journal
/var/log/auth.log
/var/log/secure
```

Examples:

```bash
journalctl -u ssh
```

or:

```bash
journalctl _COMM=sshd
```

Search recent events:

```bash
journalctl --since "1 hour ago" | grep -i ssh
```

The exact log source and message format vary by distribution and logging configuration.

---

# 31. Failed SSH Login Investigation

A failed login is not automatically an attack.

Possible reasons include:

- user typed the wrong password
- administrator used the wrong key
- automation used an expired credential
- legitimate service misconfiguration
- password spraying/brute-force attempt

Investigate patterns rather than one event.

Look for:

```text
source IP
username
frequency
success/failure
geographic/network context
authentication method
time pattern
```

---

# 32. Brute-Force Detection Concept

Imagine logs contain:

```text
Failed password for admin from 203.0.113.10
Failed password for root from 203.0.113.10
Failed password for test from 203.0.113.10
Failed password for admin from 203.0.113.10
...
```

This pattern is more suspicious than a single failed login.

Conceptual detection:

```text
Many failures
      │
      ▼
Same source
      │
      ▼
Many usernames
      │
      ▼
Short time window
      │
      ▼
Potential password spray/brute force
```

The exact detection threshold should be tuned to the environment.

---

# 33. SSH Successful Login Investigation

Successful logins deserve context too.

Questions:

1. Which account logged in?
2. From which source IP?
3. At what time?
4. Was the source expected?
5. Was the authentication method expected?
6. Was the account normally used for SSH?
7. What happened after login?

Useful commands may include:

```bash
last
```

and:

```bash
lastlog
```

Also inspect authentication logs and shell/session activity according to your environment.

---

# 34. `last` and `lastlog`

`last` can show historical login records from system accounting data.

```bash
last
```

For a specific user:

```bash
last username
```

`lastlog` reports the most recent login information for users on systems where it is available:

```bash
lastlog
```

These are useful clues, but do not assume they are a complete forensic record.

Logs can be rotated, altered, incomplete, or configured differently.

---

# 35. SSH Sessions

Find active login sessions with:

```bash
who
```

or:

```bash
w
```

These can show:

- username
- terminal
- login time
- source host/address in many configurations
- current activity/load information

Then correlate a session with processes:

```bash
ps -ef
```

---

# 36. SSH Keys as Persistence

Attackers who gain sufficient access may attempt to add their public key to an account's:

```text
~/.ssh/authorized_keys
```

This can allow future key-based authentication.

### Defensive audit

For authorized investigation:

```bash
find /home -path '*/.ssh/authorized_keys' -type f -print
```

Then inspect files and compare keys with approved administrator inventories.

Also check root's SSH configuration where appropriate.

### Important

An unfamiliar key is a **finding to validate**, not automatic proof of compromise. Shared administration, automation, deployment systems, and configuration management can legitimately create keys.

---

# 37. SSH Agent

`ssh-agent` can hold private keys in memory so users do not need to repeatedly enter a key passphrase.

Conceptually:

```text
Private key
    │
    ▼
ssh-agent
    │
    ▼
SSH client requests signature
    │
    ▼
Agent proves key possession
```

List loaded identities:

```bash
ssh-add -l
```

Add a key:

```bash
ssh-add ~/.ssh/id_ed25519
```

Remove identities:

```bash
ssh-add -D
```

Use these commands carefully in shared or sensitive environments.

---

# 38. SSH Agent Forwarding

Agent forwarding can allow a remote SSH session to use your local SSH agent for further authentication.

Conceptually:

```text
Your workstation
      │
      │ forwarded agent
      ▼
Jump server
      │
      ▼
Internal server
```

### Security concern

If the intermediate host is compromised, forwarding can expose the ability to request signatures from your agent while the connection is active.

Therefore:

```text
Agent forwarding
      ↓
Use only when needed
      ↓
Prefer safer alternatives when appropriate
```

Do not enable forwarding globally without a clear requirement.

---

# 39. SSH Config `ProxyJump`

A common safer administrative pattern for reaching an internal host through a bastion is `ProxyJump`.

Conceptually:

```text
Your PC
   │
   ▼
Bastion / Jump Host
   │
   ▼
Internal Server
```

Example:

```bash
ssh -J bastion admin@internal-server
```

This is different from blindly exposing internal SSH services to the Internet.

---

# 40. Secure File Transfer — `scp`

`scp` can copy files over SSH.

Upload:

```bash
scp report.txt user@server:/home/user/
```

Download:

```bash
scp user@server:/home/user/report.txt .
```

Recursive copying may be supported with:

```bash
scp -r directory/ user@server:/path/
```

For large/complex transfers, `sftp` or `rsync` over SSH may be more appropriate.

---

# 41. SFTP

SFTP is the SSH File Transfer Protocol.

Connect:

```bash
sftp user@server
```

Common commands inside SFTP:

```text
ls
pwd
lpwd
cd
lcd
get
put
bye
```

SFTP provides file transfer over the SSH transport rather than using traditional FTP semantics.

---

# 42. SSH Port Forwarding

SSH can create encrypted tunnels.

### Local forwarding concept

```text
Your machine
     │
 localhost:8080
     │
     ▼
SSH tunnel
     │
     ▼
Remote host:80
```

Example in an authorized lab:

```bash
ssh -L 8080:internal-server:80 user@bastion
```

Then traffic to local port `8080` can be carried through the SSH connection toward the specified destination from the SSH server's network context.

### Security relevance

Port forwarding can be legitimate administration, but unexpected SSH tunnels may also hide or enable unauthorized network paths.

---

# 43. Remote Forwarding

Remote forwarding reverses the direction of the listening endpoint.

Conceptually:

```text
Remote host port
      │
      ▼
SSH tunnel
      │
      ▼
Your machine/service
```

Because SSH tunneling can alter network reachability, organizations should control and monitor forwarding according to policy.

---

# 44. SSH Hardening Checklist

A practical hardening approach:

```text
1. Keep OpenSSH patched
2. Use strong authentication
3. Prefer public-key authentication where appropriate
4. Protect private keys with passphrases
5. Restrict SSH users/groups
6. Minimize direct root login
7. Limit network exposure
8. Use host firewalls/network controls
9. Monitor authentication events
10. Review authorized keys
11. Control forwarding features
12. Test configuration before reload/restart
```

Security is not achieved by changing one SSH setting.

---

# 45. Least Privilege for SSH

Avoid giving every SSH user administrative access.

Better model:

```text
SSH access
    │
    ▼
Normal user
    │
    ▼
Specific sudo permissions
    │
    ▼
Required administration
```

This reduces the impact of credential compromise.

---

# 46. SSH Security Monitoring

Useful telemetry includes:

- failed authentications
- successful authentications
- source IPs
- targeted usernames
- authentication methods
- unusual login times
- unusual source networks
- new `authorized_keys` entries
- SSH configuration changes
- unexpected `sshd` processes
- SSH port-forwarding activity where logged/visible

A SIEM can aggregate these events for detection and correlation.

---

# 47. SOC Scenario — SSH Brute Force

### Alert

A server generates hundreds of failed SSH authentication events.

### Step 1 — Establish scope

Determine:

```text
Source IP(s)
Target host
Target usernames
Time range
Failure count
```

### Step 2 — Review logs

```bash
journalctl --since "1 hour ago" | grep -i ssh
```

On systems using traditional auth logs, inspect the appropriate log file.

### Step 3 — Check for successful authentication

Search around the same time window for successful login events.

### Step 4 — Check active sessions

```bash
who
w
```

### Step 5 — Check historical access

```bash
last
```

### Step 6 — Determine legitimacy

Ask:

- Is the source an approved scanner/admin network?
- Was a legitimate user locked out?
- Was there a successful login after many failures?
- Were privileged accounts targeted?

### SOC conclusion

A large number of failures may indicate brute force or password spraying, but confirm the context before declaring compromise.

---

# 48. SOC Scenario — Unexpected Successful SSH Login

### Scenario

An alert reports an SSH login to a production Linux host from an unfamiliar address.

Investigate:

```text
Who?
When?
From where?
How authenticated?
What account?
Was it expected?
What happened afterward?
```

Commands:

```bash
last
lastlog
who
w
```

Then correlate authentication logs with:

```bash
ps -ef
sudo ss -antp
```

and relevant service/application logs.

### Evidence mindset

Do not immediately delete the account, kill the session, or remove keys without following the incident-response procedure. Preserve evidence first when required.

---

# 49. SOC Scenario — New `authorized_keys` Entry

### Alert

An administrator reports an unfamiliar SSH key under a privileged account.

Investigate:

```bash
sudo ls -la /root/.ssh
sudo cat /root/.ssh/authorized_keys
```

Then correlate:

```text
Key
 ↓
Comment / identifier
 ↓
Account
 ↓
Recent login
 ↓
Source IP
 ↓
Configuration management
 ↓
Change records
```

Questions:

- Was the key created by an approved administrator?
- Is it from automation?
- When did it appear?
- Was the account recently accessed?
- Are there related configuration changes?

---

# 50. SOC Scenario — Suspicious SSH Tunnel

An unexpected process or user appears to maintain a long-lived SSH connection.

Inspect:

```bash
sudo ss -antp
```

Then:

```bash
ps -fp PID
```

and:

```bash
tr '\0' ' ' < /proc/PID/cmdline
```

Look for expected vs unusual forwarding-related options and correlate with the user's role and change records.

### Important

SSH tunnels have many legitimate uses. The investigation must be contextual.

---

# 🧪 LAB 1 — Build an SSH Lab

## Objective

Create two disposable Linux VMs or use two isolated authorized hosts.

```text
VM-A                         VM-B
Client                       Server
  │                            │
  └────── SSH connection ──────┘
```

Tasks:

1. Identify the server IP.
2. Verify the SSH service.
3. Connect using `ssh`.
4. Check the server's host key.
5. Run a remote command.

Example:

```bash
ssh user@SERVER_IP
```

Then:

```bash
hostname
id
ip -br addr
```

---

# 🧪 LAB 2 — SSH Key Authentication

## Objective

Replace password-based authentication with public-key authentication in a disposable lab.

On the client:

```bash
ssh-keygen -t ed25519
```

Copy the public key:

```bash
ssh-copy-id user@SERVER_IP
```

Connect:

```bash
ssh user@SERVER_IP
```

Verify the key files:

```bash
ls -la ~/.ssh
```

### Challenge

Add a passphrase-protected key and explain why the passphrase matters.

---

# 🧪 LAB 3 — SSH Configuration Audit

On the lab server:

```bash
sudo sshd -T
```

Review settings related to:

```text
permitrootlogin
passwordauthentication
pubkeyauthentication
allowusers
allowgroups
maxauthtries
x11forwarding
allowtcpforwarding
```

Create a short report:

```text
Setting:
Current value:
Security impact:
Recommended lab value:
Reason:
```

Do not apply production recommendations blindly.

---

# 🧪 LAB 4 — SSH Authentication Investigation

Generate a few intentional failed logins in your disposable lab.

Then inspect:

```bash
journalctl --since "30 minutes ago" | grep -i ssh
```

Also:

```bash
last
lastlog
```

Identify:

```text
Timestamp
Username
Source
Success/failure
Authentication method where visible
```

### Challenge

Create a simple detection rule concept for:

```text
5+ failed SSH logins
from the same source
within 10 minutes
```

---

# 🧪 LAB 5 — SSH Network Investigation

On the server:

```bash
sudo ss -lntp | grep ssh
```

Then connect from the client and observe:

```bash
sudo ss -antp | grep ':22'
```

Identify:

- local IP
- local port
- remote IP
- remote port
- connection state
- process/PID where visible

Connect this with:

```text
Network
 ↓
Socket
 ↓
sshd
 ↓
User session
```

---

# 🛠️ Mini Project — Linux SSH Security Auditor

Build a Bash-based audit tool that reports:

```text
============================
 SSH SECURITY AUDIT
============================

SSH SERVICE
-----------

LISTENING PORTS
---------------

EFFECTIVE SSH CONFIG
--------------------

ROOT LOGIN POLICY
-----------------

PASSWORD AUTH POLICY
--------------------

PUBLIC KEY AUTH
---------------

ALLOWED USERS/GROUPS
--------------------

AUTHORIZED KEYS
---------------

ACTIVE SSH CONNECTIONS
----------------------

RECENT SSH AUTH EVENTS
----------------------

SECURITY FINDINGS
-----------------
```

Useful commands:

```bash
systemctl status ssh
sudo sshd -T
sudo ss -lntp
sudo ss -antp
last
lastlog
who
journalctl
```

### Security findings to flag for review

```text
WARNING: unexpected SSH listening address
WARNING: direct root login permitted
WARNING: password authentication enabled
WARNING: unexpected authorized key
WARNING: unexpected active SSH session
WARNING: repeated failed authentication attempts
WARNING: suspicious source network
```

The tool should produce findings for analyst review, not automatically lock accounts or kill sessions.

---

# 🔥 Troubleshooting Master Workflow

```text
SSH Problem
    │
    ▼
DNS / destination correct?
    │
    ▼
Network path works?
    │
    ▼
Port reachable?
    │
    ▼
sshd listening?
    │
    ▼
Host-key verification?
    │
    ▼
Authentication method?
    │
    ▼
User/key/password valid?
    │
    ▼
Permissions / account policy?
    │
    ▼
Session / shell problem?
```

Commands:

```bash
ip route get SERVER_IP
ssh -vv user@SERVER_IP
sudo ss -lntp
sudo sshd -t
sudo sshd -T
journalctl -u ssh
```

---

# 🧠 Essential SSH Cheat Sheet

| Goal | Command |
|---|---|
| SSH login | `ssh user@host` |
| Custom port | `ssh -p PORT user@host` |
| Debug | `ssh -vvv user@host` |
| Generate Ed25519 key | `ssh-keygen -t ed25519` |
| Copy public key | `ssh-copy-id user@host` |
| List agent keys | `ssh-add -l` |
| Add key to agent | `ssh-add ~/.ssh/id_ed25519` |
| Remove agent keys | `ssh-add -D` |
| SFTP | `sftp user@host` |
| Copy file | `scp file user@host:/path/` |
| Server config syntax | `sudo sshd -t` |
| Effective config | `sudo sshd -T` |
| Service status | `systemctl status ssh` |
| Listening SSH socket | `sudo ss -lntp` |
| Active connections | `sudo ss -antp` |
| Login history | `last` |
| Last login per user | `lastlog` |
| Current sessions | `who` / `w` |
| SSH logs | `journalctl -u ssh` |
| Client config | `~/.ssh/config` |
| Server config | `/etc/ssh/sshd_config` |
| Authorized keys | `~/.ssh/authorized_keys` |
| Known server keys | `~/.ssh/known_hosts` |

---

# 🎯 Interview Questions

## Beginner

1. What is SSH?
2. What is the difference between an SSH client and server?
3. What is `sshd`?
4. What is the default SSH port?
5. What is a host key?
6. What is `known_hosts`?
7. What is public-key authentication?
8. What is `authorized_keys`?
9. What is an SSH private key?
10. Why should an SSH private key be protected?

## Intermediate

11. Explain the basic SSH connection sequence.
12. What is the purpose of a host-key fingerprint?
13. What does `ssh -vvv` do?
14. How do you validate `sshd_config`?
15. What is the difference between `sshd -t` and `sshd -T`?
16. How do you find SSH listening ports?
17. How do you identify active SSH connections?
18. What is `ssh-agent`?
19. What is agent forwarding?
20. What is `ProxyJump`?

## Advanced / SOC

21. How would you investigate hundreds of failed SSH logins?
22. How would you investigate an unexpected successful SSH login?
23. How can `authorized_keys` be abused for persistence?
24. Why is direct root SSH login a security concern?
25. Why is changing the SSH port not a complete security control?
26. How would you investigate an unexpected SSH tunnel?
27. What evidence would you collect before terminating a suspicious SSH session?
28. How would you correlate an SSH login with process/network activity?
29. How would you distinguish password spraying from a normal administrator mistake?
30. What SSH telemetry would you send to a SIEM?

---

# 🔐 SOC / Blue Team Mental Model

Memorize this:

```text
SSH ALERT
   │
   ▼
Source IP
   │
   ▼
Destination host
   │
   ▼
Username
   │
   ▼
Authentication method
   │
   ▼
Success / Failure
   │
   ▼
Session
   │
   ▼
Processes
   │
   ▼
Network connections
   │
   ▼
Files / keys / persistence
   │
   ▼
Timeline + correlation
```

The goal is not merely to identify **"someone logged in"**.

The goal is to answer:

> **Who accessed the system, from where, how, whether it was expected, and what happened afterward?**

---

# ✅ Module 16 Checklist

- [ ] I understand SSH client/server architecture.
- [ ] I can connect to a Linux host with SSH.
- [ ] I understand SSH encryption at a high level.
- [ ] I understand server host keys.
- [ ] I understand `known_hosts`.
- [ ] I understand public-key authentication.
- [ ] I can generate an Ed25519 key pair.
- [ ] I understand `authorized_keys`.
- [ ] I know how to protect SSH private keys.
- [ ] I understand SSH client configuration.
- [ ] I understand `sshd_config`.
- [ ] I can validate SSH configuration.
- [ ] I understand root-login and password-authentication risks.
- [ ] I can troubleshoot SSH with `ssh -vvv`.
- [ ] I can inspect SSH listening ports with `ss`.
- [ ] I understand `ssh-agent`.
- [ ] I understand agent-forwarding risks.
- [ ] I understand `ProxyJump`.
- [ ] I can use `scp` and `sftp`.
- [ ] I understand SSH port forwarding at a high level.
- [ ] I can investigate failed SSH logins.
- [ ] I can investigate suspicious successful logins.
- [ ] I understand SSH keys as a potential persistence mechanism.
- [ ] I can build a basic SSH security audit.

---

# 🚀 Next Module

## **17 — Firewall and Linux Security** 🛡️

Next we connect networking and SSH to host defense:

```text
Linux Firewall
      │
      ├── Netfilter / nftables
      ├── iptables legacy concepts
      ├── UFW
      ├── firewalld
      ├── Zones & services
      ├── Rules & policies
      ├── Stateful filtering
      ├── Logging
      ├── Troubleshooting blocked traffic
      ├── Host hardening
      └── SOC firewall investigation
```

> **Goal:** understand how Linux controls network traffic locally and how a Blue Team analyst investigates firewall behavior.
