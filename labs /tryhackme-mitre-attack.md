# TryHackMe — MITRE ATT&CK Room (Learning Notes)

**Date:** 2025-09-29  
**Platform:** TryHackMe  
**Topic:** MITRE ATT&CK — mapping adversary behaviour to detection and response

---

## Overview
Completed the TryHackMe MITRE ATT&CK room to learn how the ATT&CK framework helps defenders classify adversary behaviour, identify telemetry sources, and design basic detection & response ideas.

---

## Objectives
- Understand the ATT&CK matrix and common tactic categories (Initial Access, Execution, Persistence, Privilege Escalation, Defense Evasion, etc.)
- Map specific techniques to telemetry sources (Windows Event Logs, Sysmon, process creation, network connections, DNS)
- Draft simple detection ideas and triage/playbook steps

---

## Key Concepts Learned
- **ATT&CK as a shared language:** map observable behaviour to ATT&CK technique IDs to standardize analysis and reporting.  
- **Telemetry-first detection:** choose the right log/agent to observe a technique (e.g., Sysmon/process creation for command execution; DNS logs for beaconing).  
- **Detection design:** turn an ATT&CK technique into detection logic + triage playbook (what to look for, what to enrich, when to escalate).  
- **Prioritization:** focus on high-impact techniques and signals that are available in your environment.

---

## Example mappings (high-level)

| ATT&CK Tactic        | Example Technique                 | Useful Telemetry                          |
|----------------------|-----------------------------------|-------------------------------------------|
| Execution            | Command and Scripting Interpreter | Process creation, commandline logging (Sysmon) |
| Persistence          | Scheduled Task                    | Task scheduler logs, registry run keys    |
| Command & Control    | DNS/HTTP Beaconing                | DNS logs, proxy logs, network flow metadata |
| Credential Access    | Brute Force                       | Authentication logs (failed logins, account lockouts) |

> Note: This is for defensive learning — no offensive instructions included.

---

## Detection / Playbook Idea (Template)
1. **Detection trigger:** e.g., multiple suspicious commandline invocations from a workstation within X minutes.  
2. **Enrichment:** host user, recent logins, parent process, open network connections.  
3. **Triage:** determine if activity is scheduled/expected or abnormal. If abnormal → isolate host, snapshot memory, escalate to IR.  
4. **Report:** document indicators (IOC), affected assets, mitigation steps, and follow-up detection rules.

---

## Artifacts / Notes
- `notes-mitre-mapping.md` — my raw notes mapping techniques to telemetry and example detections.  
- `images/` — screenshots: room badge, matrix mapping snippet, playbook sample.

---

## Lessons Learned
- ATT&CK helps structure thinking during triage and writeups — now I can explain *how* an alert maps to a tactic and why a given log source is important.  
- My next step: apply this mapping to a live SOC alert in TryHackMe or LetsDefend lab and write a 1-page case study.

---

## Next Steps
- Run a small hunt: look for commandline anomalies in Splunk/Wazuh logs and map any findings to ATT&CK techniques.  
- Draft a short case study with screenshots and timeline.

---
