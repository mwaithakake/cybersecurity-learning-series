# 🕵️ Wireshark: Packet Operations - Full Lab Analysis
**Date:** December 17, 2025


## 📖 The Curriculum: What this Lab Covers
The **Packet Operations** room is a comprehensive deep-dive into the "Heavy Lifting" phase of network forensics. Before getting into specific threats, it teaches a student how to manage, manipulate, and master large-scale traffic data. The lab is structured into four core learning domains:

### **1. Network Baselining & Statistics**
The lab starts by teaching you how to look at a PCAP from 30,000 feet. You learn to use the **Statistics Menu** to map out the network's logical and physical landscape.
* **Endpoints & Conversations:** Learning to identify the "Top Talkers" and map MAC addresses to manufacturers.
* **Resolved Addresses:** Bridging the gap between raw IP data and human-readable hostnames.
* **Protocol Hierarchy:** Analyzing the "weight" of the traffic—identifying how much of the capture is actual data (TCP) vs. network "chatter."

### **2. Advanced Filtering Operators**
This is where you move beyond simple IP filters. The lab introduces **Advanced Comparison Operators**:
* **`contains` vs `matches`:** Learning when to look for a specific word and when to use Regular Expressions (Regex) to find complex patterns.
* **Set Membership (`in`):** Learning to group multiple points of interest (like ports or IPs) into a single, high-performance search string.

### **3. Data Transformation & Functions**
The lab pushes you to use Wireshark as a data processor. You learn to use **Functions** to change how the tool "sees" packet headers.
* **Case Sensitivity:** Using `upper()` and `lower()` to ensure search results aren't missed due to formatting.
* **Type Casting:** Using the `string()` function to turn numerical data (like TTLs or Frame Numbers) into text so that pattern-matching logic can be applied to it.

### **4. Environmental Management (Profiles)**
Finally, the lab covers **Operational Efficiency**. You learn that an analyst's environment (Colors, Buttons, and Columns) should change based on the investigation. This includes learning to manage **Profiles** to isolate specific issues like Checksum errors without being overwhelmed by hardware artifacts.

---

## 🧠 My "Aha!" Moments: Navigating the Logic
While the curriculum is broad, the real learning happened when I hit the "Logic Walls" during the tasks. These are the moments where my understanding evolved.

### **The Logic Trap: Protocol Contradictions**
I faced a hurdle understanding why simple logic like `tcp && udp` failed. 
* **The Realization:** I learned that a packet cannot exist in two states simultaneously. This led me to master the **`in` operator**—a much cleaner and logically sound way to perform "Multi-Port Sweeps" across a network.

### **The Pattern Hunt: Teaching Wireshark to "Read"**
The challenge of finding **Even TTL numbers** was a major turning point.
* **The Struggle:** Wireshark sees TTL as a value, but Regex sees it as text. 
* **The Solution:** Using `string(ip.ttl) matches "[02468]$"` was the first time I successfully "cast" data from one type to another. It felt like unlocking a hidden layer of the tool's power.

### **The Checksum Mystery: Trusting the Triage**
In the **Checksum Control** profile, the screen was flooded with "Bad Checksum" errors.
* **The Confusion:** The errors looked real, but they were actually **Hardware Offloading** artifacts.
* **The Mastery:** I learned to move past the visual "noise" and trust the **Filter Buttons**. This taught me that a good analyst doesn't just look for errors—they look for *meaningful* errors.

---

## 🖼️ Forensic Evidence (Results Verification)
The following screenshots prove the successful application of the advanced filters and logic taught in this lab.

### **Service Fingerprinting (IIS v7.5)**
Evidence of isolating a specific legacy server version using Regex escaping.

> <img width="1015" height="823" alt="wireshark 2" src="https://github.com/user-attachments/assets/5355ac43-5834-4763-a2b5-67a40f9ecf52" />


### **The Port Sweep (`in` Operator)**
Evidence of using set membership to track multiple suspicious ports in a single string.

> <img width="1015" height="823" alt="wireshark 3" src="https://github.com/user-attachments/assets/07e3e17e-56ec-4987-b04b-04bfeda0dc2e" />


### **Mathematical Pattern Matching (Even TTLs)**
Evidence of casting integers to strings to isolate specific numerical patterns.
<img width="1015" height="823" alt="wireshark 4" src="https://github.com/user-attachments/assets/0bd4c7b0-9b5b-4e0d-8d7e-8d31499d6d74" />

### **Profile-Based Triage (Bad Checksums)**
Evidence of using the "Checksum Control" profile to isolate the 261 relevant packets from hardware noise.
<img width="1849" height="526" alt="wireshark 6" src="https://github.com/user-attachments/assets/0f2f5288-aae0-4890-a914-740a4ac4d7cf" />


---

## 🚀 Final Takeaway
This lab moved me from "searching" for data to "mining" it. The most important skill I walked away with is **Precision**. In a real-world SOC environment, being able to cut out 99% of the noise using advanced logic is the difference between finding a threat and missing it entirely.
