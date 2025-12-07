# Lab 06 Azure AD Connect – Hybrid Identity Sync (On-Prem AD → Azure AD)

---

## Overview
This lab demonstrates how to deploy a **Hybrid Identity environment** using **Azure AD Connect**, synchronizing on-premises Active Directory identities into Microsoft Entra ID (Azure AD).  
The configuration simulates a real enterprise setup where organizations maintain on-prem AD but require seamless identity sync to the cloud for authentication and access to SaaS services.

This lab reflects real-world hybrid identity implementations used by enterprises transitioning into cloud-based identity platforms.

---

## Real-World Risk
More than 70% of organizations still depend on on-prem Active Directory, making hybrid identity the top attack surface for modern ransomware groups.

Without proper synchronization, identity governance, and secure sign-in flows, companies risk:

- Identity drift  
- Credential misuse  
- Inconsistent lifecycle management  
- Failed authentication due to UPN mismatches  

A correct hybrid identity approach aligns with **Zero Trust** and **Identity Governance** best practices.

---

## What I Built
- On-Prem Active Directory Domain: `corp.contoso.local`  
- User and Group Organizational Units  
- Azure AD Connect server joined to the domain  
- Password Hash Sync (PHS)  
- Public UPN suffix routing (`contoso.com`)  
- Selective OU filtering  
- Sync verification from AD → Azure AD  
- Login flow validation  
- Complete lab cleanup  

---

## Hybrid Identity Diagram
*(Replace with your actual image)*

![Hybrid Identity Diagram](./Screenshot/hybrid_diagram.png)

---

## Azure AD Connect Sync Flow Diagram
*(Replace with your actual image)*

![Sync Flow Diagram](./Screenshot/sync_flow_diagram.png)

---

## Azure AD Connect Config (JSON Example)

```json
{
  "syncRules": {
    "source": "OnPremAD",
    "destination": "AzureAD",
    "signInMethod": "PasswordHashSync",
    "filteredOUs": [
      "OU=Users,OU=Corp,DC=corp,DC=contoso,DC=local",
      "OU=Groups,OU=Corp,DC=corp,DC=contoso,DC=local"
    ],
    "upnMapping": "userPrincipalName",
    "runProfile": "Delta Sync"
  }
}
```

---

| # | Action | Screenshot |
|---|---------|-------------|
| 1 | Create on-prem AD OUs | <img src="./Screenshot/OUs.png" width="180" height="120" style="object-fit: cover; border-radius: 6px; box-shadow: 0 1px 3px rgba(0,0,0,0.1);"/> |
| 2 | Create on-prem users | <img src="./Screenshot/AD-Users.png" width="180" height="120" style="object-fit: cover; border-radius: 6px; box-shadow: 0 1px 3px rgba(0,0,0,0.1);"/> |
| 3 | Configure UPN suffix | <img src="./Screenshot/UPN-Suffix.png" width="180" height="120" style="object-fit: cover; border-radius: 6px; box-shadow: 0 1px 3px rgba(0,0,0,0.1);"/> |
| 4 | Install Azure AD Connect | <img src="./Screenshot/AADConnect-Install.png" width="180" height="120" style="object-fit: cover; border-radius: 6px; box-shadow: 0 1px 3px rgba(0,0,0,0.1);"/> |
| 5 | Configure OU Filtering  | <img src="./Screenshot/OU-Filtering.png" width="180" height="120" style="object-fit: cover; border-radius: 6px; box-shadow: 0 1px 3px rgba(0,0,0,0.1);"/> |
| 6 | Initial Sync Completed | <img src="./Screenshot/Initial-Sync.png" width="180" height="120" style="object-fit: cover; border-radius: 6px; box-shadow: 0 1px 3px rgba(0,0,0,0.1);"/> |
| 7 | Cloud identities appear | <img src="./Screenshot/AzureAD-Users.png" width="180" height="120" style="object-fit: cover; border-radius: 6px; box-shadow: 0 1px 3px rgba(0,0,0,0.1);"/> |
| 8 | User login test | <img src="./Screenshot/Login-Test.png" width="180" height="120" style="object-fit: cover; border-radius: 6px; box-shadow: 0 1px 3px rgba(0,0,0,0.1);"/> |


