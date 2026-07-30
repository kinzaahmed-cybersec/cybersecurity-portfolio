# Lab 1 Learning Notes: Network Connectivity Verification, ARP & ICMP Analysis

## Objective

Understand how two virtual machines communicate inside the same network and analyze the communication process using networking tools.

This lab covered:

* IP addressing
* Subnet identification
* VMware NAT networking
* Network connectivity testing
* Firewall troubleshooting
* ARP resolution
* MAC address discovery
* ICMP communication
* Packet capture and analysis using Wireshark

These concepts are fundamental for SOC operations because analysts regularly investigate:

* Network connectivity problems
* Firewall events
* Suspicious communication
* Packet captures
* SIEM network alerts
* Lateral movement activity

---

# Lab Environment

| Component       | Details            |
| --------------- | ------------------ |
| Hypervisor      | VMware Workstation |
| Network Type    | VMware NAT Network |
| Machine 1       | Windows 11 VM      |
| Machine 2       | Kali Linux VM      |
| Packet Analyzer | Wireshark          |

---

# Initial Environment Verification

Before beginning any investigation, verify the current state of the environment.

In real SOC investigations, analysts should never blindly trust previous information.

DHCP leases may change after:

* VM restart
* Host restart
* Lease expiration

Always verify:

* IP address
* Network interface
* Gateway
* Connectivity status

---

# Windows 11 Network Verification

Command:

```cmd
ipconfig
```

Output:

```
IPv4 Address:
192.168.16.131

Subnet Mask:
255.255.255.0

Default Gateway:
192.168.16.2
```

## Analysis

### IPv4 Address

```
192.168.16.131
```

Private IP address assigned to the Windows VM through VMware's virtual DHCP service.

---

### Subnet Mask

```
255.255.255.0
```

Equivalent CIDR notation:

```
/24
```

Meaning:

* First 24 bits identify the network.
* Remaining 8 bits identify hosts.

Network:

```
192.168.16.0/24
```

Usable hosts:

```
192.168.16.1 - 192.168.16.254
```

Broadcast:

```
192.168.16.255
```

SOC analysts frequently encounter CIDR notation in:

* Firewall rules
* SIEM alerts
* Network logs
* Threat intelligence reports

---

### Default Gateway

```
192.168.16.2
```

This is VMware's virtual NAT router.

It:

* Provides NAT functionality
* Allows VM internet access
* Keeps the virtual lab isolated from the physical LAN

It is not the physical home router.

---

# Kali Linux Network Verification

Command:

```bash
ip addr
```

Important output:

```
Interface:
eth0

MAC Address:
00:0c:29:b8:90:4f

IPv4 Address:
192.168.16.130/24
```

---

# Network Comparison

| Machine    | IP Address     | Network         |
| ---------- | -------------- | --------------- |
| Windows 11 | 192.168.16.131 | 192.168.16.0/24 |
| Kali Linux | 192.168.16.130 | 192.168.16.0/24 |

Inference:

* Both systems are on the same subnet.
* Both systems can communicate directly.
* Routing is not required between them.

---

# Understanding Subnetting

Example:

```
192.168.16.130/24
```

The `/24` means:

* First 24 bits = network portion
* Remaining 8 bits = host portion

Therefore:

```
Network:
192.168.16.0/24
```

SOC relevance:

Logs may show:

```
10.10.50.25/24
```

Analysts must infer which network the host belongs to.

---

# Connectivity Testing

Command:

```bash
ping -c 4 192.168.16.131
```

Purpose:

Verify communication between Kali Linux and Windows.

---

# Troubleshooting Scenario

## Problem

Communication was asymmetric.

Windows → Kali:

```
Successful
```

Kali → Windows:

```
Failed
```

---

# SOC Troubleshooting Methodology

The investigation followed:

```
Observe symptom
        ↓
Eliminate possible causes
        ↓
Form hypothesis
        ↓
Collect evidence
        ↓
Verify hypothesis
        ↓
Apply fix
        ↓
Retest
```

This workflow is more important than memorizing commands.

---

# Firewall Investigation

## Hypothesis

Windows Firewall was blocking inbound ICMP Echo Requests.

Command:

```cmd
netsh advfirewall show allprofiles
```

Important output:

```
Firewall Policy:
BlockInbound,AllowOutbound
```

Meaning:

Inbound traffic:

* Blocked by default

Outbound traffic:

* Allowed by default

However, this alone does not prove ICMP is blocked.

The specific inbound rule must be checked.

---

# Windows Firewall Rule Analysis

Checked rule:

