#  Linux Hardening & Security Summary

This guide serves as a complete reference for the Linux Security Learning Series. It follows the **Defence-in-Depth** model, layering security from the physical hardware up to the application level.

---

## 1. Physical Security & Data-at-Rest
Physical access is the most direct route to a system compromise. If an attacker has the hardware, software protections can often be bypassed.

### Bootloader Security
* **The Threat:** An intruder can use **GRUB** (the Linux bootloader) to reset the root password. The adage is: **"Boot access = Root access."**
* **Mitigation:** * Set a **BIOS/UEFI password** to prevent unauthorized booting from external media.
    * Use `grub2-mkpasswd-pbkdf2` to generate a hash and password-protect the GRUB configuration. This prevents unauthorized users from editing boot parameters or gaining root shells.

### Disk Encryption (LUKS)
If physical security is breached (e.g., a stolen laptop/server), encryption makes data unreadable.
* **LUKS (Linux Unified Key Setup):** The standard for Linux disk encryption.
* **Architecture:**
    * **LUKS phdr:** Stores UUID, cipher details, and master key checksum.
    * **KM (Key Material):** Contains 8 slots (KM1-KM8) allowing multiple user passwords to unlock the same master key.
    * **Bulk Data:** The actual encrypted data.
* **Common Commands:**
    * `cryptsetup luksFormat /dev/sdb1`: Initialize encryption.
    * `cryptsetup luksOpen /dev/sdb1 my_drive`: Map the encrypted partition.
    * `mkfs.ext4 /dev/mapper/my_drive`: Format the mapped device.
    * `cryptsetup luksDump /dev/sdb1`: View header info (cipher used, iterations, salt).

---

## 2. Network Security & Firewall Management
A firewall controls the flow of packets, preventing unauthorized services from being exposed to the network.

### Netfilter Ecosystem
* **Netfilter:** The kernel framework for packet filtering.
* **iptables:** Legacy CLI front-end. Uses **Chains** (`INPUT`, `OUTPUT`, `FORWARD`).
    * Example: `iptables -A INPUT -p tcp --dport 22 -j ACCEPT`
* **nftables:** Modern successor to iptables; faster and more scalable.
* **UFW (Uncomplicated Firewall):** Simplified front-end for Debian/Ubuntu.
    * Example: `ufw allow 22/tcp`
<img width="1854" height="967" alt="firewalls1" src="https://github.com/user-attachments/assets/e80314d3-0d51-422d-9ab2-136caf7e82bb" />
<img width="1854" height="967" alt="firewall2" src="https://github.com/user-attachments/assets/077db64e-b53b-4e26-8b28-82667cd0f86f" />

### Strategy: Default Deny
The most secure posture is to **block everything** by default and only allow specific exceptions (e.g., Port 80 for HTTP, 443 for HTTPS, 53 for DNS).

---

## 3. Remote Access & User Hardening
Securing how users log in and what they can do once they are inside the system.

### SSH Hardening
* **Encryption:** Avoid cleartext protocols like Telnet; always use SSH.
* **Configuration (`/etc/ssh/sshd_config`):**
    * `PermitRootLogin no`: Force login as a standard user.
    * `PasswordAuthentication no`: Disable passwords entirely to stop brute-force attacks.
    * `PubkeyAuthentication yes`: Use SSH Key Pairs (`id_rsa` and `id_rsa.pub`).
* **Key Setup:** Use `ssh-keygen -t rsa` and `ssh-copy-id user@host`.
<img width="1854" height="967" alt="useraccounts" src="https://github.com/user-attachments/assets/66e17f6a-bf07-480b-8eb4-43acdddf4807" />


### Privilege Management & Account Hygiene
* **sudo over root:** Never work as root. Use `usermod -aG sudo` (Debian) or `wheel` (RedHat) to grant administrative rights.
* **Password Policy:** Use `libpwquality` (`/etc/security/pwquality.conf`) to enforce:
    * `minlen`: Minimum length.
    * `minclass`: Required character types.
* **Disabling Accounts:** Change the user's shell to `/sbin/nologin` in `/etc/passwd` for departed users or service accounts (like `www-data` or `nginx`) to prevent interactive logins.

---

## 4. Attack Surface Reduction & Maintenance
### Minimizing Vulnerabilities
* **Remove Unnecessary Software:** Every installed package is a potential exploit vector.
* **Close Unused Ports:** If a service is disabled, ensure the firewall blocks its ports.
* **Legacy Protocols:** Replace Telnet with SSH and TFTP with SFTP.
* **Remove Banners:** Configure services to hide version strings (identification) to prevent attackers from footprinting your specific vulnerabilities.

### Patching & Lifecycles
* **Ubuntu LTS:** Released every 2 years; offers 5 years of free updates (10 with Pro/ESM).
* **RHEL:** Offers a 10-12 year lifecycle (Full, Maintenance, and Extended phases).
* **Kernel Updates:** Critical for fixing flaws like **Dirty COW** (a race condition in the COW mechanism allowing root escalation).

---

## 5. Monitoring & Auditing
Visibility into system events is crucial for identifying threats.

### Essential Log Files (`/var/log/`)
* `auth.log` (Debian) / `secure` (RedHat): Tracks authentication and sudo attempts.
* `messages` / `syslog`: General system events.
* `utmp` / `wtmp`: Currently logged-in users and login/logout history.
* `kern.log`: Kernel-specific messages.
<img width="1854" height="967" alt="logs" src="https://github.com/user-attachments/assets/408669ef-fe1a-4711-b351-242f6300c4a6" />

### Analysis Tools
* `tail -n 20 [file]`: View the most recent events.
* `grep "FAILED" [file]`: Filter logs for specific keywords.

---

## Final Best Practices
1.  **Document everything:** Keep a record of all configuration changes.
2.  **Staging:** Always test changes in a lab environment before deploying to production.
3.  **Automation:** Use automatic updates for stable systems to ensure security patches are applied immediately.
