
# 🛡️ Networking Security & Advanced Segmentation Lab

## 📖 Overview
Networking is a critical but often overlooked component of corporate security. A well-designed network doesn't just permit communication; it provides **redundancy, optimization, and security boundaries**. This lab explores the transition from functional networking to a secure architecture using Layer 2 segmentation, Zoning, and Advanced Inspection protocols.

---

## 🏗️ Phase 1: Secure Segmentation & VLANs
Standard subnetting allows for basic communication but fails as a security boundary. A compromised **BYOD (Bring Your Own Device)** can easily traverse a subnetted network if routes exist.

### 🔹 VLANs (Virtual LANs)
VLANs segment a network at **Layer 2**. By "tagging" frames with the **IEEE 802.1q (dot1q)** standard, we can isolate devices even if they share the same physical switch.
* **Native VLANs:** Handle untagged traffic, ensuring every packet has a designated home.
* **ROAS (Router on a Stick):** Uses a single physical **Trunk** interface divided into **Virtual Sub-interfaces** (e.g., `eth0.10`) to route traffic between isolated VLANs.

### 💻 Configuration (Open vSwitch & VyOS)
```bash
# Open vSwitch: Assigning a VLAN tag to a port
ovs-vsctl set port eth2 tag=10

# VyOS: Configuring a Router-on-a-Stick sub-interface
set interfaces ethernet eth0 vif 10 address '192.168.100.1/24'
```

---

## 🗺️ Phase 2: Security Zoning
VLANs provide the walls, but **Security Zones** define the trust levels.

| Zone | Purpose | Example Assets |
| :--- | :--- | :--- |
| **External** | Untrusted/Outside control | Public Web Users |
| **DMZ** | Public-facing services | Web Servers, Mail Servers |
| **Trusted** | Internal standard devices | Employee Workstations |
| **Restricted** | High-value/High-risk data | Domain Controllers (AD), Databases |
| **Audit/Mgmt** | Monitoring & Management | SIEM, Backup Servers |

---

## 🚦 Phase 3: Traffic Filtering (ACLs)
To enforce the boundaries between zones, we use **Access Control Lists (ACLs)**. These consist of **ACEs (Access Control Entries)** that permit or deny traffic based on Source/Destination IPs and Ports.

* **Logic:** The router evaluates rules from top to bottom. Once a match is found, the action (Permit/Deny) is taken.
* **Limitation:** Basic ACLs are "Stateless," meaning they don't track the context of a conversation.

---

## 🔐 Phase 4: Advanced Inspection & Layer 2 Defense
Security must exist below the IP layer to prevent "invisible" attacks like spoofing and rogue services.

### 🔍 SSL/TLS Inspection
Attackers often hide Command & Control (C2) traffic in encrypted HTTPS. 
* **Mechanism:** The firewall acts as a **Proxy (MiTM)**, decrypting traffic to scan it via **UTM (Unified Threat Management)** before re-encrypting it.
* **Trade-off:** High visibility vs. Privacy/Performance overhead.

### 🛡️ DHCP Snooping & DAI
These protocols protect the integrity of network identities:
* **DHCP Snooping:** Builds a **Binding Database** (MAC + IP + Port). It kills "DHCP Offers" from untrusted ports to prevent **Rogue DHCP Servers**.
* **Dynamic ARP Inspection (DAI):** Uses the DHCP Binding Database to validate ARP packets. If an attacker tries to spoof a MAC address (ARP Poisoning), the switch detects the mismatch and drops the packet.

---

## 🎓 Key Takeaways
1.  **VLANs ≠ Security:** Without ACLs or Firewalls, VLANs are just organizational tools.
2.  **Layer 2 Matters:** Defenses like DAI are required to stop attackers from stealing the "Identity" of the Gateway or other hosts.
3.  **Zero Trust Foundations:** Proper Zoning ensures that even if one segment (like the DMZ) is breached, the "Restricted" zone remains isolated.


