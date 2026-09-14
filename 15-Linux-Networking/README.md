# 🌐 15 — Linux Networking

> **Linux Knowledge Base — Beginner → Advanced → SOC / Blue Team**

## 🎯 Module Objectives

By the end of this module, you should be able to:

- Explain how networking works on a Linux host.
- Understand network interfaces, MAC addresses, IP addresses, subnets, gateways, and routes.
- Inspect and configure interfaces with modern Linux networking tools.
- Understand IPv4, IPv6, loopback, and link-local addressing.
- Understand routing and the default gateway.
- Explain ARP/neighbor discovery at a practical level.
- Understand DNS resolution and troubleshoot DNS failures.
- Inspect listening ports, established connections, and sockets.
- Test connectivity systematically instead of guessing.
- Understand NetworkManager and `nmcli` on systems that use it.
- Understand network namespaces at a high level.
- Investigate suspicious network activity from a SOC/Blue Team perspective.

---

# 1. Why Linux Networking Matters

Linux is everywhere in modern infrastructure:

```text
Servers
Cloud
Containers
Firewalls
Virtual Machines
Network Appliances
Security Tools
SIEM Infrastructure
Web Servers
Databases
```

For a Linux administrator or SOC analyst, networking is not optional knowledge.

A common security investigation looks like:

```text
Process
  ↓
Socket
  ↓
IP + Port
  ↓
Remote IP
  ↓
DNS / Routing
  ↓
Network Logs
  ↓
Security Decision
```

---

# 2. Linux Network Architecture

A simplified model:

```text
Application
    │
    ▼
Socket API
    │
    ▼
TCP / UDP
    │
    ▼
IP
    │
    ▼
Routing
    │
    ▼
Network Interface
    │
    ▼
Driver / Kernel
    │
    ▼
NIC / Virtual NIC
    │
    ▼
Network
```

Applications normally do not directly manipulate Ethernet frames. They use sockets, while the kernel handles much of the protocol stack and interface operations.

---

# 3. Network Interface

A **network interface** represents a connection between the Linux system and a network.

Examples:

```text
eth0
ens33
enp0s3
wlan0
lo
```

Modern Linux systems often use predictable interface names such as `ens33` or `enp0s3` rather than always using `eth0`.

List interfaces:

```bash
ip link
```

Short form:

```bash
ip -br link
```

Example conceptual output:

```text
lo       UNKNOWN
ens33    UP
```

---

# 4. Loopback Interface

The loopback interface is normally named:

```text
lo
```

It allows a system to communicate with itself through the networking stack.

Common address:

```text
127.0.0.1
```

IPv6 loopback:

```text
::1
```

Test it:

```bash
ping -c 3 127.0.0.1
```

### Security relevance

A service listening only on loopback may be inaccessible directly from the network.

Compare:

```text
127.0.0.1:8080
```

with:

```text
0.0.0.0:8080
```

The second indicates listening on all IPv4 local interfaces, subject to other network/firewall controls.

---

# 5. MAC Address

A network interface commonly has a Layer-2 MAC address.

View it:

```bash
ip link show
```

Example:

```text
link/ether 52:54:00:12:34:56
```

A MAC address is associated with the local network interface and is used by Ethernet/LAN technologies at Layer 2.

### Important

A MAC address is not the same thing as an IP address.

```text
MAC → local Layer-2 identity/address
IP  → Layer-3 network address
```

Virtual machines, containers, Wi-Fi, bridges, and virtualization systems can make the details more complex.

---

# 6. IP Address

An IP address identifies a host/interface at the network layer.

IPv4 example:

```text
192.168.1.50
```

IPv6 example:

```text
2001:db8::50
```

View addresses:

```bash
ip addr
```

Short form:

```bash
ip -br addr
```

Example:

```text
lo      UNKNOWN  127.0.0.1/8
ens33   UP       192.168.1.50/24
```

---

# 7. IPv4 CIDR and Subnets

Consider:

```text
192.168.1.50/24
```

The `/24` is the prefix length.

