# 🔐 Lab 08 – Identity Protection & Risk Policies

## 📌 Overview

This lab demonstrates the implementation and configuration of **Identity Protection** and **risk-based policies** using **:contentReference[oaicite:0]{index=0}** (formerly Azure Active Directory).

The purpose of this lab is to showcase hands-on knowledge of **identity risk detection, automated remediation, and conditional access**, which are critical skills for roles in **Cloud Security, IAM, SOC, and Azure Administration**.

---

## 🎯 Lab Objectives

- Understand identity risk concepts (User Risk & Sign-in Risk)
- Configure Identity Protection features
- Create and enforce risk-based policies
- Apply automatic remediation controls
- Validate policy enforcement through testing

---

## 🧠 Key Concepts Covered

| Concept | Description |
|------|-------------|
| Identity Protection | Detects compromised identities using ML signals |
| User Risk | Probability that a user account is compromised |
| Sign-in Risk | Risk associated with a specific authentication attempt |
| Conditional Access | Policy-based access control |
| MFA | Multi-Factor Authentication as a mitigation control |

---

## 🛠️ Prerequisites

- Active Azure tenant
- Access to Microsoft Entra ID
- User with **Global Administrator** or **Security Administrator** role
- **Entra ID P2** license (or trial)

---

## 🎞️ GIF Flow: `identity-protection-mfa-enforcement.gif`


## 📐 Architecture & Flow Diagrams

The following diagrams provide architectural context, risk evaluation logic, and enforcement outcomes.

### Diagram 01 – High-Level Identity Architecture
**Description:** Shows where Identity Protection fits within the Azure identity architecture.

📸 File: `diagram-01-identity-architecture.png`

---

### Diagram 02 – Risk Detection & Policy Evaluation Flow
**Description:** Illustrates how sign-in signals are evaluated and mapped to risk policies.

📸 File: `diagram-02-risk-policy-flow.png`

---

### Diagram 03 – Enforcement & Remediation Actions
**Description:** Displays enforcement actions based on risk level (MFA, password reset, or block).

📸 File: `diagram-03-enforcement-remediation.png`

---


## 📊 Evidence Summary (Screenshots)

| # | Action                            | Screenshot                                                                              |
| - | --------------------------------- | --------------------------------------------------------------------------------------- |
| 1 | Create cloud-only users           | <img src="./Screenshots/protec-dashboard.png" width="180" style="border-radius:6px;"/>  |
| 2 | Create security group             | <img src="./Screenshots/risk-detected.png" width="180" style="border-radius:6px;"/>     |
| 3 | Enable FIDO2 authentication       | <img src="./Screenshots/risk-events.png" width="180" style="border-radius:6px;"/>      |
| 4 | Register FIDO2 security key       | <img src="./Screenshots/user-risk-policy.png" width="180" style="border-radius:6px;"/> |
| 5 | Configure Conditional Access      | <img src="./Screenshots/sign-in-risk-policy.png" width="180" style="border-radius:6px;"/> |
| 6 | Passwordless sign-in test         | <img src="./Screenshots/enforcement-test.png" width="180" style="border-radius:6px;"/> |

----


## 🧪 Lab Steps

### Step 1 – Access Identity Protection Dashboard
- Sign in to the Azure Portal with a security administrator account.
- Navigate to **Microsoft Entra ID** and select **Identity Protection**.
- Review the dashboard to understand current identity risk posture, including:
  - User risk detections
  - Risky sign-in events
  - Risk trends

📸 Screenshot: `dashboard.png`

**Purpose:** Establish baseline visibility of identity-related risks across the tenant.

---

### Step 2 – Analyze User Risk Detections
- Open **User risk detections** within Identity Protection.
- Identify users flagged with **Medium** or **High** risk levels.
- Review detection details such as:
  - Risk level
  - Detection type (e.g., leaked credentials, atypical behavior)
  - Detection time and status

📸 Screenshot: `risk-detected.png`

**Purpose:** Assess the likelihood that a user account has been compromised and requires remediation.

---

### Step 3 – Review Risky Sign-in Events
- Navigate to **Sign-in risk detections**.
- Examine sign-in attempts marked as risky.
- Analyze contextual signals including:
  - IP address and geolocation
  - Client and application used
  - Risk level assigned to the sign-in

📸 Screenshot: `risk-events.png`

**Purpose:** Identify suspicious authentication attempts that may indicate credential misuse or adversary activity.

---

### Step 4 – Configure User Risk Policy
- Navigate to **User risk policy** under Identity Protection.
- Create a policy targeting users with **Medium** and **High** user risk.
- Configure the control to **Require password change**.
- Scope the policy to appropriate users or groups.

📸 Screenshot: `user-risk-policy.png`

**Purpose:** Automatically remediate potentially compromised accounts by forcing credential reset.

---

### Step 5 – Configure Sign-in Risk Policy
- Navigate to **Sign-in risk policy**.
- Create a policy targeting **Medium** and **High** sign-in risk levels.
- Configure access control to **Require multi-factor authentication (MFA)**.
- Validate policy assignment and exclusions.

📸 Screenshot: `sign-in-risk-policy.png`

**Purpose:** Mitigate risky authentication attempts by enforcing strong authentication challenges.

---

### Step 6 – Validate Policy Enforcement
- Perform or simulate a risky sign-in scenario.
- Confirm that the sign-in triggers the configured controls:
  - MFA challenge for risky sign-ins
  - Password reset for high-risk users
- Review sign-in logs and Identity Protection status updates.

📸 Screenshot: `policy-enforcement.png`  

**Purpose:** Verify that identity risk detection and remediation are functioning as designed.

---

## ✅ Expected Results

- Automatic detection of identity risks
- Enforcement of security controls without manual intervention
- Improved security posture for identities
- Reduced risk of account compromise

---

## 🧩 Real-World Use Cases

- Preventing Account Takeover (ATO)
- Blocking or challenging suspicious sign-ins
- Enforcing Zero Trust security principles
- Automating IAM incident response

---

## 🧠 Skills Demonstrated

- Identity & Access Management (IAM)
- Cloud Security (Azure)
- Risk-based Authentication
- Conditional Access Policies
- Zero Trust Architecture

---
## 🛠️ Tools Used

 Identity Protection & IAM 
 Configuration & monitoring 
 Risk detection  Risk-based enforcement 
 Remediation control 
 Risk validation





