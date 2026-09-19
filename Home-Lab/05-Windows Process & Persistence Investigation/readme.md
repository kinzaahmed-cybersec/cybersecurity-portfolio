# Lab 3 — Windows Process & Persistence Investigation

## Objective

Investigate Windows process creation and basic registry-based persistence from a SOC analyst perspective.

The lab focused on:

* Enabling Windows Process Creation auditing
* Generating controlled process activity
* Investigating **Event ID 4688**
* Analyzing parent → child process relationships
* Reviewing process privilege and elevation
* Creating controlled **RunOnce** persistence
* Correlating persistence with subsequent process execution
* Performing a basic SOC assessment based on context and execution chain

Windows **Event ID 4688** is generated when a new process is created. It provides process, user, parent-process, elevation, and integrity information useful for endpoint investigation.

---

## Lab Environment

| Component             | Configuration                                            |
| --------------------- | -------------------------------------------------------- |
| Host                  | Windows 11 VM                                            |
| Hostname              | `WIN11_SOC`                                              |
| User                  | `kinza`                                                  |
| Primary Log           | Windows Security Event Log                               |
| Primary Event         | Event ID `4688`                                          |
| Persistence Mechanism | `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce` |

---

## 1. Enable Process Creation Auditing

Process Creation auditing was enabled through:

**Local Security Policy → Advanced Audit Policy Configuration → System Audit Policies → Detailed Tracking → Audit Process Creation**

This enabled Windows to generate **Event ID 4688** whenever a new process was created.

---

## 2. Generate Controlled Process Activity

Controlled processes were generated to establish known-good execution examples.

Examples included:

* `powershell.exe`
* `notepad.exe`
* `cmd.exe`

The purpose was to compare normal process execution with different parent processes and privilege levels.

---

## 3. Investigate Event ID 4688

### PowerShell Execution

A controlled PowerShell execution produced a 4688 event showing:

* **New Process:** `powershell.exe`
* **User:** `WIN11_SOC\kinza`
* **Parent Process:** `explorer.exe`
* **Token Elevation Type:** Type 3, Limited
* **Mandatory Label:** Medium
* **Command Line:** Empty

The PowerShell process was assessed as legitimate because it was intentionally launched by the user through normal interactive activity.

PowerShell itself should not automatically be classified as malicious. Its legitimacy depends on context such as the initiating user, parent process, command line, privilege level, and surrounding activity.

### Command-Line Limitation

The `Process Command Line` field was empty in the collected 4688 events.

This is expected when command-line auditing has not been separately enabled. Microsoft documents that the **Include command line in process creation events** policy must be enabled to populate command-line information in 4688 events.

---

## 4. Parent → Child Process Analysis

A controlled process chain was created:

```text
PowerShell
    ↓
Notepad
```

The resulting 4688 event showed:

```text
Parent Process:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

New Process:
...\Notepad\Notepad.exe
```

The process was executed under the expected user context with:

* Token Elevation Type: Type 3, Limited
* Mandatory Integrity Level: Medium

### SOC Interpretation

Parent-child relationships are important because they provide execution context.

A process should not be considered malicious solely because of its name. Analysts should investigate the complete execution chain, including:

* Parent process
* Child process
* User
* Command line
* Privilege/elevation
* Integrity level
* Timing
* Surrounding activity

In this controlled case, **PowerShell → Notepad** was legitimate because it was intentionally generated during the investigation.

---

## 5. Privilege & Elevation Analysis

An elevated Command Prompt was launched using **Run as administrator**.

The resulting 4688 event was compared with the normal PowerShell execution.

The investigation focused on:

* **Token Elevation Type**
* **Mandatory Integrity Level**
* User context
* Parent process

The normal PowerShell execution used:

```text
Token Elevation Type: Type 3 - Limited
Mandatory Label: Medium
```

An administrator-launched process provides evidence of elevated execution, demonstrating why privilege level is an important part of process investigation.

### SOC Interpretation

Elevated execution is **not automatically malicious**.

The analyst should determine:

> Who launched the process → what process launched it → with what privileges → for what purpose → in what surrounding context?

Microsoft documents Token Elevation Type and Mandatory Label as useful fields for assessing process privilege and integrity.

---

## 6. Controlled RunOnce Persistence

A controlled persistence mechanism was created using:

```text
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

A string value was created:

```text
Name:
SOC_Lab_Test

Value:
notepad.exe
```

This configuration was intentionally created for the lab to demonstrate how registry-based persistence can result in process execution during a subsequent logon.

### RunOnce Behavior

After restarting the Windows VM and logging back in:

* Notepad launched automatically.
* The corresponding process creation was recorded as **Event ID 4688**.
* The `SOC_Lab_Test` RunOnce entry was no longer present afterward.

This demonstrated the expected one-time execution behavior of the `RunOnce` mechanism.

---

## 7. Persistence → Process Correlation

The persistence mechanism was correlated with the resulting process execution:

```text
RunOnce Registry Entry
        ↓
User Logon / Startup
        ↓
explorer.exe
        ↓
notepad.exe
        ↓
Event ID 4688
```

The resulting 4688 event showed:

* **User:** `kinza`
* **New Process:** `Notepad.exe`
* **Parent Process:** `explorer.exe`
* **Token Elevation Type:** Type 3, Limited
* **Mandatory Integrity Level:** Medium
* **Command Line:** Empty

The process execution was considered legitimate because the persistence mechanism had been intentionally created as part of the controlled lab.

### SOC Assessment

The key lesson is that persistence investigation requires **correlation**, not simply identifying a suspicious registry key.

A SOC analyst can investigate:

1. **What persistence mechanism exists?**
2. **What executable is configured to run?**
3. **When does it execute?**
4. **Which user context executes it?**
5. **What process starts it?**
6. **What privilege level does it receive?**
7. **Does the resulting activity match expected behavior?**

A registry persistence mechanism combined with unexpected process execution would warrant further investigation.

---

## Key SOC Takeaways

* **4688** records new process creation.
* Process names alone are insufficient to determine maliciousness.
* Parent → child relationships provide important execution context.
* PowerShell is legitimate administrative tooling but can also be abused, so context matters.
* **Token Elevation Type** and **Mandatory Integrity Level** help determine process privilege.
* Registry **Run/RunOnce** mechanisms can be abused for persistence.
* Persistence should be correlated with subsequent process activity.
* A strong SOC assessment combines **user + process + parent + privilege + timing + persistence + surrounding activity**.
* Missing command-line information can limit investigation and should be recognized as a telemetry gap. Microsoft notes that command-line auditing requires a separate policy configuration.

---

## Evidence / Screenshots

1. `Registry_runonce_key_entry.png`
2. `4688_notepad.exe_process_execution.png`
