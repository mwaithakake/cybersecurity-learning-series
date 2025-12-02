# Phishing Analysis Tools – Analysis & Notes

**Date:** 2025-12-02
**File Name:** `phishing_analysis_tools.md`

## Objective

Document my learning and workflow from completing the **TryHackMe: Phishing Emails 3 – Phishing Analysis Tools** room. This room focused on using dedicated tooling to triage, analyze, and investigate suspicious emails and their associated IOCs.

---

## Tools Used

These are the specific tools referenced, demonstrated, or required throughout the room:

### 📨 Email & Header Analysis

* **MessageHeader Analyzer (Microsoft)** – Review SPF/DKIM/DMARC, sender anomalies, and routing hops.
* **MXToolbox** – DNS lookups, header parsing, blacklist checks.

### 🧪 URL & Domain Analysis

* **VirusTotal** – URL/file reputation scoring and threat intel.
* **URLScan.io** – Live URL scanning, page captures, and resource requests.
* **PhishTool** – In-depth phishing email analysis platform (headers, URLs, attachments, relationships).

### 🔗 File & Attachment Analysis

* **Hybrid Analysis** – Sandbox detonation and static analysis of samples.
* **Hatching Triage** – Behavioral analysis of suspicious files.
* **Any.Run** – Interactive malware sandboxing.

### 🔍 OSINT & Reputation Services

* **Whois** – Domain ownership & registration timeline.
* **AbuseIPDB** – IP reputation.
* **Talos Intelligence** – Domain/IP/email rep.
* **UrlHaus** – Malware URL tracking.

### 🧩 IOC Extraction & Utility Tools

* **CyberChef** – Decoding, deobfuscation, base64, string extraction.
* **IOC Parser** – Automated IOC extraction from bodies/headers.
* **PhishStats** – Phishing campaign tracking.
* **EmlAnalyzer** (Mentioned in training flow) – For .eml file dissection.

---

## What I Learned

### 1. **How to approach email analysis with dedicated tooling**

The room emphasized not just manually reviewing emails but leveraging a layered toolset: header → body → attachments → URLs → external intel.

### 2. **Header forensics is foundational**

* SPF/DKIM/DMARC can quickly invalidate a sender’s legitimacy.
* Return-Path mismatches and strange Received chains are strong red flags.
* Free tools like Microsoft Header Analyzer and MXToolbox make this process clean.

### 3. **URL investigation isn’t just about clicking**

* Using sandboxes (URLScan, VirusTotal) allows safe inspection.
* I learned what redirected chains look like and what malicious infrastructure patterns appear (fresh domains, no WHOIS data, hosting in high‑risk countries, identical TLS cert fingerprints).

### 4. **Attachment analysis using sandboxes**

Even without downloading malware, tools like Hybrid Analysis/Triage/AnyRun help reveal:

* Dropped files
* API behaviors
* Network calls
* Suspicious strings

### 5. **IOC extraction and documentation**

CyberChef and IOC extraction tools made it easier to document:

* Domains
* IP addresses
* Hashes
* URLs
* File names

This room really reinforced the workflow of pulling all IOCs together and mapping them to MITRE techniques.

---

## Workflow Summary

Here’s how I handled the exercises step‑by‑step:

1. **Loaded the suspicious email (.eml) into a tool** (PhishTool or EmlAnalyzer).
2. **Parsed the email headers** using:

   * Microsoft Header Analyzer
   * MXToolbox
3. **Extracted all URLs** from the email body.
4. **Ran each URL through**:

   * URLScan.io (live behavior)
   * VirusTotal (reputation)
5. **Checked sender domain & IP reputation** via:

   * Talos
   * AbuseIPDB
   * Whois
6. **Analyzed the attachment** using:

   * Hybrid Analysis
   * Hatching Triage
   * Any.Run (for behavioral indicators)
7. **Deobfuscated encoded strings** in CyberChef.
8. **Extracted and documented all IOCs** using IOC Parser.
9. Mapped findings to MITRE ATT&CK where relevant.

---

## Key Takeaways

* This room is heavily focused on **practical investigation techniques** using real tools, not theory.
* Learned how powerful sandboxes and automated analyzers are when dealing with phishing campaigns.
* Reinforced the discipline of maintaining a structured, repeatable workflow.
* Confirmed how to build a full chain of evidence from a single phishing email.
* Strengthened my ability to perform triage independently.

---

## Final Thoughts

Out of the three phishing rooms so far, this one felt the most realistic because it mirrors what analysts actually do in a SOC environment: analyze an email, extract IOCs, enrich data using external intel, and determine impact.

This will definitely be part of my defensive toolkit and something I’ll revisit as I advance deeper into incident response.

---

*End of Entry*
