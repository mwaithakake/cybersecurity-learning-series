# Incident Investigation Report: Windows Host-Centric Attack Analysis

**Target Host:** `Martha` (`Martha.cyberlab.local`)  
**Investigation Date:** October 5, 2026  
**Analyst:** Security Operations  
**Scope:** Host-based investigation of a controlled multi-stage attack simulation on a domain-joined Windows 11 workstation.

---

## 1. Executive Summary

A controlled attack simulation was conducted on the Windows 11 workstation `Martha` to evaluate host-level visibility and investigative capability.

The simulation generated several attacker-like activities, including system discovery, encoded PowerShell execution, file staging and concealment, and registry-based persistence. Sysmon and PowerShell Script Block Logging were used to collect and correlate evidence across the different stages.

The investigation successfully reconstructed the activity using process, file, registry, and PowerShell telemetry.

---

## 2. Environment & Telemetry

**Target Host:** `Martha.cyberlab.local`  
**Operating System:** Windows 11  
**Domain:** `cyberlab.local`  
**Investigation Account:** `CYBERLAB\Administrator`

### Telemetry Sources

* Windows Security Logs
* Sysmon
* PowerShell Script Block Logging
* Windows Event Viewer

Key telemetry used during the investigation:

* **Sysmon Event ID 1** — Process Creation
* **Sysmon Event ID 11** — File Creation
* **Sysmon Event ID 13** — Registry Value Set
* **PowerShell Event ID 4104** — Script Block Logging

### Baseline

Before the simulation, the workstation was checked for existing local accounts and running processes using:

```powershell
Get-LocalUser
Get-Process | Select-Object Name, Id

```

**Evidence:** `discovery.png`
<img width="1851" height="517" alt="discovery" src="https://github.com/user-attachments/assets/83b5f968-0913-4b82-9f61-1d14e88c86e9" />


This established a basic reference point for the host before the simulated activity occurred.

---

## 3. Incident Timeline

| Time (EAT) | Activity | Evidence | Finding |
| --- | --- | --- | --- |
| **10:33:50** | Encoded PowerShell execution | Sysmon ID 1, PowerShell ID 4104 | PowerShell executed using `-EncodedCommand`; the underlying script was captured by Script Block Logging. |
| **10:51:45** | File staging | Sysmon ID 11 | `secrets.txt` was created in `C:\Users\Public\AuditTemp\`. |
| **10:51:45** | File concealment | Sysmon ID 1 | `attrib.exe` applied the Hidden attribute to the staged file. |
| **10:52:52** | Registry persistence | Sysmon ID 1, ID 13 | `LabCheckKey` was added to the user's Registry Run location. |

---

## 4. Investigation Findings

### 4.1 Encoded PowerShell Execution

The simulation generated a Base64-encoded PowerShell command containing:

```powershell
Write-Output 'InvestigationLabActive'; Get-Service

```

The command was executed using PowerShell's `-EncodedCommand` parameter.

**Evidence:**
* `powershell_encoding.png` — encoded command generation and execution
<img width="1295" height="452" alt="powershell_encoding" src="https://github.com/user-attachments/assets/8fdf674e-a398-4e24-a709-d1d705759d3c" />

* `Screenshot 2026-10-05 114038.png` — Sysmon Process Creation event
  <img width="1305" height="361" alt="Screenshot 2026-10-05 114038" src="https://github.com/user-attachments/assets/d0eeb075-8951-486a-a146-fc2fe8e02393" />

* `powershell_encoding2.png` — PowerShell Event ID 4104
  <img width="1298" height="811" alt="powershell_encoding2" src="https://github.com/user-attachments/assets/b2254e2c-db61-4565-ba42-5b0486e723d5" />


Sysmon Event ID 1 recorded `powershell.exe` executing with the `-EncodedCommand` parameter.

PowerShell Event ID 4104 subsequently captured the underlying script:

```powershell
Write-Output 'InvestigationLabActive'; Get-Service

```

**Analysis:** Encoding can make PowerShell commands less obvious to simple string-based detection, but it does not make the activity invisible. Script Block Logging provided visibility into the underlying command.

**Finding:** Encoded PowerShell execution was successfully detected and the underlying command was recovered through Event ID 4104.

---

### 4.2 File Staging & Concealment

The simulation created a staging directory and a file representing collected data:

```text
C:\Users\Public\AuditTemp\secrets.txt

