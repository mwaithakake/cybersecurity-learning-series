


# SIEM Alert Investigation Report  
**Role:** SOC Analyst (Tier 1 – Simulation)  
**Tool:** Splunk  
**Log Sources:** Windows Security Logs, Sysmon, PowerShell Operational Logs  

---
## Incident Summary

An investigation was initiated following the detection of anomalous activity on a Windows endpoint. Initial telemetry indicated suspicious PowerShell execution and unauthorized account activity, suggesting potential persistence and backdoor creation.

The purpose of this investigation was to determine the scope, impact, and nature of the activity, identify affected assets, and outline appropriate containment actions.

---

## 1️⃣ Alert Context

### Alert Type
Suspicious local account creation followed by abnormal PowerShell execution.

### Alert Source
Security Information and Event Management (SIEM) platform – Splunk.

### Data Sources
- Windows Security Event Logs  
- Sysmon Logs  
- PowerShell Operational Logs  

### Alert Description
The SIEM generated an alert after detecting the creation of a new local user account on a Windows endpoint, followed by suspicious process execution involving command-line utilities and encoded PowerShell commands. This behavior is commonly associated with unauthorized persistence and potential remote command execution.

📸 **Screenshot:**  

<img width="1854" height="967" alt="registry" src="https://github.com/user-attachments/assets/4ffe95ce-d304-413f-b2f8-de2c98984a4a" />

*(SIEM alert details showing alert trigger and time range)*

---

## 2️⃣ Analyst Hypothesis

Based on the alert context, this activity may indicate **unauthorized persistence** through local account creation, followed by **remote command execution** using PowerShell. The behavior suggests potential attacker activity rather than legitimate administrative actions.

---

## 3️⃣ Investigation Process

### 🔍 a) Initial Alert Validation

The alert was confirmed by reviewing Windows Security logs for user account creation events.

Key findings:
- A new local account named **A1berto** was created.
- The account was created by the user **James**.
- The event occurred on the host **Micheal.Beaven**.

Relevant event:
- **Event ID 4720** – User account created.

📸 **Screenshot:**  

<img width="1854" height="967" alt="4720" src="https://github.com/user-attachments/assets/fe9e4904-cb1f-4e29-878a-62c6f432576a" />

*(Event ID 4720 showing account creation details)*

---

### 🔍 b) Timeline Analysis

The investigation window was expanded to analyze activity before and after the account creation event.

Observed sequence:
1. Local account creation (Event ID 4720)
2. Execution of `net.exe` and `net1.exe`
3. Registry modification
4. Remote process execution using WMIC
5. Encoded PowerShell command execution

This sequence indicates deliberate post-compromise activity rather than an isolated event.

📸 **Screenshot:**  
<img width="1854" height="967" alt="Screenshot_2026-01-14_03_33_16" src="https://github.com/user-attachments/assets/119a9248-d8f9-46fd-a3db-945a06cae572" />



*(Process creation showing net.exe / net1.exe execution)*

---

### 🔍 c) User Behavior Analysis

The newly created account **A1berto** was analyzed for logon activity.

Findings:
- No successful or failed interactive logons detected.
- No normal user activity associated with the account.

This suggests the account was created for persistence rather than legitimate use.

📸 **Screenshot:**  

<img width="1854" height="967" alt="no_logon" src="https://github.com/user-attachments/assets/c70a7983-ec29-43e6-8a59-61dedc310962" />

*(Search results confirming no logon activity)*

---

### 🔍 d) Host and Endpoint Activity

Further investigation focused on endpoint behavior following the account creation.

Key observations:
- Remote command execution using:
```

WMIC.exe /node:WORKSTATION6 process call create
<img width="1854" height="967" alt="logon_process" src="https://github.com/user-attachments/assets/c10dc057-0cda-4891-a643-424040ae7f83" />


```
- Execution of encoded PowerShell commands
- Registry modification consistent with persistence techniques

The encoded PowerShell execution strongly indicates an attempt to conceal malicious activity.

📸 **Screenshot:**  

<img width="1854" height="967" alt="powershell_scripts" src="https://github.com/user-attachments/assets/4e2243de-7eb2-4b71-94f6-e36e0f6ae2fb" />
<img width="1854" height="967" alt="powershell2" src="https://github.com/user-attachments/assets/d0bc3a4e-54b8-4297-b005-f5262106cb47" />
*(PowerShell logs showing encoded command execution)*

---

### 🔍 e) False Positive Validation

The activity was evaluated for possible benign explanations.

The following were ruled out:
- Legitimate administrative provisioning
- Scheduled system tasks
- Approved automation tools
- Known internal management scripts

Indicators supporting malicious intent:
- Randomized username
- Plaintext password usage
- Encoded PowerShell execution
- Remote command execution via WMIC

---

## 4️⃣ Findings and Evidence Summary

- Unauthorized local account creation detected
- No legitimate logon activity for the new account
- Multiple suspicious processes executed post-creation
- Encoded PowerShell commands observed
- Activity consistent with persistence and lateral movement techniques

All events were time-correlated and occurred on the same endpoint.

---

## 5️⃣ Verdict

✅ **True Positive – Confirmed Security Incident**

The observed behavior aligns with known attacker techniques and cannot be attributed to normal administrative activity.

---

## 6️⃣ Impact Assessment

If left unaddressed, this activity could result in:
- Persistent unauthorized access
- Lateral movement to additional systems
- Execution of arbitrary commands
- Potential data compromise

The affected asset is a Windows endpoint with possible exposure to credential misuse.

---

## 7️⃣ Recommended Actions

As a Tier 1 SOC Analyst, the following actions are recommended:

- Disable and remove the unauthorized local account
- Reset credentials associated with the compromised user
- Isolate the affected host for further forensic analysis
- Escalate the incident to Tier 2 for deeper investigation
- Review and enhance PowerShell logging and alerting rules

---

## 8️⃣ Conclusion

This investigation demonstrates effective SIEM alert validation, log correlation, and endpoint analysis. The alert was confirmed as a true security incident involving unauthorized account creation and suspicious PowerShell execution. Proper escalation and remediation are required to prevent further impact.

---

## 🔎 Skills Demonstrated

- SIEM alert triage and validation  
- Windows log analysis  
- Timeline correlation  
- Endpoint behavior analysis  
- Incident documentation and reporting  
- SOC Level 1 decision-making  


