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