```
File and Printer Sharing (Echo Request - ICMPv4-In)
```

Finding:

The inbound ICMP rule was disabled.

Correct fix:

Enabled:

```
File and Printer Sharing
(Echo Request - ICMPv4-In)
```

Why?

Disabling the complete firewall would be poor security practice.

SOC analysts should modify only the required security control.

---

# Retesting Connectivity

Command:

```bash
ping -c 4 192.168.16.131
```

Result:

```
4 packets transmitted
4 received
0% packet loss
```

Communication restored.

---

# Understanding Ping Output

Example:

```
64 bytes from 192.168.16.131:
icmp_seq=1 ttl=128 time=1.42 ms
```

## ICMP Sequence Number

Example:

```
icmp_seq=1
icmp_seq=2
icmp_seq=3
icmp_seq=4
```

Purpose:

Shows packet order.

Missing sequence numbers indicate packet loss.

---

## TTL (Time To Live)

TTL is a hop counter.

Each router decreases TTL by 1.

Typical values:

Windows:

```
128
```

Linux:

```
64
```

SOC relevance:

TTL values can help identify operating systems during packet analysis.

---

## Latency

Example:

```
time=1.42 ms
```

Shows round-trip communication delay.

---

## Ping Statistics

Example:

```
rtt min/avg/max/mdev
```

| Field | Meaning           |
| ----- | ----------------- |
| min   | Fastest response  |
| avg   | Average latency   |
| max   | Slowest response  |
| mdev  | Latency variation |

Low variation indicates stable communication.

---

# Day 2 Environment Verification

After reopening the lab:

Verify again:

Windows:

```cmd
ipconfig
```

Kali:

```bash
ip addr
```

Reason:

DHCP leases can change.

Never assume yesterday's environment is identical.

---

# Destination Host Unreachable Investigation

Initial Kali output:

```
Destination Host Unreachable
```

Possible causes:

* Windows VM asleep
* Network adapter inactive
* VM networking issue
* ARP resolution failure

After waking Windows, communication worked.

Lesson:

Always investigate the actual state before changing configurations.

---

# ARP (Address Resolution Protocol)

## Purpose

ARP resolves:

```
IP Address
     ↓
MAC Address
```

Example:

```
192.168.16.131

↓

00:0c:29:0f:3d:7d
```

ARP operates within the local network.

---

# Why MAC Address Is Required

IP addresses operate at Layer 3.

MAC addresses operate at Layer 2.

Ethernet frames require:

```
Destination MAC Address
```

A network interface cannot deliver frames using only an IP address.

---

# IP vs MAC

## IP Address

* Logical address
* Identifies destination device/network

## MAC Address

* Layer 2 hardware address
* Identifies network interface on the local network

---

# ARP Communication Process

Kali wants to contact:

```
Windows IP:
192.168.16.131
```

But does not know the MAC address.

## Step 1: ARP Request

Broadcast:

```
Who has 192.168.16.131?
Tell 192.168.16.130
```

Destination MAC:

```
FF:FF:FF:FF:FF:FF
```

---

## Step 2: ARP Reply

Windows responds:

```
192.168.16.131 is at 00:0c:29:0f:3d:7d
```

---

## Step 3: ARP Cache Update

Kali stores:

```
192.168.16.131
        ↓
00:0c:29:0f:3d:7d
```

Now Ethernet frames can be sent.

---

# ARP Cache Investigation

Commands:

```bash
arp -a
```

or:

```bash
ip neigh
```

Initial entry:

```
192.168.16.2
00:50:56:f1:47:3a
```

This was VMware's NAT gateway.

Important:

ARP cache is not a complete network inventory.

It contains:

* Recently contacted devices
* Dynamic mappings
* Validated neighbors

Entries expire automatically.

---

# ip neigh States

Example:

```
192.168.16.131 REACHABLE
```

Meaning:

The device was recently confirmed reachable.

Example:

```
192.168.16.2 STALE
```

Meaning:

The entry exists but has not recently been verified.

STALE does not mean broken.

---

# Clearing ARP Cache

Command:

```bash
sudo ip neigh del 192.168.16.131 dev eth0
```

Purpose:

* Removes existing MAC mapping
* Forces fresh ARP resolution

Possible output:

```
RTNETLINK answers:
No such file or directory
```

Meaning:

No ARP entry existed to delete.

Not a critical error.

---

# Wireshark Packet Capture

Interface:

```
eth0
```

Traffic generated:

```bash
ping -c 1 192.168.16.131
```

Filter:

```
arp || icmp
```

Displays:

* ARP packets
* ICMP packets

