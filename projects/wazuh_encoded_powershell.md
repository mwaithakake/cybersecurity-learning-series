# Wazuh SIEM Detection and Investigation: Encoded PowerShell Activity

## 1. Project Overview

**Project type:** Blue Team / SIEM Detection and Incident Investigation  
**Tools:** Wazuh, Sysmon, Windows PowerShell, Windows Event Logs, Ubuntu  
**Environment:** Isolated VirtualBox cybersecurity lab  
**Endpoint investigated:** Windows 11 — `Martha` (`192.168.56.106`)  
**SIEM manager:** Wazuh manager on Ubuntu (`192.168.56.107`)  
**MITRE ATT&CK technique:** T1059.001 — PowerShell

### Objective

The objective of this project was to simulate suspicious PowerShell activity on a Windows endpoint, collect and analyze security telemetry using Wazuh, investigate the resulting alert, reconstruct the observed activity, assess its significance, and recommend appropriate response actions.

The investigation focused on understanding how encoded PowerShell execution appears in endpoint telemetry and how an analyst can distinguish suspicious behavior from confirmed malicious activity.

## 2. Lab Environment and Data Sources

The investigation used a Windows 11 virtual machine monitored by a Wazuh agent connected to a Wazuh manager running on Ubuntu.

The relevant data sources were:

- **Sysmon:** Process-creation events and file-related telemetry.
- **Windows PowerShell Operational log:** Script Block Logging, including Event ID 4104.
- **Wazuh alerts:** Detection rules and event information generated from endpoint telemetry.

The endpoint was configured to forward relevant security events to Wazuh. Wazuh alerts were examined alongside locally available Windows event logs and filesystem evidence.

## 3. Simulated Activity

A controlled PowerShell command was executed using the `-EncodedCommand` parameter. The decoded command was:

`Set-Content -Path 'C:\Users\Public\siem_investigation_marker.txt' -Value 'SIEM investigation test'`

The command wrote a benign text value to a marker file in the Public user directory.

Base64 encoding was used to represent the command. Encoding is not encryption and does not inherently indicate malicious intent. However, encoded PowerShell execution can be suspicious during security monitoring because the command's contents may not be immediately apparent.

The activity was intentionally generated for this lab; no real malware was required.

## 4. Detection

Wazuh generated an alert for the encoded PowerShell execution.

**Primary alert details**

- **Rule ID:** 92057
- **Rule description:** PowerShell spawned a PowerShell process that executed a Base64-encoded command.
- **Severity level:** 12
- **MITRE ATT&CK:** T1059.001 — PowerShell
- **Endpoint:** Martha (`192.168.56.106`)
- **Process ID:** 5288
- **Parent process ID:** 7776
- **Account:** `CYBERLAB\Administrator`
- **Sysmon event:** Event ID 1 — Process Creation
- **Event time:** 07:45:06 EAT, 17 October 2026
- **Wazuh alert time:** 07:45:08 EAT, 17 October 2026

The alert identified encoded PowerShell execution and provided process metadata for further investigation.

The severity assigned by Wazuh indicated that the detection warranted attention. It did not, by itself, establish that the activity was malicious.

<img width="1792" height="693" alt="wazuh-detection(1)" src="https://github.com/user-attachments/assets/7379d0cf-97b1-46a8-9a17-a954940f10e4" />
*Figure 1. Wazuh alert 92057 showing the encoded PowerShell detection.*

## 5. Investigation and Evidence

### 5.1 Process Investigation

The Sysmon process-creation record associated with the alert identified the executable as:

`C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`

The recorded command line included the `-EncodedCommand` parameter.

The event identified process ID 5288, parent process ID 7776, and the account `CYBERLAB\Administrator`. The record also contained a SHA256 hash for the PowerShell executable.

These details helped establish which process triggered the detection, which process launched it, and which account was associated with the execution.

The process metadata supported the investigation but did not independently demonstrate malicious intent.

<img width="1786" height="721" alt="04-process-investigation" src="https://github.com/user-attachments/assets/f0a590e1-89ac-4de3-a0ca-457e3ed72f2c" />
*Figure 2. Process investigation details from the Wazuh alert JSON.*

### 5.2 PowerShell Script-Block Evidence

Windows PowerShell Operational logs contained Event ID 4104 records around the time of execution.

The records included the encoded PowerShell invocation and the decoded `Set-Content` command. This provided local endpoint evidence of the command's intended operation.

The recorded timestamps were approximately 07:45:06–07:45:09 EAT on 17 October 2026.

The original local PowerShell event was confirmed on the endpoint. The available evidence did not establish that this exact Event ID 4104 record was also present in Wazuh's alert records.

<img width="1212" height="710" alt="02-powershell-investigation" src="https://github.com/user-attachments/assets/b7af43f2-060d-4eb8-b117-57470ef499ab" />
*Figure 3. PowerShell Event ID 4104 records showing the encoded invocation and the command writing the marker file.*

### 5.3 Marker File Verification

The marker file was verified at:

`C:\Users\Public\siem_investigation_marker.txt`

The filesystem inspection returned:

- **File size:** 25 bytes
- **Creation time:** 07:45:09 EAT, 17 October 2026
- **Last modification time:** 07:45:09 EAT, 17 October 2026

The file's existence and timestamps were consistent with the intended test command.

However, a Sysmon Event ID 11 record directly identifying the creation of this specific marker file was not found during the investigation. Therefore, the file's existence and timestamp were treated as supporting evidence rather than direct Sysmon confirmation of its creation.