```

The file contained simulated data rather than real sensitive information.

The file was then hidden using:

```cmd
attrib +h C:\Users\Public\AuditTemp\secrets.txt

```

**Evidence:**

* `file staging.png` — staging and concealment commands
* <img width="1848" height="406" alt="file staging" src="https://github.com/user-attachments/assets/a28b3168-7dcc-4ab3-bcf2-132c7d5164cb" />

* `filestaging.png` — Sysmon Event ID 11 showing file creation
* <img width="1300" height="785" alt="filestaging" src="https://github.com/user-attachments/assets/34e5bff3-109b-4e59-8f91-69e2b39f4527" />

* `registrykeys.png` — Sysmon Event ID 1 showing `attrib.exe` execution
  <img width="1846" height="430" alt="registrykeys" src="https://github.com/user-attachments/assets/f5a39b50-73ab-4fb4-97b0-8dc5c94ddcff" />


Sysmon Event ID 11 confirmed creation of `secrets.txt`, while Event ID 1 captured execution of `attrib.exe` against the file.

**Analysis:** The sequence simulates an attacker staging and concealing a file before potential further activity. The evidence confirms file creation and concealment but does not demonstrate actual data theft or exfiltration.

**Finding:** A staged file was created and subsequently hidden using `attrib.exe`.

---

### 4.3 Registry Persistence

The simulation modified the current user's Registry Run location:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run

```

A value named `LabCheckKey` was created to execute:

```text
powershell.exe -WindowStyle Hidden -Command Write-Output 'Persisted'

```

**Evidence:**

* `registry1.png` — Sysmon Event ID 1 showing `reg.exe`
  <img width="1272" height="772" alt="registry1" src="https://github.com/user-attachments/assets/5f4e1296-c2aa-4d12-84e9-32d5bb322b10" />

* `registry2.png` — Sysmon Event ID 13 showing the registry modification
  <img width="1226" height="788" alt="registry2" src="https://github.com/user-attachments/assets/c8c96901-27f2-408d-b820-71d8b29d871d" />


Sysmon Event ID 13 recorded the creation of the `LabCheckKey` value under the user's Run key.

The Sysmon configuration identifies this activity using the older `T1060` label. The current MITRE ATT&CK technique is **T1547.001 — Registry Run Keys / Startup Folder**.

**Analysis:** Registry Run keys can be used to establish user-level persistence by causing a program or command to execute when the user logs in.

**Finding:** A PowerShell command was configured for execution through the user's Registry Run key.

---

## 5. Investigation Conclusion

The investigation reconstructed the controlled activity on workstation `Martha` across three primary stages:

```text
Encoded PowerShell Execution
          ↓
File Staging & Concealment
          ↓
Registry Persistence

```

Multiple telemetry sources were successfully correlated:

* **Sysmon Event ID 1** → process execution
* **Sysmon Event ID 11** → file creation
* **Sysmon Event ID 13** → registry modification
* **PowerShell Event ID 4104** → underlying PowerShell script

The investigation demonstrated that combining these telemetry sources provides greater investigative visibility than relying on a single event source.

Because this was a controlled laboratory simulation, the findings represent **simulated attacker activity rather than an actual compromise**.

---

## 6. Defensive Recommendations

1. **Maintain PowerShell Script Block Logging** to improve visibility into PowerShell activity, including obfuscated commands.
2. **Monitor Registry Run Keys** for unexpected modifications that could indicate persistence.
3. **Monitor suspicious file creation and concealment**, particularly in public or temporary directories.
4. **Correlate process, file, registry, and PowerShell events** to reconstruct activity and establish an accurate timeline.

---

## Appendix: Evidence Index

| Screenshot | Evidence |
| --- | --- |
| `discovery.png` | Baseline accounts and running processes |
| `powershell_encoding.png` | Encoded PowerShell command generation |
| `powershell_encoding2.png` | PowerShell Event ID 4104 |
| `Screenshot 2026-10-05 114038.png` |	Sysmon Process Creation for PowerShell
| `file staging.png` | File staging and concealment commands |
| `filestaging.png` | Sysmon Event ID 11 |
| `registrykeys.png` | Sysmon Process Creation for `attrib.exe` |
| `registry1.png` | Sysmon Event ID 1 for `reg.exe` |
| `registry2.png` | Sysmon Event ID 13 for Registry Run key modification |



