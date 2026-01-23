## Snort Room Write-Up
**Date:** January 23, 2026

**Tools:** Snort v2.9.7.0, Linux CLI, Traffic Generator, Nano

---

## 🚀 Overview
In this lab, I mastered **Snort**, a powerful open-source Network Intrusion Detection and Prevention System (IDS/IPS). I moved beyond basic packet sniffing into advanced traffic analysis, custom rule creation, and active threat prevention.

## 🛠️ Phase 1: Environment Setup & Validation
First, I verified the Snort installation and tested the configuration file to ensure the engine was ready for traffic analysis.

<img width="986" height="568" alt="snort 1" src="https://github.com/user-attachments/assets/5d4cce34-8c44-4283-b7df-f20ec1f09098" />

> **Screenshot 1: Confirming Snort Version** > `snort -V`


I used the `-T` flag to validate the configuration. This is a critical step in a production environment to prevent the service from crashing due to syntax errors.
```bash
sudo snort -c /etc/snort/snort.conf -T

```
---

## 🔍 Phase 2: Traffic Analysis (IDS Mode)

I explored how Snort processes pre-recorded traffic (`.pcap` files). This is vital for forensic analysis after a security incident.

```bash
snort -r task9.pcap -n 10

```
<img width="997" height="706" alt="Screenshot from 2026-01-23 10-13-05" src="https://github.com/user-attachments/assets/fce7eef8-2b63-41e5-bfbb-9ecad4bee7cd" />


> **Screenshot 3: Reading PCAP data**

### 💡 The "Aha!" Moment: Solving Permission Issues

During the "ASCII Logging" task, I ran into a common real-world hurdle. When running Snort with `sudo`, the output log folders were owned by the `root` user, preventing my standard `ubuntu` user from accessing them.

**The Fix:** I used the `chown` (Change Ownership) command with the `-R` (recursive) flag to take ownership of the logs back, allowing me to analyze the data without staying in a dangerous root shell.

```bash
# Giving my user ownership of the generated logs
sudo chown ubuntu:ubuntu -R .

```

This allowed me to enter the `145.254.160.237` folder and identify that the **Source Port** connecting to **DNS (Port 53)** was found in the `UDP` log files.

---

## 🛡️ Phase 3: Moving to IPS (Prevention) Mode

I shifted from passive observation to active defense. Using the **Data Acquisition (DAQ)** module, I turned Snort into a bridge that can drop malicious packets.

```bash
sudo snort -c /etc/snort/snort.conf -q -Q --daq afpacket -i eth0:eth1 -A console

```

In this mode, instead of just "Alerting," Snort showed `[Drop]`, effectively stopping the simulated ICMP/HTTP attacks from reaching their target.

---

## ✍️ Phase 4: Custom Rule Engineering

The highlight of the lab was writing my own detection signatures in `local.rules`.
<img width="1022" height="843" alt="snort 4" src="https://github.com/user-attachments/assets/6318702c-fe67-4c9c-bf85-96eccfe7c360" />
<img width="1855" height="883" alt="snort3" src="https://github.com/user-attachments/assets/05d1cfa8-08cc-4bdb-bb4e-a6becad16bf6" />

> **Screenshot 4: Writing Custom Rules in Nano**

I experimented with several rule types:

1. **Basic Alert:** Detecting any TCP traffic.
2. **Flag Detection:** Filtering for specific TCP flags like `S` (SYN) or `PA` (PUSH-ACK).
3. **Logical Filters:** Using the `sameip` option to detect packets where the source and destination are identical.

```bash
# Example of my custom UDP rule with same-IP detection
alert udp any any <> any any (msg:"SAME-IP"; sameip; sid:100001; rev:1; )

```
<img width="1852" height="773" alt="snort 5" src="https://github.com/user-attachments/assets/ab88df5a-9907-47b2-bc13-112704cd6500" />

> **Screenshot 5: Successful Rule Loading**

---

## 🧠 Key Takeaways

* **Ownership Matters:** Running security tools with `sudo` often creates "permission silos." Knowing how to use `chown -R` is just as important as knowing the tool itself.
* **The Power of `-K ASCII`:** Binary logs are great for Wireshark, but ASCII logging is superior for quick grep-ing and human-readable analysis of specific IP-based sessions.
* **Inline vs. Passive:** IDS (Passive) is for visibility; IPS (Inline/DAQ) is for control. Moving traffic through interfaces like `eth0:eth1` is what enables the "Drop" action.
* **Rule Precision:** Using `sid` (Signature ID) starting from `1000001` ensures custom rules don't conflict with built-in Snort signatures.

---

