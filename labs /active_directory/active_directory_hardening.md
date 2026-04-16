# Active Directory Hardening: Security Fundamentals & Implementation

This technical write-up covers the core logical structure of Active Directory (AD), essential hardening configurations via Group Policy, and the implementation of modern security models to defend against common attack vectors.

## 1. Active Directory Logical Structure
Understanding the hierarchy is the first step in securing an AD environment:
* **Domain:** The core logical unit storing critical information about objects (users, computers, etc.).
* **Domain Controller (DC):** The "brain" of the network that handles authentication and authorization.
* **Trees & Forests:** Trees allow resource sharing between domains via trusts. A Forest is the highest security boundary, sharing a global catalog and schema.
* **Trusts:** Communication bridges between domains.
    * **Transitive:** If A trusts B, and B trusts C, then A trusts C.
    * **Directional:** Can be One-way or Two-way.

---

## 2. Hardening Configurations via Group Policy
To mitigate vulnerabilities, specific security policies must be enforced using the **Group Policy Management Editor**.

### **A. Eliminating Weak Hashes (LM Hash)**
By default, Windows may store the LAN Manager (LM) hash for passwords under 15 characters. LM hashes are weak and easily cracked.
* **Policy:** `Network security: Do not store LAN Manager hash value on next password change`
* **Action:** Set to **Enabled**.

> **[Screenshot Reference: Network security: Do not store LAN Manager hash value on next password change - Enabled]**
<img width="717" height="811" alt="Screenshot from 2026-04-16 08-10-58" src="https://github.com/user-attachments/assets/0c853aa6-108e-4f85-869b-04228d349f8a" />


### **B. SMB & LDAP Signing**
To prevent Man-in-the-Middle (MiTM) attacks, communication between clients and servers must be digitally signed to ensure integrity.
* **SMB Signing:** Ensures that SMB traffic (file/print sharing) cannot be modified in transit.
* **LDAP Signing:** Requires a Simple Authentication and Security Layer (SASL) to ignore unsigned or plain-text LDAP requests.

> **[Screenshot Reference: Microsoft network server: Digitally sign communications (always) - Enabled]**
<img width="717" height="811" alt="Screenshot from 2026-04-16 08-13-52" src="https://github.com/user-attachments/assets/d2bb28da-1b29-469e-aa01-046aaf8ae071" />

---

## 3. The Tiered Access Model (TAM)
The TAM is a security framework designed to protect "Tier 0" identities by isolating them from high-risk environments.

| Tier | Description | Included Assets |
| :--- | :--- | :--- |
| **Tier 0** | Highest Security Zone | Domain Controllers, Admin Accounts, Enterprise Admin Groups. |
| **Tier 1** | Server/App Layer | Domain member servers and enterprise applications. |
| **Tier 2** | End-User Layer | HR/Sales workstations, non-IT personnel devices. |

**Golden Rule:** Privileged credentials must never cross boundaries. A Tier 0 admin should never log into a Tier 2 device, as this prevents lateral movement and credential harvesting.

---

## 4. Microsoft Security Compliance Toolkit (MSCT)
The MSCT provides official security baselines to standardize hardening without custom scripting.

* **Security Baselines:** Consumable GPO backups that apply Microsoft-recommended settings.
* **Policy Analyzer:** A tool to compare GPOs and identify redundant or conflicting settings.

### **Implementation via PowerShell**
Baselines can be quickly deployed by executing the provided installation scripts with administrative privileges.

> **[Screenshot Reference: Running BaselineLocalInstall.ps1 with PowerShell]**
<img width="1328" height="803" alt="Screenshot from 2026-04-16 09-52-23" src="https://github.com/user-attachments/assets/3eef0236-88c2-46e7-b0f0-c22ae4444013" />


---

## 5. Known Attack Vectors & Defenses

| Attack | Description | Mitigation |
| :--- | :--- | :--- |
| **Kerberoasting** | Requesting TGS tickets and cracking service account hashes offline. | Use MFA and rotate KDC service account passwords regularly. |
| **RDP Brute Force** | Automated credential guessing on exposed RDP ports. | Disable public RDP access; use VPNs and account lockout policies. |
| **SMB Shares** | Exploiting unauthenticated or "Everyone" access on network shares. | Audit shares using `Get-SmbOpenFile` and restrict permissions. |
| **Password Attacks** | Brute force or dictionary attacks against weak passwords. | Enforce complexity, minimum length (10-14 chars), and history. |

---

## 6. Continuous Auditing
Hardening is not a one-time event. Regular audits are required:
1.  **Usage Audits:** Validate that access rights match current job duties.
2.  **Privilege Audits:** Ensure the **Principle of Least Privilege** is maintained.
3.  **Change Audits:** Monitor for unauthorized modifications to GPOs or account permissions.
