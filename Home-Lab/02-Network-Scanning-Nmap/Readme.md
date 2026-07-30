# Lab 2: Network Scanning and Enumeration with Nmap

## Overview

This lab focuses on network reconnaissance using Nmap. The objective was to discover live hosts, identify open ports, fingerprint services, infer the operating system, and understand how different scan types reveal different levels of information.

These skills form an essential part of SOC investigations, vulnerability assessments, and asset discovery.

---

# Lab Objectives

- Discover live hosts on the local network
- Identify open TCP ports
- Perform service version detection
- Perform operating system fingerprinting
- Compare default and full port scans
- Understand aggressive scanning and its components
- Interpret Nmap scan results from a SOC analyst's perspective

---

# Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation |
| Attacker / Analyst | Kali Linux |
| Target | Windows 11 VM |
| Network | VMware NAT |

---

# Step 1: Host Discovery

Command:

```bash
nmap -sn 192.168.16.0/24
```

### Purpose

Identify active hosts without performing port scans.

### Hosts Discovered

| IP Address | Description |
|---|---|
| 192.168.16.1 | VMware Virtual Network |
| 192.168.16.2 | VMware NAT Gateway |
| 192.168.16.130 | Kali Linux |
| 192.168.16.131 | Windows 11 VM |
| 192.168.16.254 | VMware DHCP |

---

# Step 2: Default TCP Scan

Command

```bash
nmap 192.168.16.131
```

Open Ports

| Port | Service |
|---|---|
|135|MSRPC|
|139|NetBIOS|
|445|Microsoft-DS (SMB)|
|5357|WSDAPI|

Observation

- 996 TCP ports were filtered.
- Windows Firewall silently dropped probes to those ports.

---

# Step 3: Service Version Detection

Command

```bash
nmap -sV 192.168.16.131
```

Additional Information Obtained

- Microsoft Windows RPC
- Microsoft Windows NetBIOS
- Microsoft HTTPAPI 2.0
- Service fingerprinting inferred Windows operating system

---

# Step 4: Full TCP Port Scan

Command

```bash
nmap -p- 192.168.16.131
```

Additional Ports Discovered

| Port | Service |
|---|---|
|5040|Unknown|
|49668|Unknown|

Observation

The default Nmap scan checks only the 1,000 most common TCP ports. A full scan examines all 65,535 TCP ports, revealing additional exposed services.

---

# Step 5: Operating System Fingerprinting

Command

```bash
sudo nmap -O 192.168.16.131
```

Result

- Likely Operating System:
    - Microsoft Windows 11 (96%)
- Network Distance:
    - 1 hop

Observation

Nmap inferred the operating system using TCP/IP fingerprinting rather than reading it directly.

---

# Step 6: Aggressive Scan

Command

```bash
sudo nmap -A 192.168.16.131
```

Additional Information Collected

- HTTP Server Header
- SMB Security Mode
- NetBIOS Hostname
- SMB Time
- Traceroute
- Enhanced OS Fingerprinting

Important Findings

NetBIOS Hostname

```
WIN11_SOC
```

SMB Version

```
SMB 3.1.1
```

Security

```
Message signing enabled and required
```

---

# Key Concepts Learned

## Host Discovery

Uses ICMP/ARP and other techniques to determine whether hosts are alive.

---

## Port Scanning

Determines which TCP ports are listening for incoming connections.

---

## Service Fingerprinting

Identifies applications listening behind open ports.

---

## OS Fingerprinting

Infers the operating system by analyzing responses to specially crafted packets.

---

## Default vs Full Scan

Default Scan

- Top 1000 TCP ports
- Faster

Full Scan

- All 65535 TCP ports
- More comprehensive
- Slower

---

## Aggressive Scan

Combines:

- Service Detection
- OS Detection
- Default NSE Scripts
- Traceroute

Produces significantly more information but is much noisier.

---

# SOC Analyst Relevance

Nmap assists SOC analysts in:

- Asset discovery
- Exposure assessment
- Network reconnaissance
- Attack surface analysis
- Validation of security configurations
- Incident investigations

---

# Commands Used

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

Full Port Scan

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

---

# Screenshots

```
screenshots/
│
├── host-discovery.png
├── default-scan.png
├── service-detection.png
├── full-port-scan.png
├── os-detection.png
└── aggressive-scan.png
```

---

# Lab Outcome

✅ Identified live hosts

✅ Enumerated open TCP ports

✅ Performed service version detection

✅ Performed operating system fingerprinting

✅ Compared default and full port scans

✅ Collected additional information using aggressive scanning

---

# Conclusion

This lab introduced practical network reconnaissance using Nmap. Multiple scan techniques were compared to understand how different levels of enumeration reveal progressively more information about a target system. These techniques form a foundational skillset for SOC analysts during network investigations and security assessments.
