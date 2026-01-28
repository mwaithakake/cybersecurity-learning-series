
# 🐚 Web Shells: Deployment, Anatomy, and Detection
**Date:** January 28, 2026
**Topic:** Web Shell Lifecycle, Log Analysis, and Forensic Hunting

---

## 📑 What is a Web Shell?
A **Web Shell** is a malicious script uploaded to a web server that allows an attacker to execute OS commands remotely. 
* **Dual Purpose:** Used for **Initial Access** (via file upload flaws) and **Persistence** (maintaining access even if the original bug is patched).
* **Kill Chain:** Allows attackers to perform reconnaissance, escalate privileges, and exfiltrate data.

> <img width="1854" height="967" alt="Screenshot_2026-01-28_04_36_15" src="https://github.com/user-attachments/assets/cc203176-393a-4442-a32f-bccb67671ddc" />
> *Example of a simple PHP web shell interface executing the 'whoami' command, returning 'www-data'.*

---

## 🧬 Anatomy of a Web Shell
Most web shells abuse legitimate language functions to interact with the system shell.

### Legitimate Functions Abused (PHP):
* `shell_exec()`
* `exec()`
* `system()`
* `passthru()`

### How it works:
1. **Input:** The shell checks for a parameter (e.g., `?cmd=`).
2. **Execution:** It passes that parameter into a system function.
3. **Output:** It displays the system's response back in the browser.

> <img width="1854" height="967" alt="Screenshot_2026-01-28_04_38_02" src="https://github.com/user-attachments/assets/f71572ab-ec32-44c7-881f-d68c92f95da0" />
> *Interacting with 'awebshell.php' to list directory contents (`ls`), revealing the 'flag.txt' file.*

---

## 🕵️ Hunting for Web Shells in Logs
Web server logs are the first line of evidence. Look for deviations from normal traffic patterns.

### 1. Unusual Request Patterns
* **Fuzzing:** Repeated `GET` requests in quick succession (often resulting in `404` errors) as attackers probe for upload directories.
* **Interaction:** Frequent `POST` or `GET` requests to a single, unusual `.php` or `.aspx` file.

> <img width="1854" height="967" alt="Screenshot_2026-01-28_05_20_28" src="https://github.com/user-attachments/assets/e044b9bd-982b-4f92-9c3d-2228b07c54f9" />
> *Log Analysis: Notice the high volume of requests from the same IP using 'ashadyagent/1.1' targeting various WordPress admin paths.*

### 2. Suspicious Indicators
| Indicator | What to look for |
| :--- | :--- |
| **User-Agent** | Blacklisted strings like `curl`, `wget`, or outdated/shortened strings (e.g., `Mozilla/4.0`). |
| **Query Strings** | Look for keywords like `cmd=`, `exec=`, or Base64 encoded strings in the URL. |
| **Missing Referrer** | Indicates a file was accessed directly via the URL bar rather than clicking a link. |

---

## 🛠️ File System Analysis
Since web shells must be stored on the server, we can hunt for them using command-line tools.

### Finding Recently Modified Scripts
Attackers often hide shells in common upload directories like `/uploads/`, `/images/`, or `/temp/`.

```bash
# Search for .php files modified between July and August 2025
find /var/www -type f -name "*.php" -newerct "2025-07-01" ! -newerct "2025-08-01"

# Search recursively for the dangerous 'eval(' function
grep -r "eval(" /var/www/html/wp-content/

```

> <img width="1854" height="967" alt="Screenshot_2026-01-28_05_36_46" src="https://github.com/user-attachments/assets/0cc135a7-08df-41a9-accb-4202394f4ca7" />
> *Correlating logs: Here we see successful **200 OK** responses for `shadyshell.php`, confirming the web shell is active and responding to `curl` commands.*

---

## 🛡️ Best Practices for Detection

To effectively defend against web shells, a layered defense strategy is required:

* **Log Correlation:** Link Web Access logs with `auditd` logs to see if a `POST` request to a script resulted in an `execve` syscall (system command execution).
* **SIEM Integration:** Centralize logs to alert on unusual HTTP methods (like `PUT` or `DELETE`) or spikes in `404` errors from a single IP.
* **WAF Rules:** Configure the Web Application Firewall to block common web shell parameters and known malicious User-Agents.
* **Hardening Uploads:** Enforce strict file type validation (MIME types) and prevent execution permissions in upload directories.

---

## 🚀 Conclusion

Detecting web shells is about connecting the dots. A single `404` error is noise, but a series of `404`s followed by a successful `200 OK` to a new `.php` file is a high-fidelity indicator of a compromise. Always verify the **Integrity** of your web directory and treat every new script as untrusted until proven otherwise.



