# Windows Threat Detection Series — Part 1: Initial Access

## 1. Overview of Initial Access

**Initial Access** marks the critical first step where a threat actor successfully breaches the perimeter of a target environment. These methods are generally divided into two main categories:

* **Exposed Services:** Exploiting systems directly connected to the internet (e.g., RDP, mail servers, websites).
* **User-Driven Methods:** Relying on human interaction to introduce malware (e.g., phishing, malicious links, removable media).

---

## 2. Exposed Services & RDP Attacks

Exposing management services like **Remote Desktop Protocol (RDP)** to the public internet introduces severe security risks. Without proper controls, automated bots scan and brute-force these interfaces within hours.

### MITRE ATT&CK Mapping
* **`T1133`** – External Remote Services
* **`T1190`** – Exploit Public-Facing Application

> [!WARNING]
> **"Ransomware Deployment Protocol"**: RDP misconfigurations frequently lead directly to full-scale ransomware deployments (utilized by major ransomware groups such as **Medusa** and **Akira**).

### Detecting an RDP Breach via Windows Security Logs

| Step | Objective | Filter / Action | Key Event ID & Details |
| :--- | :--- | :--- | :--- |
| **1** | **Spot Brute Force** | Filter Security logs for failed remote logins | **Event ID `4625`**<br>• Logon Types: `3` (Network) & `10` (RemoteInteractive)<br>• Review `Source IP` for external addresses |
| **2** | **Confirm Access** | Identify the compromised account | **Event ID `4624`** (Successful Logon)<br>• Match timestamp following brute-force attempts |
| **3** | **Track Actions** | Trace attacker process spawning | **Event ID `4624`** (Logon Type `10`)<br>• Copy the `Logon ID`<br>• Pivot to **Sysmon** logs and filter by same `Logon ID` |

---

## 3. User-Driven Methods: Phishing

Phishing remains a dominant threat vector because it bypasses network firewalls entirely by tricking internal users into executing malicious code.

### Binary Attachments
* Threat actors frequently use less common executable extensions (e.g., `.com`, `.scr`, `.cpl`) rather than standard `.exe` files to bypass user scrutiny (e.g., `tryhatme.com` masquerading as a meeting link).
* Windows' default setting to **hide known file extensions** is heavily exploited using double extensions (e.g., `invoice.pdf.exe`).

### LNK (Shortcut) Attachments
* Attackers hide scripts (`PowerShell`, `VBScript`, `BAT`) inside `.lnk` shortcut files inside compressed archives (`.zip`, `.rar`) to evade antivirus detection.
* **Execution Flow:**
  ```text
  User double-clicks .lnk -> Explorer.exe parses Target field -> Powershell.exe executed -> Remote Payload (e.g., RemcosRAT) Downloaded

```

### Sysmon Execution Chain for Phishing Payloads

```text
[Sysmon Event ID 1]  Web Browser Launch
                     └─ Image: C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe
                     └─ ParentImage: C:\Windows\Explorer.EXE

[Sysmon Event ID 11] File Downloaded
                     └─ TargetFilename: C:\Users\User\Downloads\invoice.zip

[Sysmon Event ID 11] Payload Unarchived
                     └─ TargetFilename: C:\Users\User\Downloads\invoice.pdf.exe

[Sysmon Event ID 1]  User Executes Payload
                     └─ Image: C:\Users\User\Downloads\invoice.pdf.exe
                     └─ ParentImage: C:\Windows\Explorer.EXE

```

---

## 4. Removable Media (USB)

USB-based attacks (observed in campaigns like **Camaro Dragon** and **Raspberry Robin**) bypass network perimeters entirely and require no active internet connection to propagate or execute.

* **MITRE ATT&CK Technique:** **`T1091`** – Replication Through Removable Media

### Common Tactics & Detection

* **Shortcuts:** Hiding legitimate contents and replacing them with malicious shortcuts (e.g., `RECOVERY.lnk`).
* **Icon Masquerading:** Disguising malicious binaries as harmless folders or media files (e.g., `Photos.exe` using a folder icon).
* **Double Extensions:** Creating copies of files such as `photo_2024_1_12.jpg.exe`.

> [!NOTE]
> USB-driven initial access closely mirrors phishing execution patterns in logs because both rely on a user executing binaries via `explorer.exe`. Look specifically for execution footprints originating from secondary drive letters (e.g., `E:\malware.exe`).

```
