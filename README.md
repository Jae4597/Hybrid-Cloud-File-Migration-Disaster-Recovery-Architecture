# Hybrid Cloud File Migration & Disaster Recovery Architecture

> Migrated corporate file shares to Azure Files with near real-time sync from on-premises Windows Server, secured remote access via Azure VPN with Private Endpoints, and built a 3-tier backup topology for disaster recovery.

**Duration:** January 2026 – February 2026
**Role:** Cyber Security Specialist (Sole Architect & Implementer)

---

## 📌 Overview

### Background
The company faced two operational pain points:

1. **Limited remote access** — Employees on business trips, working from home during snow days, or impacted by natural disasters could not access internal file shares hosted on the on-premises Windows Server.
2. **Single point of failure for backups** — The existing backup topology relied on a single location, exposing the company to significant data-loss risk in the event of hardware failure, ransomware, or physical disaster at the main office.

### Goal
Deliver a hybrid storage architecture that:
- Enables **secure remote access** to corporate file shares from any location.
- Maintains **near real-time consistency** between on-premises and cloud storage.
- Provides a **resilient, multi-tier backup strategy** to meet disaster recovery (DR) requirements.

---

## 🏗️ Architecture

### 1. Access Flow (In-Office vs. Out-of-Office)

![Access Flow Diagram](./diagrams/access-flow.png)

| Scenario | Access Path |
|---|---|
| **In-Office** | Employee → Company Main Server (direct LAN access) |
| **Out-of-Office** | Employee → Azure VPN Client (P2S, Entra ID auth) → Azure Files (via Private Endpoint) |

### 2. Backup Topology (3-Tier DR Strategy)

![Backup Topology Diagram](./diagrams/backup-topology.png)

| Tier | Source | Destination | Schedule | Method |
|---|---|---|---|---|
| **Tier 1 — Real-time Sync** | Company Main Server | Azure Files | Continuous | Azure File Sync |
| **Tier 2 — Cloud Backup** | Azure Files | Microsoft Backup (Azure Backup) | Daily 7:00 PM | Scheduled snapshot |
| **Tier 3a — Local Backup** | Company Main Server | Main Office NAS (Synology) | Daily 6:00 PM | Hyper Backup |
| **Tier 3b — Off-site Replication** | Main Office NAS | Backup Office NAS (Synology) | Daily 10:00 PM | Hyper Backup Vault |

This staggered scheduling ensures that even if one backup tier fails, the others remain unaffected, and recovery is possible from multiple geographically distinct points.

---

## 🛠️ Tech Stack

### Cloud (Microsoft Azure)
- **Azure Files** — Cloud-hosted SMB file share
- **Azure File Sync** — Near real-time sync agent on on-premises Windows Server
- **Azure VPN Gateway** — Point-to-Site (P2S) configuration with **Microsoft Entra ID** authentication
- **Azure Virtual Network (VNet)** — Network segmentation with **Private Endpoint** to keep Azure Files off the public internet
- **Azure Backup** — Scheduled cloud-side snapshot protection

### On-Premises
- **Windows Server 2016** — File server + Azure File Sync agent host
- **Active Directory (Windows Server 2016)** — Domain controller for identity and NTFS permissions

### Storage / Backup Hardware
- **Synology RS422+** (Main Office NAS)
- **Synology RS422+** (Backup Office NAS — off-site)
- **Hyper Backup / Hyper Backup Vault** — Synology-native backup tooling

---

## ⚙️ Implementation Details

### Step 1 — Azure Environment Setup
- Provisioned Resource Group, VNet, and subnets with network segmentation aligned to internal security policy.
- Created Storage Account and Azure Files share.
- Configured **Private Endpoint** for the Storage Account so that all access to Azure Files traverses the VNet privately, with no public endpoint exposure.

### Step 2 — Point-to-Site VPN Connectivity
- Deployed Azure VPN Gateway in P2S mode.
- Configured **Microsoft Entra ID authentication** so that VPN access is tied to corporate identity (no per-device certificate or shared key management overhead).
- Distributed VPN client configurations to remote-eligible employees.

### Step 3 — On-Premises ↔ Azure Synchronization
- Installed **Azure File Sync Agent** on Windows Server 2016.
- Created Storage Sync Service and registered the on-premises server.
- Configured Sync Group so that the on-premises file share and Azure Files maintain near real-time consistency.
- Mapped Azure Files to client devices using Storage Account access key for direct SMB access when needed.

### Step 4 — Access Control (Least Privilege)
- Applied **NTFS permissions** governed by **Active Directory security groups** to enforce department-level folder access.
- Each department's users can only access folders relevant to their role, following the principle of least privilege.

### Step 5 — Multi-Tier Backup Configuration
- **Azure Backup** — Configured daily snapshots of Azure Files at 7:00 PM.
- **Hyper Backup (Main NAS)** — Scheduled daily backup from Company Main Server at 6:00 PM.
- **Hyper Backup Vault (Backup NAS)** — Replicated Main NAS to off-site Backup NAS at 10:00 PM.
- Schedules were intentionally staggered to avoid contention and to ensure that a failure or compromise during one backup window does not propagate across all tiers simultaneously.

---

## 📈 Outcome

- **Remote access enabled** — Employees can securely access corporate file shares from any location, eliminating downtime during business trips, snow days, and other remote work scenarios.
- **No public exposure of file shares** — Private Endpoint ensures Azure Files is never reachable from the public internet, reducing the attack surface.
- **Identity-bound VPN access** — Entra ID authentication ties remote access to corporate identity lifecycle, allowing immediate revocation upon offboarding.
- **DR-ready backup posture** — 3-tier backup topology (Azure Backup + Main NAS + off-site Backup NAS) eliminates single point of failure and supports recovery from cloud, on-prem, or off-site sources.
- **Least-privilege access** — NTFS + AD group-based folder permissions ensure department-scoped data access.

---

## 🔐 Security Considerations

| Concern | Mitigation |
|---|---|
| Public exposure of cloud file share | Azure Files accessed only via **Private Endpoint** inside VNet |
| Unauthorized remote access | P2S VPN with **Microsoft Entra ID** authentication |
| Credential sprawl | Centralized identity via Entra ID (no per-device certs) |
| Over-permissioned file access | **NTFS + AD group** based least-privilege model |
| Single point of failure | **3-tier** geographically distributed backup |
| Backup chain compromise | **Staggered schedules** (6PM / 7PM / 10PM) across independent systems |

---

## 📁 Repository Structure

```
.
├── README.md
└── diagrams/
    ├── access-flow.png
    ├── access-flow.drawio
    ├── backup-topology.png
    └── backup-topology.drawio
```

---

## 👤 Author

**Changjae Chung** — Cyber Security Specialist
🔗 [LinkedIn](https://www.linkedin.com/in/changjae-chung-374821176)
