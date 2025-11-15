# 04-Entitlement-Management
## TL;DR (Executive Summary)
Implemented and validated **Microsoft Entra ID Entitlement Management** with **catalogs**, **access packages**, **approval workflows**, and **automatic lifecycle expiration**.  
Successfully simulated the **end-to-end user access request experience** via **MyAccess portal**, including **approval**, **assignment**, and **auto-revocation**.  
This lab demonstrates production-ready **Identity Governance** skills — essential for **SC-300** and modern **IAM** roles.

---
## Learning Objectives
By completing this lab, I was able to:
* Configure **Entitlement Management** from scratch in Microsoft Entra ID.
* Build **access packages** with bundled resources (groups, apps, SharePoint).
* Enforce **approval policies** and **time-bound access**.
* Validate **self-service access** via **MyAccess portal**.
* Prove **automatic access revocation** on expiration.
* Document full **audit trail** and **governance compliance**.

---
## Key Concepts
- **Catalog**: Container for resources (groups, apps, sites)
- **Access Package**: Bundle of entitlements for self-service
- **Policy**: Controls who can request, approval, and expiration
- **MyAccess Portal**: User-facing self-service interface
- **Lifecycle Automation**: No manual cleanup — access expires automatically
- **Access Reviews**: (Future lab) Periodic entitlement validation