A `/24` corresponds to the traditional subnet mask:

```text
255.255.255.0
```

Conceptually:

```text
192.168.1.50/24
│────────────│
  network + host
```

The prefix tells Linux which destinations are directly reachable through the local network and which require routing.

---

# 8. Private IPv4 Addresses

Common private IPv4 ranges include:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

These are commonly used inside private networks and are not globally routable on the public Internet in the normal way.

### Example

```text
Laptop       192.168.1.20
Linux VM     192.168.1.50
Router       192.168.1.1
```

All may be part of the same local subnet.

---

# 9. IPv6 Basics

Linux supports IPv6 alongside IPv4.

View IPv6 addresses:

```bash
ip -6 addr
```

Important categories include:

- loopback: `::1`
- link-local: typically `fe80::/10`
- global unicast addresses

View IPv6 routes:

```bash
ip -6 route
```

Do not assume that a host is IPv4-only just because `ip addr` shows a familiar IPv4 address.

---

# 10. Interface State

Inspect an interface:

```bash
ip link show ens33
```

Bring an interface administratively up:

```bash
sudo ip link set ens33 up
```

Bring it down:

```bash
sudo ip link set ens33 down
```

> Use configuration commands carefully, especially over SSH. Bringing down your active interface can disconnect your session.

---

# 11. NetworkManager

Many desktop and enterprise Linux distributions use **NetworkManager** to manage network connections.

Check whether it is active:

```bash
systemctl status NetworkManager
```

View connections:

```bash
nmcli connection show
```

View devices:

```bash
nmcli device status
```

Show general networking state:

```bash
nmcli general status
```

### Important

Not every Linux environment uses NetworkManager. Servers, minimal distributions, containers, and specialized systems may use other network configuration methods.

---

# 12. `ip` vs `ifconfig`

Modern Linux administration should prioritize the `ip` command from the `iproute2` suite.

Preferred:

```bash
ip addr
ip link
ip route
ip neigh
```

You may still encounter:

```bash
ifconfig
```

on older systems or legacy documentation.

For modern Linux troubleshooting, learn `ip` first.

---

# 13. Default Gateway

A host needs a route to reach networks that are not directly connected.

Usually, this is handled by a **default route**.

View routes:

```bash
ip route
```

Example:

```text
default via 192.168.1.1 dev ens33
192.168.1.0/24 dev ens33 proto kernel scope link src 192.168.1.50
```

Meaning:

```text
Unknown destination
       │
       ▼
default route
       │
       ▼
192.168.1.1
       │
       ▼
   external network
```

---

# 14. Routing Table

The routing table tells Linux where to send packets.

View it:

```bash
ip route
```

Useful IPv6 form:

```bash
ip -6 route
```

A simplified decision:

```text
Destination
    │
    ▼
Routing table lookup
    │
    ├── Specific matching route?
    │       ↓
    │     use it
    │
    └── Otherwise
            ↓
       default route
```

Linux generally chooses the most specific matching route before a less-specific/default route.

---

# 15. Ask Linux Which Route It Will Use

One of the most useful troubleshooting commands is:

```bash
ip route get 8.8.8.8
```

This can show information such as:

- selected route
- interface
- source address
- next hop

Example conceptual result:

```text
8.8.8.8 via 192.168.1.1 dev ens33 src 192.168.1.50
```

This is often more useful than simply staring at the whole routing table.

---

# 16. ARP and Neighbor Discovery

IPv4 hosts on an Ethernet-like LAN need to map an IP address to a Layer-2 address such as a MAC address.

ARP performs this function for IPv4.

Linux exposes neighbor information through:

```bash
ip neigh
```

Example:

```text
192.168.1.1 dev ens33 lladdr aa:bb:cc:dd:ee:ff REACHABLE
```

For IPv6, Neighbor Discovery is used instead of ARP.

```bash
ip -6 neigh
```

---

# 17. Understanding Neighbor States

You may see states such as:

```text
REACHABLE
STALE
DELAY
PROBE
FAILED
INCOMPLETE
```

