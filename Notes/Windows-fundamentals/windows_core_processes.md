# Windows Core Processes

## 2. Important Windows Core Processes

| Process | Function | Security Relevance |
|---|---|---|
| `System` | Handles kernel and driver activity. | Investigate unusual system-level behaviour. |
| `smss.exe` | Initializes Windows sessions. | Check unusual executable locations. |
| `csrss.exe` | Supports important Windows functions. | Can be impersonated by malware. |
| `wininit.exe` | Initializes background processes during startup. | Useful for investigating startup activity. |
| `services.exe` | Manages Windows services. | Can be abused for persistence. |
| `lsass.exe` | Handles authentication and security policies. | Targeted for credential theft. |
| `winlogon.exe` | Manages user sign-in and sign-out. | Investigate unusual behaviour or locations. |
| `explorer.exe` | Manages the desktop and File Explorer. | Can appear in suspicious process chains. |

## 3. Commonly Abused Windows Processes

| Process | Normal Function | Potential Malicious Use |
|---|---|---|
| `svchost.exe` | Hosts Windows services. | Hiding malicious services. |
| `rundll32.exe` | Executes DLL functions. | Executing malicious DLL code. |
| `dllhost.exe` | Hosts COM objects. | Abuse of COM execution. |
| `taskhostw.exe` | Hosts background tasks. | Suspicious task execution. |
| `cmd.exe` | Windows command interpreter. | Executing malicious commands. |
| `powershell.exe` | Windows scripting environment. | Downloading payloads and executing malicious scripts. |
| `conhost.exe` | Supports console applications. | Appears alongside command-line activity. |
| `taskmgr.exe` | Manages and displays running processes. | Relevant during process investigations. |

**Note:** These processes are legitimate Windows components. Their activity must be evaluated using execution context and behaviour.

## 5. Suspicious Process Indicators

- **Unusual executable location:** A system process running from Downloads or a temporary directory.
- **Suspicious parent process:** Microsoft Word unexpectedly launching PowerShell.
- **Encoded commands:** PowerShell using `-EncodedCommand`.
- **Unexpected network connections:** Connections to unfamiliar external addresses.
- **Suspicious process access:** Unexpected access to LSASS memory.
- **Unusual service creation:** New services executing from suspicious directories.
- **Process masquerading:** Malicious executables using legitimate Windows process names.

These indicators require further investigation and do not automatically confirm malicious activity.

## 6. Important Sysmon Event IDs

| Event ID | Event | Investigation Purpose |
|---|---|---|
| 1 | Process Creation | Identifies processes, parent processes and command lines. |
| 3 | Network Connection | Identifies network connections initiated by processes. |
| 7 | Image Loaded | Records DLL and image loading activity when configured. |
| 10 | Process Access | Helps investigate access to other processes, including LSASS. |
| 11 | File Create | Records file creation activity. |
| 13 | Registry Value Set | Helps investigate registry modifications and persistence. |

**Additional Windows Event ID:**

- **4688:** Records process creation when Windows process creation auditing is enabled.

## 7. Practical Investigation Example

**Scenario:** An analyst receives an alert involving suspicious PowerShell execution.

**Available Evidence:**

```text
Parent Process: WINWORD.EXE
Process: powershell.exe
Command Line: powershell.exe -EncodedCommand [encoded content]
```

**Investigation Steps:**

1. Establish why Microsoft Word launched PowerShell.
2. Examine the PowerShell command-line arguments.
3. Decode and inspect the command safely.
4. Check whether PowerShell downloaded or created additional files.
5. Examine subsequent process creation events.
6. Review related network connections.
7. Determine whether persistence or further malicious execution occurred.

**Objective:** Establish what happened, how it happened and whether the activity represents a security incident.
