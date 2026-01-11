
**Date:** 2026-01-11  
**Category:** SOC Operations / Traffic Analysis / Exfiltration Detection

---

## 🎯 Objective
Identify and analyze common techniques used by adversaries to smuggle data out of a compromised network. This lab focused on detecting exfiltration via **FTP**, **HTTP**, **ICMP**, and **DNS** using tools like **Wireshark** and **Splunk**.

---

## 🛠️ Technical Breakdown & Methodology

### 1. FTP Exfiltration (Protocol Abuse)
* **Concept:** Attackers often use plain-text protocols like FTP to transfer files because they are common and sometimes overlooked by basic firewalls.
* **Analysis:** Using Wireshark, I identified multiple `USER` login attempts followed by a successful file transfer (`STOR`).
* **Detection:** By following the TCP stream, I recovered the contents of the exfiltrated file and an associated flag.

![Identifying FTP requests and file storage commands (STOR) in Wireshark](Screenshot_2026-01-11_05_50_38.png)
<img width="1854" height="967" alt="ftp 1" src="https://github.com/user-attachments/assets/9c09c89e-c61f-4428-a9ef-097cdd241ea3" />


![Following the TCP Stream to recover the exfiltrated data: internal_passwords.csv](Screenshot_2026-01-11_05_50_50.png)
<img width="1854" height="967" alt=" ftp 2" src="https://github.com/user-attachments/assets/75c9c433-ac19-45a4-b3a5-3f333730670a" />

---

### 2. HTTP Exfiltration (Web Post Requests)
* **Concept:** Data is often "posted" to an external web server. It looks like normal web traffic but contains sensitive payloads.
* **Splunk Analysis:** I used Splunk to calculate the average and maximum bytes sent per domain. One outlier stood out with a very high byte count compared to its frequency.
* **Wireshark Verification:** Inspected the `POST` request to `api.cloudsync-services.com`.

![Using Splunk to hunt for outliers in HTTP traffic by analyzing bytes_sent](Screenshot_2026-01-11_05_58_11.png)
<img width="1854" height="967" alt="http_logs" src="https://github.com/user-attachments/assets/e94da6e3-ed9b-4a90-9ac1-b782c51f9cf9" />


![Recovering sensitive "Finance Department" credentials from an HTTP POST request](Screenshot_2026-01-11_06_02_03.png)
<img width="1854" height="967" alt="http_2" src="https://github.com/user-attachments/assets/09c4c279-3289-426a-a038-c34358311442" />

---

### 3. DNS & ICMP Tunneling (Stealth Techniques)
* **Concept:** When standard ports are blocked, attackers use "non-data" protocols like DNS queries or ICMP (pings) to carry small chunks of data.
* **DNS:** I analyzed query volume per source IP in Splunk to find high-frequency beaconing.
* **ICMP:** Found ICMP echo requests with unusually large data payloads containing system information.

![Monitoring DNS query counts per Source IP in Splunk to detect potential tunneling](Screenshot_2026-01-09_10_19_10.png)
<img width="1854" height="967" alt="dns splunk" src="https://github.com/user-attachments/assets/94f17e7a-2ebf-4bab-8da7-273ba274c198" />


![Identifying ICMP packets carrying clear-text sensitive data (database credentials)](Screenshot_2026-01-11_07_18_09.png)
<img width="1854" height="967" alt="icmp" src="https://github.com/user-attachments/assets/fd03e57b-6d9d-4e7e-8e0b-646a0b6348e5" />

---

## 🧠 Forensic Insights

### Why Outlier Analysis Matters
In the HTTP task, the attacker used a domain that looked legitimate (`cloudsync-services.com`). However, by using the Splunk query:

```splunk
index="data_exfil" sourcetype="http_logs" method=POST 
| stats count avg(bytes_sent) max(bytes_sent) min(bytes_sent) by domain 
| sort - count

```

The anomaly became clear. A single event with a massive `max(bytes_sent)` is a classic indicator of a file upload rather than a simple web request.

### Protocol Disguise

* **ICMP:** Normally used for connectivity checks. Seeing `R3@d0nly!2025` (a password) inside a ping packet is a 100% confirmation of data exfiltration.
* **FTP:** Plaintext protocols are a gift to SOC analysts. If it isn't encrypted (SFTP), we can see every password and file content in the packet bytes.

---

## 💡 Lessons Learned

* **Baselines are key:** You can't find an "unusually large" packet if you don't know what a "normal" packet looks like.
* **Contextual Analysis:** High DNS traffic isn't always a scan; it could be exfiltration if the subdomains look like encoded strings.
* **Defense in Depth:** This lab reinforced why sensitive data should be encrypted at rest; if the attacker exfiltrates it via plaintext, the data is immediately compromised.

