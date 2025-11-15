
# 03-Single-Sign-On (SSO) & SCIM Provisioning Lab

## 🧾 TL;DR (Executive Summary)
Implemented and validated **SAML 2.0 Single Sign-On (SSO)** between **Microsoft Entra ID (Azure AD)** and **Salesforce Developer Edition (SP-initiated flow)**.

### 🛑 Key Finding on SCIM Provisioning
Due to platform restrictions on the Salesforce Developer Edition (or Trailhead Playgrounds), **SCIM provisioning was blocked**. This project successfully documents the SAML SSO configuration and provides a **manual simulation** and detailed steps to understand the full user lifecycle (Create, Update, Disable, Delete) as it *would* occur with a working SCIM endpoint.

TThis lab validates proficiency in Identity Federation, SAML configuration, and user provisioning concepts essential for modern **Identity and Access Management (IAM)**.

---

## 🎯 Learning Objectives
By completing this lab, I was able to:
* ✅ Configure a secure **SAML 2.0 trust relationship** between an IdP and an SP.
* ✅ Demonstrate **SP-initiated SSO login flow**.
* ✅ Define and use **Dynamic Groups** to automate application assignment.
* ✅ Understand and document the process for **SCIM provisioning** and **attribute mapping**.
* ✅ Manually simulate the **full user lifecycle (Create, Update, Disable, Delete)** to meet the project's learning goal despite platform limitations.

---
## Key Concepts
- **IdP (Identity Provider):** Microsoft Entra ID – issues SAML assertions
- **SP (Service Provider):** Salesforce – consumes SAML for login
- **SCIM:** Automates user/group sync across domains
- **JIT Assignment:** Access granted via dynamic group membership
- **Federation ID:** Links Entra user to Salesforce (`userPrincipalName`)

---
## What I Built
| Component | Details |
|---------|---------|
| **SaaS App** | Salesforce Developer Edition (`speed-velocity-336`) |
| **SSO Protocol** | SAML 2.0 (SP-initiated) |
| **IdP** | Microsoft Entra ID |
| **Dynamic Group** | `GRP-Salesforce-Users` (`user.department -eq "Salesforce Users"`) |
| **Test Users** | Alice, Bob, Carol (`@practicecyber.onmicrosoft.com`) |
| **Lifecycle** | Create → Update → Disable → Delete (manual) |
| **Cleanup** | 100% – app, users, group removed |

----

## Architecture Diagram
This diagram illustrates the intended full implementation flow (SAML for authentication, SCIM for provisioning).

<img width="900" src="./Screenshots/sso_scim_architecture.png" alt="SAML/OIDC SSO + SCIM Provisioning Architecture" />


*Dynamic group → JIT → SSO → Manual sync simulation*


## 🔄 SCIM Provisioning Decision Flow 

<img width="800" src="./Screenshots/scim_provisioning_flow.png" alt="SCIM Lifecycle: Create, Update, Disable, Delete" />


***Note:** The SCIM provisioning path was simulated for this specific lab environment.*

---

### Part 1: SAML SSO Implementation (Successful)

## 💻 Full Implementation Workflow & Evidence

This table chronologically documents all steps, from the successful SAML SSO configuration to the SCIM provisioning failure and the subsequent manual simulation.

