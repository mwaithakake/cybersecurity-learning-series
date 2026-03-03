# Project: Active Directory Infrastructure Lifecycle & Security Hardening
**Date:** March 3, 2026

**Role:** Systems Administrator (Simulation)

**Objective:** Transition a legacy "flat" network into a structured, delegated, and policy-enforced Windows Domain.

---

## 1. OU Creation & Advanced Object Cleanup
**Business Problem:** Legacy OUs and defunct department data created a cluttered and insecure administrative environment. 
**Practical Action:**
* Enabled **Advanced Features** in Active Directory Users and Computers (ADUC) to access the **Object** properties.
* Modified the "Protect object from accidental deletion" flag to successfully decommission obsolete OUs.
* Created a clean, logical OU hierarchy mirroring the modern business structure.


<img width="1006" height="835" alt="deleting_OU(1)" src="https://github.com/user-attachments/assets/c154db1f-135d-448a-96b2-92b63c81a7ed" />

<img width="1006" height="835" alt="deleting_OU(2)" src="https://github.com/user-attachments/assets/b26e3af3-ec7a-4d37-a82e-d26d2f936072" />

<img width="1006" height="835" alt="deleting_OU(3)" src="https://github.com/user-attachments/assets/f17cbf64-3727-49b3-b47e-6a23dbd6a477" />

---

## 2. Managed Object Migration (Workstations & Servers)
**Business Problem:** New computers joining the domain default to the `Computers` container, where they receive no specific security policies.
**Practical Action:**
* Identified all active workstations and laptops in the default container.
* Performed a **Managed Migration** by manually moving machine objects into the newly created `Workstations` and `Servers` OUs.
**Business Value:** This ensures that assets are immediately placed under the correct GPO scope (e.g., standard users don't get server-level access).


<img width="1002" height="833" alt="used1" src="https://github.com/user-attachments/assets/0af66d32-6d5a-4195-848e-e4a82939bb63" />
<img width="1002" height="833" alt="place" src="https://github.com/user-attachments/assets/977884f5-28c5-49b0-a370-9e7adb75b38a" />
<img width="1002" height="833" alt="place2" src="https://github.com/user-attachments/assets/848b01b7-c8d6-4903-8fd0-c69511aa8ef5" />

---

## 3. Delegation of Control (The Helpdesk Model)
**Business Problem:** Domain Administrators were overwhelmed with low-level tickets (password resets), creating a productivity bottleneck.
**Practical Action:**
* Utilized the **Delegation of Control Wizard** on the Sales OU.
* Assigned the user `phillip` the specific right to **Reset User Passwords** without granting him any other administrative privileges.


<img width="1006" height="835" alt="place4" src="https://github.com/user-attachments/assets/4674c2a0-e964-4792-bcfb-643776408ad1" />
<img width="1006" height="835" alt="place5" src="https://github.com/user-attachments/assets/e878af23-7b73-4909-af4f-7b759c2698a8" />

---

## 4. PowerShell Administration & Security
**Business Problem:** Delegated users often lack access to GUI tools, requiring command-line proficiency to fulfill their roles.
**Practical Action:**
* Executed the `Set-ADAccountPassword` cmdlet as `phillip` to reset the user `sophie`'s account.
* Used the `-ChangePasswordAtLogon $true` flag to ensure user privacy and compliance.
**Technical Proof:** Utilized `-AsSecureString` to prevent cleartext password exposure in the terminal history.


<img width="1006" height="835" alt="place6" src="https://github.com/user-attachments/assets/88f834d7-2955-4127-a2ed-45c4d3f06761" />
<img width="1006" height="835" alt="delegating_OU(6)" src="https://github.com/user-attachments/assets/84dcbc20-1460-4b9c-9a95-12b6cb5a144a" />
<img width="1006" height="835" alt="delegating_OU(5)" src="https://github.com/user-attachments/assets/9d88a7e6-bf5a-4675-a1aa-ef68148942a6" />
<img width="1006" height="835" alt="delegating_OU(7)" src="https://github.com/user-attachments/assets/e04fdb29-cc69-49b1-93f1-7eb3820f9196" />

---

## 5. Security GPO: Control Panel Restriction
**Business Problem:** Non-technical staff in Sales and Marketing were making unauthorized changes to system configurations.
**Practical Action:**
* Engineered a User-Side GPO: **Prohibit Access to Control Panel and PC Settings**.
* Linked the GPO specifically to the Sales, Management, and Marketing OUs while excluding the IT OU to maintain administrative flow.

<img width="1002" height="833" alt="gpo_1" src="https://github.com/user-attachments/assets/8553ac5c-e1ec-4a8a-88ed-a6ec307a4c9c" />
<img width="1002" height="833" alt="gpo_3" src="https://github.com/user-attachments/assets/64ba1503-1023-4806-ac2d-48839c698418" />
<img width="1002" height="833" alt="gpo_4" src="https://github.com/user-attachments/assets/cf4391e7-220d-4d61-b0fe-2c59d99762ae" />
<img width="1002" height="833" alt="gpo_2" src="https://github.com/user-attachments/assets/f5265bf2-9d3d-442e-b484-f5c97612929b" />
<img width="1002" height="833" alt="gpo_5" src="https://github.com/user-attachments/assets/f956353b-2270-4624-986d-b3408ffcfef1" />
<img width="1191" height="845" alt="gpo_6" src="https://github.com/user-attachments/assets/9522b636-73fc-4e25-8361-18be5346872b" />
<img width="1191" height="845" alt="gpo_7" src="https://github.com/user-attachments/assets/6f7c5b94-3461-4f94-9932-48c975ea0ed2" />

---

## 6. Security GPO: Password Policy Hardening
**Business Problem:** The default password length (7 characters) was insufficient for modern security standards.
**Practical Action:**
* Modified the **Default Domain Policy**.
* Navigated to `Account Policies -> Password Policy` and increased the **Minimum Password Length to 10 characters**.


<img width="1002" height="833" alt="char_1" src="https://github.com/user-attachments/assets/8af27e5e-f57d-4078-a934-55fa7e703084" />
<img width="1002" height="833" alt="char_2" src="https://github.com/user-attachments/assets/5fafa79e-44dc-471b-b89c-09cd44651cb0" />
<img width="1002" height="833" alt="char_3" src="https://github.com/user-attachments/assets/2b657307-666e-4779-97e3-6f6b74462ab4" />

---

## Key Results & Skillsets
* **Identity Management:** Managed the full lifecycle of users and machine accounts.
* **Security Engineering:** Implemented Least Privilege and GPO-based hardening.
* **Troubleshooting:** Used CLI tools to bypass propagation lag and verify environment health.
