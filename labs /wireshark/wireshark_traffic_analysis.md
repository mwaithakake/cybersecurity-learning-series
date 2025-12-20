# Project: Network Traffic Analysis & Incident Response (Wireshark)
Date: December 20, 2025

## 🛡️ Objective

This project documents a deep-dive investigation into various network compromise scenarios. The goal was to move beyond basic packet capture and perform advanced host identification, tunnel detection, traffic decryption, and actionable threat mitigation.

---

## 🛠️ Tooling & Environment

* **Tool:** Wireshark (v3.1+)
* **Protocols Investigated:** DHCP, NBNS, Kerberos, ICMP, DNS, FTP, HTTP/2, TLS.
* **Key Skills:** Protocol Tunneling Detection, SSL/TLS Decryption, Cleartext Credential Hunting, Firewall Rule Generation.

---

## 🔍 Investigation Modules

### 1. Host & User Identification

In a compromise, an IP address is just a number. I used protocol-specific metadata to map IP addresses to physical devices and human users.

* **DHCP (Option 12):** Identified hostnames of mobile devices (e.g., "Galaxy A30").
* **NetBIOS (NBNS):** Used to map workstation names in environments where DHCP logs might be missing.
* **Kerberos (CNameString):** Distinguished between computer accounts (ending in `$`) and actual human usernames to find the person behind the traffic.

 <img width="1001" height="825" alt="netbios" src="https://github.com/user-attachments/assets/9eac4846-1f0b-4a6e-9809-c0aec5aba24e" />
 
<img width="1001" height="825" alt="dhcp" src="https://github.com/user-attachments/assets/5126eef4-b80c-4bd4-b489-1f33379b2172" />

### 2. Tunneling & Data Exfiltration

I investigated how attackers "smuggle" data through trusted protocols to bypass firewalls.

* **ICMP Tunneling:** Identified SSH sessions hidden inside Ping packets by looking for anomalous packet sizes (`data.len > 64`) and finding the `SSH-2.0` handshake in the raw bytes.
* **DNS Tunneling:** Spotted data exfiltration by filtering for unusually long query names (`dns.qry.name.len > 15`) and identifying a high volume of Base64 encoded subdomains pointing to the malicious root domain: `dataexf[.]com`.

<img width="1010" height="860" alt="icmp" src="https://github.com/user-attachments/assets/43f1fa67-4b9b-40f7-9faa-32945f3d6964" />

### 3. Cleartext Protocol Analysis (FTP & HTTP)

Investigated "unsecured" traffic where credentials and commands are sent in plain text.

* **FTP Investigation:** Analyzed response codes (e.g., `530` for failed logins) to identify brute-force attempts and captured cleartext passwords using the `ftp.request.command == "PASS"` filter.
* **HTTP & Log4j:** Hunted for modern exploits by searching for JNDI lookup strings (`${jndi:ldap...}`) in User-Agent headers. Identified automated attack tools like `sqlmap`, `Nikto`, and `Wfuzz` by analyzing User-Agent "signatures."

<img width="1858" height="382" alt="http" src="https://github.com/user-attachments/assets/bf72ea98-1b11-4504-bd81-dbc1bf47f9d3" />

### 4. Decrypting HTTPS Traffic

Modern threats often hide in encrypted TLS streams. I utilized **SSL/TLS Key Log Files** to "unlock" the traffic.

* **Process:** Applied the `(Pre)-Master-Secret` log file within Wireshark preferences.
* **Outcome:** Successfully transformed encrypted "Application Data" into readable **HTTP/2** streams, allowing for the inspection of full URIs and file transfers that were previously invisible.

### 5. Hunting Cleartext Credentials

Used the **Tools → Credentials** feature to rapidly aggregate all unencrypted login attempts across the pcap.

* **Key Discovery:** Identified a specific packet (#170) where a user submitted an empty password, a common sign of automated script errors or anonymous login attempts.

<img width="1861" height="794" alt="bonus 1" src="https://github.com/user-attachments/assets/c676ddc0-ba4b-4b7b-8f48-36c4510e4c44" />

---

## ⚡ Actionable Mitigation

The final step of the investigation was moving from analysis to defense. I utilized Wireshark’s **Firewall ACL Rules** tool to automatically generate blocklist syntax for:

* **Netfilter (iptables)**
* **Cisco IOS (Standard/Extended)**
* **Windows Firewall (netsh)**

<img width="1861" height="794" alt="bonus 2" src="https://github.com/user-attachments/assets/6c34c9d1-955d-4fb7-afe6-6c550e34a04c" />

---

## 📋 Technical Filter Cheat Sheet

| Goal | Wireshark Filter |
| --- | --- |
| **Find Hostname (DHCP)** | `dhcp.option.hostname contains "name"` |
| **Find Real Usernames** | `kerberos.CNameString and !(kerberos.CNameString contains "$")` |
| **Detect ICMP Tunnel** | `data.len > 64 and icmp` |
| **Detect Long DNS Queries** | `dns.qry.name.len > 15 and !mdns` |
| **Find HTTP POST Data** | `http.request.method == "POST"` |
| **Log4j Exploit Hunting** | `frame contains "jndi"` |

---

## 🧠 Analyst Reflection

"This lab reinforced that 'trust' is a vulnerability. Attackers use DNS and ICMP because they are usually allowed by default. Effective analysis is not just about knowing filters, but about recognizing **protocol anomalies**—like a Ping packet that is too heavy or a DNS query that looks like a secret code. Decryption and automated credential tools are powerful, but the analyst's ability to interpret the raw bytes remains the most critical skill."

---
