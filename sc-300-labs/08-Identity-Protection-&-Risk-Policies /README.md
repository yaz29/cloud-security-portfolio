# 🔐 Lab 08 – Identity Protection & Risk Policies

## 📌 Overview

This lab demonstrates the configuration and implementation of Microsoft Entra ID Identity Protection, including risk detection and risk-based Conditional Access policies.
The goal is to showcase hands-on expertise in detecting compromised identities using machine learning signals and automatically enforcing remediation controls — a critical capability for modern Zero Trust security architectures.
This implementation aligns with enterprise requirements for automated identity threat response and is highly relevant for roles in  **Cloud Security, IAM, SOC, and Azure Administration**.

---

## 🎯 Lab Objectives

- Understand identity risk concepts (User Risk & Sign-in Risk)
- Configure Identity Protection features
- Create and enforce risk-based policies
- Apply automatic remediation controls
- Validate policy enforcement through testing

---
## ⚠️ Real-World Risk
81% of breaches involve weak, stolen, or reused credentials (Verizon DBIR 2024).
Traditional authentication relies on static signals that adversaries can bypass. Identity Protection uses AI/ML-driven risk detection to identify anomalies in real time, enabling proactive remediation.

## This lab mitigates:

Account takeover (ATO) via leaked credentials
Impossible travel or anomalous sign-in patterns
Credential stuffing and spray attacks
Lateral movement from compromised accounts

## Security alignment:

Zero Trust principles
NIST 800-63B (risk-based authentication)
Microsoft recommended identity security practices


## 🛠 What I Built

Reviewed Identity Protection dashboard and risk detections
Configured User Risk policy (remediate High/Medium risk with password reset)
Configured Sign-in Risk policy (require MFA for High/Medium risk sign-ins)
Validated automated enforcement through simulated risky scenarios
Full cleanup performed (policies disabled, no persistent artifacts)

---
## 🎥 Full Authentication Flow identity-protection-mfa-enforcement.gif
<img src="./Screenshots/demo_FIDO2.gif" width="600"/>

---
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
| 1 | Create cloud-only users           | <img src="./Screenshots/dashboard.png" width="180" style="border-radius:6px;"/>  |
| 2 | Create security group             | <img src="./Screenshots/risk-detected.png" width="180" style="border-radius:6px;"/>     |
| 3 | Enable FIDO2 authentication       | <img src="./Screenshots/risk-events.png" width="180" style="border-radius:6px;"/>      |
| 4 | Register FIDO2 security key       | <img src="./Screenshots/user-risk-policy.png" width="180" style="border-radius:6px;"/> |
| 5 | Configure Conditional Access      | <img src="./Screenshots/sign-in-risk-policy.png" width="180" style="border-radius:6px;"/> |
| 6 | Passwordless sign-in test         | <img src="./Screenshots/enforcement-test.png" width="180" style="border-radius:6px;"/> |

----


## 🧪 Step-by-Step Implementation 
## 1️⃣ Access Identity Protection Dashboard
Purpose (Security reasoning):
Gain visibility into current identity risk posture using ML-powered detections.
## Actions:
Sign in to Microsoft Entra admin center
Navigate to Identity Protection > Overview

## Validation:
Dashboard shows risk summary, trends, and flagged users/sign-ins
📸 Screenshot: dashboard.pngç

---
## 2️⃣ Review Risky Users
Purpose (Security reasoning):
Identify accounts likely compromised (e.g., leaked credentials detected on dark web).

## Actions:
Go to Identity Protection > Risky users
Filter for Medium/High risk
Review detection details and risk factors

📸 Screenshot: risky-users.png

## 3️⃣ Review Risky Sign-ins
Purpose (Security reasoning):
Detect anomalous authentication attempts in real time.

## Actions:
Go to Identity Protection > Risky sign-ins
Analyze signals: unfamiliar location, impossible travel, anonymous IP, etc.

📸 Screenshot: risky-signins.png

## 4️⃣ Configure User Risk Policy
Purpose (Security reasoning):
Automatically remediate compromised accounts by forcing secure password reset.

## Actions:
Navigate to Identity Protection > User risk policy
Set policy to Medium and High risk
Control: Require password change
Scope: All users (or pilot group)

📸 Screenshot: user-risk-policy.png

## 5️⃣ Configure Sign-in Risk Policy
Purpose (Security reasoning):
Dynamically challenge suspicious sign-in attempts without blocking legitimate access.

## Actions:
Navigate to Identity Protection > Sign-in risk policy
Set policy to Medium and High risk
Control: Require multi-factor authentication
Scope: All users (or selected apps/groups)

📸 Screenshot: signin-risk-policy.png

## 6️⃣ Validate Policy Enforcement
Purpose (Security reasoning):
Confirm automated controls trigger correctly in response to risk signals.

## Actions:
Simulate risky sign-in (e.g., via VPN from unusual location or Tor)
Observe MFA challenge for medium/high sign-in risk
Trigger user risk (if possible via leaked credentials simulation)
Review updated risk status and sign-in logs

📸 Screenshot: enforcement-test.png
---

## ✅ Expected Results

- Automatic detection of identity risks
- Enforcement of security controls without manual intervention
- Improved security posture for identities
- Reduced risk of account compromise

---

## 🔐 License Requirement
This lab requires Microsoft Entra ID P2 licenses (or EMS E5) to enable:

Identity Protection risk detections
Risk-based Conditional Access policies
Automated remediation controls

Licenses were assigned temporarily for lab purposes and removed during cleanup.

---
## 🧰 Tools & Services Used

Microsoft Entra ID (formerly Azure AD)
Identity Protection
Risk-based Conditional Access policies
Microsoft Entra admin center
Sign-in and audit logs




