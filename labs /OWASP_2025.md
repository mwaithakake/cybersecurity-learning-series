# 🛡️ OWASP Top 10: 2025 
**Date:** January 27, 2026
**Topic:** Core Vulnerabilities & IAAA Framework

---

## 🏛️ The Foundation: IAAA Framework
The **IAAA model** is the non-negotiable hierarchy for verifying users and actions. Skipping a level compromises the entire chain.

1.  **Identity:** The unique account (User ID/Email).
2.  **Authentication:** Proving identity (Passwords, OTP, Passkeys).
3.  **Authorisation:** Permission levels (What can they do?).
4.  **Accountability:** Logging and alerting (Who did what and when?).

---

## 🔍 Vulnerability Deep-Dive

### 1. Broken Access Control (A01) & Auth Failures (A07)
Failures in the IAAA implementation allow attackers to jump privilege levels.
* **Key Issues:** IDOR (Insecure Direct Object Reference), horizontal/vertical privilege escalation, and logic flaws in login flows.
* **Prevention:** Enforce server-side checks on **every** request; rate-limit/lock out brute force attempts.

### 2. Security Misconfigurations (A02)
Mistakes in environment setup rather than code bugs.
* **Patterns:** Default credentials, exposed S3 buckets (e.g., Uber 2017), and uncurated AI/ML endpoints.
* **Prevention:** Harden defaults, remove unused services, and audit cloud permissions regularly.

### 3. Software Supply Chain Failures (A03)
Risks inherited from third-party components, libraries, or AI models.
* **Impact:** A single compromised dependency (e.g., SolarWinds) can bypass all internal code security.
* **Prevention:** Verify all third-party models/libraries; lock down CI/CD pipelines.

### 4. Cryptographic Failures (A04)
Incorrect or absent encryption for data at rest and in transit.
* **Patterns:** Using MD5/SHA-1, hard-coded secrets, or invalid TLS certificates.
* **Prevention:** Use AES-GCM or TLS 1.3; utilize Key Management Services (AWS KMS, Azure Key Vault).

### 5. Insecure Design (A05)
Architectural flaws that cannot be "patched" because they are built into the logic.
* **AI Era Risks:** Prompt injection and "blind trust" in model outputs.
* **Prevention:** Threat modeling at every stage; separate system prompts from user content; require human review for high-risk AI actions.

### 6. Injection (A06)
Mishandling user input by passing it directly to interpreters (SQL, Shell, SSTI).
* **Prevention:** Use **parameterized queries** and prepared statements. Never build queries through string concatenation.

### 7. Software & Data Integrity Failures (A08)
Trusting code or data without verifying its authenticity or origin.
* **Key Issues:** Unsigned updates or unverified JSON/binary files.
* **Prevention:** Use cryptographic checksums and establish strict trust boundaries within CI/CD processes.

### 8. Logging & Alerting Failures (A09)
The breakdown of **Accountability**. Without logs, breaches go undetected.
* **Prevention:** Log full auth lifecycles and centralize logs off-host to prevent attacker tampering.

---

> **Summary Note:** The 2025 landscape shifts heavily toward **AI-specific threats** (Prompt Injection, Model Poisoning) and **Supply Chain integrity**, reflecting the modern "modular" way we build software.