These states describe Linux's knowledge of neighbor reachability and resolution.

A `STALE` entry is not automatically an error. It can simply mean that the cached information has not recently been confirmed.

---

# 18. DNS — Domain Name System

DNS translates names into network information.

Example:

```text
example.com
    ↓
DNS
    ↓
IP address
```

A typical Linux application may use the system resolver configuration and resolver libraries rather than manually querying a DNS server itself.

Inspect resolver configuration:

```bash
cat /etc/resolv.conf
```

On systems managed by `systemd-resolved`, you may also inspect:

```bash
resolvectl status
```

Not every Linux distribution uses `systemd-resolved`.

---

# 19. Test DNS Resolution

Useful commands:

```bash
getent hosts example.com
```

If available:

```bash
dig example.com
```

or:

```bash
nslookup example.com
```

For troubleshooting, `getent` is useful because it tests the system's configured name-service path rather than only talking directly to DNS in isolation.

---

# 20. DNS Troubleshooting Logic

If this works:

```bash
ping -c 3 1.1.1.1
```

but this fails:

```bash
ping -c 3 example.com
```

one possibility is DNS resolution failure.

Use:

```bash
getent hosts example.com
```

Then inspect:

```bash
cat /etc/resolv.conf
```

Do not conclude that every failed `ping example.com` is DNS-related: ICMP may be blocked, the destination may not respond to ping, or another issue may exist.

---

# 21. TCP and UDP

Linux applications commonly communicate using TCP or UDP.

## TCP

TCP provides connection-oriented communication with mechanisms such as:

- reliable delivery
- sequencing
- retransmission
- flow/congestion control

## UDP

UDP is connectionless at the transport protocol level and has lower protocol overhead, but does not itself provide TCP-style reliability.

Common examples:

```text
TCP 22   → SSH
TCP 80   → HTTP
TCP 443  → HTTPS
UDP 53   → DNS (commonly)
```

These are conventions, not guarantees. A service can use different ports.

---

# 22. Ports

A port helps identify a transport-layer endpoint associated with an application/service.

Conceptually:

```text
IP Address + Protocol + Port
          │
          ▼
       Endpoint
```

Example:

```text
192.168.1.50:22/TCP
```

Port ranges are commonly described as:

```text
0–1023       Well-known
1024–49151   Registered
49152–65535  Dynamic/private
```

Exact assignments and operating-system behavior can vary, so treat the ranges as conventions rather than application guarantees.

---

# 23. Sockets

A socket is an operating-system abstraction used by applications for network communication.

A socket can represent:

- listening service
- established connection
- local endpoint
- remote endpoint
- transport protocol

This is why socket inspection is extremely useful in SOC investigations.

---

# 24. `ss` — Essential Socket Tool

List listening TCP/UDP sockets:

```bash
sudo ss -lntup
```

Useful flags:

| Flag | Meaning |
|---|---|
| `-l` | Listening |
| `-n` | Numeric addresses/ports |
| `-t` | TCP |
| `-u` | UDP |
| `-p` | Process information where permitted |

All TCP connections:

```bash
ss -ant
```

All UDP sockets:

```bash
ss -anu
```

---

# 25. Listening vs Established

A listening socket waits for incoming connections.

```text
LISTEN
   │
   ▼
Waiting for clients
```

An established TCP connection represents an active TCP session.

```text
ESTABLISHED
   │
   ├── local IP:port
   └── remote IP:port
```

For a SOC analyst:

```bash
ss -antp
```

can help connect network activity to local processes when process attribution is available.

---

# 26. Which Process Owns the Connection?

Try:

```bash
sudo ss -lntup
```

You may see:

```text
users:(("sshd",pid=1234,fd=3))
```

Then investigate the process:

```bash
ps -fp 1234
```

And, when appropriate:

```bash
readlink -f /proc/1234/exe
```

This creates a useful correlation:

```text
Port
 ↓
Socket
 ↓
PID
 ↓
Process
 ↓
Executable
 ↓
User / Service
```

---

# 27. Network Connectivity Testing

