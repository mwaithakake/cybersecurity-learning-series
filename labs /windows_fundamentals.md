# TryHackMe: Windows Fundamentals Summary
**Last Updated:** March 2, 2026

A comprehensive overview of the core concepts covered in the Windows Fundamentals 1, 2, and 3 modules.

---

## 🖥️ Windows Fundamentals 1: The Basics
*This module focuses on the user interface, the file system, and basic management tools.*

### Windows Editions
* **Home:** Basic version for personal use.
* **Pro:** Includes advanced features like **BitLocker** and **Domain Join** for business environments.
* **Enterprise:** Designed for large organizations with advanced security and management capabilities.

### User Interface & Navigation
* **Desktop & Taskbar:** Managing the Start Menu, Search, Task View, and the Notification Area (System Tray).
* **System Folders:**
    * `C:\Windows`: Contains the Operating System files.
    * `C:\Program Files`: Installation location for 64-bit applications.
    * `C:\Users`: Contains user-specific profiles and data.



### The NTFS File System
* **Key Features:** Supports files larger than 4GB, file/folder permissions (ACLs), and encryption (EFS).
* **Alternate Data Streams (ADS):** A feature where a file can hide metadata or malicious code in a hidden stream (e.g., `file.txt:secret.txt`).

### Security & Management
* **UAC (User Account Control):** Prevents unauthorized system changes by requiring Administrator approval for high-level tasks.
* **Task Manager (`Ctrl+Shift+Esc`):** Used to monitor CPU, RAM, and disk usage, or to kill unresponsive processes.

---

## 🛠️ Windows Fundamentals 2: System Utilities
*This module dives into administrative tools used for troubleshooting and configuration.*

### Core Administrative Tools
* **Computer Management (`compmgmt.msc`):** A central console for managing various system tools.
* **Task Scheduler:** Used to automate scripts or programs based on specific times or triggers.
* **Event Viewer:** The system’s audit trail. Logs events categorized as *Critical, Error, Warning,* or *Information*.
* **Device Manager:** Manages hardware drivers and resolves hardware conflicts.

### Monitoring & Information
* **Resource Monitor (`resmon`):** Provides detailed, real-time data on CPU, Network, and Disk impact per process.
* **System Information (`msinfo32`):** A complete summary of hardware specs, environment variables, and software drivers.
* **System Configuration (`msconfig`):** Useful for troubleshooting the boot process and managing startup services.

### Advanced Configuration
* **Command Prompt (`cmd`):** Standard CLI for executing system commands.
* **Registry Editor (`regedit`):** A hierarchical database storing system and app settings. Organized into five "Hives."



---

## 🛡️ Windows Fundamentals 3: Security & Protection
*This module focuses on the built-in defensive mechanisms of the Windows OS.*

### Windows Security & Defender
* **Virus & Threat Protection:** Offers Quick, Full, and Custom scans to identify malware.
* **Real-time Protection:** Constantly monitors system activity for suspicious behavior.
* **Windows Updates:** Critical for maintaining security by patching known vulnerabilities.

### Network & Firewall
* **Firewall Profiles:**
    * **Domain:** Used when connected to a corporate network.
    * **Private:** Used for trusted home or work networks.
    * **Public:** Strictest settings for untrusted networks (e.g., airports).
* **Port Control:** Managing inbound and outbound rules to block or allow specific traffic.

### Hardware & Data Security
* **TPM (Trusted Platform Module):** A dedicated hardware chip that stores encryption keys.
* **BitLocker:** Full-disk encryption used to protect data on lost or stolen devices.
* **VSS (Volume Shadow Copy Service):** Creates snapshots of files even while they are in use; used for **System Restore**.

---

## ⌨️ Essential Commands Reference
| Tool | Run Command | Purpose |
| :--- | :--- | :--- |
| **System Info** | `msinfo32` | View hardware/software specs |
| **Registry** | `regedit` | Modify system configuration |
| **Management** | `compmgmt.msc` | Access Computer Management |
| **Events** | `eventvwr.msc` | View system logs |
| **Resources** | `resmon` | Detailed resource monitoring |
| **Configuration** | `msconfig` | Manage boot and services |

---
*Summary generated for educational purposes.*
