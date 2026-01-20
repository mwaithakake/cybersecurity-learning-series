# Cybersecurity Learning Series: Firewalls 🛡️
**Date:** January 20, 2026

## 📝 Overview
Today I explored the fundamental role of firewalls in digital security. Much like security guards at a physical building entrance, a firewall acts as a barrier between a trusted internal network and untrusted external networks (like the Internet). It inspects incoming and outgoing traffic and decides whether to allow or deny it based on a set of defined security rules.

## 🎯 Learning Objectives
* Understood the different types of firewalls and their OSI layer operations.
* Learned the core components of firewall rules (Source, Destination, Port, Protocol, Action).
* Performed hands-on configuration of the **Windows Defender Firewall**.
* Explored Linux firewall utilities, specifically **UFW (Uncomplicated Firewall)**.

---

## 🧱 Types of Firewalls
Firewalls are categorized by how they handle data and which OSI layer they operate on:

| Type | OSI Layer | Characteristics |
| :--- | :--- | :--- |
| **Stateless** | Layer 3 & 4 | Filters based on rules; doesn't remember previous connection states. Fast but less secure. |
| **Stateful** | Layer 3 & 4 | Keeps a "state table" of active connections. Recognizes patterns and is more secure than stateless. |
| **Proxy (App-level)** | Layer 7 | Acts as an intermediary; inspects actual packet content and masks internal IP addresses. |
| **Next-Gen (NGFW)** | Layer 3 - 7 | Advanced protection including Deep Packet Inspection (DPI) and Intrusion Prevention Systems (IPS). |

---

## ⚙️ Firewall Rule Components
A rule is an instruction given to the firewall. Key components include:
* **Action:** Allow, Deny, or Forward.
* **Direction:** Inbound (incoming) or Outbound (outgoing).
* **Scope:** Source/Destination IP addresses and Port numbers (e.g., Port 80 for HTTP).

---

## 🖥️ Practical Lab: Windows Defender Firewall
Windows uses "Network Profiles" (Private vs. Public) to apply different security levels automatically.

### Blocking HTTP/HTTPS Traffic
I practiced creating a **Custom Outbound Rule** to block web browsing (Ports 80 and 443).

1. **Rule Setup:** Defined the protocol as TCP and targeted specific remote ports: 80, 443.
   <img width="540" height="685" alt="f1" src="https://github.com/user-attachments/assets/d9e93f42-ec6e-4fd7-ab8d-1e8a0b36a7d2" />

*(Applied here: Protocols and Ports configuration showing TCP 80, 443)*

3. **Verification:** Before the rule, websites were accessible. After enabling the "Block" action, the browser correctly showed a "Can't reach this page" error.
<img width="1265" height="277" alt="f2" src="https://github.com/user-attachments/assets/fe292e2a-b0cb-4c3c-965c-6d908d346230" />

*(Applied here: View of the Outbound Rules list showing the active block rule)*

<img width="841" height="673" alt="f3" src="https://github.com/user-attachments/assets/0872a3bc-89c4-48fd-b96c-db9abac9c75e" />

*(Applied here: The result of the block rule in a browser)*

---

## 🐧 Practical Lab: Linux (UFW)
Linux firewalls often sit on the **Netfilter** framework. I used **UFW** (Uncomplicated Firewall) for easier management.

### Common Commands Used:
* **Check Status:** `sudo ufw status`
<img width="720" height="157" alt="Screenshot from 2026-01-20 10-52-02" src="https://github.com/user-attachments/assets/3cb5d4a6-9147-407d-b1bd-cd8bf37429b4" />


* **Enable Firewall:** `sudo ufw enable`
<img width="728" height="169" alt="Screenshot from 2026-01-20 10-52-29" src="https://github.com/user-attachments/assets/0922a703-630e-4697-9497-24c736230c76" />


* **Set Default Policy:** To allow all outgoing traffic by default.
<img width="720" height="157" alt="Screenshot from 2026-01-20 10-52-11" src="https://github.com/user-attachments/assets/e6d17da0-c059-45ba-a702-67abe5196d98" />


* **Deny Specific Traffic:** To block incoming SSH (Port 22) connections.
<img width="728" height="131" alt="f4" src="https://github.com/user-attachments/assets/7374066f-6556-4bf0-a7df-db83e0b21e32" />


---

## 💡 Key Takeaway
A firewall is only as good as its rules. Whether using Windows or Linux, the ability to control exactly what enters and leaves a system is one of the most vital skills in network security.

```