Use a layered troubleshooting approach.

### Step 1 — Interface

```bash
ip link
```

### Step 2 — IP address

```bash
ip addr
```

### Step 3 — Route

```bash
ip route
```

### Step 4 — Neighbor/gateway

```bash
ip neigh
```

### Step 5 — DNS

```bash
getent hosts example.com
```

### Step 6 — Application/service

Use an appropriate protocol-specific test.

```text
Interface
   ↓
Address
   ↓
Route
   ↓
Neighbor
   ↓
DNS
   ↓
Port
   ↓
Application
```

This prevents random command guessing.

---

# 28. `ping`

`ping` commonly uses ICMP Echo messages for IPv4/IPv6 connectivity testing.

Example:

```bash
ping -c 4 192.168.1.1
```

### Important

A failed ping does **not** prove that a host is unreachable.

Reasons include:

- ICMP filtering
- host firewall
- routing problems
- destination policy
- service-specific behavior

Use `ping` as one test in a larger troubleshooting workflow.

---

# 29. `traceroute` / `tracepath`

These tools help investigate the path toward a destination.

Examples:

```bash
tracepath example.com
```

or, where installed:

```bash
traceroute example.com
```

A path may contain devices that do not respond to diagnostic probes, so `*` does not automatically mean the path is broken.

---

# 30. Application-Level Testing

If DNS and routing work, test the actual application.

For an HTTP/HTTPS service, for example:

```bash
curl -I https://example.com
```

This tests more than ICMP because it exercises the application protocol.

Conceptually:

```text
ping
  ↓
ICMP reachability test

curl
  ↓
DNS + TCP + TLS + HTTP
```

The exact layers involved depend on the command and destination.

---

# 31. Hostname

View hostname:

```bash
hostname
```

More detailed information:

```bash
hostnamectl
```

Hostname can be useful during administration and SOC investigations, especially when multiple servers are involved.

---

# 32. `/etc/hosts`

Linux can use a local hosts file for static name mappings.

View it:

```bash
cat /etc/hosts
```

Example:

```text
127.0.0.1   localhost
192.168.1.20   lab-server
```

A manipulated `/etc/hosts` file can redirect names locally.

### SOC relevance

If a known domain unexpectedly resolves to an unusual local IP, inspect:

```bash
cat /etc/hosts
```

and the system's resolver configuration.

---

# 33. Network Configuration Files

Linux networking configuration varies significantly by distribution and network stack.

You may encounter:

```text
NetworkManager
systemd-networkd
netplan
ifupdown
```

Do not assume that a configuration file from one distribution applies to another.

First identify the networking system in use.

Useful checks:

```bash
systemctl is-active NetworkManager
systemctl is-active systemd-networkd
```

And:

```bash
nmcli general status
```

when NetworkManager is present.

---

# 34. Persistent vs Temporary Configuration

Commands such as:

```bash
ip addr add ...
ip route add ...
```

can change the current runtime network configuration.

Such changes may disappear after reboot or after the network-management service reapplies its configuration.

Persistent configuration should be managed using the system's actual networking framework.

### Principle

```text
Runtime state
     ≠
Persistent configuration
```

This distinction is important during troubleshooting.

---

# 35. Network Namespaces

A **network namespace** provides an isolated network environment inside the same Linux kernel.

Conceptually:

```text
Linux Kernel
│
├── Namespace A
│    ├── interfaces
│    ├── routes
│    └── sockets
│
└── Namespace B
     ├── interfaces
     ├── routes
     └── sockets
```

Containers commonly use network namespaces to isolate network resources.

List namespaces when the `ip` tooling supports it:

```bash
ip netns list
```

### Why SOC analysts should care

A process may exist inside a container or namespace and therefore not appear in the same network view as a host-level service.

---

# 36. Bridges and Virtual Networking

Virtualization and containers commonly introduce interfaces such as:

```text
br0
virbr0
docker0
vethXXXX
```

A bridge behaves conceptually like a virtual Layer-2 switch.