---

# Wireshark ARP Analysis

## ARP Request

Captured:

```
Who has 192.168.16.131?
Tell 192.168.16.130
```

Analysis:

Sender:

```
Kali Linux
192.168.16.130
```

Target:

```
Windows
192.168.16.131
```

Destination:

```
Broadcast
```

Reason:

Kali did not know Windows' MAC address.

---

## ARP Reply

Captured:

```
192.168.16.131 is at 00:0c:29:0f:3d:7d
```

Windows provided:

* IP address
* MAC address

Kali could now communicate at Layer 2.

---

# ICMP Analysis

Captured:

| Packet Type       | Count |
| ----------------- | ----- |
| ICMP Echo Request | 4     |
| ICMP Echo Reply   | 4     |

Flow:

```
Kali
192.168.16.130

ICMP Echo Request
        ↓

Windows
192.168.16.131

ICMP Echo Reply
        ↓

Kali
```

---

# VMware Network Components Observed

ARP table contained:

Gateway:

```
192.168.16.2
```

Another VMware component:

```
192.168.16.254
```

These are VMware virtual networking components.

---

# Commands Practiced

## Windows

```cmd
ipconfig
```

```cmd
netsh advfirewall show allprofiles
```

```cmd
ping 192.168.16.130
```

## Kali Linux

```bash
ip addr
```

```bash
ping -c 4 192.168.16.131
```

```bash
arp -a
```

```bash
ip neigh
```

```bash
sudo ip neigh del <IP> dev eth0
```

```bash
wireshark
```

```bash
# Check VMware shared folder (not part of networking lab, so exclude from Lab 1 notes)
vmware-hgfsclient
```

```bash
# Creates the mount point directory where VMware Shared Folders will be accessed.
# /mnt/hgfs is the standard location used for VMware shared folder mounts.
sudo mkdir -p /mnt/hgfs


# Mounts VMware Shared Folders from the host machine into Kali Linux.
# .host:/ tells Kali to access shared folders configured in VMware.
# /mnt/hgfs is where those folders become accessible.
# -o allow_other allows other users/processes to access the mounted folder.
sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other


# Verifies that the VMware Shared Folder was successfully mounted.
# Output showed "Snapshots", confirming that the host folder was accessible from Kali.
ls /mnt/hgfs/


# Copies the screenshot from Kali's local Pictures directory
# into the VMware Shared Folder named Snapshots.
# This transferred the file from the Kali VM to the host PC folder.
cp ~/Pictures/wireshark-arp-icmp-analysis.png /mnt/hgfs/Snapshots/
```

---

# Interview Questions

## What is ARP?

ARP maps IPv4 addresses to MAC addresses within the local network.

## Why is ARP required?

Because Ethernet frames require destination MAC addresses.

## Difference between IP and MAC?

IP identifies the destination logically.

MAC identifies the local network interface.

## Why did Kali initially fail to ping Windows?

Windows Firewall blocked inbound ICMP Echo Requests.

## Why did Windows ping Kali successfully?

Outbound traffic was allowed by default.

## What does TTL indicate?

TTL indicates the maximum number of router hops before a packet is discarded.

## What is ARP spoofing?

ARP spoofing is an attack where false ARP mappings associate an attacker MAC address with another IP address.

Possible impacts:

* Man-in-the-middle attacks
* Traffic interception
* Session hijacking

---

# SOC Relevance

This lab developed skills required for:

* Network troubleshooting
* Packet analysis
* Firewall investigation
* ARP spoofing detection
* ICMP reconnaissance analysis
* SIEM network alert investigation

---

# Final Lab Takeaways

Completed:

✅ Verified Windows and Kali network configuration
✅ Understood VMware NAT networking
✅ Learned subnet basics
✅ Tested connectivity using ICMP
✅ Investigated firewall-related connectivity failure
✅ Followed SOC troubleshooting methodology
✅ Learned ARP IP-to-MAC resolution
✅ Analyzed ARP request and reply packets
✅ Used Wireshark for packet analysis
✅ Interpreted ICMP traffic


# Lab 2 Notes: Network Discovery & Enumeration with Nmap

> **Objective:** Learn how to discover hosts, enumerate services, fingerprint operating systems, and interpret Nmap results from a SOC Level 1 analyst's perspective.

---

# What is Nmap?

Nmap (Network Mapper) is a network reconnaissance tool used to discover what a **remote machine exposes** to the network.

It helps answer questions such as:

- Which hosts are alive?
- Which ports are open?
- Which services are listening?
- Which operating system is likely running?

### Important

