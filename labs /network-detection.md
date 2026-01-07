# Lab: Network Detection & Log Analysis (TryHackMe)
**Date:** 2026-01-07  
**Category:** Network Security / SOC Operations  

## 🎯 Objective
Analyze raw network logs (Zeek/Conn logs) to identify and distinguish between different stages of an adversary's reconnaissance phase: horizontal, vertical, and targeted scanning.

---

## 🛠️ Technical Breakdown & Methodology

### 1. Identifying the Horizontal Scan
**Concept:** Mapping "aliveness" across an entire network segment.
* **Pattern:** One source IP hitting multiple unique destination IPs on a single port.
* **Command:** `cut -d ',' -f5 *.csv | sort | uniq -c | sort -nr`
* **Observation:** A sequence of IPs from `203.0.113.1` through `.254` appeared with low hit counts.
* **Conclusion:** The scanned range was **203.0.113.0/24**.



### 2. Identifying the Vertical Scan
**Concept:** A "deep dive" into a single host to find all open ports/services.
* **Pattern:** A single destination IP appearing thousands of times with varying destination ports.
* **Command:** `cut -d ',' -f5 *.csv | sort | uniq -c | head`
* **Observation:** IP **192.168.230.145** showed over 6,000 log entries.
* **Conclusion:** This host was the primary target of a comprehensive port sweep.

### 3. Targeted "Common Services" Scan
**Concept:** A focused scan targeting specific high-value management ports to minimize detection.
* **Command:** `grep -w "192.168.230.1" log-session-2.csv | cut -d "," -f6 | sort -n | uniq`
* **Analytical Logic:** The `-w` (whole word) flag was critical here to isolate `.1` and ignore the noise from `.145`.
* **Results:** The attacker targeted ports **80 (HTTP)**, **445 (SMB)**, and **3389 (RDP)**.



---

## 🧠 Forensic Insights: Connection States

The lab required identifying the scan type based on the Zeek connection state **`S0`**.

* **What is S0?** This state indicates that a connection attempt (SYN packet) was seen, but no reply was ever sent back.
* **Analysis:** Because this occurred across thousands of ports, it confirms a **TCP SYN Scan** (also known as a Stealth or Half-Open scan).
* **TCP vs UDP:** UDP is a connectionless protocol; it does not use the SYN/ACK handshake. Therefore, the `S0` state is a specific indicator of a TCP-based handshake failure.



---

## 💡 Lessons Learned
1. **The Power of Grep Flags:** `-v` (invert) and `-w` (whole word) are essential for cleaning "dirty" logs where IP addresses overlap.
2. **Behavioral Analysis:** High-volume traffic (Vertical) is easy to spot, but low-volume traffic (Targeted) often requires deliberate filtering to uncover.
3. **Zeek States:** Understanding connection states like `S0`, `REJ`, and `SF` allows an analyst to determine not just *that* a scan happened, but *how* it was executed.

---
