# Windows Hardening & Workstation Security

## 🛡️ Overview
This project focuses on the basic concepts and practical techniques required to harden a Windows workstation. It covers identity management, network security, application control, and storage protection to mitigate the risk of hacking and data breaches.

---

## 🏗️ Core Management Tools & Monitoring
Windows provides several built-in tools for system management and visibility into malicious activity.

* **Windows Services (`services.msc`):** Manages critical background functions (Local, Network, and System). 
* **Windows Registry (`regedit`):** A database for configuration settings. 
    * **Security Risk:** Malicious programs often target the **Run & Run Once** keys to execute during boot-up.
* **Event Viewer (`eventvwr`):** Centralized log database.
    * **Default Location:** `C:\WINDOWS\system32\config\folder`
    * **Categories:** Application (installed programs), System (OS components), and Security (authentication/security events).
* **Telemetry:** Data collection used by Microsoft via Universal Telemetry Client (UTC) and `diagtrack.dll`.
    * **Storage Path:** `%ProgramData%\Microsoft\Diagnosis`

---

## 👤 Identity & Access Management (IAM)
Protecting system access through the **Principle of Least Privilege**.

* **Account Types:** * **Standard:** For routine functions (browsing, MS Office).
    * **Admin:** Strictly for management (software installation, registry editing).
* **User Account Control (UAC):** Ensures applications run in non-administrator accounts to mitigate privilege escalation.
    * **Hardening:** Keep notification level at **"Always Notify."**
* **Local & Group Policy Editor:**
    * **Password Policies:** Set via `Security settings > Account Policies > Password policy`. Requires complexity and prevents leaked password use.
    * **Account Lockout Policy:** Configured via `Account Policies > Account Lockout Policy`. 
    * **Hardening:** Set to lock out after **3 invalid attempts**.

---

## 🌐 Network Management & Hardening
Reducing the network attack surface by disabling vulnerable protocols and securing traffic.

* **Windows Defender Firewall (`WF.msc`):**
    * **Profiles:** Domain, Private, and Public.
    * **Hardening:** Use a **"Default Deny"** rule for all incoming connections. Activate Private profile with "Blocked Incoming Connections."
* **SMB Protocol:** A file-sharing protocol often exploited in the wild.
    * **Disable Command (PowerShell):** `Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol`
* **Local DNS & ARP Security:**
    * **Hosts File:** Located at `C:\Windows\System32\Drivers\etc\hosts`. Attackers edit this to reroute traffic.
    * **ARP Poisoning:** Accepting unauthenticated ARP responses.
    * **Check ARP Table:** `arp -a`
    * **Clear ARP Cache:** `arp -d`
* **Remote Desktop (RDP):** Disable via `Settings > Remote Desktop` to prevent vulnerabilities like **BlueKeep**.
* **Device Manager:** Disable unused networking devices (routers, ethernet cards, WiFi adapters).

---

## 💻 Application Management
Securing software installation and execution environments.

* **Trusted Stores:** Only allow installations from the **Microsoft Store** (`ms-windows-store:`) to ensure software is vetted.
* **AppLocker:** Blocks specific executables, scripts, and installers based on publisher names or rules.
* **Windows Security (Defender):**
    * **Real-time protection:** Periodic system scanning.
    * **Application Guard:** Sandboxing web sessions.
    * **Controlled Folder Access:** Protects memory and folders from unwanted apps.
* **Microsoft Office Hardening:** Managed via `office.bat` to reduce the attack surface (e.g., disabling macros).
* **Browser (Edge):**
    * **SmartScreen:** Protects against phishing and malicious sites.
    * **Tracking Prevention:** Set to **"Strict"** in Settings.

---

## 🔒 Storage, Compute & Integrity
Ensuring data is encrypted and the system boots in a known-good state.

* **BitLocker:** Drive encryption requiring a **Trusted Platform Module (TPM)** chip.
* **Windows Sandbox:** A lightweight, isolated desktop environment for safely running suspicious applications.
* **Secure Boot:** UEFI-based standard that checks for trusted hardware and firmware before booting.
* **Backups:** Use **File History** (`Settings > Update and Security > Backup`) to prevent data loss from hardware failure or malware.
* **Windows Updates:** The most critical hardening part. Ensure auto-updates are enabled via `Settings > Update & Security > Windows Updates`.
