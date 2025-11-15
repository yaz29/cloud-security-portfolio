
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

## 🏗️ Architecture Diagram
This diagram illustrates the intended full implementation flow (SAML for authentication, SCIM for provisioning).

<img width="900" src="./Screenshots/sso_scim_architecture.png" alt="SAML/OIDC SSO + SCIM Provisioning Architecture" />

***Note:** The SCIM provisioning path was simulated for this specific lab environment.*

---

## 💻 Project Flow & Outcome (Evidence)

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
| **8** | **SCIM Failure** | **Root Cause:** SCIM Provisioning is not supported in the Salesforce Developer Edition/Trial Org. | 🛑 **Blocked** | *Provisioning logs show "SCIM endpoint not supported."* |
| **9** | **SCIM Mapping** | **Conceptual:** Document required attribute mapping (e.g., `userPrincipalName` $\rightarrow$ `userName`) despite UI being inaccessible. | **Skipped** | [View Conceptual Mapping](./Exports/SCIM_Mapping.json) |
| **10** | **Lifecycle** | **Manual Simulation:** Update Alice's job title in Entra $\rightarrow$ manually update in Salesforce. | ✅ Success (Manual) | <img src="./Screenshots/Lifecycle_Manual.png" width="180" height="120"/> |
| **11** | **Deprovisioning** | **Manual Simulation:** Disable Bob and Delete Carol in Entra $\rightarrow$ manually deactivate/delete in Salesforce. | ✅ Success (Manual) | <img src="./Screenshots/User_Disabled.png" width="180" height="120"/> |
| **12** | **Audit Trail** | Export Entra ID Provisioning Logs showing the continuous SCIM connection errors. | ✅ Captured | <img src="./Screenshots/Provisioning_Logs_Error.png" width="180" height="120"/> / [Download CSV](./Exports/Provisioning_Logs_No_SCIM.csv) |

---

## 🪜 Step-by-Step Implementation Guide (Avoiding Pitfalls)

### Part A: Initial Setup in Microsoft Entra ID (Azure AD)

#### 1. Create Test Users
* **Path:** Microsoft Entra Admin Center $\rightarrow$ **Users** $\rightarrow$ **New user**.
* **Users:** Create the three test accounts (Alice, Bob, Carol).
* **Crucial Step:** Ensure all three users have their **Department** attribute set to `Salesforce Users`.

#### 2. Create Dynamic Group
* **Path:** Microsoft Entra Admin Center $\rightarrow$ **Groups** $\rightarrow$ **New group**.
* **Type:** Security $\rightarrow$ **Membership type:** Dynamic User.
* **Name:** `GRP-Salesforce-Users`
* **Dynamic Query Rule:** `(user.department -eq "Salesforce Users")`
* **Wait:** Allow 5-10 minutes for group membership to fully propagate before testing SSO.

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

#### 7. Manually Create Users in Salesforce (Crucial Workaround for SCIM Limitation)
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
