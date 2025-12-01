Perfect, I can help you expand your GitHub README to include a full summary of **all content and tasks in the Phishing Emails 1 room** from TryHackMe. Here’s a professional way to document it based on the topics covered:

---

# `/labs/phishing_emails_lab.md`

````markdown
# TryHackMe: Phishing Emails Lab

**Date:** 2025-12-01  
**Platform:** TryHackMe  
**Room:** Phishing Analysis Fundamentals  
**Objective:** Learn the components of emails, analyze headers, and reconstruct attachments safely.

---

## 📝 Overview

This lab covers:

- Email history and basics
- Email protocols (SMTP, POP3, IMAP)
- Ports and secure transport
- Email headers and body
- Identifying phishing indicators
- Base64 attachment extraction and PDF reconstruction

---

## 🌱 Step-by-Step Lab Notes

### 1. Introduction

- Phishing is a social engineering attack via email, SMS, or phone.  
- Even with layered defenses, attackers exploit user mistakes.  
- Security analysts need to manually analyze emails to identify malicious content.

---

### 2. The Email Address

- **Inventor:** Ray Tomlinson (1970s)  
- **Components of an email address:**
  1. User mailbox / username
  2. `@` symbol
  3. Domain  

> Example: `billy@johndoe.com`  
> - `billy` → user mailbox  
> - `johndoe.com` → domain  

---

### 3. Email Delivery & Protocols

**Protocols for sending/receiving emails:**

| Protocol | Purpose | Notes |
|----------|--------|------|
| SMTP     | Sending emails | Port 25 default, STARTTLS for secure transport |
| POP3     | Retrieving emails | Downloads messages to a single device; optional server storage |
| IMAP     | Retrieving emails | Messages stay on server; can sync across multiple devices |

**How email travels:**

1. Sender composes email.  
2. SMTP queries DNS for recipient domain.  
3. Email relayed via SMTP servers.  
4. Destination server receives the email.  
5. POP3/IMAP client retrieves the email.

**Ports (Secure Transport / STARTTLS):**  
- SMTP: 587  
- IMAP: 993  
- POP3: 995  

---

### 4. Email Headers

**Two main parts of an email:**

1. **Header** – metadata about the email (from, to, subject, date, servers involved).  
2. **Body** – actual message content (text or HTML).  

**Common header fields to analyze:**

- From → sender email  
- To → recipient email  
- Subject → email subject line  
- Date → sent date  

**Tip:** Always view emails in raw format to see the full header and routing information.

---

### 5. Analyzing Base64 Attachments

**Task:** Extract and reconstruct a PDF attached in Base64 format.  

**Commands used:**

```bash
# Filter only valid Base64 lines and decode to PDF
grep -E '^[A-Za-z0-9+/=]+$' email2.txt | base64 -d > output.pdf

# Extract text from PDF
pdftotext output.pdf output.txt
cat output.txt
````

**Alternative (if pdftotext not installed):**

```bash
strings output.pdf
```

**Lesson learned:** Always filter for valid Base64 lines to avoid "invalid input" errors.

---

### 6. Observations / Results

* PDF text was successfully reconstructed from the Base64 email attachment.
* Learned safe handling of suspicious attachments and basic email forensics.
* Reinforced the workflow for analyzing headers and identifying phishing indicators.

---

### 🔧 Tools Used

* Linux Terminal: `cat`, `grep`, `base64`
* `pdftotext` / `strings`
* TryHackMe lab environment

---

### 💡 Key Takeaways

* Email analysis is critical for detecting phishing attacks.
* Understand email protocols and header information to trace sender behavior.
* Base64 attachments must be carefully handled to avoid errors.
* Security analysts need both manual and automated methods for email investigation.


