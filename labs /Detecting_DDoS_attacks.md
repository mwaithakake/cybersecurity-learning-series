# 🛡️ Detecting Web DDoS: Forensic Log Analysis
**Date:** February 3, 2026
**Topic:** Application Layer (Layer 7) DDoS Detection and Mitigation

---

## 📑 Understanding Web DDoS
Unlike network-layer attacks that flood bandwidth, a **Web DDoS** targets the **Application Layer (Layer 7)**. The goal is to overwhelm server-side resources like CPU, memory, or database connections by abusing legitimate web functions.

### Common Layer 7 Attack Types:
* **HTTP Flood:** Sending a massive volume of GET/POST requests to exhaust server threads.
* **Oversized Queries:** Forcing complex database lookups (e.g., through search bars) to lock up the backend.
* **Slowloris:** Keeping many connections open simultaneously by sending partial HTTP requests.
* **Cache Bypass:** Appending random parameters (e.g., `?cache=bypass`) to force the origin server to process every request instead of using a CDN.

---

## 🕵️ Forensic Investigation: Manual Log Analysis
Web server logs are the primary evidence in a DDoS investigation. When analyzing `access.log`, we look for high request rates and anomalies in headers.

### Key Indicators Found:
* **Attacker IP:** `203.12.23.195` identified as a primary source of high-frequency requests.
* **Targeted Page:** The `/login` endpoint was repeatedly hammered to abuse authentication logic.
* **Server Impact:** Legitimate users began receiving **HTTP 503 (Service Unavailable)** errors, indicating the server could no longer handle the load.


<img width="1854" height="967" alt="Screenshot_2026-02-03_05_36_36" src="https://github.com/user-attachments/assets/6a106a3d-b6ef-40d0-8dec-d6e209fa5a5f" />

> *Log Evidence: Massive spike of GET requests to /login.php from a single IP, resulting in service exhaustion.*

---

## 📊 SIEM Analysis (Splunk)
Scaling the investigation using a SIEM allows us to identify the full scope of a botnet.

### 1. Botnet Profiling
By analyzing the `uri_path`, we discovered that **81.8%** of all traffic was directed at the `/search` endpoint. This indicates a highly targeted Layer 7 flood.

* **Botnet Size:** Forensic filtering revealed **60 unique IP addresses** were part of the attacking cluster.
* **Top Attacking IP:** `203.0.113.7`.
* **Malicious User-Agent:** The most frequent agent used by the bots was `Java/1.8.0_181`.


<img width="1854" height="967" alt="Screenshot_2026-02-03_05_50_33" src="https://github.com/user-attachments/assets/fe0a31cc-f70f-4b3d-bae2-f87334c2e4b3" />
<img width="1854" height="967" alt="Screenshot_2026-02-03_05_44_56" src="https://github.com/user-attachments/assets/e79cad92-f7c1-44a9-abfa-a303095c8dd1" />

> *SIEM Statistics: Breakdown of the most active IPs and User-Agents involved in the flood.*

### 2. Visualizing the Attack Spike
Using the `timechart` command in Splunk, we can visualize the intensity of the flood.

* **Peak Intensity:** During the height of the attack, the server was hit with a peak of **207 requests per second**.
* **Victim Impact:** The first legitimate internal user (IP `10.10.0.27`) received a 503 error immediately following this peak.


<img width="1854" height="967" alt="Screenshot_2026-02-03_05_58_55" src="https://github.com/user-attachments/assets/ecf7336c-263e-4f9a-ba1d-659964f58d5b" />

> *Visualization: The vertical "wall" of traffic at 1:12 PM represents the exact moment the botnet began the search-flood.*

---

## 🛡️ Mitigation Strategies
Defending against Layer 7 attacks requires a combination of network and application-level controls:

1. **WAF (Web Application Firewall):** Implement rate-limiting rules (e.g., "Block IP if requests to /login exceed 10 per minute").
2. **CAPTCHAs:** Use puzzles to distinguish between automated bot traffic and real human users.
3. **Load Balancing:** Distribute incoming traffic across multiple backend servers to prevent a single point of failure.
4. **CDN (Content Delivery Network):** Cache static content at the edge to reduce the load on the origin server.

---

## 🚀 Conclusion
DDoS detection isn't just about high traffic—it's about identifying **intent**. By correlating specific URI paths (`/search`, `/login`) with unusual User-Agents (`Java`, `python-requests`), we can distinguish a botnet from a legitimate traffic spike.

---
