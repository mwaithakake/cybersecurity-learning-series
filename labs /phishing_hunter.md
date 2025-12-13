### 🎣 Phishing Hunter Module: Consolidated Expertise in Threat Triage, Forensics, and Mitigation

**Status:** Badge Earned – Signifying comprehensive, hands-on skill in defending against one of the industry's highest-risk vectors.

**Evidence:** [TryHackMe: Phishing Hunter Badge](https://tryhackme.com/marthanyangatiwa/badges/phishing?utm_campaign=social_share&utm_medium=social&utm_content=badge&utm_source=copy&sharerId=67a462635ff13e077bef0f38)

---

## 🔬 Phase 1: Deep Email Forensics and Triage

This phase focused on breaking down the email structure to validate its origin and integrity, moving beyond surface-level observation, incorporating skills from **Phishing Analysis Fundamentals** and **Emails in Action**.

| Analytical Focus | Skill Demonstrated | Real-World Application |
| :--- | :--- | :--- |
| **Header Tracing** | **Non-Repudiation & Traceback** | Deep analysis of `Received` and `Return-Path` headers to identify the true server origin, trace the message path through intermediate mail relays, and confirm the sender's IP. |
| **Authentication** | **DMARC, SPF, & DKIM Validation** | Used external tools (e.g., MXToolbox, Header Analyzer) to check the validity of the sender's domain policy, immediately flagging policy failures (`Fail`) indicative of spoofing. |
| **Lure Classification** | **Social Engineering Recognition** | Categorized the psychological trigger used (urgency, authority, financial incentive) and identified common link manipulation tactics (pixel tracking, shortened URLs). |
| **Protocol Knowledge** | **SMTP/POP3/IMAP Flow** | Demonstrated understanding of the underlying protocols and how emails are relayed, which is foundational for understanding where and how an attack can be injected or stopped. |

---

## 🔗 Phase 2: Advanced Payload Analysis and IOC Enrichment

This phase focused on the safe inspection of malicious content, leveraging advanced tooling as covered in the **Phishing Analysis Tools** room.

### **A. Integrated Toolset Proficiency**

I used a layered approach across multiple specialized tools:

* **Email Forensics:** MessageHeader Analyzer, MXToolbox, PhishTool.
* **Deobfuscation:** **CyberChef** (Proficient in Base64, Hex, and URL encoding/decoding operations).
* **Sandboxing:** **Any.Run** and **Hybrid Analysis** for safe file detonation and behavioral monitoring.
* **OSINT/Reputation:** VirusTotal, AbuseIPDB, UrlHaus, and Whois for comprehensive IOC enrichment.

### **B. Key Analytical Tasks**

* **Behavioral Monitoring:** Observed HTTP redirects and network calls within sandboxes to capture the full attack chain and final landing page URL.
* **IOC Generation:** Extracted critical Indicators of Compromise (IOCs)—including file hashes (SHA256), malicious registry keys, and Dropper URLs—ready for insertion into SIEM/EDR systems.
* **Adversary TTPs:** Identified phishing infrastructure patterns (fresh domains, typosquatting, high-risk hosting) and matched them against known adversary techniques.

---

## 🛡️ Phase 3: Mitigation, Prevention, and Response Strategy

This phase incorporates lessons from **Phishing Prevention** and the real-world challenge rooms, focusing on defense hardening and full incident closure.

### **Structured Incident Response Workflow**

| Step | Action and Tool | Defense Outcome |
| :--- | :--- | :--- |
| **Containment** | IOC Blocking & Endpoint Isolation | Immediately pushed extracted IPs, domains, and hashes into organizational blocklists (Firewall/Email Gateway). |
| **Eradication/Recovery** | System Cleanup & Account Reset | Identified malicious file paths (e.g., in downloads) and advocated for user account password resets if credentials were confirmed harvested. |
| **Knowledge Mapping** | **MITRE ATT&CK Mapping** | Identified the specific Adversary Tactics, Techniques, and Procedures (TTPs) used (e.g., T1566: Phishing) to improve proactive threat hunting rules. |
| **Organizational Defense** | **User Awareness Training** | Developed training points based on the identified lures and TTPs, strengthening the human layer of defense against future campaigns. |

### **Final Conclusion**

The Phishing Hunter Module provided the full, practical scope necessary for a SOC Level 1/2 Analyst. My experience ranges from manual email header forensics and safe payload analysis using advanced tools to the final steps of containment, IOC generation, and organizational hardening. I am effectively equipped to act as a **Phishing Triage Specialist**, minimizing dwell time and mitigating organizational risk.