```text
VM ──┐
VM ──┼── virtual bridge ── physical/virtual uplink
Container ─┘
```

Do not label an unfamiliar interface as malicious simply because it is virtual.

First determine which virtualization/container platform created it.

---

# 37. Network Troubleshooting Master Workflow

When a Linux host cannot reach a destination:

```text
1. Is interface UP?
        ↓
   ip link
        ↓
2. Does it have an IP?
        ↓
   ip addr
        ↓
3. Is there a route?
        ↓
   ip route
        ↓
4. Which route is selected?
        ↓
   ip route get DEST
        ↓
5. Can local neighbor/gateway resolve?
        ↓
   ip neigh
        ↓
6. Does DNS work?
        ↓
   getent hosts NAME
        ↓
7. Is the destination port reachable?
        ↓
   application-specific test
        ↓
8. Is the application healthy?
```

---

# 38. Common Networking Failures

| Symptom | Possible cause |
|---|---|
| No interface | Driver/device/configuration issue |
| Interface DOWN | Administrative or network-manager state |
| No IP address | DHCP/static configuration issue |
| Local subnet unreachable | Interface, subnet, VLAN, ARP/neighbor issue |
| Internet unreachable | Route/gateway issue |
| IP works, hostname fails | DNS/resolver issue |
| Port closed | Service not listening/firewall/policy |
| Connection times out | Routing/filtering/firewall/service issue |
| Connection refused | Host reachable but endpoint is not accepting connection |
| Intermittent connectivity | Link, congestion, DNS, routing, MTU, or service issue |

Never diagnose solely from one symptom.

---

# 39. Connection Refused vs Timeout

These are useful clues.

### Connection refused

Often means the network path reached the destination host, but the destination rejected the connection because nothing was accepting it or a device actively rejected it.

### Timeout

Can indicate:

- packet filtering
- routing failure
- host unavailable
- firewall policy
- service/network path problem

Neither message alone identifies the exact root cause.

---

# 40. MTU Basics

MTU is the maximum size of a packet/frame payload handled at a particular network layer/interface context.

Typical Ethernet MTU:

```text
1500 bytes
```

but other values are common in tunnels, VPNs, virtualization, and specialized networks.

Check interface information:

```bash
ip link show
```

MTU problems can cause strange symptoms where some connections work while larger packets or specific applications fail.

---

# 41. Network Security — Host Firewall vs Service

A service listening on a port does not automatically mean the port is reachable from every network.

```text
Application
    │
    ▼
Listening socket
    │
    ▼
Host firewall / security policy
    │
    ▼
Network path
    │
    ▼
Remote host
```

This distinction is important when troubleshooting and during security assessments.

---

# 42. Identify Exposed Services

Start with:

```bash
sudo ss -lntup
```

For each listening service ask:

1. What port?
2. TCP or UDP?
3. Which local address?
4. Which process?
5. Which user?
6. Which executable?
7. Is it expected?
8. Is remote access required?
9. What firewall policy applies?

This is an excellent baseline for Linux hardening.

---

# 43. SOC Investigation — Suspicious Outbound Connection

### Scenario

A Linux server is making an unexpected outbound connection to a public IP.

Do not immediately kill the process.

### Step 1 — Identify connections

```bash
sudo ss -antp
```

### Step 2 — Identify PID

Suppose the connection belongs to PID `2468`.

```bash
ps -fp 2468
```

### Step 3 — Inspect executable

```bash
readlink -f /proc/2468/exe
```

### Step 4 — Inspect command line

```bash
tr '\0' ' ' < /proc/2468/cmdline
```

### Step 5 — Identify owner

```bash
ps -o pid,ppid,user,group,etime,cmd -p 2468
```

### Step 6 — Identify parent process

```bash
ps -o pid,ppid,cmd -p 2468
```

Then inspect the parent PID.

### Step 7 — Correlate

Look at:

```text
Process
 ↓
Executable
 ↓
User
 ↓
Parent
 ↓
Service / Scheduled task
 ↓
Destination IP/domain
 ↓
Logs
```

### Security principle

