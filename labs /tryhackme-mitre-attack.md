# TryHackMe — MITRE ATT&CK Room (Learning Notes)

**Date:** 2025-09-29  
**Platform:** TryHackMe  
**Room:** MITRE **
**Difficulty:** medium  

---

## Overview
Completed the TryHackMe **MITRE** room to learn how the ATT&CK framework and related MITRE resources help defenders classify adversary behaviour, identify useful telemetry sources, and design basic detection & response ideas. The room covers multiple MITRE projects and how they support defensive workflows.


---

## Room structure / Tasks (as listed on TryHackMe)
1. **Task 1 — Introduction to MITRE**  
2. **Task 2 — Basic Terminology**  
3. **Task 3 — ATT&CK® Framework**  
4. **Task 4 — CAR Knowledge Base**  
5. **Task 5 — MITRE Engage**  
6. **Task 6 — MITRE D3FEND**  
7. **Task 7 — ATT&CK® Emulation Plans**  
8. **Task 8 — ATT&CK® and Threat Intelligence**  
9. **Task 9 — Conclusion**

---

## Objectives
- Understand the ATT&CK matrix and related MITRE projects (CAR, Engage, D3FEND, Emulation Plans).  
- Identify which telemetry sources support detection of common ATT&CK techniques (e.g., Windows event logs, Sysmon, network/DNS logs).  
- Learn how ATT&CK integrates with threat intelligence and emulation planning for defensive testing.  
- Draft basic detection and triage/playbook ideas informed by ATT&CK mappings.

---

## Key Concepts Learned
- **ATT&CK as a defender’s language:** how to map observed behaviour to ATT&CK technique IDs to standardize reporting and detection.  
- **Complementary MITRE projects:** CAR (detection content), Engage (adversary engagement frameworks), D3FEND (defensive techniques), and Emulation Plans (testing/validation).  
- **Telemetry-first approach:** select the most appropriate logs/agents for observing a technique (e.g., process creation & commandline via Sysmon, DNS via DNS logs).  
- **Detection design & validation:** use CAR/Emulation Plans to build and test detection logic.

---

## Example mappings (high-level)
| ATT&CK Tactic     | Example Technique                 | Useful Telemetry                                  |
|-------------------|-----------------------------------|---------------------------------------------------|
| Execution         | Command and Scripting Interpreter | Process creation, commandline logging (Sysmon)    |
| Persistence       | Scheduled Task                    | Task scheduler logs, registry run keys            |
| Command & Control | DNS/HTTP Beaconing                | DNS logs, proxy logs, network flow metadata       |
| Credential Access  | Brute Force                      | Authentication logs (failed logins, lockouts)     |

> Defensive focus only — no offensive instructions included.

---

## Detection / Playbook idea (template)
1. **Detection trigger:** e.g., repeated suspicious commandline invocations on a host within X minutes.  
2. **Enrichment:** host owner, recent logins, parent process, network connections.  
3. **Triage:** determine whether activity is scheduled/expected vs anomalous. If suspicious → isolate host, capture memory, escalate to IR.  
4. **Report:** document IOCs, impacted assets, mitigation actions, and follow-up detection rules.



## Lessons Learned
- MITRE ATT&CK plus CAR/D3FEND/Emulation Plans provides a practical ecosystem for detection design, testing, and validation.  
- Mapping techniques to telemetry makes triage faster and more consistent across SOC analysts.  
- Next actionable step: apply ATT&CK mappings to a live alert in Splunk/Wazuh and prepare a 1-page case study.

---



