# Lab 1: Network Connectivity Verification & ARP Analysis

## Overview

This lab focused on establishing and verifying network communication between a Windows 11 virtual machine and a Kali Linux virtual machine in a VMware NAT environment. The lab also introduced ARP (Address Resolution Protocol) and ICMP (Internet Control Message Protocol), with packet-level analysis performed using Wireshark.

---

## Lab Objectives

- Verify IP configuration of both virtual machines.
- Test connectivity using ICMP.
- Troubleshoot connectivity issues.
- Understand the relationship between IP and MAC addresses.
- Observe ARP address resolution.
- Analyze ARP and ICMP packets using Wireshark.

---

## Lab Environment

| Component | Details |
|-----------|---------|
| Hypervisor | VMware Workstation Pro 17.6.3 |
| Host OS | Windows 11 |
| Guest OS 1 | Windows 11 VM |
| Guest OS 2 | Kali Linux 2025.2 |
| Network Mode | NAT |
| Tool | Wireshark |

---

## Network Configuration

### Windows VM

| Parameter | Value |
|-----------|-------|
| IPv4 Address | 192.168.16.131 |
| Subnet Mask | 255.255.255.0 (/24) |
| Default Gateway | 192.168.16.2 |

### Kali Linux VM

| Parameter | Value |
|-----------|-------|
| IPv4 Address | 192.168.16.130 |
| Subnet | /24 |
| MAC Address | 00:0c:29:b8:90:4f |

---

## Activities Performed

### 1. Verified Network Configuration

Validated IP addressing using:

- `ipconfig`
- `ip addr`

Confirmed both virtual machines were connected to the same subnet:

```
192.168.16.0/24
```

---

### 2. Tested Connectivity

Performed ICMP communication tests between Windows and Kali.

Command:

```bash
ping -c 4 192.168.16.131
```

Result:

```
4 transmitted
4 received
0% packet loss
```

---

### 3. Troubleshooting

Initial communication from Kali to Windows failed.

Investigation revealed Windows Defender Firewall was blocking inbound ICMP Echo Requests.

Resolution:

Enabled the **File and Printer Sharing (Echo Request - ICMPv4-In)** inbound firewall rule.

Connectivity was successfully restored.

---

### 4. ARP Analysis

Captured ARP traffic in Wireshark.

Observed:

- ARP Request
- ARP Reply

Verified successful mapping of the Windows IP address to its MAC address before ICMP communication.

---

### 5. ICMP Analysis

Captured and analyzed:

- Echo Request
- Echo Reply

Verified successful two-way communication.

---

## Skills Practiced

- Network troubleshooting
- Windows Firewall configuration
- IPv4 addressing
- Subnet verification
- ARP analysis
- ICMP analysis
- Wireshark packet capture
- VMware NAT networking

---

## Tools Used

- VMware Workstation
- Windows Command Prompt
- Kali Linux Terminal
- Wireshark

---

## Commands Used

### Windows

```cmd
ipconfig
ping 192.168.16.130
```

### Kali

```bash
ip addr
ping -c 4 192.168.16.131
ip neigh
```

---

## Screenshots

```
screenshots/
├── windows-ipconfig.png
├── kali-ip-address.png
├── ping-success.png
└── wireshark-arp-icmp-analysis.png
```

---

## Outcome

- Successfully configured communication between Windows and Kali virtual machines.
- Identified and resolved a Windows Firewall issue.
- Understood how ARP resolves IPv4 addresses into MAC addresses.
- Captured and analyzed ARP and ICMP traffic using Wireshark.
- Built foundational networking knowledge required for SOC operations.