Nmap shows the **outside view** of a target.

It **does not** show:

- Running processes
- Outbound connections
- Which application owns a connection
- Internal system information

For those, endpoint tools such as **netstat**, **PowerShell**, **Task Manager**, or an **EDR** are used.

---

# Mental Model

Think of a computer as an apartment building.

- Computer = Building
- Ports = Apartment doors
- Services = People inside the apartments
- Nmap = Someone knocking on every door to see who answers

Nmap only discovers services **waiting for incoming connections**.

---

# Host Discovery

Command:

```bash
nmap -sn 192.168.16.0/24
```

## Purpose

Discover live hosts without scanning ports.

The `-sn` option performs **Ping Scan (Host Discovery)**.

It identifies:

- Live devices
- Their IP addresses
- MAC addresses (on local networks)
- Vendor information (when available)

---

## Hosts Found in Lab

| IP Address | Device |
|------------|---------|
|192.168.16.1|VMware Virtual Network|
|192.168.16.2|VMware NAT Gateway|
|192.168.16.130|Kali Linux|
|192.168.16.131|Windows 11 VM|
|192.168.16.254|VMware DHCP Server|

### SOC Insight

Vendor information helps identify:

- VMware
- Cisco
- Dell
- HP
- Apple

This can quickly reveal virtual infrastructure during investigations.

---

# Default Port Scan

Command

```bash
nmap 192.168.16.131
```

## Purpose

Scans the **top 1000 most common TCP ports**.

Fast reconnaissance.

---

## Open Ports Found

| Port | Service | Purpose | SOC Importance |
|------|----------|----------|---------------|
|135|MSRPC|Remote Procedure Call|Core Windows service used for remote management. Frequently seen in enterprise environments.|
|139|NetBIOS Session Service|Legacy Windows file/printer sharing|Older protocol largely replaced by SMB over TCP, but still encountered.|
|445|SMB (Microsoft-DS)|Windows file sharing|Very high-value attack target. Used in lateral movement, ransomware, pass-the-hash, and EternalBlue-style attacks.|
|5357|HTTPAPI / WSDAPI|Web Services for Devices|Used for device discovery and management in Windows.|

---

# Open vs Closed vs Filtered Ports

### Open

A service is actively listening.

Example:

```
445/tcp open microsoft-ds
```

The machine accepted the connection.

---

### Closed

No service is listening.

The machine replies saying:

> "Nothing is running here."

---

### Filtered

The firewall silently discards packets.

No reply is received.

Example:

```
996 filtered tcp ports
```

Meaning:

Nmap sent probes.

Windows Firewall dropped them.

Nmap therefore labels them as **filtered**.

---

# Why Port 445 Matters

SMB (Server Message Block)

Purpose:

- File sharing
- Printer sharing
- Windows administration

Common attack vectors:

- WannaCry
- EternalBlue
- Pass-the-Hash
- Lateral Movement
- SMB Brute Force

### SOC Relevance

Monitor SMB for:

- Excessive authentication failures
- Unusual file access
- Lateral movement
- Suspicious SMB sessions

---

# Service Fingerprinting

Command

```bash
nmap -sV 192.168.16.131
```

## Purpose

Identify the software running behind open ports.

Instead of only reporting:

```
135 open
```

Nmap attempts to identify:

```
Microsoft Windows RPC
```

---

## Services Identified

| Port | Service |
|------|----------|
|135|Microsoft Windows RPC|
|139|Microsoft Windows NetBIOS|
|5357|Microsoft HTTPAPI 2.0|

---

## Why SMB Version Wasn't Identified

Output:

```
445/tcp open microsoft-ds?
```

The `?` means Nmap couldn't confidently identify the exact SMB version.

Possible reasons:

- Firewall restrictions
- Limited fingerprint information
- SMB negotiation requires additional communication
- Version intentionally hidden

### SOC Takeaway

Even without the exact version, knowing **SMB exists** is already valuable during investigations.

---

# Unknown Services

Full scan discovered:

```
5040/tcp
49668/tcp
```

shown as:

```
unknown
```

This **does not** mean the ports are unknown.

It means Nmap couldn't determine **which application** was listening.

Possible reasons:

- Custom application
- Rare service
- Service hides its identity
- No matching fingerprint

---

## How SOC Analysts Identify Unknown Services

Nmap provides an external view.

Endpoint tools provide the answer.

Examples:

Windows

```cmd
netstat -abno
```

or

```powershell
Get-NetTCPConnection
```

These reveal:

- Process name
- PID
- Local port
- Remote connection
- Listening status

