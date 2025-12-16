
# 🗺️ Nmap Deep Dive: Host Discovery, Scanning & Reporting


**TryHackMe: Nmap Fundamentals - Deep Dive** (2025-12-16)

---

### 🌟 Purpose

This documentation summarizes key features of Nmap (Network Mapper), focusing on network reconnaissance techniques, scan types (Connect vs. SYN), advanced detection, and efficient output handling necessary for security assessments and network audits. Running Nmap as `root` (or using `sudo`) is essential to avoid restricting its abilities, such as the SYN scan.

### 🎯 I. Host Discovery (Who is Online?)

The initial step in any scan is finding live hosts. Nmap provides several ways to define and discover targets.

#### **Target Specification Methods**

| Method | Example | Description |
| :--- | :--- | :--- |
| **IP Range** | `192.168.0.1-10` | Scans IPs from 1 to 10 in the specified range. |
| **IP Subnet (CIDR)** | `192.168.0.1/24` | Scans all 256 IPs in the subnet (0-255). |
| **Hostname** | `example.thm` | Scans the target by its domain/host name. |

#### **The Ping Scan (`-sn`)**

The `-sn` (Skip Port Scan, or Ping Scan) option is used to quickly identify active hosts without performing a full port scan. 

* **Local Network Scanning:** When scanning a local (directly connected) network, Nmap starts by sending **ARP requests**. When a device responds, Nmap labels the host as "up" and can determine its MAC address and vendor.
* **Remote Network Scanning:** When separated by one or more routers, Nmap cannot use ARP. Instead, it sends multiple probes (e.g., ICMP echo requests, TCP SYN to port 443, TCP ACK to port 80, etc.). The host is marked "up" if it responds to any of these probes.

| Option | Explanation |
| :--- | :--- |
| **`-sn`** | **Ping Scan:** Discover live hosts without running a port scan. |
| **`-sL`** | **List Scan:** Only lists the target IPs to be scanned without sending any packets; useful for confirming target range. |
| **`-Pn`** | **No Ping:** Forces Nmap to treat *all* hosts as online, bypassing the discovery phase to start port scanning immediately, which is useful for hosts that block ICMP. |

---

### 🔎 II. Port Scanning Techniques

Nmap focuses on discovering network services by checking the 65,535 possible TCP/UDP ports.

#### **A. TCP Connect Scan (`-sT`)**

* **Description:** This is the most basic and "loudest" scan. It works by completing the full **TCP three-way handshake** (SYN $\rightarrow$ SYN-ACK $\rightarrow$ ACK) with the target port.
* **Open Port Logic:** If the port is open, the full handshake completes. Nmap then immediately **"tears down"** the established connection by sending a **RST-ACK** packet.
* **Closed Port Logic:** The target immediately sends a **RST-ACK** packet in response to the initial SYN.
* **Vulnerability:** Since the full handshake occurs, this scan leaves clear session entries in the target system's logs, making it easy to detect.

#### **B. TCP SYN Scan (Stealth) (`-sS`)**

* **Description:** Considered "stealthy" because it only executes the first step of the handshake (often called a "half-open" scan).
* **Open Port Logic:** 
    1.  Nmap sends a **SYN** packet.
    2.  The target sends **SYN-ACK** (confirming the port is open).
    3.  Nmap immediately responds with a **RST** (Reset) packet instead of the final ACK.
* **Advantage:** Since the connection is never fully established, this process generates far fewer application-level logs on the target system, making it less detectable than a Connect Scan.

#### **C. UDP Scan (`-sU`)**

* **Description:** Used to find services listening on connectionless UDP ports (e.g., DNS, DHCP, NTP).
* **Open Port Logic:**
    * If Nmap sends a UDP packet to a port and gets **no response**, the port is typically considered **Open|Filtered** (the port may be open, or a firewall may be blocking the response).
    * If the target sends an **ICMP Destination Unreachable (Port Unreachable)** message, the port is confirmed **Closed**.

| Port Scan Option | Protocol | Speed / Stealth |
| :--- | :--- | :--- |
| **`-sT`** | TCP | Slow, Loud (Full Handshake) |
| **`-sS`** | TCP | Fast, Stealthy (Half-Open) |
| **`-sU`** | UDP | Often Slow, Complex Response |
| **`-F`** | Fast Mode | Scans the 100 most common ports instead of the default 1000. |
| **`-p-`** | All Ports | Scans all 65,535 ports (`-p1-65535`). |

