### 🔥 Latest Entry

**Splunk Log Analysis & SPL Mastery** (2026-01-08)

#### [Splunk: The Basics](https://tryhackme.com/room/splunkbasics)
* **Core Concepts:** Explored the three-tier architecture of Splunk: **Forwarders** for data collection, **Indexers** for storage/parsing, and **Search Heads** for visualization.
* **Log Ingestion:** Successfully ingested raw JSON VPN logs by creating a custom index (`vpn_logs`). This process involved configuring data types and ensuring proper field extraction for analysis.
* **Security Investigation:**
    * Filtered traffic to identify anomalies; found **2,814 events** originating from countries outside of France using the `!=` (not equal) operator.
    * Isolated **14 security events** tied to the suspicious IP `107.3.206.58` to track potential unauthorized access.

      
<img width="1854" height="967" alt="Screenshot_2026-01-08_04_02_41" src="https://github.com/user-attachments/assets/17edf9f4-fdb7-4778-9d89-eb45654824cb" />

<img width="1854" height="967" alt="Screenshot_2026-01-08_04_31_26" src="https://github.com/user-attachments/assets/cb17fac5-730e-4d24-8b72-4b8e2c5c5813" />

> *Search results for IP 107.3.206.58 and VPN_logs index overview.*

#### [Splunk: Exploring SPL](https://tryhackme.com/room/splunkexploringspl)
* **Search Optimization:** Moved beyond simple keywords to using **Boolean operators** (AND, OR, NOT) and **Wildcards** (`*`) to filter complex Windows Event Logs.
* **Data Transformation:** Practiced using the pipe operator (`|`) to chain commands. Specifically used `table` to clean up noisy data and `stats` to aggregate event counts by user and host.
* **Result Structuring:** Applied ordering commands including `sort`, `dedup`, `head`, and `tail` to organize forensic evidence chronologically.
    * **Example Query:** `index=windowslogs | table _time, EventID, Hostname | sort - _time`

> <img width="1854" height="967" alt="Screenshot_2026-01-08_04_54_15" src="https://github.com/user-attachments/assets/742e9dda-55f5-41ff-881a-aa5ddd2bc184" />

<img width="1854" height="967" alt="Screenshot_2026-01-08_07_18_40" src="https://github.com/user-attachments/assets/1c1dcfd9-263a-460b-9515-dfa62d3f65a9" />



---