## Detailed Step-by-Step Breakdown

## 1. On-Prem Active Directory Setup
Installed Active Directory Domain Services (AD DS).
Created the domain: corp.contoso.local.
Configured DNS to support the new domain.
Created the following OU structure:
OU=Corp
OU=Users
OU=Groups
Evidence: ./Screenshot/OUs.png.

## 2. User Creation in On-Prem AD
Created test user accounts in OU=Users, for example:
John Doe (jdoe)
Jane Smith (jsmith)
Assigned strong initial passwords.
Evidence: ./Screenshot/AD-Users.png.

## 3. UPN Suffix Configuration
Added public UPN suffix contoso.com in Active Directory Domains and Trusts.
Updated user accounts so their UPN matches the public domain (for Azure sign-in), e.g. jdoe@contoso.com.
Evidence: ./Screenshot/UPN-Suffix.png.

## 4. Azure AD Connect Installation
Joined a Windows Server to the corp.contoso.local domain to host Azure AD Connect.
Downloaded and installed Azure AD Connect.
Chose the Customize option during setup.
Selected Password Hash Sync as the sign-in method.
Provided:
Azure AD Global Administrator credentials.
On-prem AD DS Enterprise/Domain Admin credentials.
Evidence: ./Screenshot/AADConnect-Install.png.

## 5. OU Filtering Configuration
In the Azure AD Connect wizard, limited synchronization to lab-specific OUs:
OU=Users
OU=Groups
This avoids syncing system accounts or unwanted objects.
Evidence: ./Screenshot/OU-Filtering.png.

## 6. Initial Sync Execution
Completed the Azure AD Connect wizard and triggered the initial synchronization.
Verified that synchronization profiles (Import → Sync → Export) ran successfully.
Evidence: ./Screenshot/Initial-Sync.png.

## 7. Cloud Identity Verification
In the Microsoft Entra admin center, navigated to:
Azure Active Directory → Users
Confirmed that on-prem users appeared as:
Source: Windows Server AD
Correct UPN: e.g. jdoe@contoso.com
Evidence: ./Screenshot/AzureAD-Users.png.

## 8. Login Test with Synced User
Opened a private browser session.
Browsed to https://portal.office.com (or another cloud app).
Signed in using the synced user, e.g. jdoe@contoso.com with the on-prem AD password.
Confirmed successful authentication.
Evidence: ./Screenshot/Login-Test.png.

## Tools Used
- Windows Server (Active Directory Domain Services)
- Azure AD Connect
- Microsoft Entra ID (Azure AD)
- Azure Portal
- Microsoft 365 Admin Center
- PowerShell (ADSync module)
## Outcome
This lab demonstrates a complete hybrid identity solution with:
- Identity synchronization from on-prem AD to Azure AD.
- Governance via OU filtering.
- Password Hash Sync as a secure and simple sign-in method.
- Correct UPN configuration for cloud sign-in.
- End-to-end login validation using a synced identity.
- The lab provides practical, job-ready experience for roles in Identity & Access Management, Cloud Engineering, and Security Engineering.

## Troubleshooting Tip
If users synchronize with a .local UPN or cannot sign in:
Verify that the public domain (for example, contoso.com) is added and verified in Microsoft Entra ID.
Ensure the public UPN suffix is configured and applied to users in on-prem AD.
Run a delta sync using PowerShell on the Azure AD Connect server:
Import-Module ADSync
Start-ADSyncSyncCycle -PolicyType Delta


## 👩‍💻 *Author:* [Yaz.](https://www.linkedin.com/in/yasmina-g-p-227576a)

