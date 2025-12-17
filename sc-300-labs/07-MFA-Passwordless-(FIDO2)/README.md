# Microsoft Entra ID – MFA + Passwordless Authentication (FIDO2)

## 🧩 Overview
This lab demonstrates how to implement **Multi-Factor Authentication (MFA)** combined with **Passwordless authentication using FIDO2 security keys** in Microsoft Entra ID (Azure AD).

The goal is to showcase a **modern, phishing-resistant identity security design** where users authenticate **without passwords**, significantly reducing the risk of credential theft, phishing, and account takeover.

This lab reflects a **real-world enterprise implementation** aligned with **Zero Trust principles**, **FIDO2 standards**, and **NIST 800-63B** recommendations.

---

## ⚠️ Real-World Risk

> **Over 80% of breaches involve stolen or weak credentials** (Verizon DBIR).

Traditional MFA still relies on passwords, which means:
- Passwords can be phished
- Passwords can be reused
- Passwords can be leaked or brute-forced

**Passwordless authentication removes the password entirely**, eliminating the most exploited attack surface.

### This lab reduces:
- Credential phishing  
- Password spray attacks  
- MFA fatigue attacks  
- Account takeover (ATO)

### Security alignment:
- Zero Trust architecture  
- NIST 800-63B (Phishing-resistant MFA)  
- Microsoft passwordless strategy  

---

## 🛠 What I Built
- **2 cloud-only users** (Jasmine, Aaron)
- **Security Group:** `Passwordless-FIDO2-Users`
- **FIDO2 authentication method enabled**
- **Passwordless sign-in tested using FIDO2 security key**
- **Conditional Access Policy** enforcing MFA
- **End-to-end authentication flow validated**
- **Full cleanup performed** (no persistent lab artifacts)

---

## 🎥 Full Authentication Flow Demo (Passwordless Sign-In)
![Passwordless Demo](./Screenshot/demo_FIDO2.gif)

---

## 🛠 Architecture Diagram
<img src="./Screenshot/FIDO2-Architecture.png" width="400"/>

---

## 🛠 Authentication Decision Flow
<img src="./Screenshot/FIDO2-Auth-Flow.png" width="400"/>

---

## 🔐 Conditional Access Policy (JSON – Exportable)

```json
{
  "displayName": "Require Passwordless MFA (FIDO2)",
  "state": "enabled",
  "conditions": {
    "clientAppTypes": ["all"],
    "applications": {
      "includeApplications": ["All"]
    },
    "users": {
      "includeGroups": ["<GROUP_GUID_PASSWORDLESS_USERS>"]
    }
  },
  "grantControls": {
    "operator": "OR",
    "builtInControls": ["mfa"]
  }
}


```
----


### 🧪 Step-by-Step Evidence

| # | Action                            | Screenshot                                                                              |
| - | --------------------------------- | --------------------------------------------------------------------------------------- |
| 1 | Create cloud-only users           | <img src="./Screenshot/Users.png" width="180" style="border-radius:6px;"/>              |
| 2 | Create security group             | <img src="./Screenshot/Group.png" width="180" style="border-radius:6px;"/>              |
| 3 | Enable FIDO2 authentication       | <img src="./Screenshot/FIDO2-enabled.png" width="180" style="border-radius:6px;"/>      |
| 4 | Register FIDO2 security key       | <img src="./Screenshot/FIDO2-registration.png" width="180" style="border-radius:6px;"/> |
| 5 | Configure Conditional Access      | <img src="./Screenshot/Conditional-Access.png" width="180" style="border-radius:6px;"/> |
| 6 | Passwordless sign-in test         | <img src="./Screenshot/Passwordless-login.png" width="180" style="border-radius:6px;"/> |
| 7 | Access granted (no password used) | <img src="./Screenshot/Access-granted.png" width="180" style="border-radius:6px;"/>     |


----

## 🔎 IAM Implementation Details

## 1️⃣ Create Cloud-Only Identities
Purpose (IAM reasoning):
Cloud-only identities remove on-prem dependencies and reflect modern cloud-first IAM architectures.
Actions:
- Navigate to Microsoft Entra ID → Users
- Create two users:
    Jasmine Demo
    Aaron Demo
- Set strong temporary passwords (for initial sign-in only)
- Mark users as cloud-only
Validation:
  - Users appear in Entra ID
No hybrid or directory sync attributes present
📸 Screenshot: Users.png