An unknown outbound connection is a **signal to investigate**, not automatic proof of compromise.

---

# 44. SOC Investigation — Unexpected Listening Port

Run:

```bash
sudo ss -lntup
```

Suppose you find an unexpected port.

Investigate:

```bash
ps -fp PID
readlink -f /proc/PID/exe
tr '\0' ' ' < /proc/PID/cmdline
```

Then determine:

- Is the process legitimate?
- Which package installed it?
- Which service manages it?
- Which user runs it?
- Is the port documented?
- Is it exposed beyond localhost?
- When did it start?
- Are there related logs?

Do not assume that a high-numbered port is malicious.

---

# 45. Network Logs

Network-related evidence can exist in many locations:

```text
systemd journal
application logs
firewall logs
DNS logs
proxy logs
EDR telemetry
cloud flow logs
authentication logs
```

View kernel/network-related journal entries:

```bash
journalctl -k
```

Search the journal for a service:

```bash
journalctl -u SERVICE
```

For an actual incident, correlate host telemetry with network telemetry whenever possible.

---

# 46. Network Baseline

A baseline answers:

```text
What is normal for this host?
```

Record items such as:

- expected interfaces
- expected IPs
- expected routes
- expected DNS configuration
- expected listening ports
- expected services
- expected outbound destinations
- expected container/VM interfaces

Then deviations become easier to investigate.

---

# 47. Defensive Network Audit Commands

Useful read-only inspection commands:

```bash
hostname
ip -br link
ip -br addr
ip route
ip -6 route
ip neigh
cat /etc/resolv.conf
cat /etc/hosts
sudo ss -lntup
sudo ss -antp
```

This gives a compact host-network snapshot.

---

# 🧪 LAB 1 — Linux Network Discovery

## Objective

Build a complete picture of your Linux host's network configuration.

Run:

```bash
hostname
ip -br link
ip -br addr
ip route
ip -6 route
ip neigh
```

Answer:

1. What is your active interface?
2. What is its MAC address?
3. What IPv4 address does it have?
4. Does it have IPv6?
5. What is the default gateway?
6. What subnet is directly connected?
7. What neighbor entries exist?

---

# 🧪 LAB 2 — DNS Investigation

## Objective

Understand the difference between network connectivity and name resolution.

Run:

```bash
cat /etc/resolv.conf
getent hosts example.com
```

If available:

```bash
dig example.com
```

Then compare:

```bash
ping -c 3 1.1.1.1
ping -c 3 example.com
```

### Challenge

Explain what could cause:

```text
IP connectivity works
BUT
DNS name resolution fails
```

---

# 🧪 LAB 3 — Socket & Listening-Port Audit

## Objective

Identify services exposed by your lab machine.

Run:

```bash
sudo ss -lntup
```

For each listening port, document:

```text
Port
Protocol
Local address
PID
Process
User
Expected?
```

### Security challenge

Identify any service listening on a non-loopback address that does not need to be remotely reachable in your lab.

Do not disable services blindly. Determine what owns them first.

---

# 🧪 LAB 4 — Route Investigation

## Objective

Understand how Linux chooses a route.

Run:

```bash
ip route
```

Then test:

```bash
ip route get 8.8.8.8
```

Try another destination relevant to your lab.

Answer:

- Which interface is selected?
- Which source IP is selected?
- Is a gateway used?
- Which route matched?

---

# 🧪 LAB 5 — Network Troubleshooting Scenario

### Scenario

A web application on a lab server cannot reach an external API.

Work through:

```text
Interface
 ↓
IP
 ↓
Route
 ↓
Gateway
 ↓
DNS
 ↓
Remote port
 ↓
Application
```

Commands you may use:

```bash
ip -br link
ip -br addr
ip route
ip route get DESTINATION
ip neigh
getent hosts API_HOSTNAME
ss -antp
curl -v https://API_HOSTNAME
```

### Deliverable

Write a short incident note:

```text
Problem:

Evidence:

Tests performed:

Finding:

Root cause:

Recommended action:
```

---

