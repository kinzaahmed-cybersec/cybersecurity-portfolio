# Lab 3: Windows Authentication Investigation

## Objective

Investigate Windows authentication and logon activity using the Windows Security Event Log and understand how a SOC analyst distinguishes normal system activity from potentially suspicious authentication events.

## Environment

- Windows 11 VM (`WIN11_SOC`) - Target
- Kali Linux - Future attacker machine
- Wireshark - Network visibility
- Windows Event Viewer - Primary log source

## Key Windows Security Events

| Event ID | Meaning |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4634 | Logoff |
| 4648 | Explicit credentials used |

---

## Investigation

### 1. Event ID 4624 - Successful Logon

Multiple successful logon events were examined.

An important observation was that **4624 does not necessarily mean a human user logged into Windows**.

Examples observed:

- `SYSTEM`
- `DWM-0`
- `DWM-1`
- `UMFD-0`
- `UMFD-1`
- `NETWORK SERVICE`

### Example: Logon Type 5

One event contained:

- Event ID: `4624`
- Logon Type: `5`
- Account: `SYSTEM`
- Process: `services.exe`
- Source Network Address: `-`

#### Interpretation

Logon Type 5 represents a service logon. Combined with `SYSTEM` and `services.exe`, this was consistent with normal Windows service activity rather than a human login.

### Important Lesson

A SOC analyst should not classify an event based only on its Event ID. The surrounding context is essential.

---

## 2. Event ID 4625 - Failed Logon

A failed interactive logon was observed.

Key fields:

- Event ID: `4625`
- Logon Type: `2`
- Account: Unknown (`-`)
- Caller Process: `svchost.exe`
- Source Address: `127.0.0.1`
- Logon Process: `User32`
- Authentication Package: `Negotiate`

### Interpretation

The system recorded a failed interactive logon attempt originating from the local host.

`127.0.0.1` is the loopback address, meaning the source was the same computer.

The event alone does not establish malicious activity. Additional context such as process behavior, timing, frequency, and related events would be required.

---

## 3. Event ID 4634 - Successful Logoff

A genuine local user logoff was identified.

Key fields:

- Event ID: `4634`
- Account: `WIN11_SOC\kinza`
- Logon Type: `2`
- Logon ID: `0x11A6272`

### Interpretation

This event represents the destruction of a local interactive logon session.

The Logon ID can be used to correlate the logoff with the corresponding authentication event.

### SOC Observation

This was the clearest example in the investigation of a human user's actual Windows session.

---

## 4. Event ID 4648 - Explicit Credentials

An explicit credential-use event was observed.

Key fields:

- Event ID: `4648`
- Account whose credentials were used: `kinza.cyberlab@outlook.com`
- Account Domain: `MicrosoftAccount`
- Target Server: `localhost`
- Process: `lsass.exe`
- Network Address: `-`

### Interpretation

Windows recorded an authentication attempt in which credentials were explicitly specified.

This is **not automatically malicious**. Legitimate Windows operations, applications, scheduled tasks, or commands can generate this event.

However, unexpected 4648 events can be useful for detecting credential misuse and should be investigated in context.

---

# SOC Analysis Lessons

### 1. Event ID is only the starting point

A `4624` event means successful logon, but it does not tell us by itself whether a human, service, or Windows component created the session.

### 2. Logon Type provides important context

Examples observed:

- Type `2` - Interactive
- Type `5` - Service

However, Type 2 alone should not automatically be interpreted as a human login because Windows can create interactive-type sessions for internal components.

### 3. Process context matters

Examples:

- `services.exe` → strongly supported service-related activity
- `winlogon.exe` → Windows logon/session management
- `svchost.exe` → Windows service-hosting process
- `lsass.exe` → Windows security/authentication subsystem

### 4. Network information helps determine origin

- `127.0.0.1` → local host / loopback
- `-` → no network address recorded

### 5. Suspicious does not mean malicious

A SOC analyst should investigate unusual authentication activity rather than immediately labeling it an attack.

---

# Key Takeaway

The main lesson from this lab was:

> **Authentication events must be interpreted using context, not Event ID alone.**

A SOC analyst should examine:

**Account → Logon Type → Process → Source Address → Target → Timestamp → Related Events**

before determining whether authentication activity is normal or suspicious.

## Skills Practiced

- Windows Event Viewer
- Windows Security Logs
- Event ID analysis
- Authentication investigation
- Logon Type analysis
- Process context analysis
- Source IP interpretation
- Event correlation
- Basic SOC triage

- ---

## Evidence / Screenshots

### Event ID 4624 - Successful Logon

![Event ID 4624](./Event%20ID%204624.png)

### Event ID 4625 - Failed Logon

![Event ID 4625](./Event%20ID%204625.png)

### Event ID 4634 - Successful Logoff

![Event ID 4634](./Event%20ID%204634.png)

### Event ID 4648 - Explicit Credential Use

![Event ID 4648](./Event%20ID%204648.png)