## 2️⃣ Create IAM Security Group
Purpose (IAM reasoning):
Groups enable scalable policy enforcement and reduce administrative overhead.
Actions:
  - Go to Microsoft Entra ID → Groups
  - Create security group:
    Name: Passwordless-FIDO2-Users
    Type: Security
    Add Jasmine and Aaron as members
Validation:
- Group membership confirmed
- Group ready for Conditional Access & auth method scoping
📸 Screenshot: Group.png

## 3️⃣ Enable FIDO2 Authentication Method
Purpose (IAM reasoning):
Authentication Methods policy defines which credentials are allowed, where, and for whom.
Actions:
  - Navigate to Microsoft Entra ID → Security → Authentication methods
  - Select FIDO2 Security Keys
Enable the method
Configure:
  - Allow self-service registration
  - Restrict usage to Passwordless-FIDO2-Users
  - Enforce key restrictions (recommended defaults)
Validation:
  - FIDO2 enabled
  - Scoped to IAM-controlled group only
📸 Screenshot: FIDO2-enabled.png

## 4️⃣ Register FIDO2 Security Key
Purpose (IAM reasoning):
Credential registration binds a cryptographic key pair to a user identity.
Actions:
  - Sign in as Jasmine
  - Go to My Security Info
  - Register a FIDO2 security key
Complete:
  - PIN setup
  - Physical key challenge
Validation:
  - Security key appears under authentication methods
  - No password involved in authentication
📸 Screenshot: FIDO2-registration.png

## 5️⃣ Configure Conditional Access Policy
Purpose (IAM reasoning):
Conditional Access enforces policy-based authentication decisions at runtime.
Actions:
  - Navigate to Microsoft Entra ID → Security → Conditional Access
Create policy:
  - Name: IAM - Require Passwordless MFA (FIDO2)
Assign:
  - Users: Passwordless-FIDO2-Users
Apps: All cloud apps
  Grant:
  - Require multi-factor authentication
  - Enable policy
Validation:
  - Policy appears as enabled
  - Scoped correctly to group
📸 Screenshot: Conditional-Access.png

## 6️⃣ Validate Passwordless Authentication Flow
Purpose (IAM reasoning):
Verification ensures that FIDO2 satisfies MFA requirements and removes password dependency.
Actions:
  - Sign out completely
  - Start a new sign-in as Jasmine
Observe authentication flow:
  - Username entered
  - FIDO2 challenge presented
  - No password prompt
  - Access granted
Validation:
  - MFA satisfied via FIDO2
  - Passwordless authentication confirmed
📸 Screenshot: Passwordless-login.png

## 7️⃣ Access Validation & Policy Enforcement
Purpose (IAM reasoning):
Final confirmation that Conditional Access is enforcing intended controls.
Actions:
  - Access a cloud application
  - Confirm no bypass or fallback to password
Validation:
  - Access granted only after phishing-resistant MFA
📸 Screenshot: Access-granted.png


-----

## 🔐 License Assignment & IAM Validation
To enable Conditional Access and advanced authentication methods, Microsoft Entra ID P2 licenses were assigned.
Steps performed:
  - Microsoft 365 Admin Center → Billing → Licenses
  - Assigned Microsoft Entra ID P2 to:
    Jasmine@PracticeCyber.onmicrosoft.com
    Aaron@PracticeCyber.onmicrosoft.com
  - Verified license assignment in Entra Admin Center
    
## IAM Validation Results:
  - Conditional Access policy successfully enforced
  - FIDO2 accepted as phishing-resistant MFA
  - Passwordless authentication confirmed

## 🧰 IAM Tools & Services Used
  - Microsoft Entra ID (Azure AD)
  - Conditional Access
  - Authentication Methods (FIDO2)
  - Microsoft Azure Portal
  - Microsoft 365 Admin Center
  - Microsoft Graph Dev Center

## 💡 IAM Outcome
This lab demonstrates an enterprise-grade IAM control implementing phishing-resistant, passwordless authentication using FIDO2 and Conditional Access.
From an IAM perspective, this solution:
  - Strengthens identity assurance
  - Reduces credential-based risk
  - Scales through group-based governance
  - Aligns with Zero Trust and compliance frameworks
    
## 🔐 IAM Key Takeaway:
Passwordless FIDO2 authentication is a core IAM capability, not an optional enhancement — it is essential for modern identity security.

## 👩‍💻 *Author:* [Yaz.](https://www.linkedin.com/in/yasmina-g-p-227576a)