# 🛠️ Mini Project — Linux Network Auditor

Build a Bash script that produces a host-network report.

## Required output

```text
============================
 LINUX NETWORK AUDIT
============================

HOSTNAME
--------

INTERFACES
----------

IP ADDRESSES
------------

ROUTING
-------

NEIGHBORS
---------

DNS
---

LISTENING PORTS
---------------

ACTIVE CONNECTIONS
------------------

SECURITY FINDINGS
-----------------
```

Suggested commands:

```bash
hostname
ip -br link
ip -br addr
ip route
ip neigh
cat /etc/resolv.conf
sudo ss -lntup
sudo ss -antp
```

### Security enhancement

Flag for analyst review:

```text
WARNING: unexpected listening service
WARNING: service exposed on non-loopback address
WARNING: unexpected default route
WARNING: unexpected DNS configuration
WARNING: unexpected outbound connection
```

The script should report findings rather than automatically taking disruptive action.

---

# 🔥 Troubleshooting Scenarios

## Scenario 1 — No IP Address

Check:

```bash
ip -br addr
ip link
```

Then identify the network-management system:

```bash
nmcli device status
```

if applicable.

Possible causes:

- DHCP failure
- incorrect static configuration
- interface down
- VLAN/network issue
- virtualization problem

---

## Scenario 2 — Can Reach Gateway, Cannot Reach Internet

Check:

```bash
ip route
ip route get 1.1.1.1
```

Then test an IP address:

```bash
ping -c 3 1.1.1.1
```

If gateway works but external IP does not, investigate routing/upstream connectivity/firewall policy.

---

## Scenario 3 — IP Works, Domain Does Not

Check:

```bash
getent hosts example.com
cat /etc/resolv.conf
```

Investigate DNS configuration and resolver reachability.

---

## Scenario 4 — Service Works Locally but Not Remotely

Check the listening address:

```bash
sudo ss -lntup
```

Compare:

```text
127.0.0.1:8080
```

vs.

```text
0.0.0.0:8080
```

Then investigate host firewall and network policy.

---

## Scenario 5 — Unexpected Outbound Connection

Use:

```bash
sudo ss -antp
```

Then correlate:

```text
Remote IP
 ↓
Local port
 ↓
PID
 ↓
Process
 ↓
Executable
 ↓
User
 ↓
Parent/service
 ↓
Logs
```

Preserve evidence and follow your incident-response process before taking disruptive action.

---

# 🧠 Essential Command Cheat Sheet

| Goal | Command |
|---|---|
| Show interfaces | `ip link` |
| Compact interfaces | `ip -br link` |
| Show IP addresses | `ip addr` |
| Compact addresses | `ip -br addr` |
| Show routes | `ip route` |
| Show IPv6 routes | `ip -6 route` |
| Predict route | `ip route get DEST` |
| Show neighbors | `ip neigh` |
| Show IPv6 neighbors | `ip -6 neigh` |
| Show NetworkManager devices | `nmcli device status` |
| Show connections | `nmcli connection show` |
| Show hostname | `hostname` |
| Resolver config | `cat /etc/resolv.conf` |
| Static host mappings | `cat /etc/hosts` |
| Resolve using system config | `getent hosts NAME` |
| DNS query | `dig NAME` |
| Test ICMP | `ping -c 4 HOST` |
| Trace path | `tracepath HOST` |
| Listening TCP/UDP | `sudo ss -lntup` |
| TCP connections | `ss -ant` |
| UDP sockets | `ss -anu` |
| Process/network correlation | `sudo ss -antp` |
| Network namespaces | `ip netns list` |
| Kernel logs | `journalctl -k` |

---

# 🎯 Interview Questions

## Beginner

1. What is a network interface?
2. What is the loopback interface?
3. What is an IP address?
4. What is a MAC address?
5. What is a subnet?
6. What is a default gateway?
7. What is DNS?
8. What is a port?
9. What is a socket?
10. What is the difference between TCP and UDP?

## Intermediate