---
## What I Built
| Component | Details |
|---------|---------|
| **Tenant** | `practicecyber.onmicrosoft.com` |
| **Catalog** | `Lab-Catalog-Entitlement` |
| **Access Package** | `Finance Full Access Bundle` |
| **Resources** | Security Group (`Finance-Team`), App (`Finance Analytics`), SharePoint (`Finance Docs`) |
| **Policy** | 1-stage approval + 30-day expiration |
| **Test User** | `test.user@practicecyber.onmicrosoft.com` |
| **Portal** | [myaccess.microsoft.com](https://myaccess.microsoft.com) |
| **Cleanup** | 100% – catalog, package, resources removed |

----
## Architecture Diagram
<img width="600" src="./Screenshots/entitlement_architecture.png" alt="Entitlement Management Architecture" />
*Catalog → Access Package → Policy → MyAccess → Lifecycle Automation*

---
### Part 1: Entitlement Management Implementation (100% Success)
## Full Implementation Workflow & Evidence
| # | **Feature** | **Action** | **Status** | **Screenshot / Evidence** |
|:-:|:------------|:---------------|:---------------|:---------------|
| **1** | **Catalog** | Created `Lab-Catalog-Entitlement` in Entra ID Governance. | Success | <img src="./Screenshots/catalog-created.png" width="180" height="120"/> |
| **2** | **Resources** | Added Group, App, and SharePoint site to catalog. | Success | <img src="./Screenshots/catalog-resources.png" width="180" height="120"/> |
| **3** | **Access Package** | Created `Finance Full Access Bundle` with all 3 resources. | Success | <img src="./Screenshots/access-package.png" width="180" height="120"/> |
| **4** | **Policy** | Configured: Specific user + 1-stage approval + 30-day expiration. | Success | <img src="./Screenshots/policy-settings.png" width="180" height="120"/> |
| **5** | **User Request** | Test user submitted request via MyAccess portal. | Success | <img src="./Screenshots/myaccess-request.png" width="180" height="120"/> |
| **6** | **Approval** | Admin approved request with comment. | Success | <img src="./Screenshots/approval-flow.png" width="180" height="120"/> |
| **7** | **Access Granted** | Group membership, app, and SharePoint access validated. | Success | <img src="./Screenshots/access-assigned.png" width="180" height="120"/> |
| **8** | **Auto-Revocation** | Access removed after policy expiration (1-min test). | Success | <img src="./Screenshots/access-expired.png" width="180" height="120"/> |
| **GIF** | **Full Flow** | User request → approval → access → expiration. | Success | <img src="./Screenshots/demo-request-flow.gif" width="180" height="120"/> |

---
## Step-by-Step Implementation Guide (Avoiding Pitfalls)

### 1. Create Catalog
- **Path:** Entra Admin Center → **Identity Governance** → **Entitlement Management** → **Catalogs**
- **Action:** `+ New catalog` → Name: `Lab-Catalog-Entitlement`
- **Tip:** Enable **"Resources can be added by admins"**
- **URL:** [Catalogs](https://entra.microsoft.com/#view/Microsoft_AAD_IG/EntitlementManagementMenuBlade/~/Catalogs)
→ See: `catalog-created.png`

### 2. Add Resources to Catalog
- **Path:** Catalog → **Resources** → **+ Add resources**
- **Add:**
  - Group: `Finance-Team` (Security)
  - App: `Finance Analytics` (Enterprise App)
  - SharePoint: `https://practicecyber.sharepoint.com/sites/FinanceDocs`
- **URL:** [Resources](https://entra.microsoft.com/#view/Microsoft_AAD_IG/EntitlementManagementMenuBlade/~/Catalogs/catalogId/~)
→ See: `catalog-resources.png`

### 3. Create Access Package
- **Path:** **Access packages** → **+ New access package**
- **Details:**
  - Name: `Finance Full Access Bundle`
  - Catalog: `Lab-Catalog-Entitlement`
  - Add all 3 resources
- **URL:** [Access Packages](https://entra.microsoft.com/#view/Microsoft_AAD_IG/EntitlementManagementMenuBlade/~/AccessPackages)
→ See: `access-package.png`

### 4. Configure Policy
- **Path:** Access Package → **Policies** → **+ Add policy**
- **Settings:**
  - **For users**: `Specific users` → Add `test.user@practicecyber.onmicrosoft.com`
  - **Require approval**: `Yes` → Approver: You
  - **Expiration**: `30 days` (use `1 minute` for quick demo)
- **URL:** [Policy Settings](https://entra.microsoft.com/#view/Microsoft_AAD_IG/EntitlementManagementMenuBlade/~/Policies)
→ See: `policy-settings.png`

### 5. User Submits Request (MyAccess)
- **Action:**
  1. Open **incognito** → [myaccess.microsoft.com](https://myaccess.microsoft.com)
  2. Sign in as `test.user@practicecyber.onmicrosoft.com`
  3. Find **Finance Full Access Bundle**
  4. Click **Request** → Justification: "Q4 reporting access required"
- **URL:** [MyAccess](https://myaccess.microsoft.com)
→ See: `myaccess-request.png`

### 6. Admin Approves Request
- **Path:** Entra → **Entitlement Management** → **Requests**
- **Action:** Find request → **Approve** → Comment: "Approved for reporting"
- **URL:** [Requests](https://entra.microsoft.com/#view/Microsoft_AAD_IG/EntitlementManagementMenuBlade/~/Requests)
→ See: `approval-flow.png`

### 7. Validate Access Granted
- **Check:**
  - `test.user` in `Finance-Team` group
  - App in **My Apps**
  - SharePoint site accessible
- **URL:** [Azure AD Users](https://entra.microsoft.com/#view/Microsoft_AAD_Users/UserManagementMenuBlade/~/AllUsers)
→ See: `access-assigned.png`

### 8. Validate Auto-Revocation
- **Action:** Wait for expiration (or force 1-min policy)
- **Verify:**
  - Access removed from MyAccess
  - Group membership gone
  - App/SharePoint inaccessible
- **URL:** [MyAccess](https://myaccess.microsoft.com)
→ See: `access-expired.png`

### GIF: Full End-to-End Flow
> **File**: `Screenshots/demo-request-flow.gif`  
> **Tool**: ScreenToGif  
> **Duration**: 22s  
> **Steps Recorded**:  
> 1. User opens MyAccess  
> 2. Requests package  
> 3. Admin approves  
> 4. Access appears  
> 5. Expires after 1 min → removed

→ See: `demo-request-flow.gif`

---
## Troubleshooting: Entitlement Management
| Error / Symptom | Root Cause | Resolution Steps |
|------------------|------------|------------------|
| **Package not visible in MyAccess** | User not in policy scope | Add user to **"For users"** in policy |
| **Request stuck in "Pending"** | Approver not assigned | Set yourself as approver in policy |
| **Access not granted after approval** | Resource sync delay | Wait 2–5 mins or refresh My Apps |
| **Expiration not working** | Policy set to "No expiration" | Change to **30 days** or **1 minute** |
| **SharePoint access denied** | Site permissions not inherited | Ensure site allows group members |

---
## Personal Note
After over a year out of formal employment, I’ve dedicated **300+ hours** to **hands-on IAM labs**, **SC-300 prep**, and **real-world simulations** like this one.  
This project proves I can deliver **production-grade Identity Governance** — from policy design to user experience — even in self-directed environments.  
I’m ready to bring **automation, compliance, and user-centric IAM** to your team.

**Author:** **Yaz**  
*Preparing: SC-300 (Dec 2025), SC-200 (Jan 2026), AWS Security Specialty*

---

![SC-300 Ready](https://img.shields.io/badge/SC--300-Ready-blue)
![Self-Taught IAM](https://img.shields.io/badge/IAM-Self%20Taught-brightgreen)
![Available](https://img.shields.io/badge/Status-Available%20for%20Hire-green)