<img width="769" height="231" alt="03-marker-file" src="https://github.com/user-attachments/assets/1e31a2ef-869b-44e1-806a-1f65dccf1271" />

*Figure 4. PowerShell output verifying the marker file's path, size and timestamps.*

### 5.4 Assessment of a Secondary Alert

Wazuh also generated rule 92213, titled *Executable file dropped in folder commonly used by malware*, at severity level 15.

The relevant Sysmon Event ID 11 record identified this file:

`C:\Users\Administrator\AppData\Local\Temp\__PSScriptPolicyTest_pebb3hno.13x.ps1`

The event shared the process GUID associated with the encoded PowerShell execution. However, the file path was different from the marker file's path.

The filename is consistent with a temporary PowerShell script-policy test file. The available evidence did not establish that this file was malicious. Consequently, the alert was treated as requiring contextual assessment rather than as proof of malware or successful malicious file transfer.

This finding illustrates why an analyst should examine the underlying event, file path, process context and surrounding activity rather than relying on alert severity alone.

<img width="1145" height="796" alt="05-secondary-alert" src="https://github.com/user-attachments/assets/88787052-6f5f-40c4-90e5-0d32200425a4" />
*Figure 5. Wazuh rule 92213 showing the temporary PowerShell script-policy test file.*

## 6. Event Timeline

| Time (EAT) | Evidence | Interpretation |
|---|---|---|
| 07:45:06 | Sysmon Event ID 1; Wazuh rule 92057 | Encoded PowerShell process execution was recorded. |
| 07:45:06–07:45:09 | Local PowerShell Event ID 4104 records | The records captured the encoded invocation, command preparation, and marker-file writing command. |
| 07:45:08 | Wazuh rule 92057 | Wazuh generated an alert for the encoded command. |
| 07:45:08 | Sysmon Event ID 11; Wazuh rule 92213 | A temporary PowerShell script-policy test file was recorded. |
| 07:45:09 | Filesystem inspection | The marker file was present with creation and modification timestamps consistent with the test timeline. |

The timestamps and process metadata support a reconstruction of the controlled activity. The marker file's creation was not directly confirmed by a matching Sysmon Event ID 11 record, and the secondary alert concerned a different file.

## 7. Findings and Conclusion

The investigation established that Wazuh detected a PowerShell process executing a Base64-encoded command on the monitored Windows endpoint.

Local PowerShell logs revealed that the command wrote a benign text value to the marker file, which was subsequently verified on disk. The observed behavior was consistent with the intentionally generated lab activity.

The investigation also identified a second, high-severity alert concerning a temporary PowerShell script-policy test file. The available evidence was insufficient to classify that file as malicious, demonstrating the importance of contextual alert investigation.

**Conclusion:** The primary alert was generated by an authorized, benign lab simulation. The observed encoded PowerShell execution warranted investigation, but encoding and alert severity alone were not evidence of compromise.

This conclusion applies to the controlled activity and evidence reviewed. It does not constitute a comprehensive assessment of the endpoint or prove the absence of unrelated malicious activity.

## 8. Scope and Limitations

The investigation was limited to one Windows 11 endpoint and selected telemetry around the controlled execution.

The following limitations were identified:

- The specific marker-file creation event was not found in the reviewed Sysmon Event ID 11 records.
- The original PowerShell Event ID 4104 evidence was confirmed locally, but its ingestion into Wazuh's alert records was not confirmed.
- The secondary alert referred to a different temporary file and could not be classified as malicious from the available evidence.
- The investigation did not establish whether unrelated persistence, network communication, credential access or lateral movement existed elsewhere on the endpoint.
- The exercise was a focused detection and investigation lab, not a full incident response engagement.

These limitations define what can reasonably be concluded from the available evidence.

## 9. Recommended Response

For a comparable alert in a real environment, an analyst should:

1. **Validate the activity:** Determine whether the execution was expected and whether the user or administrator can explain it.
2. **Inspect the command:** Decode and review the command's actual behavior, arguments and intended effects.
3. **Examine the process chain:** Review the parent process, account, process metadata and surrounding process activity.
4. **Correlate additional telemetry:** Review adjacent process, file, registry, network and authentication events for evidence of broader activity.
5. **Assess related alerts:** Investigate associated alerts individually and determine whether their underlying events support the detection descriptions.
6. **Preserve evidence:** Retain relevant event records, timestamps, command lines and hashes according to the organization's procedures.
7. **Contain when justified:** If the activity remains unexplained and evidence indicates a credible threat, consider endpoint isolation and escalation under the incident response process.
8. **Document and improve:** Record the conclusion, response actions and any confirmed false positives. Tune detection rules only after validating the underlying behavior and considering the risk of missed detections.

In this lab, the known activity was intentional and benign, so disruptive containment was not warranted.

## 10. Skills Demonstrated

- SIEM alert triage and investigation using Wazuh
- Windows endpoint telemetry analysis
- Sysmon process-creation investigation
- PowerShell Script Block Logging analysis
- Command decoding and behavioral interpretation
- Process and parent-process analysis
- Timeline construction and evidence correlation
- Contextual assessment of high-severity alerts
- MITRE ATT&CK mapping
- Evidence-based conclusions and response recommendations
- Documentation of investigation scope and limitations

## 11. Project Summary

This project demonstrated a practical workflow for investigating suspicious PowerShell activity in a controlled Windows lab. It connected a SIEM detection to endpoint process evidence, local PowerShell logs and filesystem observations, then assessed the findings and documented an appropriate response.

The central lesson was that **a detection identifies behavior worth investigating; the analyst must establish what the evidence actually supports before deciding whether an incident is malicious.**