In enterprise environments, EDR/SIEM telemetry often provides this information automatically.

---

# Full Port Scan

Command

```bash
nmap -p- 192.168.16.131
```

## Purpose

Scan **all 65,535 TCP ports**.

---

## Default vs Full Scan

### Default Scan

- Top 1000 TCP ports
- Faster
- May miss uncommon services

---

### Full Scan

- All 65,535 ports
- Slower
- Finds uncommon services

### SOC Insight

Use:

- Default scan for quick reconnaissance.
- Full scan when deeper enumeration is required.

---

# OS Fingerprinting

Command

```bash
sudo nmap -O 192.168.16.131
```

## Purpose

Infer the operating system by analyzing TCP/IP behavior.

Nmap compares responses to a fingerprint database.

It **does not** read the OS directly.

---

## Why Multiple Windows Versions Appeared

Example:

- Windows 11
- Windows 10
- Windows Server

Many Microsoft operating systems behave similarly on the network.

Nmap therefore reports confidence scores.

Example:

```
Windows 11 (96%)
```

Meaning:

Windows 11 is the **best match**, not a certainty.

---

## Warning Explained

```
OSScan results may be unreliable...
```

For accurate fingerprinting, Nmap prefers:

- At least one open port
- At least one closed port

Windows Firewall filtered most ports instead of replying "closed."

Less information = less confidence.

---

# Network Distance

Output:

```
1 hop
```

Meaning:

Target is one network device away.

Example:

```
Kali
↓

VMware NAT

↓

Windows
```

---

# Aggressive Scan

Command

```bash
sudo nmap -A 192.168.16.131
```

## Includes

- Service Detection
- Version Detection
- OS Detection
- Default NSE Scripts
- Traceroute

---

## Additional Information Found

- NetBIOS hostname
- SMB security mode
- HTTP headers
- Traceroute
- SMB time

Example:

```
Hostname:
WIN11_SOC
```

Example:

```
SMB 3.1.1

Message signing enabled and required
```

### SOC Insight

Aggressive scans provide richer information but generate significantly more traffic and are more detectable.

Use only on authorized systems.

---

# Nmap vs Endpoint Tools

## Nmap

Shows:

- Open ports
- Listening services
- Service versions
- OS guess

Perspective:

**Outside the machine**

---

## Endpoint Tools

Examples:

```cmd
netstat -abno
```

Show:

- Running processes
- Outbound connections
- Listening ports
- PIDs
- Established sessions

Perspective:

**Inside the machine**

---

# Listening Services vs Outbound Connections

This distinction is critical.

### Nmap Finds

Services waiting for incoming connections.

Example:

```
Windows VM

445 (SMB)

Listening
```

---

### Nmap Does NOT Show

Example:

```
Edge

↓

NetMirror Website
```

Because Edge is the **client**, not the server.

The listening service belongs to the **NetMirror server**, not your Windows VM.

---

## Simplified Rule

Nmap asks:

> "What are you offering to the network?"

It does **not** ask:

> "What websites are you connected to?"

---

# Common SOC Use Cases

Nmap helps with:

- Asset discovery
- Attack surface mapping
- Service identification
- Exposure assessment
- Security validation
- Investigation support

---

# Interview Nuggets

- Nmap discovers **what a remote machine exposes**, not what it consumes.
- Service fingerprinting identifies applications.
- OS fingerprinting estimates the operating system.
- Fingerprinting is an **inference**, not proof.
- Default scans are fast.
- Full scans are comprehensive.
- Aggressive scans generate more network traffic.
- SMB (445) is one of the most targeted Windows services.

---

# Commands Cheat Sheet

Host Discovery

```bash
nmap -sn 192.168.16.0/24
```

Default Scan

```bash
nmap 192.168.16.131
```

Service Detection

```bash
nmap -sV 192.168.16.131
```

Full Scan

```bash
nmap -p- 192.168.16.131
```

OS Detection

```bash
sudo nmap -O 192.168.16.131
```

Aggressive Scan

```bash
sudo nmap -A 192.168.16.131
```

Multiple Host Discovery

```bash
nmap -sn 192.168.16.130 192.168.16.131
```

---

# Key Takeaways

- Nmap provides an **external perspective** of a target.
- Endpoint tools provide an **internal perspective**.
- Ports represent communication endpoints where services listen.
- Service fingerprinting identifies software behind open ports.
- OS fingerprinting estimates the operating system from network behavior.
- Default scans trade depth for speed.
- Full scans trade speed for completeness.
- Understanding **what a host offers** versus **what it consumes** is fundamental to networking and SOC investigations.
