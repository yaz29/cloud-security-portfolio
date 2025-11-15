# 04-Entitlement-Management  
Lab Guide – Microsoft Entra ID Governance

This lab explains how to configure and test Entitlement Management in Microsoft Entra ID. You will create catalogs, access packages, policies, and test the end-user request process.

---

## Overview

Entitlement Management allows organizations to manage access to resources through catalogs and access packages. Users can request access through the MyAccess portal, and administrators can control approvals, assignments, and expiration.

---

## Objectives

- Create a catalog with groups, applications, and SharePoint sites.  
- Create an access package using the catalog.  
- Configure request, approval, and lifecycle policies.  
- Request the access package as an end user.  
- Validate access assignment and expiration.  
- Provide screenshot evidence of each step.

---

## Lab Steps

### 1. Create a Catalog

1. Open the Entra Admin Center.  
2. Navigate to **Identity Governance → Entitlement Management → Catalogs**.  
3. Select **New Catalog** and provide a name and description.  
4. Add resources:
   - Groups  
   - Applications  
   - SharePoint sites  

---

### 2. Create an Access Package

1. Go to **Access Packages → New Access Package**.  
2. Select the previously created catalog.  
3. Add required resources:
   - Group role assignments  
   - App roles  
   - SharePoint permissions  

---

### 3. Configure a Policy

1. Define who can request:
   - Internal users  
   - External (B2B) users  
   - Specific users or groups  

2. Configure approval workflow:
   - No approval  
   - Single-stage approval  
   - Multi-stage approval  

3. Configure lifecycle settings:
   - Assignment expiration  
   - Justification requirement  
   - Optional access reviews  

---

### 4. Publish the Access Package

Users can request access via the MyAccess portal:  
https://myaccess.microsoft.com

---

### 5. Test User Request

1. Sign in as a test user.  
2. Go to MyAccess and locate the access package.  
3. Submit a request.  
4. Complete approval steps (if required).  
5. Validate:
   - Group assignment  
   - Application access  
   - SharePoint permissions  

---

### 6. Validate Expiration

1. Confirm expiration settings.  
2. Validate that access is removed automatically after expiration.

---

## Screenshot Evidence

| Step | Description | Screenshot |
|------|-------------|------------|
| 1 | Catalog created | ![](screenshots/catalog-created.png) |
| 2 | Resources added to catalog | ![](screenshots/catalog-resources.png) |
| 3 | Access package created | ![](screenshots/access-package.png) |
| 4 | Policy configuration | ![](screenshots/policy-settings.png) |
| 5 | MyAccess request page | ![](screenshots/myaccess-request.png) |
| 6 | Approval workflow (if applicable) | ![](screenshots/approval-flow.png) |
| 7 | Access assignment confirmation | ![](screenshots/access-assigned.png) |
| 8 | Access removal after expiration | ![](screenshots/access-expired.png) |

> Replace each screenshot path with your own images (e.g., `/images/01.png`).

---

## Expected Results

- Users can request and receive access via MyAccess.  
- Approval workflow functions correctly.  
- Access is automatically removed based on lifecycle rules.  

---

## Useful Links

- MyAccess Portal: https://myaccess.microsoft.com  
- Entra Governance Documentation: https://learn.microsoft.com/entra/id-governance/
