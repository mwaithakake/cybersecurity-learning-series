### **Date:** February 5, 2026
### **Topic:** Windows Event Logging System (evtx) and Blue Team Investigation

---

## 📑 Core Concepts of Windows Event Logs
Windows Event Logs are a detailed record of system, security, and application activities. They are stored in the `.evtx` format and can be accessed via **Event Viewer** (`eventvwr.msc`) or programmatically via **PowerShell**.

### Standard Windows Logs:
* **System Logs:** Events logged by Windows system components (e.g., driver failures).
* **Security Logs:** Audit events like successful/failed logons and resource access.
* **Application Logs:** Events logged by specific software applications.
* **Directory Service Logs:** (Domain Controllers only) Active Directory activities.
<img width="1853" height="840" alt="Screenshot from 2026-02-04 11-44-36" src="https://github.com/user-attachments/assets/430589f5-a4a4-4eb1-a802-c4c74cb0dc50" />

---

## 🧬 Key Event IDs for Forensic Analysis


Understanding common Event IDs is critical for rapid threat hunting:

| Category | Event ID | Description |
| :--- | :--- | :--- |
| **Logon** | 4624 | Successful Logon (Look for Logon Type 2 or 10) |
| **Logon** | 4625 | Failed Logon (Brute Force indicator) |
| **Privilege** | 4672 | Special Privileges assigned to a new logon |
| **Process** | 4688 | A new process has been created |
| **Logs** | 1102 | The audit log was cleared (Persistence/Anti-forensics) |
| **PowerShell**| 4104 | PowerShell Script Block Logging (Shows actual code) |

---

## 🛠️ Scenario-Based Investigation
This section reflects hands-on analysis of a compromised Windows system.

### Scenario 1: PowerShell Command Analysis
**Task:** Identify the command executed by an attacker via PowerShell.
**Method:** Navigate to `Applications and Services Logs > Microsoft > Windows > PowerShell > Operational`.
* **Finding:** Event ID **4104** captured the script block text.
* **Finding:** Investigated the encoded payload to reveal C2 communication.

> **[PLACEHOLDER: Add Screenshot of PowerShell Operational Log showing Event ID 4104 here]**
> <img width="1056" height="715" alt="4104" src="https://github.com/user-attachments/assets/88dc1f27-084f-4b2e-a9d4-eb6ba98e6574" />

> *Forensic Evidence: The ScriptBlockText shows the exact commands executed in the session.*

### Scenario 2: User Account Creation & Persistence
**Task:** Identify when a suspicious account named "Sam" was created.
**Method:** Query the Security Log using PowerShell for Event ID 4720 (A user account was created).

```powershell
# Querying for the creation of account "Sam"
Get-WinEvent -LogName Security -FilterXPath '*/EventData/Data[@Name="TargetUserName"]="Sam" and */System/EventID=4720'


```

* **Observation:** Two results were returned, indicating multiple attempts or a specific creation/modification chain.
* **Observation:** Followed up with Event ID **4724** (An attempt was made to reset an account's password).
> <img width="1848" height="597" alt="4720" src="https://github.com/user-attachments/assets/884fba92-4054-4baa-9016-c604746e620d" />
> *Analysis: The timestamp matches the period of suspected compromise.*

---

## 🕵️ Advanced Querying with Get-WinEvent

The `Get-WinEvent` cmdlet is more powerful than the standard GUI for large log files.

* **FilterXML:** Allows for complex multi-parameter queries.
* **FilterXPath:** Efficiently searches the structured XML data within the log entries.

**Example: Finding Special Privileges**

```powershell
# Hunting for high-privilege logons (Event ID 4672)
Get-WinEvent -FilterXPath "*[System[EventID=4672]]" -LogName Security | Select-Object -First 5

```
<img width="1445" height="725" alt="here" src="https://github.com/user-attachments/assets/97118a7b-0c84-4cb4-a973-5369f8eb4346" />

---

## 🛡️ Mitigation & Best Practices

1. **Enable Script Block Logging:** Ensure Event ID 4104 is active via Group Policy to capture malicious PowerShell.
2. **Centralized Logging:** Forward `.evtx` files to a SIEM (Splunk/ELK) to prevent attackers from deleting logs locally.
3. **Monitor for Log Clearing:** Set alerts for Event ID **1102** and **104**, as these are classic signs of an attacker covering their tracks.




