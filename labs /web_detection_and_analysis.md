# 🛡️ Web Attack Detection & Analysis
**Date:** January 28, 2026
**Topic:** Client-side vs. Server-side Attacks, Log Analysis, and WAFs

---

## 🌐 Overview
Web applications are high-value targets. Detecting these threats requires a combination of log analysis, network traffic inspection, and automated defenses like Web Application Firewalls (WAF).

## ⚔️ Client-Side vs. Server-Side Attacks

### 1. Client-Side Attacks
These exploit the **user's browser** or behavior.
* **Common Types:** Cross-Site Scripting (XSS), CSRF, and Clickjacking.
* **The SOC Blindspot:** Analysts have little visibility here because the malicious code executes on the user's device, often leaving no traces in server logs.

### 2. Server-Side Attacks
These target the **server logic, database, or OS**.
* **Common Types:** Brute-force, SQL Injection (SQLi), and Command Injection.
* **The Advantage:** These leave a "paper trail" in access logs and network packets.

---

## 📜 Log-Based Detection
Access logs provide a chronological record of requests. While they don't usually show the "body" of a POST request, they reveal patterns of abuse.

### Key Indicators in Logs:
| Log Field | Malicious Indicator |
| :--- | :--- |
| **Client IP** | Unusual geo-location or known malicious reputation. |
| **Status Code** | High volume of `404` (fuzzing) or `403` (unauthorized). |
| **User-Agent** | Automated tools like `sqlmap`, `wpscan`, or `Hydra`. |

<img width="1837" height="471" alt="logs1" src="https://github.com/user-attachments/assets/26982ccd-5ea2-4a64-ab74-163521e063fe" />

> *Note: In this log snippet, notice the repeated "FFUF" User-Agent, indicating a directory discovery/fuzzing attack.*

---

## 🔍 Network Traffic Analysis (Wireshark)
Unlike logs, network captures (PCAPs) allow us to see the **full payload**, including credentials and data exfiltrated in the request body.

### Identifying an Attack Sequence:
1.  **Fuzzing:** Repeated `GET` requests to non-existent pages.
2.  **Brute Force:** Rapid-fire `POST` requests to `/login.php`.
    > <img width="788" height="745" alt="Screenshot from 2026-01-28 11-45-46" src="https://github.com/user-attachments/assets/c611bd54-0710-403a-a294-2aed50f912c4" />

    > *In this packet capture, we can see the successful login attempt with the password 'astronpasssword123' and a '302 Found' response.*
3.  **Exploitation:** Following a successful login, the attacker attempts SQL Injection.
    > <img width="1841" height="113" alt="Screenshot from 2026-01-28 11-32-02" src="https://github.com/user-attachments/assets/ca17aac1-2858-430a-a854-292654b42efd" />

    > *The log shows a GET request containing an encoded SQLi payload targeting 'changeusername.php'.*

### Decoding Payloads
Attackers often URL-encode their payloads to bypass simple filters. 
> <img width="928" height="595" alt="Screenshot from 2026-01-28 11-32-31" src="https://github.com/user-attachments/assets/7e8f1997-e1e0-4046-b8c3-a0c0d0c47de7" />

> *Using a decoder reveals the logic: `%2527+OR+%271%27%3D%271` translates to the classic `' OR '1'='1` bypass.*

---

## 🛡️ Web Application Firewalls (WAF)
A WAF sits in front of the application to filter traffic based on rules. 

* **Pattern Matching:** Blocking strings like `sqlmap` or common SQLi keywords.
* **Reputation:** Denying IPs known to be part of botnets.
* **Challenge-Response:** Using CAPTCHAs to weed out the 37% of global traffic that is bot-driven.
* **Protocol Analysis:** Observing traffic in real-time.
    > <img width="1854" height="81" alt="Screenshot from 2026-01-28 11-48-08" src="https://github.com/user-attachments/assets/19b20b0b-2f94-4fe9-84e7-87985d84e811" />

    > *Wireshark view of the SQLi attack being transmitted over HTTP before any WAF intervention.*

---

### 🚀 Investigation Summary
By correlating **Access Logs** (which show the "When" and "Where") with **Network Traffic** (which shows the "What" and "How"), defenders can reconstruct the full lifecycle of a breach—from initial fuzzing to final data exfiltration.
