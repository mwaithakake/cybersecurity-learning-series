
# PowerShell Command Reference & Quick Guide for Linux Users

## Quick Command Reference Table (PowerShell vs. Linux & Windows CLI)

| Task / Operation | PowerShell Cmdlet | Linux Equivalent (Bash) | Windows CMD Equivalent |
| --- | --- | --- | --- |
| **List directory contents** | `Get-ChildItem` | `ls` / `dir` | `dir` |
| **Change directory** | `Set-Location` | `cd` | `cd` / `chdir` |
| **Read file contents** | `Get-Content` | `cat` | `type` |
| **Create file/directory** | `New-Item` | `touch` (files) / `mkdir` (dirs) | `mkdir` (dirs) / `type nul >` (files) |
| **Delete file/directory** | `Remove-Item` | `rm` / `rmdir` | `del` / `rmdir` |
| **Copy file/directory** | `Copy-Item` | `cp` | `copy` |
| **Move/Rename file** | `Move-Item` | `mv` | `move` |
| **Search text in files** | `Select-String` | `grep` | `findstr` |
| **Display command history** | `Get-History` | `history` | `doskey /history` |
| **View network configuration** | `Get-NetIPConfiguration` | `ipconfig` / `ip a` | `ipconfig` |
| **List running processes** | `Get-Process` | `ps aux` | `tasklist` |
| **Stop a running process** | `Stop-Process` | `kill` / `pkill` | `taskkill` |
| **Find command manual/help** | `Get-Help` | `man` / `apropos` | `help` / `<cmd> /?` |

---

## 1. Basic Syntax & Discovery Cmdlets

PowerShell commands follow a consistent **`Verb-Noun`** naming convention (e.g., `Get-Content`, `Set-Location`), making their purpose intuitive.

* **Discovery & Help:**
* `Get-Command`: Lists all available cmdlets, functions, aliases, and scripts in the current session. Can be filtered (e.g., `Get-Command -CommandType "Function"`).
* `Get-Help <Cmdlet>`: Retrieves detailed usage, parameters, and syntax. Append `-examples` or `-detailed` for more depth.
* `Get-Alias`: Displays shortcuts/alternative names for cmdlets (e.g., `dir` maps to `Get-ChildItem`, `cat` maps to `Get-Content`).



---

## 2. File System Management & Navigation

PowerShell handles files and directories using unified cmdlets rather than separate commands.

* `Get-ChildItem`: Lists files and directories in a specified path.
* `Set-Location`: Changes the current working directory.
* `New-Item`: Creates a new file or directory (via `-ItemType "File"` or `-ItemType "Directory"`).
* `Remove-Item`: Deletes files or directories.
* `Copy-Item` / `Move-Item`: Copies or moves files and directories.
* `Get-Content`: Reads and displays file contents.

---

## 3. The Power of Piping & Data Filtering

The pipeline (`|`) passes entire objects (including properties and methods) from one command to the next.

* **`Sort-Object`**: Sorts incoming objects by specified properties (e.g., `Get-ChildItem | Sort-Object Length -Descending`).
* **`Where-Object`**: Filters objects based on conditions using comparison operators (`-eq`, `-ne`, `-gt`, `-ge`, `-lt`, `-le`, `-like`).
* *Example:* `Get-ChildItem | Where-Object -Property "Extension" -eq ".txt"`


* **`Select-Object`**: Selects specific properties or limits the number of returned objects (e.g., `-First 1`).
* **`Select-String`**: Searches for text patterns within files, supporting full regular expressions (regex).

---

## 4. System & Network Enumeration

* **`Get-ComputerInfo`**: Retrieves a comprehensive snapshot of system configurations, hardware specifications, and OS build details.
* **`Get-LocalUser`**: Lists local user accounts, statuses, and descriptions for security audits.
* **`Get-NetIPConfiguration`**: Provides detailed networking stats (IP addresses, DNS servers, gateways) per interface.
* **`Get-NetIPAddress`**: Shows detailed properties for all configured IP addresses, including inactive ones.

---

## 5. Scripting & Remote Management

* **Scripting Utility:** Scripts automate repetitive tasks, configuration checks, and log parsing.
* *Blue Team:* Used for log analysis, anomaly detection, and extracting Indicators of Compromise (IOCs).
* *Red Team:* Used for system enumeration, remote execution, and simulation testing.


* **`Invoke-Command`**: Executes commands or scripts on local or remote computers using parameters like `-ComputerName`, `-FilePath`, and `-ScriptBlock`.
