
# 🛡️ Incident Investigation Report  
**Incident Type:** Man-in-the-Middle (MITM) Attack  
**Organization:** Acme Corp  
**Analyst Role:** SOC Analyst  
**Data Sources:** Network PCAP, Network Logs, Splunk (index=network_logs)

---

## 1. Executive Summary

Acme Corp’s security monitoring systems detected unusual internal network traffic patterns indicative of a potential Man-in-the-Middle (MITM) attack within the corporate LAN.

A detailed investigation confirmed that an attacker successfully intercepted internal network traffic using **ARP spoofing**, redirected user connections through **DNS spoofing**, and downgraded encrypted traffic via **SSL stripping**, resulting in the exposure of user credentials in plaintext.

The incident represents a **confirmed compromise of authentication credentials** and poses a significant risk to internal systems and user accounts.

---

## 2. Incident Context & Initial Alert

The investigation began following alerts indicating abnormal internal traffic patterns, including:

- Unexpected ARP activity  
- Irregular DNS responses for internal services  
- Unencrypted authentication traffic observed on the network  

Given the nature of the alerts, a potential MITM scenario was suspected and a deeper packet-level investigation was initiated.

---

## 3. Data Sources & Tools Used

The following data sources were analyzed:

- **Packet Capture (PCAP):** Full network traffic capture from the affected LAN segment  
- **Network Logs:** `mitm_attack.log`  
- **Splunk SIEM:** Logs ingested under `index=network_logs`  
- **Analysis Tools:** Wireshark, Splunk Search  

---

## 4. Investigation Findings

### 4.1 ARP Spoofing – Network Interception Established

**Observation**

Analysis of ARP traffic revealed repeated gratuitous ARP replies advertising conflicting MAC addresses for the same IP address (`192.168.10.1`, the default gateway).

Wireshark flagged duplicate IP address usage, indicating ARP cache poisoning.

**Evidence**

- Multiple ARP replies claiming ownership of the gateway IP  
- Inconsistent MAC address mappings for the same IP  
- Continuous ARP poisoning activity over time  

📸 **Screenshot:**  
`arp_mitm.png`  
<img width="1854" height="967" alt="arp_mitm" src="https://github.com/user-attachments/assets/df24dc49-b9b5-4432-a0a6-d0cf0de509bf" />

*Gratuitous ARP replies claiming default gateway IP and duplicate IP detection*

**Conclusion**

An attacker successfully poisoned ARP caches on the network, positioning themselves between victim hosts and the default gateway, enabling traffic interception.

---

### 4.2 DNS Spoofing – Malicious Redirection Confirmed

**Observation**

DNS queries for an internal authentication service (`corp-login.acme-corp.local`) were answered with responses resolving to `192.168.10.55`.

The responses originated from a host that was not a legitimate DNS server, indicating forged DNS replies.

**Evidence**

- DNS query and response pairs resolving the login domain to an attacker-controlled IP  
- Repeated responses confirming consistent redirection  
- DNS responses aligned temporally with ARP spoofing activity  

📸 **Screenshots:**  
- `dns1_mitm.png`
- <img width="1854" height="967" alt="dns1_mitm" src="https://github.com/user-attachments/assets/3d76d4d0-b53d-4af7-b0e7-2dc505b3fd46" />

-  — *Forged DNS response redirecting authentication domain*  
- `dns2_mitm.png`
- <img width="1854" height="967" alt="dns2_mitm" src="https://github.com/user-attachments/assets/16903d1a-a8b1-448d-9047-3f2e8710a7e6" />
 — *DNS resolution to attacker-controlled host*

**Conclusion**

The attacker manipulated DNS responses to redirect victims to a malicious internal host, enabling control over authentication traffic.

---

### 4.3 SSL Stripping – Credential Exposure Observed

**Observation**

Initial TLS connection attempts were observed for the internal login service. However, subsequent authentication traffic occurred over unencrypted HTTP rather than HTTPS.

Captured HTTP traffic revealed user credentials transmitted in plaintext.

**Evidence**

- TLS Client Hello observed without successful encrypted session establishment  
- HTTP POST requests to `/login` endpoint  
- Cleartext credentials visible within packet payloads  

📸 **Screenshots:**  
- `ssl1_mitm.png`
-<img width="1854" height="967" alt="ssl1_mitm" src="https://github.com/user-attachments/assets/f23095ac-74b5-4f24-8d59-bad4179e2205" />

-   — *Initial encrypted connection attempt (TLS Client Hello)*  
- `ssl2_mitm.png`
- <img width="1854" height="967" alt="ssl2_mitm" src="https://github.com/user-attachments/assets/d158d540-bc86-48d3-b7ba-03d2a3155f9b" />

-  — *Downgraded authentication traffic over HTTP*  
- `ssl3_mitm.png`
-<img width="1854" height="967" alt="ssl3_mitm" src="https://github.com/user-attachments/assets/04d2141f-4e20-4e7f-afde-3ba818410abd" />

-   — *HTTP POST request containing credentials*  
- `ssl4_mitm.png`
-<img width="1854" height="967" alt="ssl4_mitm" src="https://github.com/user-attachments/assets/177f1313-d94c-411a-9381-80fdb94a075d" />

-   — *Cleartext credential exposure in TCP stream*

**Conclusion**

The attacker successfully downgraded encrypted sessions to plaintext using SSL stripping, resulting in credential compromise.

---

## 5. Attack Chain Reconstruction

Based on the collected evidence, the following attack sequence was reconstructed:

1. Attacker performs ARP spoofing to intercept LAN traffic  
2. Attacker forges DNS responses to redirect victims  
3. Victims connect to attacker-controlled host  
4. SSL stripping downgrades HTTPS to HTTP  
5. User credentials are captured in plaintext  

---

## 6. Impact Assessment

- **Confidentiality:** Compromised (credentials exposed)  
- **Integrity:** At risk due to traffic manipulation  
- **Availability:** Not directly impacted  

**Potential Risks**

- Unauthorized access to internal systems  
- Lateral movement using stolen credentials  
- Escalation to broader internal compromise  

---

## 7. Analyst Recommendations

- Implement Dynamic ARP Inspection (DAI)  
- Enforce HTTPS with HSTS for internal applications  
- Monitor for anomalous ARP and DNS activity  
- Educate users on certificate warnings  
- Rotate compromised credentials immediately  

---

## 8. Final Assessment

This investigation confirms a coordinated Man-in-the-Middle attack leveraging multiple techniques to intercept and compromise internal authentication traffic. Immediate remediation and enhanced network monitoring are required to prevent recurrence.
