# TryHackMe — Phishing Emails in Action 🌐  
**Date Started:** 2025‑12‑01  
**Date Completed:** 2025‑12‑01  
**Room:** Phishing Emails in Action (Phishing Emails 2)  
**Learning Focus:** Email‑based threats, phishing analysis, social engineering, IOC documentation  

---

## 🔍 Lab Overview

This room gives a practical, hands‑on exposure to real‑world phishingemail samples. The aim is to dissect each email to uncover:

- spoofed senders and header anomalies  
- malicious or obfuscated links (shortened URLs, redirect chains)  
- social‑engineering techniques (urgency, fear, brand impersonation)  
- malicious attachments (macros, executables), and  
- Indicators of Compromise (IOCs) that can be tracked or used for detection.

It reflects real-world phishing campaigns and helps build a workflow for safe email triage and threat detection.

---

## 🛠️ What I Did — My Analysis Workflow

For each suspicious email sample in the lab, I followed this process:

1. **Initial Review**  
   - Read the email as rendered (looked at sender name, display email, subject line, body text, branding logos if present).  
   - Sensed initial red flags: mismatched sender, poor grammar, urgent language, generic greetings.

2. **Header & Sender Analysis**  
   - Viewed full raw headers / metadata (Return‑Path, Received: chain, originating IP if visible).  
   - Checked mismatches between displayed sender name vs real sender domain — a classic spoofing sign.  

3. **Link & URL Inspection (Safe Mode)**  
   - Hovered over each link without clicking; copied links in a “defanged” format (e.g. `hxxp://`, replacing `.` with `[.]`).  
   - Investigated URL structure: saw URL‑shorteners, suspicious domains, odd subdomains, redirect patterns, and domains unrelated to the spoofed brand.  
   - Used external tools (e.g. URL expanders, URLScan / VirusTotal) to preview link destination safely (without clicking).  

4. **Attachment Review (Static Only)**  
   - Checked file name and extension: noticed suspicious names like “Invoice”, “Payment_Details”, “Shipping_Notice”, “Delivery_Info”, etc.  
   - Did **not** enable macros or execute any attachment — safe preview only.  
   - For Excel/Word templates or macros, noted that if macros were enabled, payloads would likely run executables (`.exe`, `.bat`, etc.).  

5. **Social Engineering & Psychological Context Analysis**  
   - Documented persuasion strategies: urgency (“Your payment failed — act now!”), fear (“Your package will be returned!”), authority/brand trust (PayPal, DHL, Microsoft, Citrix, etc.), curiosity (“Open your invoice/receipt/attachment”).  
   - Recognized how attackers combine technical tricks with psychological manipulation to increase success rates.  

6. **IOC & Artifact Documentation**  
   - Recorded suspicious domains, defanged URLs, sender addresses, attachment filenames — all as IOCs.  
   - Prepared these for potential reporting, alert rules, or future threat intelligence ingestion.  

7. **Reflection & Threat Context Mapping**  
   - For each email, considered what the attacker’s ultimate goal might be: credential theft, malware delivery, financial fraud, data harvesting, or network intrusion.  
   - Thought about real‑world relevance: how such an email could bypass spam filters, trick an unsuspecting user, and lead to compromise.  

---

## 🧠 Key Takeaways & Threat‑Analysis Lessons

- Phishing emails today are often **well crafted** — not just spam with bad grammar: they use **realistic branding**, professional-looking HTML, and plausible contexts (payments, deliveries, shared documents).  
- **Spoofed headers and sender‑domain mismatches** remain a reliable early red flag — always check raw headers, not just the “From:” display name.  
- **URL shorteners, redirects, and obfuscated domains** are common to hide malicious infrastructure. Always treat any shortened link with suspicion.  
- **Attachments — especially Office documents with macros** — continue to be a major attack vector. Attackers rely on social engineering (e.g. “enable editing”) rather than zero‑day exploits.  
- **Social engineering techniques** (urgency, fear, authority, scarcity) are often as important as technical tricks — most phishing depends on human psychology more than technical prowess.  
- Building a **systematic triage workflow** (header analysis → link/URL inspection → attachment review → IOC logging → risk evaluation) is critical — this is how real SOCs approach phishing incidents.  

---

## 💡 Personal Reflection & How This Shapes My SOC‑Readiness

Working through this room helped me internalize the **mindset of an analyst** — always skeptical, always defensive, always verifying. I came away with:  

- A repeatable, safe process for investigating suspicious emails.  
- Confidence identifying and documenting IOCs and red flags without executing malicious content.  
- Appreciation for the role of social engineering in modern cyberattacks — reminding me that defense isn’t just technical, but also behavioral and procedural.  
- Motivation to integrate phishing‑analysis habits (defanging URLs, logging IOCs, cautious handling of attachments) into daily security practice.  

---

## 📂 Suggestions for Repo / Writing Style

- This write‑up can serve as a **template** for future phishing‑analysis labs or real‑world exercise logs.  
- Keep focusing on **methodology, observations, lessons learned** — not just answers to prompts.  
- A writeup like this shows recruiters you know not just “what phishing is” but *how to analyze it* — a good sign for SOC / defensive roles.  
