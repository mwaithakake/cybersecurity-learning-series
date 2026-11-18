# Linux Threat Detection & Incident Response

## 1. Initial Access & Entry Vectors

Linux servers—frequently deployed as edge firewalls, web servers, databases, or container hypervisors—are prime targets for both automated botnets and advanced persistent threat (APT) groups. Attackers exploit several primary vectors to gain a foothold:

* **Exposed SSH Services:** Shodan indexes tens of millions of internet-facing SSH ports. Attackers leverage automated botnets to brute-force weak passwords or authenticate using stolen private keys harvested from compromised developer environments (e.g., public GitHub repositories, automation servers, or administrator endpoints).
* **Vulnerable Public-Facing Applications:** Exploiting unpatched vulnerabilities in web apps, email gateways, or container APIs (e.g., Zimbra, Docker API, Palo Alto firewalls, or vulnerable WordPress plugins) under MITRE technique **T1190**.
* **Supply Chain & Human-Led Risks:** Unverified script execution (`curl | bash`), typosquatted packages (`pip3 install`), and compromised software dependencies or CI/CD secrets (e.g., XZ Utils backdoor, leaked access tokens in CI pipelines).

---

## 2. Reconnaissance & The Discovery Phase

Immediately following initial access, automated tools or manual actors execute enumeration commands to map the operating system, environment, and privileges:

* **General Discovery:** `pwd`, `ls /`, `env`, `uname -a`, `hostname`, `id`, `whoami`, `w`, `last`, and network checks (`ss -tnlp`, `netstat`, `ip a`).
* **Specialized Discovery:**
* *Data Stealers:* Searching for sensitive files (`find / -name .env`, `find /home -name id_rsa`, `history | grep pass`).
* *Cryptominers:* Querying CPU/GPU specifications (`cat /proc/cpuinfo`, `lscpu`, `free -m`).
* *Network Scanners:* Internal probing using `ping` loops or netcat scans (`nc`).


* **Analyst Insight:** The unexpected execution of `whoami` by a background service (like a web application) is a high-confidence indicator of compromise (IoC) and a primary trigger for security alerts.

---

## 3. Payload Delivery & Execution (Ingress & Reverse Shells)

Once a foothold is secured, attackers move to download malware or establish stable communication channels:

* **Ingress Tool Transfer:** Utilizing pre-installed utilities like `wget`, `curl`, or secure copy (`scp`/`sftp`) to pull down payloads from external servers or code-hosting platforms.
* **Reverse Shells:** Because web exploits and command injection vulnerabilities lack interactive features (like color, autocompletion, or Ctrl+C support), attackers spawn reverse shells (`bash -i >& /dev/tcp/...`, `socat`, or Python equivalents) to establish a reliable interactive terminal connecting back to their command-and-control (C2) server.

---

## 4. Privilege Escalation

Initial compromise typically lands an analyst or attacker inside a low-privilege service account (e.g., `www-data` or `serviceuser`). To achieve full control, attackers execute privilege escalation paths:

* **Kernel & Binary Exploits:** Leveraging local privilege escalation scripts (e.g., PwnKit), vulnerable SUID binaries (`/bin/env /bin/bash -p`), or misconfigured file permissions.
* **Effective UID Tracking:** Analysts can verify privilege jumps by tracking the effective user ID (`uid`) transition in process execution logs before and after exploit execution.

---

## 5. Persistence Mechanisms

To survive reboots or maintain long-term access without active malware, threat actors establish persistence across multiple layers:

* **Cron Jobs:** Scheduled tasks added to `/etc/crontab`, `/etc/cron.d/`, or user spool files (`/var/spool/cron/*`) to execute payloads periodically or `@reboot`.
* **Systemd Services:** Crafting malicious `.service` files inside `/etc/systemd/system/` (disguised as legitimate background daemons like `cloud-online` or `syslogd`) to launch malware automatically on system startup.
* **Account & SSH Backdoors:** Creating unauthorized local user accounts with sudo privileges or appending attacker-controlled public keys directly into target user `~/.ssh/authorized_keys` files.
* **Application-Layer Persistence:** Installing web shells (e.g., WSO web shell) inside vulnerable CMS platforms (WordPress, Joomla), bypassing OS-level system monitoring entirely.

---

## 6. Targeted Attacks: Espionage & Ransomware

While commodity botnets run automated "Hack and Forget" scripts for cryptomining or DDoS, targeted campaigns present severe business risks:

* **Espionage (APT Groups):** State-sponsored groups like Kimsuky deploy customized backdoors and legitimate-sounding systemd services on critical government or corporate infrastructure.
* **Hypervisor Ransomware:** Modern threat actors target Linux-based hypervisors (ESXi). Compromising a single physical hypervisor allows attackers to simultaneously encrypt dozens or hundreds of hosted virtual machines and internal servers.

---

## 7. Incident Detection & Analyst Playbook

Detecting sophisticated Linux attacks requires correlating data across multiple system telemetry sources:

1. **Authentication Logs (`/var/log/auth.log`):** Review SSH logins for anomalous source IPs, odd login hours, and suspicious user account creation or modification events (`useradd`, `usermod`).
2. **Process Tree Analysis (`auditd` & `ausearch`):** The gold standard for Linux forensics. By querying system calls (`-x`, `--pid`, `--ppid`), analysts walk up and down the process hierarchy from PID 1 to uncover the exact parent script, web request, or compromised binary that spawned unauthorized commands.
3. **File System Auditing:** Monitor changes to critical configuration directories (`/etc/systemd/system/`, `/etc/cron*`, and user `.ssh/authorized_keys`) to catch unauthorized persistence mechanisms immediately.

---
## 8. Quick-Reference Investigation Cheat Sheet

| Investigation Goal | Command / Log Source | What to Look For |
|---|---|---|
| **Audit SSH Logins** | `cat /var/log/auth.log \| grep Accepted` | External source IPs, odd access hours, password vs key auth |
| **Search Command History** | `history` / `ausearch -i -x <cmd>` | `whoami`, `uname -a`, privilege escalation checks |
| **Trace Process Lineage** | `ausearch -i --pid <PID>` / `--ppid` | Walk up process tree to PID 1 to find parent script/app |
| **Check Cron Persistence** | `ls -la /etc/cron* /var/spool/cron/` | Unauthorized jobs, `@reboot` entries, outbound curl/sh pipes |
| **Check Systemd Services** | `ls -la /etc/systemd/system/` | Unfamiliar service descriptions or hidden binary paths |
| **Audit Authorized Keys** | `cat ~/.ssh/authorized_keys` | Rogue public keys appended without administrative approval |

