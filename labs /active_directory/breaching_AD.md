# Lab Report: Active Directory Initial Access & Infrastructure Exploitation

**Date:** April 2026  
**Environment:** Kali Linux (Attacker), Windows 10 (THMJMP1), TryHackMe AD Lab

---

## 1. Phase 1: Initial Reconnaissance & Network Poisoning
The engagement began with identifying misconfigured services on the local network to harvest initial credentials.

### 1.1 Web Portal Discovery (Printer Server)
I located a printer management portal (`printer.za.tryhackme.com`) that supported LDAP authentication for administrative settings. 
* **The Vulnerability:** The "LDAP Server" field could be pointed to an arbitrary IP. By changing this to my Kali IP (**10.150.70.10**), I could force the server to attempt an LDAP bind against my machine, exposing credentials or hashes.


<img width="1854" height="967" alt="printer_server" src="https://github.com/user-attachments/assets/63e11b2d-580e-4f10-9124-0732821fd49a" />

### 1.2 Packet Analysis with tcpdump
To verify the connection attempts from the printer server, I utilized `tcpdump` to monitor incoming traffic on port 389 (LDAP).
* **Command:** `sudo tcpdump -SX -i breachad tcp port 389`
* **Observation:** The packet capture confirmed the printer server attempting to authenticate. I observed the transition from the initial TCP three-way handshake to the LDAP bind request, identifying clear-text strings and protocol headers.


<img width="1854" height="967" alt="tcpdump" src="https://github.com/user-attachments/assets/fb70b015-7f5c-4254-b474-a92a5cd90209" />


### 1.3 LLMNR/NBT-NS Poisoning (Responder)
I launched `Responder` to poison legacy name resolution requests.
* **Execution:** `sudo responder -I breachad`
* **Result:** Captured a **Net-NTLMv2** hash for the user `ZA\svcFileCopy`. 
* **Cracking:** Using `hashcat` with mode `5600`, I cracked the hash against `passwordlist.txt`.
* **Loot:** Recovered the password **`Password1!`**.


<img width="1854" height="967" alt="hash" src="https://github.com/user-attachments/assets/f192f4f7-d725-4e93-81a2-4d07db893bc8" />


---

## 2. Phase 2: Domain Foothold via Password Spraying
With a known valid password format and a list of domain users, I transitioned to a password spray attack to escalate access.

* **Attack Vector:** Exploiting the `ntlm_passwordspray.py` script to test the password `Changeme123` across the `ZA.TRYHACKME.COM` domain.
* **Success:** The spray identified **4 valid credential pairs**, including `hollie.powell`, `heather.smith`, `gordon.stevens`, and `georgina.edwards`.
* **Outcome:** This provided multiple entry points into the domain without triggering account lockout thresholds.


<img width="1854" height="967" alt="password_spraying" src="https://github.com/user-attachments/assets/1c281cfa-e26e-404c-80da-ca7b8bdaacd4" />


---

## 3. Phase 3: Infrastructure Scraping (MDT/PXE)
I targeted the Microsoft Deployment Toolkit (MDT) to find high-value service account credentials stored in deployment images.

* **Recon:** Used `PowerPXE.ps1` on the `THMJMP1` host to parse the Boot Configuration Data (BCD).
* **Extraction:** 1. Identified the path: `\Boot\x64\Images\LiteTouchPE_x64.wim`.
    2. Used `tftp` to exfiltrate the 350MB `.wim` file.
    3. Ran `Get-FindCredentials` against the exfiltrated image.
* **Loot:** Scraped clear-text credentials for `svcMDT`: **`PXEBootSecure1@`**.


<img width="1854" height="967" alt="power_pxe(2)" src="https://github.com/user-attachments/assets/eabe86ef-29ab-4e43-aded-6b13aa2ec9ce" />


---

## 4. Phase 4: Local Post-Exploitation (McAfee ma.db)
The final phase involved extracting credentials from the McAfee Agent local database on the compromised Windows 10 host.

### 4.1 Environment and Script Debugging
I exfiltrated `ma.db` using `scp`. To facilitate local LDAP authentication changes required for the attackbox, I modified `olcSaslSecProps.ldif` and restarted the `slapd` service.

> **[INSERT SCREENSHOT: ldif(2).jpg]**
<img width="1854" height="967" alt="ldif(2)" src="https://github.com/user-attachments/assets/97d70b0a-b1dc-4144-b96b-2c4a892929b6" />

### 4.2 Python 3 Decryption Pivot
The legacy decryption script failed with a `TypeError` due to Python 3's handling of integers vs strings in XOR operations.
* **Code Correction:** Manually edited `mcafee_sitelist_pwd_decrypt.py` to handle the byte-encoding properly.
* **Result:** Successfully decrypted the `AUTH_PASSWD` blob.
* **Loot:** Recovered the password for `svcMcAfee`: **`MyStrongPassword!`**.

> **[INSERT SCREENSHOT: ma.db(2).jpg]**
<img width="1854" height="967" alt="ma db(2)" src="https://github.com/user-attachments/assets/d3cafefb-167e-4dfe-9f31-ee8927c57770" />


---

## 5. Technical Skills Summary
* **Protocol Exploitation:** Poisoning LLMNR/NBNS and intercepting LDAP traffic with `tcpdump`.
* **Credential Maneuvering:** Executing successful password sprays and hash cracking with `hashcat`.
* **Forensic Scraping:** Mounting and analyzing WIM images for deployment secrets.
* **Developer Pivot:** Debugging and refactoring legacy Python cryptographic code for modern environments.
