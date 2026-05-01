# Cloud Security & Operations: Learning Series Summary

This summary covers the core pillars of cloud computing, deployment strategies, and the shared responsibility model for securing data and infrastructure.

---

## 1. Cloud Fundamentals & Architecture
Cloud computing is the delivery of computing services over the internet using a **pay-as-you-go** model. Key characteristics include **Scalability** (adjusting resources to demand), **Simplicity** (managed interfaces), and **Automation** (reducing human administration).

### Service Models
*   **IaaS (Infrastructure as a Service):** Provider handles hardware; User handles OS and Apps.
*   **PaaS (Platform as a Service):** Provider handles OS/Platform; User handles Apps.
*   **SaaS (Software as a Service):** Provider handles everything; User simply uses the software.

### Deployment Models
*   **Public:** Shared resources (e.g., AWS, Azure). *Risk: Vendor lock-in.*
*   **Private:** Dedicated resources for one organization. *Risk: Physical/Personnel threats.*
*   **Hybrid:** Mix of Public and Private for flexibility and sensitive data handling.
*   **Community:** Shared between specific organizations with common goals. *Risk: Configuration consistency.*

---

## 2. The Cloud Data Lifecycle & Security
Data must be protected at every stage of its life. Data is generally classified as **Confidential**, **Internal**, or **Public**.

| Phase | Key Security Action |
| :--- | :--- |
| **Create/Update** | Define owners, classify data, and use SSL/TLS for transit. |
| **Store** | Encryption at rest and regular backups. |
| **Use** | Strict permissions and secure virtualization (isolation). |
| **Share** | DLP (Data Loss Prevention) and jurisdictional compliance. |
| **Archive** | Physical security and long-term encryption. |
| **Destroy** | **Crypto-shredding** (destroying keys to make data unreadable). |

---

## 3. Identity & Access Management (IAM)
IAM ensures the "right people" have the "right access."
*   **Identities:** Digital representations of users, services, or APIs.
*   **Authentication:** Using MFA, biometrics, or passwords to verify identity.
*   **Policies:** 
    *   *Identity-based:* Permissions follow the user.
    *   *Resource-based:* Permissions stay with the data/service.
    *   *Session-based:* Temporary access for a specific timeframe.

---

## 4. Layered Network Security
Cloud security relies on a "Defense in Depth" strategy:
1.  **Layer 1: Security Groups:** Instance-level firewalls. Operates on a "Deny All" principle (only "Allow" rules).
2.  **Layer 2: NACLs (Network Access Control Lists):** VPC/Subnet-level firewalls. Supports both "Allow" and "Deny" rules.
3.  **Layer 3: Vendor Solutions:** Specialized tools like AWS DNS Firewall or Network Firewall.

---

## 5. Continuity & Maintenance
### Disaster Recovery (DR)
*   **Cold DR:** Cheapest, slowest (High RTO). Uses snapshots.
*   **Warm DR:** Mid-range cost. Near real-time sync, ready to scale.
*   **Hot DR:** Most expensive, zero downtime. Parallel sites with load balancers.

### Monitoring & Patching
*   **AWS CloudTrail:** Logs API calls (audit trail).
*   **AWS CloudWatch:** Monitors performance and resource health.
*   **AWS GuardDuty:** Continuous threat detection.
*   **Systems Manager (Patch Manager):** Automates OS and application updates to close security gaps.