| # | **Feature** | **Action** | **Status** | **Screenshot / Evidence** |
|:-:|:------------|:---------------|:---------------|:---------------|
| **1** | **Users & Group** | Create 3 Test Users (Alice, Bob, Carol) in Entra ID with `Department = Salesforce Users`. | ✅ Success | <img src="./Screenshots/Users_Created.png" width="180" height="120"/> |
| **2** | **Users & Group** | Create Dynamic Group `GRP-Salesforce-Users` with rule based on the `Department` attribute. | ✅ Success | <img src="./Screenshots/Dynamic_Group.png" width="180" height="120"/> |
| **3** | **App Setup** | Add Salesforce Enterprise Application and assign it to the Dynamic Group. | ✅ Success | <img src="./Screenshots/App_Assignment.png" width="180" height="120"/> |
| **4** | **SAML Config** | Configure SAML 2.0 in Entra ID and upload Metadata XML to Salesforce. | ✅ Success | <img src="./Screenshots/SAML_Config.png" width="180" height="120"/> |
| **5** | **User Creation** | **Manual Step:** Create test users in Salesforce and set **Federation ID** to match the Entra ID UPN (`user.userprincipalname`). | ✅ Success (Manual) | <img src="./Screenshots/Users_Manual.png" width="180" height="120"/> |
| **6** | **SSO Test** | Test SP-initiated login flow from Salesforce URL $\rightarrow$ Entra ID Login $\rightarrow$ Salesforce Dashboard. | ✅ Success | <img src="./Screenshots/SSO_Login.png" width="180" height="120"/> |
| **7** | **SCIM Attempt** | Attempt to configure Automatic Provisioning (SCIM) in Entra ID and test connection. | ❌ **Failed** | <img src="./Screenshots/SCIM_Disabled.png" width="180" height="120"/> |
| **8** | **SCIM Failure** | **Root Cause:** SCIM Provisioning is not supported in the Salesforce Developer Edition/Trial Org. | 🛑 **Blocked** | *Provisioning logs show "SCIM endpoint not supported."* || <img src="./Screenshots/App_Assignment.png" width="180" height="120"/> |
| **9** | **SCIM Mapping** | **Conceptual:** Document required attribute mapping (e.g., `userPrincipalName` $\rightarrow$ `userName`) despite UI being inaccessible. | **Skipped** | [View Conceptual Mapping](./Exports/SCIM_Mapping.json) |
| **10** | **Lifecycle** | **Manual Simulation:** Update Alice's job title in Entra $\rightarrow$ manually update in Salesforce. | ✅ Success (Manual) | <img src="./Screenshots/SSO_Login.png" width="180" height="120"/> | <img src="./Screenshots/SSO_Login1.png" width="180" height="120"/> |
| **11** | **Audit Trail** | Export Entra ID Provisioning Logs showing the continuous SCIM connection errors. | 🛑 **Blocked** | <img src="./Screenshots/Provisioning_Logs_Disabled.png" width="180" height="120"/>  |

---

## 🪜 Step-by-Step Implementation Guide (Avoiding Pitfalls)

### Part A: Initial Setup in Microsoft Entra ID (Azure AD)

## Step-by-Step Implementation

### 1. Create Test Users
- **Path:** Microsoft Entra Admin Center → **Identity** → **Users** → **All users** → **New user** → **Create user**  
- **Action:** Create three test accounts:  
  - `alice.johnson@practicecyber.onmicrosoft.com`  
  - `bob.smith@practicecyber.onmicrosoft.com`  
  - `carol.lee@practicecyber.onmicrosoft.com`  
