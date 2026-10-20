Here is the updated cheat sheet with a quick reference table mapping the Windows CMD commands to their Linux equivalents to help you cross-reference quickly:

---

# Windows Command-Line Cheat Sheet & Reference Guide

A collection of essential Windows Command Prompt (CMD) commands for system enumeration, network troubleshooting, file/directory navigation, and process management.

---

## Windows vs. Linux Command Quick Reference

| Windows CMD | Linux Equivalent | Description / Notes |
| --- | --- | --- |
| **`set`** | `printenv` / `env` | View environment variables and system Path |
| **`ver`** | `cat /etc/os-release` | View OS version |
| **`systeminfo`** | `uname -a` / `lscpu` / `free` | Enumerate system, CPU, and memory details |
| **`ipconfig`** | `ip a` (or `ifconfig`) | Network interface configuration |
| **`ping`** | `ping` | ICMP reachability test |
| **`tracert`** | `traceroute` | Trace network path/hops |
| **`nslookup`** | `nslookup` / `dig` | DNS query and record lookup |
| **`netstat`** | `ss` / `netstat` | Active connections and listening ports |
| **`cd`** | `pwd` / `cd` | Print current directory / change directory |
| **`dir`** | `ls` | List directory contents |
| **`tree`** | `tree` | Visual directory tree structure |
| **`mkdir`** | `mkdir` | Create a new directory |
| **`rmdir`** | `rmdir` / `rm -r` | Remove a directory |
| **`type`** | `cat` | Dump file contents to the terminal |
| **`copy`** | `cp` | Copy files |
| **`move`** | `mv` | Move or rename files |
| **`del` / `erase**` | `rm` | Delete files |
| **`tasklist`** | `ps aux` | List running processes |
| **`taskkill`** | `kill` / `killall` | Terminate a running process |
| **`help` / `/?**` | `man` or `--help` | Display command help/manual |
| **`cls`** | `clear` | Clear terminal screen |
| **`more`** | `more` / `less` | Page-by-page output viewing |
| **`chkdsk`** | `fsck` | Check file systems for errors |

---

## 1. System & Environment Information

* **`set`**: Displays or modifies environment variables. Use `set` to view the system **Path** variable, which defines where Windows looks for executable commands.
* **`ver`**: Displays the current Microsoft Windows operating system version.
* **`systeminfo`**: Enumerates detailed system information, including OS name, build, version, manufacturer, processor details, and memory status.
* **`driverquery`**: Displays a list of installed device drivers.

---

## 2. Networking & Troubleshooting

* **`ipconfig`**: Displays basic network configuration (IPv4/IPv6 addresses, subnet mask, default gateway).
* `ipconfig /all`: Displays verbose network details, including DNS servers, MAC address, and DHCP status.


* **`ping <target>`**: Sends ICMP echo request packets to verify network reachability and measure round-trip time (RTT).
* **`tracert <target>`**: Traces the network route (hops) a packet takes to reach a destination by observing TTL expiration.
* **`nslookup <domain> [dns_server]`**: Queries DNS records to resolve domain names to IP addresses, optionally specifying a custom name server (e.g., `1.1.1.1`).
* **`netstat`**: Displays active network connections and listening ports.
* **Common Options**:
* `-a`: Displays all active connections and listening ports.
* `-b`: Shows the executable program associated with each connection.
* `-o`: Reveals the Process ID (PID) associated with the connection.
* `-n`: Displays addresses and port numbers in numerical form.


* *Example*: `netstat -abon` to identify which processes (and PIDs) are bound to specific ports.



---

## 3. Directory & File Operations

### Directory Navigation

* **`cd`**: Displays the current working directory (or changes directories when followed by a path).
* `cd ..`: Moves up one directory level.


* **`dir`**: Lists files and folders in the current directory.
* `dir /a`: Displays hidden and system files.
* `dir /s`: Displays files in the current directory and all subdirectories.


* **`tree`**: Visually displays the folder path structure in a tree diagram.
* **`mkdir <name>`**: Creates a new directory (Make Directory).
* **`rmdir <name>`**: Removes an empty directory (Remove Directory).

### File Management

* **`type <file>`**: Dumps the contents of a text file directly to the terminal window.
* **`copy <source> <destination>`**: Copies files from one location to another. Supports wildcards (e.g., `copy *.md C:\Backup`).
* **`move <source> <destination>`**: Moves or renames files.
* **`del` / `erase**`: Deletes one or more files.

---

## 4. Process Management

* **`tasklist`**: Lists all currently running processes and their corresponding PIDs and memory usage.
* `tasklist /FI "imagename eq <process.exe>"`: Filters the process list by executable image name.


* **`taskkill /PID <pid>`**: Terminates a running process by its Process ID.

---

## 5. Useful Tricks & Utilities

* **`help <command>`** or **`<command> /?`**: Displays the built-in help page and manual for any command.
* **`cls`**: Clears the Command Prompt screen.
* **Paging with `more**`: Append `| more` to any long command output (or use `more file.txt`) to view results page-by-page. Press the **Spacebar** to advance a page, **Enter** to advance a line, or **Ctrl + C** to exit.
* **`chkdsk`**: Checks file systems and disk volumes for errors and bad sectors.
* **`sfc /scannow`**: Scans critical system files for corruption and attempts automatic repairs.