11. How do you find a Linux host's IP address?
12. How do you view the routing table?
13. How do you determine which route Linux will use for a destination?
14. What does `ip neigh` show?
15. What is ARP?
16. What is IPv6 Neighbor Discovery?
17. How do you find listening ports?
18. How do you identify which process owns a port?
19. What is the difference between listening and established sockets?
20. What is the difference between runtime and persistent network configuration?

## Advanced / SOC

21. How would you troubleshoot a host that has an IP but cannot access the Internet?
22. How would you troubleshoot DNS separately from general connectivity?
23. How would you investigate an unexpected listening port?
24. How would you investigate an unexpected outbound connection?
25. Why is `0.0.0.0:PORT` different from `127.0.0.1:PORT`?
26. Why can a failed ping still mean the host is reachable?
27. What are network namespaces?
28. Why might a container's network activity not be obvious from a simple host-level view?
29. How can `/etc/hosts` create a security concern?
30. What information would you collect before terminating a suspicious network-connected process?

---

# 🔐 SOC / Blue Team Takeaways

Linux networking gives you the foundation for host-based network investigation.

A useful SOC workflow is:

```text
ALERT
  │
  ▼
Network Connection
  │
  ├── Source IP/Port
  ├── Destination IP/Port
  └── Protocol
  │
  ▼
Local PID
  │
  ▼
Process
  │
  ├── User
  ├── Parent
  ├── Executable
  └── Command line
  │
  ▼
Service / Scheduled Origin
  │
  ▼
Logs + EDR + DNS + Firewall
  │
  ▼
Expected or Suspicious?
```

### Skills to practice

- interface enumeration
- IP/subnet understanding
- route analysis
- neighbor inspection
- DNS troubleshooting
- port/socket enumeration
- process-to-network correlation
- network baselining
- container/namespace awareness
- defensive incident investigation

---

# 🧩 Quick Mental Model

Memorize this sequence:

```text
NIC
 ↓
MAC
 ↓
IP
 ↓
Subnet
 ↓
Route
 ↓
Gateway
 ↓
DNS
 ↓
Port
 ↓
Socket
 ↓
Process
```

For SOC work, reverse the last part:

```text
Suspicious IP/Port
       ↓
Socket
       ↓
PID
       ↓
Process
       ↓
Executable
       ↓
User
       ↓
Parent / Service
       ↓
Logs
```

---

# ✅ Module 15 Checklist

- [ ] I understand Linux network interfaces.
- [ ] I can identify the loopback interface.
- [ ] I understand MAC vs IP addresses.
- [ ] I understand IPv4 CIDR/subnets.
- [ ] I know common private IPv4 ranges.
- [ ] I understand basic IPv6 addressing.
- [ ] I can inspect interfaces with `ip`.
- [ ] I understand NetworkManager at a high level.
- [ ] I can inspect routing tables.
- [ ] I can use `ip route get`.
- [ ] I understand ARP and neighbor tables.
- [ ] I understand DNS resolution.
- [ ] I can troubleshoot DNS separately from connectivity.
- [ ] I understand TCP, UDP, ports, and sockets.
- [ ] I can identify listening services with `ss`.
- [ ] I can correlate sockets with processes.
- [ ] I understand runtime vs persistent network configuration.
- [ ] I understand network namespaces at a high level.
- [ ] I can troubleshoot Linux connectivity systematically.
- [ ] I can investigate suspicious inbound/outbound network activity.

---

# 🚀 Next Module

## **16 — SSH and Remote Access**

Next we will go deeper into one of the most important Linux administration and SOC protocols:

```text
SSH
 │
 ├── Client / Server architecture
 ├── Authentication
 ├── Password vs key authentication
 ├── SSH keys
 ├── ssh-agent
 ├── known_hosts
 ├── sshd configuration
 ├── Secure remote administration
 ├── SSH troubleshooting
 ├── Failed-login investigation
 ├── Brute-force detection
 └── SOC investigation of suspicious SSH activity
```

> **Goal:** understand SSH not just as a remote-login command, but as an authentication system and a major source of security telemetry.
