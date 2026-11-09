# Linux Logging for SOC Operations: Summary

## 1. Introduction to Linux Logging

- **Context:** The room focuses on Linux servers without a GUI (Debian, Ubuntu, CentOS, RHEL). Linux is not immune to malware, and Linux-targeted intrusions are a growing problem, so SOC analysts need to know how to investigate Linux alerts.
- **Storage and structure:** Unlike Windows, which uses structured event IDs and tools like Event Viewer, Linux writes events to **plain text files**, mostly in `/var/log/`. Any text editor or command-line tool can read them, but there are no event codes and no strict format.
- **Variation:** Linux lets you change log format, verbosity, and storage location, and distributions customise logging differently. Logs on one system may look different (or be missing) on another.

## 2. Core Log Files and Discovery

- **`/var/log/syslog`** (`/var/log/messages` on RHEL): An aggregated stream of general system events such as cron jobs, boot messages, and time synchronisation.
- **`/var/log/auth.log`** (`/var/log/secure` on RHEL): The most useful log for SOC. Despite the name, it holds authentication events, user management actions, and `sudo` commands.
- **`/var/log/kern.log`:** Kernel messages and errors, useful for advanced DFIR work.
- **Package manager logs:** `/var/log/dpkg.log` and `/var/log/apt` (Debian-based), or `/var/log/dnf.log` and `/var/log/yum.log` (RHEL-based), tracking software installs and removals.
- **`/var/log/audit/audit.log`:** Runtime events recorded by auditd (covered in section 5).
- **App-specific logs:** Dedicated logs for services such as web servers (`/var/log/nginx/access.log`), databases, mail servers, and containers. Each line in a web access log is one request.
- **Log discovery:** When you don't know where something is logged, search the whole directory for keywords:

```bash
grep -R -E "auth|login|session" /var/log
```

## 3. Searching and Filtering with `grep`

Logs contain thousands of lines, so filtering is the core skill.

| Flag / command | Purpose |
|---|---|
| `grep "text" file` | Search a single file |
| `-R` (or `-r`) | **Recursive**: search every file in a directory tree. `-R` also follows symlinks |
| `-E` | **Extended** regex, so `\|` works as "or" without escaping |
| `-i` | Ignore case |
| `-n` | Show line numbers |
| `-l` | List only matching file names |
| `-v` | Invert: exclude matching lines (e.g. `grep -v CRON`) |
| `2>/dev/null` | Hide "Permission denied" errors |
| `zgrep` | Search compressed `.gz` rotated logs |

- **Practical habit:** Use `-R` when you don't know which file holds the evidence, then narrow down to the specific file.
- **Permissions:** Many logs are readable only by root or the `adm` group, so you often need `sudo`.
- **Rotated logs:** Older entries may sit in files like `syslog.1` or `syslog.2.gz`.

### Examples from `syslog`

- **Time synchronisation:** `systemd-timesyncd` logs which time server the machine contacted. Search with `grep -i "time server" /var/log/syslog`. The domain name (not the IP) identifies the server.
- **Boot timeline:** Each boot writes kernel lines such as `LSM: initializing lsm=...` followed by `Yama: becoming mindful.` Each pair marks one boot, which gives a quick reboot history.

## 4. Key Detection Techniques

### Authentication events (`auth.log`)

- **Logins and logouts:** Filter for `session opened` and `session closed`. The text in brackets shows how the login happened: `login:session` (keyboard), `sshd:session` (SSH), `cron:session`, or `sudo:session`.
- **SSH activity:** Filter for `sshd` plus `Accepted` or `Failed`. Entries follow the pattern *result, method, user, source IP, port*. Many `Failed password` lines from one IP point to **brute forcing**.
- **User management:** Search for `passwd`, `useradd`, `usermod`, and `userdel`. Look for suspicious patterns like a new user being created and then added to the `sudo` group (a common persistence technique).
- **Sudo commands:** Search for `COMMAND=` to see what was run with elevated privileges, for example someone stopping an EDR service before escalating to root.

### Bash history

