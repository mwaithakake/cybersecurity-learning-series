# Part 3: Command & Control, Persistence & Impact

## 1. Command and Control (C2)

Once a threat actor establishes Initial Access, they must maintain continuous control over the compromised host. This is tracked under the **MITRE Command and Control (C2)** tactic.

* **Attacks Without C2:** In interactive sessions like an RDP breach, threat actors can type commands directly. However, because this access can be terminated quickly, most attackers prefer establishing a dedicated C2 channel.
* **The Simplest C2:** The initial phishing payload or malware binary directly acts as the communication channel back to the attacker's infrastructure (e.g., CobaltStrike C2 frameworks).
* **Advanced C2:** The initial entry point downloads a separate, stealthy C2 binary (often staged in directories like `C:\Temp`), ensuring persistence even if the user deletes the original phishing attachment.

---

## 2. Persistence Overview

While data stealers have a short lifespan (breach, extract, exit), advanced campaigns require long-term access that can survive reboots and password changes. This is mapped to the **MITRE Persistence** tactic.

### A. Persisting via RDP (User Accounts & Privileges)
If an attacker breaches a system via exposed RDP, they may create a new backdoor account or compromise an existing one to ensure future access.

* **Key MITRE Techniques:** `T1136` (Create Account), `T1098` (Account Manipulation).
* **Management Commands:**
  * *CMD:* `net user "mr.backd00r" "p@ssw0rd!" /add` & `net localgroup Administrators "mr.backd00r" /add`
  * *PowerShell:* `New-LocalUser "mr.backd00r" -Password [...]` & `Add-LocalGroupMember "Administrators" -Member "mr.backd00r"`

#### Detecting Account-Based Persistence via Windows Security Logs
* **Event ID `4720`:** A new user account was created. (Investigate the creator, source IP, and timestamp).
* **Event ID `4732`:** A member was added to a security-enabled local group (such as *Administrators* or *Remote Desktop Users*).
* **Event ID `4724`:** An account's password was reset.

---

### B. Malware Persistence: Services & Tasks
For attacks initiated via phishing or USB vectors where RDP is unavailable, malware must persist independently across reboots.

#### 1. Windows Services
* Runs automatically after OS startup, usually requiring administrative privileges.
* **Attack Example:** `sc create "BadService" binpath= "C:\malware.exe" start= auto`
* **Detection Opportunities:**
  * Launch of `sc.exe create` via **Sysmon Event ID 1**.
  * Service creation via **Security Event ID `4697`** or **System Event ID `7045`**.
  * Suspicious processes spawned with a parent of `services.exe`.

#### 2. Scheduled Tasks
* Frequently abused by threat actors (e.g., APT28, APT41) because they are easy to configure and hide.
* **Attack Example:** `schtasks /create /tn "BadTask" /tr "C:\malware.exe" /sc onstart /ru System`
* **Detection Opportunities:**
  * Launch of `schtasks.exe /create` via **Sysmon Event ID 1**.
  * Task creation events via **Security Event ID `4698`**.
  * Suspicious processes spawned with a parent of `svchost.exe [-s Schedule]`.

---

### C. Malware Persistence: Startup Folders & Run Keys
For per-user persistence that runs upon a specific user's login without requiring administrative rights:

#### 1. Startup Folder
* **Path:** `C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\`
* **Detection:** Monitor file creation events inside Startup directories via **Sysmon Event ID 11**. Note that parent processes will typically be `explorer.exe`.

#### 2. Run Keys (Registry Persistence)
* **Path:** `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run` (or `HKLM` for all users).
* **Detection:** Monitor registry value creation or modification events targeting Run keys via **Sysmon Event ID 13**.

---

## 3. Why Persistence Matters (The Threat Landscape)

Threat actors maintain persistent access for several strategic reasons:
1. **Botnet Recruitment:** Incorporating the host into larger botnets (combining crypto-mining, stealing, and C2 routing).
2. **Espionage:** Long-term, quiet surveillance (e.g., Volt Typhoon maintaining stealth access within critical infrastructure grids for months).
3. **Lateral Movement & Ransomware:** Utilizing the breached workstation as a stepping stone to map and compromise the broader Active Directory domain.

### Impact & Ransomware
The ultimate goal of many disruptive intrusions is **Impact**—most notably via **Ransomware**. By encrypting domain controllers, endpoints, and backups, threat actors can paralyze entire organizations (such as hospital networks), resulting in operational standstills, extortion, and systemic business failure.

> [!NOTE]
> Complex attacks always begin with a single, small breach. Understanding the chain from **Initial Access ➔ Discovery ➔ Collection ➔ C2 ➔ Persistence** allows SOC analysts to intercept threats before they result in catastrophic impact.
