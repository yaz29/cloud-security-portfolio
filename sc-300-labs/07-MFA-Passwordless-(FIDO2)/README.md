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
## 1️⃣ Identity Creation 
- Created two cloud-only identities:
    Jasmine Demo
    Aaron Demo
No on-prem or hybrid identity dependency, reflecting modern cloud-first IAM.
📸 Users.png

## 2️⃣ Group-Based Access Control
- Created security group: Passwordless-FIDO2-Users
- Used group membership to scope:
- Authentication methods
- Conditional Access policies
- Demonstrates scalable IAM design.
📸 Group.png

## 3️⃣ Authentication Methods Governance (FIDO2)
- Enabled FIDO2 Security Keys under Authentication Methods
- Scoped registration and usage to the IAM-controlled group
- Allowed self-service registration with controlled policy boundaries
📸 FIDO2-enabled.png

## 4️⃣ Passwordless Credential Registration
- Registered a FIDO2 security key via My Security Info
- Completed user verification using:
- PIN
- Physical security key challenge
- Verified credential binding to user identity
📸 FIDO2-registration.png

## 5️⃣ Conditional Access Policy Enforcement
- Targeted Passwordless-FIDO2-Users
- Applied to All cloud apps
- Required Multi-Factor Authentication
- Validated that FIDO2 satisfies MFA requirements
- Enabled policy and confirmed enforcement
📸 Conditional-Access.png

## 6️⃣ Authentication Validation (IAM Perspective)
Performed clean sign-in tests
Observed authentication sequence:
- Username
- FIDO2 challenge
- No password prompt
- Access granted
Confirmed phishing-resistant MFA flow
📸 Passwordless-login.png

## 🔐 License Assignment & IAM Validation
To enable Conditional Access and advanced authentication methods, Microsoft Entra ID P2 licenses were assigned.
Steps performed:
  - Microsoft 365 Admin Center → Billing → Licenses
  - Assigned Microsoft Entra ID P2 to:
    Jasmine@PracticeCyber.onmicrosoft.com
    Aaron@PracticeCyber.onmicrosoft.com
  - Verified license assignment in Entra Admin Center
IAM Validation Results:
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
🔐 IAM Key Takeaway:
Passwordless FIDO2 authentication is a core IAM capability, not an optional enhancement — it is essential for modern identity security.