- **Crucial Step:** Set **Department** = `Salesforce Users` for **all three users**  
- **URL:** [Entra Users](https://entra.microsoft.com/#view/Microsoft_AAD_Users/UserManagementMenuBlade/~/AllUsers)  
→ See: `Users_Created.png`

### 2. Create Dynamic Group
- **Path:** Microsoft Entra Admin Center → **Identity** → **Groups** → **All groups** → **New group**  
- **Group type:** `Security`  
- **Membership type:** `Dynamic User`  
- **Group name:** `GRP-Salesforce-Users`  
- **Dynamic user rule:** `(user.department -eq "Salesforce Users")`  
- **Wait:** Allow **5–10 minutes** for membership to propagate before testing SSO  
- **URL:** [Entra Groups](https://entra.microsoft.com/#view/Microsoft_AAD_IAM/GroupsManagementMenuBlade/~/AllGroups)  
→ See: `Dynamic_Group.png`

### 3. Add Salesforce Enterprise Application
- **Path:** Microsoft Entra Admin Center → **Identity** → **Applications** → **Enterprise applications** → **New application**  
- **Action:** Click **Create your own application** → Name: `Salesforce-Lab` → **Integrate any other application you don't find in the gallery (Non-gallery)** → **Create**  
- **URL:** [Enterprise Apps](https://entra.microsoft.com/#view/Microsoft_AAD_IAM/EnterpriseAppsMenuBlade/~/Overview)  
→ See: `App_Added.png`

### 4. Configure SAML SSO in Salesforce (SP-Initiated)
- **Path:** Salesforce → **Setup** → Search `Single Sign-On Settings` → **Single Sign-On Settings** → **New** (SAML Enabled must be checked)  
- **SAML SSO Settings:**  
  - **Name:** `STS - Microsoft Entra SSO`  
  - **API Name:** `STS_Microsoft_Entra_SSO`  
  - **Issuer:** `https://sts.windows.net/<your-tenant-id>/` *(from Entra metadata)*  
  - **Entity Id:** `urn:salesforce:lab`  
  - **SAML Identity Type:** `Assertion contains the Federation ID from the User object`  
  - **Identity Provider Certificate:** *(upload later)*  
  - **Request Signing Certificate:** `Default`  
  - **Assertion Decryption Certificate:** `Default`  
  - **SAML Identity Location:** `Identity is in the NameIdentifier element of the Subject statement`  
- **Crucial:** Copy the **Login URL** and **Assertion Consumer Service (ACS) URL** (e.g., `https://speed-velocity-336.my.salesforce.com`)  
- **URL:** Salesforce Setup → Single Sign-On  
→ See: `SAML_Config_Salesforce.png`

### 5. Configure SAML SSO in Microsoft Entra ID (IdP)
- **Path:** Enterprise App `Salesforce-Lab` → **Single sign-on** → **SAML**  
- **Basic SAML Configuration:**  
  - **Identifier (Entity ID):** `urn:salesforce:lab`  
  - **Reply URL (ACS URL):** `https://speed-velocity-336.my.salesforce.com` *(from Salesforce)*  
  - **Sign-on URL:** `https://speed-velocity-336.lightning.force.com`  
- **User Attributes & Claims:**  
  - **Unique User Identifier (NameID):** `user.userprincipalname`  
- **SAML Signing Certificate:**  
  - Download **Federation Metadata XML**  
- **URL:** [Entra SAML Config](https://entra.microsoft.com/#view/Microsoft_AAD_IAM/EnterpriseAppMenuBlade/~/SingleSignOn)  
→ See: `SAML_Config_Entra.png`

### 6. Finalize SAML in Salesforce
- **Path:** Salesforce → **Single Sign-On Settings** → Edit `STS - Microsoft Entra SSO`  
- **Action:** Upload the **Federation Metadata XML** downloaded from Entra ID  
- **Enable:** Check **SAML Enabled** → **Save**  
- **My Domain SSO Enforcement (Optional):**  
  - Setup → **My Domain** → **Authentication Configuration** → Edit → Check `STS - Microsoft Entra SSO` → Save  
→ See: `SAML_Certificate_Uploaded.png`

### 7. Manually Create Users in Salesforce (SCIM Workaround)
- **Path:** Salesforce → **Setup** → **Users** → **Users** → **New User**  
- **Action:** Create 3 users with:  
  | Name         | Email                                | Federation ID                        | Profile       | Active |
  |--------------|--------------------------------------|--------------------------------------|---------------|--------|
  | Alice Johnson| `alice.johnson@practicecyber.onmicrosoft.com` | `alice.johnson@practicecyber.onmicrosoft.com` | Standard User | Yes |
  | Bob Smith    | `bob.smith@practicecyber.onmicrosoft.com`     | `bob.smith@practicecyber.onmicrosoft.com`     | Standard User | Yes |
  | Carol Lee    | `carol.lee@practicecyber.onmicrosoft.com`     | `carol.lee@practicecyber.onmicrosoft.com`     | Standard User | Yes |
- **Crucial:** **Federation ID must exactly match** the Entra `userPrincipalName`  
- **URL:** Salesforce Users Setup  
→ See: `Users_Manual.png`

### 8. Assign App to Dynamic Group
- **Path:** Enterprise App `Salesforce-Lab` → **Users and groups** → **Add user/group**  
- **Select:** `GRP-Salesforce-Users`  
- **Assignment required:** `Yes`  
- **URL:** [Entra Users & Groups](https://entra.microsoft.com/#view/Microsoft_AAD_IAM/EnterpriseAppMenuBlade/~/UsersAndGroups)  
→ See: `App_Assignment.png`

### 9. Test SSO (SP-Initiated)
- **Action:**  
  1. Open **incognito tab**  
  2. Go to: `https://speed-velocity-336.lightning.force.com`  
  3. Redirects to **Microsoft Login**  
  4. Enter `alice.johnson@practicecyber.onmicrosoft.com` → **SSO Success** → Salesforce Dashboard  
- **URL:** [Salesforce Login](https://speed-velocity-336.lightning.force.com)  
→ See: `SSO_Login.png`

### 10. Attempt SCIM Provisioning → **BLOCKED**
- **Path:** Enterprise App `Salesforce-Lab` → **Provisioning**  
- **Issue:** **Provisioning menu is disabled (grayed out)** in Entra ID  
- **Root Cause:** Salesforce Developer Edition **does not expose a SCIM endpoint**  
- **Result:** No Tenant URL, no Secret Token, no Test Connection possible  
- **URL:** [Entra Provisioning](https://entra.microsoft.com/#view/Microsoft_AAD_IAM/EnterpriseAppMenuBlade/~/Provisioning)  
→ See: `SCIM_Disabled_Entra.png`

### 11. SCIM Attribute Mapping → **SKIPPED**
> **No SCIM → no mapping interface available**  
**Expected mappings (conceptual):**  
```json
{
  "userPrincipalName": "userName",
  "displayName": "name.formatted",
  "mail": "emails[type eq \"work\"].value",
  "jobTitle": "title",
  "department": "department"
}

```



----------------------------------  
#### 3. Add Salesforce Enterprise Application
* **Path:** Microsoft Entra Admin Center $\rightarrow$ **Enterprise applications** $\rightarrow$ **New application**.
* **Action:** Select **Create your own application** $\rightarrow$ **Name:** `Salesforce-Lab`.

### Part B: Configuring SAML 2.0 (The Successful Part)

#### 4. Configure SAML SSO in Salesforce (SP)
* **Path:** Salesforce Setup $\rightarrow$ **Identity** $\rightarrow$ **Single Sign-On Settings** $\rightarrow$ **New** (SAML).
* **Name:** `STS - Microsoft Entra SSO`.
* **SAML Identity Type:** **Assertion contains the Federation ID from the User object** (This must match the NameID from Entra).
* **Entity ID:** `urn:salesforce:lab` (or similar).
* **Crucial Step:** Note the **Assertion Consumer Service (ACS) URL** from this page—you need it for Entra ID.

#### 5. Configure SAML SSO in Microsoft Entra ID (IdP)
* **Path:** Entra Enterprise Application `Salesforce-Lab` $\rightarrow$ **Single sign-on** $\rightarrow$ **SAML**.
* **Basic SAML Configuration (Section 1):**
    * **Identifier (Entity ID):** Must match the value in Salesforce (e.g., `urn:salesforce:lab`).
    * **Reply URL (Assertion Consumer Service URL):** Use the ACS URL noted from Salesforce.
    * **Sign on URL:** Use your Salesforce domain login URL.
* **Attributes & Claims (Section 2):** Ensure the **Unique User Identifier (Name ID)** is set to `user.userprincipalname`.
* **SAML Signing Certificate (Section 3):** Download the **Federation Metadata XML**.

#### 6. Finalize SAML Configuration in Salesforce
* **Path:** Return to the Salesforce **Single Sign-On Settings** page.
* **Identity Provider Certificate:** Upload the **Federation Metadata XML** downloaded from Entra ID.
-------------
## Step-by-Step Implementation

### 1. Create Test Users
- Emails: `alice.johnson@...`, `bob.smith@...`, `carol.lee@...`  
- Department: `Salesforce Users`  
→ See: `Users_Created.png`

### 2. Create Dynamic Group
- Name: `GRP-Salesforce-Users`  
- Rule: `(user.department -eq "Salesforce Users")`  
→ See: `Dynamic_Group.png`

### 3. Add Salesforce App
- Path: **Enterprise Apps → New → Create own app → Salesforce-Lab**  
→ See: `App_Added.png`

### 4. Configure SAML SSO
- Identifier: `urn:salesforce:lab`  
- Reply URL: `https://speed-velocity-336.my.salesforce.com`  
- Metadata exchanged (Entra ↔ Salesforce)  
→ See: `SAML_Config.png`

### 5. Enable SCIM → **BLOCKED**
> **SCIM not supported in Developer/Trial orgs**  
> - No “Provisioning” menu  
> - No SCIM endpoint or token  
> **Actual result:** `"SCIM endpoint not supported"`  
→ See: `SCIM_Disabled.png`

### 6. Attribute Mapping → **SKIPPED**
> No SCIM → no mapping UI  
**Expected mappings:**
```json
userPrincipalName → userName
displayName       → name.formatted
mail              → emails[type eq "work"].value
jobTitle          → title
department        → department
```
------------

#### 7. Manually Create Users in Salesforce **(Crucial Workaround for SCIM Limitation)** 
* **Path:** Salesforce Setup $\rightarrow$ **Users** $\rightarrow$ **New User**.
* **Action:** Create the Alice, Bob, and Carol user records.
* **Crucial Step:** Set the Salesforce **Federation ID** for each user to their full Entra ID email address (e.g., `alice.johnson@practicecyber.onmicrosoft.com`). **This is what allows SAML login to work.**

#### 8. Assign Application to Dynamic Group
* **Path:** Entra Enterprise Application `Salesforce-Lab` $\rightarrow$ **Users and groups** $\rightarrow$ **Add user/group**.
* **Group:** Select `GRP-Salesforce-Users`.
* **Assignment required:** **Yes**.

#### 9. Test SSO (SP-Initiated)
* **Path:** Open an incognito browser tab and navigate to your Salesforce domain URL (`https://<instance>.my.salesforce.com`).
* **Result:** Should successfully redirect through Microsoft Login and land you on the Salesforce dashboard.

### Part C: SCIM Provisioning (Limitation and Simulation)

#### 10. Attempt to Enable SCIM Provisioning (The Expected Failure)
* **Path:** Entra Enterprise Application `Salesforce-Lab` $\rightarrow$ **Provisioning**.
* **Provisioning Mode:** Set to **Automatic**.
* **Issue:** You **cannot** obtain the Tenant URL or Secret Token from Salesforce. The required **User Provisioning Settings** under Connected Apps are missing in Developer Edition.
* **Result:** The **Test Connection** in Entra ID will **Fail** with an error like `"SCIM endpoint not supported"`.

#### 11. SCIM Attribute Mapping (Conceptual Step)
* **Note:** The mapping interface is inaccessible due to the connection failure.
* **Conceptual Goal:** Map Entra attributes to Salesforce fields (e.g., `userPrincipalName` $\rightarrow$ `userName`, `mail` $\rightarrow$ `emails[type eq "work"].value`, etc.).
* [**View SCIM Mapping Schema (Conceptual)**](./Exports/SCIM_Mapping.json)

#### 12. Manual Lifecycle Simulation
To demonstrate understanding of the full lifecycle:

| Action | Azure Action | Salesforce Manual Action | SCIM Expected Result |
|:---------|:---------------|:-------------------------|:---------------------|
| **Update** Alice | Change Alice's `jobTitle` in Entra ID. | Manually edit Alice's user record in Salesforce and update the **Title** field. | **Update** (PATCH) operation. |
| **Disable** Bob | Uncheck the **Account enabled** box for Bob in Entra ID. | Manually edit Bob's user record in Salesforce and uncheck **Active**. | **Deprovision** (Disable) operation. |
| **Delete** Carol | Hard delete the user Carol in Entra ID. | Manually delete Carol's user record in Salesforce. | **Deprovision** (DELETE) operation. |

#### 13. Export Provisioning Logs (Documentation of Failure)
* **Path:** Entra Enterprise Application `Salesforce-Lab` $\rightarrow$ **Provisioning logs**.
* **Filter:** Filter by **Failure** status.
* **Action:** Download the logs. This file proves the SCIM endpoint was the limiting factor.
* [**View Provisioning Error Logs**](./Exports/Provisioning_Logs_No_SCIM.csv)

---

## 🧠 Troubleshooting: Salesforce $\leftrightarrow$ Microsoft Entra ID (SAML SSO)

This section documents the main issues encountered and their resolution.

| ❌ Error / Symptom | 🔍 Root Cause | 🛠️ Resolution Steps |
|--------------------|--------------|---------------------|
| **AADSTS50105:** *User not assigned to the application.* | User was not part of the assigned group, or dynamic membership hadn't propagated. | $\rightarrow$ Verified user met the Dynamic Group rule. $\rightarrow$ **Wait 5-10 minutes** for dynamic group membership to sync. |
| **Salesforce SSO Error:** *Cannot log you in because of an issue.* | The Salesforce **Federation ID** did not match the **NameID** sent in the SAML assertion. | $\rightarrow$ Manually set the **Federation ID** in Salesforce to exactly match the Entra ID **User Principal Name**. |
| Redirect goes to **Salesforce classic login** | The Salesforce domain was not configured to use the newly created SSO provider. | $\rightarrow$ In **Salesforce Setup $\rightarrow$ My Domain $\rightarrow$ Authentication Configuration**, ensure the new SSO provider is checked. |
| **SAML login loop** | Mismatch in **Reply URL (ACS)** or **Entity ID** between IdP and SP. | $\rightarrow$ Re-upload the latest **Federation Metadata XML** from Entra ID to Salesforce. |

---

## 🌟 Personal Note
I executed this project to grow my skills as an **Identity & Access Management professional**, demonstrating that **curiosity, consistency, and technical depth** are key—even when faced with platform limitations. Documenting the SCIM limitation and manually simulating the lifecycle was a valuable exercise in understanding the full process. I look forward to bringing this problem-solving mindset to a professional team.

**Author:** **Yaz**
