
# 🔍 TryHackMe: Tcpdump Traffic Analysis - Deep Dive

## 🔥 Latest Entry

**TryHackMe: Tcpdump Traffic Analysis - Deep Dive** (2025-12-16)

---

### 🌟 Purpose

This document serves as a comprehensive guide and technical documentation of the `tcpdump` command-line tool, detailing its use for capturing, filtering, and analyzing fundamental network protocol exchanges. This knowledge is foundational for roles in threat detection and security operations.

### 📚 I. Fundamentals: Capture and Display Arguments

Modern interfaces abstract network protocol complexities. `tcpdump` is built on the `libpcap` library and provides the essential window into these conversations. To use it, we must specify **what** to listen to and **how** to display the output.

| Argument | Syntax | Description |
| :--- | :--- | :--- |
| **Interface** | `-i INTERFACE` / `-i any` | Specifies which network interface to monitor (e.g., `eth0` or `ens5`). |
| **File Operations** | `-w FILE.pcap` / `-r FILE.pcap` | Saves captured packets to a file (`-w`) or reads from an existing file (`-r`). |
| **Limit Count** | `-c COUNT` | Stops the capture after a specific number of packets. |
| **No Resolution** | `-n` / `-nn` | Prevents DNS and/or port number resolution (improves speed). |
| **Verbose Output** | `-v` / `-vv` / `-vvv` | Increases the level of detail displayed. |
| **Link-Level** | `-e` | Includes MAC addresses (link-level header) in the output. |
| **Raw Data** | `-A` / `-xx` / `-X` | Displays packet data in ASCII (`-A`), Hexadecimal (`-xx`), or both (`-X`). |

### 🔎 II. Advanced Filtering with BPF Syntax

Filtering is critical for reducing thousands of packets down to a relevant dataset. Filters use **BPF (Berkeley Packet Filter)** syntax.

#### **Core Filter Primitives**

| Filter Category | Syntax | Example |
| :--- | :--- | :--- |
| **By Host/Port** | `host IP and dst port 80` | Filters for HTTP traffic destined for a specific IP. |
| **By Protocol** | `PROTOCOL` | `tcpdump icmp` (filters for ICMP traffic) |
| **Length Filter** | `'greater LENGTH'` | `tcpdump 'greater 15000'` (finds large packets) |

#### **Logical Operators**

Filters are combined using `and`, `or`, and `not`.

```bash
# Example: Capture all traffic except TCP segments
tcpdump not tcp
````

#### **Filtering by TCP Flags**

This powerful technique uses the `tcp[tcpflags]` primitive and binary logic to check for specific connection states.

| Use Case | Filter Expression |
| :--- | :--- |
| **Check for ONLY SYN** | `tcp "tcp[tcpflags] == tcp-syn"` |
| **Check for AT LEAST SYN** | `tcp "tcp[tcpflags] & tcp-syn != 0"` |
| **Check for SYN or ACK** | `tcp "tcp[tcpflags] & (tcp-syn|tcp-ack) != 0"` |

-----

### 🔬 III. Evidence Screenshots & Protocol Analysis

The following section presents the practical outputs from the lab, confirming the successful use of advanced `tcpdump` filtering to isolate and analyze specific protocols within the `traffic.pcap` file.

#### **A. ARP Analysis: Link-Level Exchange**

The command used the `-e` (link-level header) and `arp` filter to isolate MAC address resolution traffic.

```bash
tcpdump -r traffic.pcap -e arp
```
<img width="738" height="249" alt="Screenshot from 2025-12-16 12-43-32" src="https://github.com/user-attachments/assets/a1cf9e8d-6965-4c86-a06e-252cedc92c6d" />


  * **Observation:** The output clearly shows the **ARP Request** (Broadcast) asking "who-has" an IP, followed by the **ARP Reply** providing the target's MAC address. This verifies the fundamental IP-to-MAC mapping process.

#### **B. DNS Analysis: Query Inspection**

To inspect the raw DNS query structure, the command filtered for `udp port 53` and used high verbosity (`-vvv`) with no name resolution (`-n`).

```bash
tcpdump -r traffic.pcap -n -vvv udp port 53 -c 1
```
<img width="1690" height="155" alt="Screenshot from 2025-12-16 11-20-53" src="https://github.com/user-attachments/assets/62c55000-f53d-47db-a2d0-532329b06e32" />


  * **Observation:** The packet log confirms a DNS query for an A record (`A? mirrors.rockylinux.org`). The source port is high (`33672`), and the destination is the standard DNS port (`53`), confirming a standard client-server query.

#### **C. TCP Flag Count: Detecting Resets (RST)**

To count the number of TCP connection resets, the `tcp[tcpflags]` primitive was used to filter for packets with **only** the RST flag set, and the output was piped to `wc` for a count.

```bash
sudo tcpdump -r traffic.pcap 'tcp[tcpflags] == tcp-rst' | wc
```
<img width="991" height="156" alt="Screenshot from 2025-12-16 12-08-00" src="https://github.com/user-attachments/assets/7418dacb-f247-456b-b821-25be0d4b9b99" />


  * **Observation:** The command returned **57 lines**, confirming that **57 packets** in the capture file had **only** the RST flag set. This signifies abrupt connection closures, often indicating a port scanner, a firewall blocking access, or a host refusing the connection.


<!-- end list -->

```