- **What it is:** `~/.bash_history` stores commands each user typed. Each user has their own file (root's is `/root/.bash_history`), and the `history` command shows the current user's commands.
- **Why it's useful:** In the lab, root's history showed setup activity such as editing `sshd_config`, installing auditd, and running `rm *` in `/etc/logrotate.d`. Deleting logging configuration is a red flag in a real investigation.
- **Why it's unreliable for SOC:**
  - **Leading space:** With `HISTCONTROL=ignoreboth`, any command that starts with a space is not saved.
  - **Scripts:** Commands hidden inside a script (`nano legit.sh && ./legit.sh`) don't appear individually.
  - **Other shells:** `/bin/sh` and similar shells don't write Bash history.
  - **Non-interactive activity:** Cron jobs, web servers, and OS tasks are not recorded.
  - **Tampering:** The file can be edited or deleted.
- **Takeaway:** Absence of history is not proof that nothing happened.

## 5. Runtime Monitoring and System Calls

### System calls

- **Definition:** A system call (syscall) is how a program asks the **kernel** to do something it can't do directly: open a file, start a process, make a network connection, and so on. Analogy: you can't walk into the restaurant kitchen, so you order through the waiter.
- **Common examples:** `execve` (start a program), `openat` (open a file), `connect` (network connection), `ptrace` (inspect another process). Linux has over 300.
- **Why it matters for security:** Attackers can hide history, rename files, and delete logs, but they can't avoid the kernel if they want to act on the system. Logging the right syscalls shows what really happened. All modern EDRs and runtime tools work this way.

### Yama (Linux Security Module)

- **What it does:** Yama restricts the `ptrace` syscall, so by default a process can only inspect its own children. This stops malware from peeking into other processes to steal credentials or keys.
- **The boot message:** `Yama: becoming mindful.` is just its normal "loaded successfully" message, not a warning.
- **Other modules** in the `lsm=` list: `lockdown`, `capability`, `landlock`, `apparmor`, `ima`, and `evm`.

### Auditd (Audit Daemon)

- **Purpose:** By default Linux does not log process creation, file changes, or network events (runtime events). Auditd fills that gap, similar to Sysmon on Windows.
- **Rules:** Stored in `/etc/audit/rules.d/`, they define which syscalls and files to monitor and tag events with a **key** (e.g. `proc_wget`, `file_sshconf`).
- **Logs:** Written to `/var/log/audit/audit.log`. Use `ausearch -i -k <key>` for readable, filtered output, or `grep` the raw file.
- **Noise control:** Logging everything produces gigabytes of data, so teams focus on high-risk events such as program execution and edits to critical files (SSH config, cron jobs).

### Reading an auditd event

One event is split across several records:

- **`PROCTITLE`:** The full command line.
- **`CWD`:** The working directory.
- **`EXECVE`:** The command split into arguments (`a0`, `a1`, ...).
- **`PATH`:** The file involved (for file events).
- **`SYSCALL`:** The technical details:
  - `syscall=execve` means a program was started; `syscall=openat` means a file was opened.
  - `pid` / `ppid`: process and parent process IDs, used to build a process tree.
  - `auid`: the **audit user**, the account originally logged in (locally or via SSH).
  - `uid`: the user the command ran as. It differs from `auid` after `sudo` or `su`, which reveals the real person behind a root session.
  - `tty`: the terminal session, which separates users on a shared server.
  - `exe`: the full path of the binary that ran.
  - `key`: the label from the rule, used for filtering.

### Worked example: investigating a download

- A `proc_wget` event showed `wget` downloading a ZIP from a GitHub release URL and saving it under a different name with `-O`.
- The **original file name** is the last part of the URL. The `-O` value is only where it was saved.
- The tool was **naabu**, a legitimate port scanner that attackers also use for reconnaissance.
- To find what it was used against, search the audit log for the tool's name and read the target in its command line. There was no dedicated key, but the command line was still captured.
- **Investigation flow:** find the download, find the execution, read the arguments, and use timestamps and `auid` to tie it to a user.

### Auditd alternatives

Raw auditd output is verbose and hard to ingest into a SIEM, so teams often use alternatives that work on the same syscall-monitoring principle:

- **Sysmon for Linux:** familiar if you already use Sysmon on Windows.
- **Falco:** open source, well suited to containers.
- **Osquery:** flexible, with broad security uses.
- **EDR products:** track Linux runtime events.

## 6. Key Takeaways

- Linux logging can be chaotic, but it usually holds enough detail to detect threats.
- Logs live in `/var/log/` and are mostly plain text.
- The top three log sources for SOC are **auth.log, app-specific logs, and runtime logs (auditd)**.
- Bash history is unreliable. Use auditd or an alternative.
- Understanding system calls makes every runtime monitoring tool easier to learn.

## Quick Command Reference

```bash
grep -R -E "auth|login|session" /var/log        # discover logs
grep -iE "timesyncd|ntp" /var/log/syslog        # time sync events
grep -i yama /var/log/syslog                    # Yama boot messages
grep -E 'session opened|session closed' /var/log/auth.log
grep sshd /var/log/auth.log | grep -E 'Accepted|Failed'
grep 'COMMAND=' /var/log/auth.log               # sudo commands
cat /root/.bash_history                         # a user's command history
ausearch -i -k <key>                            # search auditd by key
grep -i <keyword> /var/log/audit/audit.log      # raw audit search
```
