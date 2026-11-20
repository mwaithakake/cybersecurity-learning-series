##  EDR and SOAR — Series Summary
### Why EDR Matters Beyond Network Perimeters and Antivirus

* **Remote & Cloud Exposure:** As organizations adopt remote work, endpoints operate outside traditional network perimeters, requiring continuous host-level protection regardless of location.
* **Market Examples:** Common EDR platforms include CrowdStrike Falcon, Symantec EDR, SentinelOne EDR, and Microsoft Defender for Endpoint.
* **EDR vs. Antivirus (AV):** Antivirus relies primarily on signature matching against known threats and often misses stealthy or novel attacks. EDR monitors endpoint activity and behavior continuously across the environment, providing broader context and cross-host visibility.
* **Host Scope Limit:** EDR is a host-focused solution and does not detect network-level threats on its own; it works alongside tools such as firewalls, DLPs, email security gateways, and IAMs, typically integrated into a SIEM.

### Attack Chain Comparison: Malicious Macro to Remote Access

In a multi-stage attack (phishing email -> macro execution spawning PowerShell -> obfuscated command downloading a second-stage payload -> injection into `svchost.exe` -> outbound connection for remote access):

* **Antivirus:** Often misses unknown signatures, legitimate host utilities (`winword.exe`), obfuscated PowerShell, memory injection into `svchost.exe`, and network-level activity, potentially marking the activity clean.
* **EDR:** Logs and flags suspicious parent-child relationships (`winword.exe` -> `powershell.exe`), obfuscated execution, process injection into `svchost.exe`, and anomalous outbound connections, generating an alert with the attack chain.

### Core Architecture, Telemetry, and Detection

* **Three Pillars:** **Visibility**, **Detection**, and **Response**.
* **Agents & Central Console:** Lightweight endpoint sensors/agents monitor activity and stream data in real time to a central console where correlation, threat intelligence matching, machine learning, and alert generation occur. Alerts are triaged by severity (Critical, High, Medium, Low, Informational) and classified by analysts as true or false positives.
* **Endpoint Telemetry:** Collected data includes process executions/terminations (parent-child relationships and process trees), network connections, command-line activity (CMD/PowerShell), file/folder modifications, and Windows registry changes.
* **Detection Techniques:** Behavioral detection, anomaly detection against baselines, IOC matching, MITRE ATT&CK tactic/technique mapping, and machine learning models trained on normal and malicious behavior chains.

### Response & Investigation Actions

* **Containment & Remediation:** Analysts can isolate a compromised host from the network to stop lateral movement, terminate malicious processes (carefully to avoid disrupting critical business operations), and quarantine malicious files.
* **Remote Access & Forensics:** Analysts can use remote shells (such as CrowdStrike Falcon Real Time Response) to run commands/scripts and collect forensic artifacts—including memory dumps, event logs, specific folder contents, and registry hives—without physical access to the endpoint.

  ## Traditional SOCs & SOAR 
### Traditional SOC Capabilities & Challenges

* **Core Capabilities:** Centralize monitoring and incident response through **Monitoring and Detection** (largely via a SIEM, such as tracking failed logins or unusual sign-in locations), **Recovery and Remediation** (isolating hosts via EDR, blocking IPs on firewalls, disabling accounts in IAM), **Threat Intelligence** (ingesting IPs, hashes, and domains to block known bad indicators), and **Communication** (coordinating with IT and leadership, such as ticketing for patch verification).
* **Key Challenges:** **Alert fatigue** from high volumes of noisy or low-context alerts, **disconnected tools** requiring manual log hopping, **manual/undocumented processes** reliant on tribal knowledge, and a **talent shortage** that compounds burnout and slows response times.

### SOAR Architecture: Orchestration, Automation & Response

* **Unified Console & Case Management:** SOAR connects disparate SOC tools (SIEM, EDR, firewall, IAM, threat intelligence) into a single interface with structured ticketing and case tracking.
* **Orchestration:** Links tools across vendors and defines conditional investigation workflows called **playbooks** (for example, checking historical logins in a SIEM and IP reputation in threat intelligence during a VPN brute-force alert, stopping early if activity matches normal user behavior).
* **Automation:** Executes playbook steps automatically without repetitive manual clicks—such as querying logs, checking threat feeds, disabling compromised accounts in IAM, and opening detailed tickets.
* **Response:** Triggers containment and remediation actions directly or automatically—such as blocking an IP on a firewall, disabling a user, or opening a tracked incident ticket.
* **Human Analyst Role:** SOAR reduces repetitive toil and alert overload, but analysts remain essential for complex investigations, business-context judgment calls, critical approvals, and building/refining playbooks.

### Playbook Workflows: Phishing & CVE Patching

* **Phishing Playbook:** Starts on a suspicious email alert, creates a ticket, checks for URLs or attachments, branches conditionally ("if/else") for analysis and threat intelligence lookups, notifies users or remediates confirmed threats, and hands off decisions where human review is needed.
* **CVE Patching Playbook:** Tracks newly disclosed Common Vulnerabilities and Exposures, checks exposure across the environment, evaluates risk thresholds, opens patching tickets, and supports testing before production rollout so frequent CVEs do not create an unpatched backlog.
