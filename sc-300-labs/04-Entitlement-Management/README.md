# 04-Entitlement-Management
**Microsoft Entra ID Governance – Hands-On Lab**  
*Identity & Access Management (IAM) | Self-Directed Learning Project*
---

## Lab Objective

Configure and validate **Entitlement Management** in Microsoft Entra ID:  
- Build **catalogs** and **access packages**  
- Enforce **approval + expiration policies**  
- Simulate **end-to-end user request flow**  
- Prove **automatic access revocation**

---

## Evidence Table (All Screenshots & GIF Included)

> **Folder**: `screenshots/`  
> **All files uploaded and verified**

| Step | Description | Evidence |
|------|-------------|----------|
| 1 | Catalog created | `screenshots/catalog-created.png` |
| 2 | Resources added (Group, App, SharePoint) | `screenshots/catalog-resources.png` |
| 3 | Access package created | `screenshots/access-package.png` |
| 4 | Policy: Approval + 30-day expiration | `screenshots/policy-settings.png` |
| 5 | User submits request in MyAccess | `screenshots/myaccess-request.png` |
| 6 | Admin approves request | `screenshots/approval-flow.png` |
| 7 | Access granted (validated in Azure AD) | `screenshots/access-assigned.png` |
| 8 | Access auto-removed after expiration | `screenshots/access-expired.png` |
| **GIF** | **End-to-end flow: Request → Approval → Access → Revocation** | `screenshots/demo-request-flow.gif` |

---

## Step-by-Step Lab Guide

### 1. Create Catalog
1. Open **Microsoft Entra admin center**  
2. Navigate to **Identity Governance** → **Entitlement Management** → **Catalogs**  
3. Click **+ New catalog**  
4. Name: `Lab-Catalog-Entitlement` | Description: `Demo for IAM automation`  
5. **Screenshot**: `catalog-created.png`

### 2. Add Resources
- Security Group: `Finance-Team`  
- App: `Finance Analytics`  
- SharePoint Site: `Finance Docs`  
- **Screenshot**: `catalog-resources.png`

### 3. Create Access Package
- Name: `Finance Full Access Bundle`  
- Catalog: `Lab-Catalog-Entitlement`  
- Add all 3 resources  
- **Screenshot**: `access-package.png`

### 4. Configure Policy
- **Who can request**: Specific users (test user)  
- **Approval**: Required (1 stage, you as approver)  
- **Expiration**: 30 days (or 1 min for demo)  
- **Screenshot**: `policy-settings.png`

### 5. User Requests Access
1. Open **Incognito** → [myaccess.microsoft.com](https://myaccess.microsoft.com)  
2. Sign in as `test.user@tenant.onmicrosoft.com`  
3. Find **Finance Full Access Bundle**  
4. Click **Request** → Justification: "Q4 financial reporting access"  
5. **Screenshot**: `myaccess-request.png`

### 6. Admin Approves
1. Back to Entra admin center → **Requests**  
2. Locate request → **Approve** with comment  
3. **Screenshot**: `approval-flow.png`

### 7. Validate Access Granted
- Group membership added in Azure AD  
- App appears in **My Apps**  
- SharePoint site accessible  
- **Screenshot**: `access-assigned.png`

### 8. Validate Auto-Removal
- Wait for expiration (or force 1-min policy)  
- Access removed from:  
  - Group  
  - App  
  - SharePoint  
- **Screenshot**: `access-expired.png`

---

## Demo GIF: Full User Journey

> **File**: `screenshots/demo-request-flow.gif`  
> **Duration**: 22 seconds  
> **Tool**: ScreenToGif

![End-to-End Access Request Flow](screenshots/demo-request-flow.gif)

---

## Key Takeaways 

| Principle | Applied Here |
|---------|--------------|
| **Least Privilege** | Bundled only required resources |
| **Zero Trust** | Approval + time-bound access |
| **Automation** | No manual cleanup — lifecycle rules |
| **Auditability** | Full logging in Entra ID |
| **User Experience** | Seamless MyAccess portal |

---
**Skills Demonstrated**:
| Skill | Level |
|------|-------|
| Microsoft Entra ID Governance | Advanced |
| Entitlement Management (Catalogs, Access Packages, Policies) | Expert |
| MyAccess Portal & UX | Advanced |
| Access Lifecycle Automation | Advanced |
| IAM Troubleshooting & Validation | Strong |

---

## Useful Links
- [MyAccess Portal](https://myaccess.microsoft.com)
- [Entitlement Management Docs](https://learn.microsoft.com/entra/id-governance/entitlement-management-overview)
- [SC-300 Exam Guide](https://learn.microsoft.com/certifications/exams/sc-300)

---

## Let's Connect
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue)](https://www.linkedin.com/in/your-profile)  
*Open to **Identity Engineer**, **IAM Consultant**, or **Cloud Security** roles*

---

## Reference 

![Microsoft Certified Ready](https://img.shields.io/badge/Microsoft-Certification%20Ready-blue)
![Self-Taught](https://img.shields.io/badge/Skill-Self%20Taught-brightgreen)
![Available for Hire](https://img.shields.io/badge/Status-Available%20for%20Hire-green)
