# Active Directory Comprehensive Learning Series
**Date:** March 3, 2026
**Topic:** Windows Domain Administration & Active Directory Domain Services (AD DS)

---

## 1. Introduction to Active Directory
Active Directory (AD) is the centralized management system for Windows networks. As a network grows, manual configuration becomes impossible. AD solves this by using a **Domain Controller (DC)** to host a single repository of all network data.

### Key Advantages:
* **Centralized Identity Management:** One set of credentials for all resources.
* **Security Policy Deployment:** Pushing configurations (GPOs) to thousands of machines at once.

---

## 2. Core Components & Objects
Active Directory treats everything on the network as an **Object**.

| Object Type | Description |
| :--- | :--- |
| **Users** | Security principals representing people or services. |
| **Machines** | Accounts for computers (named `Name$`). Passwords rotate automatically. |
| **Security Groups** | Collections of users/machines used to grant **permissions** to resources. |
| **Organizational Units (OU)** | Containers used to organize objects and apply **Group Policies**. |

### OUs vs. Groups: The Critical Distinction
* **OUs** are for **Administration**: Applying GPOs (e.g., "All Sales users get this wallpaper"). A user belongs to only **one** OU.
* **Groups** are for **Access**: Granting permissions (e.g., "This user can access the Payroll folder"). A user can belong to **many** groups.



---

## 3. Administrative Management
### Accidental Deletion Protection
By default, OUs are protected. To delete one:
1. Enable **Advanced Features** in the 'View' menu.
2. Uncheck "Protect object from accidental deletion" in the **Object** tab of the OU properties.

### Delegation of Control
The principle of **Least Privilege**. Instead of making every IT person a "Domain Admin," we delegate specific tasks (e.g., "Reset User Passwords") over specific OUs using the **Delegation of Control Wizard**.

---

## 4. Group Policy Objects (GPO)
GPOs are the "rules" of the domain. They consist of **Computer Configurations** (applied at boot) and **User Configurations** (applied at logon).

* **Inheritance:** Policies applied at the root domain flow down to all sub-OUs.
* **SYSVOL:** A shared folder on the DC used to distribute GPO data to the rest of the network.
* **GPUpdate:** Use `gpupdate /force` in PowerShell to bypass the 2-hour refresh cycle and apply policies immediately.



---

## 5. Authentication Protocols
AD uses two primary protocols to verify identities:

1. **Kerberos (Modern):** Ticket-based system.
   - **TGT (Ticket Granting Ticket):** Proves you are authenticated.
   - **TGS (Service Ticket):** Allows access to a specific service (SQL, File Share).
2. **NetNTLM (Legacy):** A challenge-response mechanism. It is kept for compatibility but is more vulnerable to certain attacks than Kerberos.



---

## 6. Scaling: Trees, Forests, and Trusts
* **Tree:** Multiple domains sharing a contiguous namespace (e.g., `sales.thm.local` and `dev.thm.local`).
* **Forest:** A collection of trees. This is the highest security boundary in AD.
* **Trusts:** Relationships that allow users in one domain to access resources in another.
  - **One-Way:** Domain A trusts Domain B (B's users access A's stuff).
  - **Two-Way:** Mutual access.
  - **Transitive:** If A trusts B, and B trusts C, then A trusts C.

---