---

### 🧠 III. Advanced Detection and Screenshots

#### **A. Service and Version Detection (`-sV`)**

The `-sV` flag is used to determine the exact application and version of the service running on an open port.

**Target: 10.81.168.135**

The scan reveals critical service information:

| PORT | STATE | SERVICE | VERSION |
| :--- | :--- | :--- | :--- |
| 7/tcp | open | echo | |
| 9/tcp | open | tcpwrapped | |
| 13/tcp | open | nagios-nsca | Nagios NSCA |
| 17/tcp | open | daytime | |
| 22/tcp | open | ssh | OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0) |
| **8008/tcp** | **open** | **http** | **lighttpd 1.4.74** |

Screenshot available here:
<img width="1849" height="455" alt="Screenshot from 2025-12-16 22-06-55" src="https://github.com/user-attachments/assets/f252841b-2582-455d-adf8-28303326fd7e" />


* **Observation:** The web server is confirmed to be **lighttpd version 1.4.74** on port **8008**. This specific version information is crucial for later vulnerability research.

#### **B. OS Detection (`-O`) and Aggressive Scan (`-A`)**

* **`-O` (OS Detection):** Triggers Nmap to guess the target operating system based on network stack responses (e.g., "Linux 4.15 - 5.8"). Note that this relies on various indicators and may not be perfectly accurate.
* **`-A` (Aggressive Scan):** Enables OS detection, version detection, traceroute, and common script scanning simultaneously, providing the most exhaustive results.

#### **C. Connect Scan Proof**

The initial Connect Scan (`-sT`) confirmed the following open service ports on `10.80.173.224`:

```bash
nmap -sT 10.80.173.224

```

| PORT | STATE | SERVICE |
| --- | --- | --- |
| 22/tcp | open | ssh |
| **8008/tcp** | **open** | **http** |

<img width="917" height="428" alt="Screenshot from 2025-12-16 18-42-34" src="https://github.com/user-attachments/assets/dfb75a86-3891-47c1-b561-ce5434a3a360" />


* **Observation:** This scan confirmed the web server on port **8008**, which was accessed via `http://10.80.173.224:8008`.

---

###🕰️ IV. Timing and Stealth ControlControlling scan speed is essential to avoid triggering security defenses like IDS/IPS.

| Option | Template | Explanation |
| --- | --- | --- |
| **`-T0`** | **Paranoid (0)** | Extremely slow (e.g., 5 minutes between probes). Highest stealth. |
| **`-T1`** | **Sneaky (1)** | Very slow (e.g., 15 seconds between probes). |
| **`-T3`** | **Normal (3)** | Default setting. Balances speed and reliability. |
| **`-T4`** | **Aggressive (4)** | Faster, assumes fast network, ideal for quick results. |
| **`--min-rate`** | (Number) | Sets the minimum packets per second for the scan. |
| **`--host-timeout`** | (Time) | Maximum amount of time to wait for a target host. |

---

###💾 V. Output and Verbosity| Option | Function | Format |
| --- | --- | --- |
| **`-v`** | **Verbosity:** Increases the level of detail shown during the scan (use `-vv` for more detail). | Terminal Output |
| **`-oA <basename>`** | **Output All:** Saves results in three major formats using a base filename. | `.nmap`, `.xml`, `.gnmap` |
| **`-oX <filename>`** | **XML Output:** Saves results in machine-readable XML format, useful for parsing by other tools. | `.xml` |

---

###📝 VI. Reflection: SYN vs. ConnectMy initial confusion between the Connect (`-sT`) and SYN (`-sS`) scans was a key learning moment in understanding network logging and stealth.

* **The Difference is the Packet:** The difference boils down to the final packet exchange after an open port is detected.
* **Connect Scan (`-sT`):** Completes the full TCP handshake (SYN, SYN-ACK, ACK). This forces the host to fully commit resources to the session, which is easily logged by the operating system and applications.
* **SYN Scan (`-sS`):** Sends a RST packet immediately after receiving the SYN-ACK. This **resets the session** before the handshake is finalized, leading to a much lower chance of being logged by higher-level security solutions. This is why it is truly considered a stealth scan.

```

```
