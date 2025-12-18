# 🔐 Lab 12 – Admin Consent & Workflow Approval

## 📌 Overview
This lab showcases advanced **Identity Governance** expertise by implementing **Admin Consent Controls** and **Admin Consent Workflow** in Microsoft Entra ID. These features prevent end-users from independently granting high-risk permissions to applications, instead routing sensitive consent requests through a formal approval process.

The configuration enforces strict governance over OAuth consent, significantly reducing risks from malicious apps, over-privileged permissions, and illicit consent grants — a critical Zero Trust control for enterprise application security.

Highly valuable for **IAM Engineer, Identity Governance Specialist, Cloud Security Architect, and Compliance Officer** roles.

---
## 🎯 Lab Objectives
- Block or restrict user consent for applications
- Enable and configure Admin Consent Workflow
- Create targeted approval policies for high-risk permissions
- Simulate consent requests as an end user/developer
- Review, approve, and deny requests as an administrator
- Validate full audit trail of consent decisions

---
## ⚠️ Real-World Risk
Illicit consent grants are a top attack vector: attackers trick users into approving malicious apps that request broad permissions (e.g., `Mail.ReadWrite`, `Files.Read.All`), leading to data theft and account compromise.

This lab mitigates:
- Phishing-driven OAuth consent attacks
- Over-privileged third-party applications
- Lack of visibility into consent grants

Aligned with:
- Microsoft Zero Trust guidance
- NIST 800-63B (Authenticator Assurance Levels)
- Regulatory requirements (GDPR, SOX, HIPAA)

## 🛠 What I Built
- Completely blocked user consent
- Enabled Admin Consent Workflow
- Created a workflow requiring approval for all application permissions containing ".All"
- Tested the end-to-end request flow using Microsoft Graph Explorer
- Approved a request as admin
- Verified detailed audit logging
- Cleanup: Disabled workflow and restored safe consent settings

---
## 🎥 Suggested Demo Animation
Create a GIF (e.g., `admin-consent-workflow.gif`) demonstrating:
- User blocked from consenting → Submits request → Admin reviews & approves → Access granted

<img src="./Screenshots/admin-consent-workflow.gif" width="800"/>

---
## 📐 Architecture & Flow Diagrams (Mermaid)

### Diagram 01 – Consent Governance Models
**Description:** Comparison of consent approaches and their security posture.

flowchart TD


    subgraph UserConsent["User Consent (High Risk)"]
        UC1[User accepts prompt]
        UC2[App immediately receives tokens]
        UC3[No admin oversight]
    end
    subgraph AdminOnly["Admin Consent Only"]
        AC1[User consent blocked]
        AC2[Global admin grants directly]
        AC3[Limited scalability]
    end
    subgraph Workflow["Admin Consent Workflow (Best Practice)"]
        WF1[User blocked for risky perms]
        WF2[Request + justification submitted]
        WF3[Designated approvers review]
        WF4[Consent granted only after approval]
    end
    style UserConsent fill:#f8d7da,stroke:#dc3545
    style AdminOnly fill:#fff3cd,stroke:#ffc107
    style Workflow fill:#d4edda,stroke:#28a745

---

## Diagram 02 – Admin Consent Workflow Process Flowlow
 **Description:**  End-to-end lifecycle of a consent request

sequenceDiagram

    participant User as End User/Developer
    participant App as Third-Party App
    participant Entra as Microsoft Entra ID
    participant Approver as Consent Approver
    User->>App: Interact with app
    App->>Entra: Request permissions (e.g., User.Read.All)
    Entra-->>User: Consent blocked – "Request admin approval"
    User->>Entra: Submit request with business justification
    Entra->>Approver: Email notification + Portal alert
    Approver->>Entra: Review app risk, permissions, justification
    Approver->>Entra: Approve or Deny
    alt Approved
        Entra-->>App: Consent granted
        Entra-->>User: Access now works
    else Denied
        Entra-->>User: Access denied
    end


## Example Admin Consent Workflow Policy (Key Configuration)
**Description:** Sample policy targeting high-risk application permissions (proof of precise control).
```
{  "displayName": "Require Approval for All Application Permissions with '.All'",
  "isEnabled": true,
  "permissionType": "application",
  "permissionClassification": "all",
  "requireJustification": true,
  "notifyReviewers": true,
  "remindersEnabled": true,
  "requestDurationInDays": 30,
  "reviewers": [
    {
      "query": "/groups/{id-of-app-consent-approvers-group}",
      "queryType": "MicrosoftGraph"
    }
  ]
}


```

## 📊 Evidence Summary (Screenshots)

| # | Action | Screenshot |
| - | ------ | ---------- |
| 1 | User Consent Settings (Blocked) | <img src="./Screenshots/01-user-consent-settings-blocked.png" width="180" style="border-radius:6px;"/> |
| 2 | Enable Admin Consent Workflow | <img src="./Screenshots/02-enable-admin-consent-workflow.png" width="180" style="border-radius:6px;"/> |
| 3 | Create New Workflow Policy | <img src="./Screenshots/03-create-workflow-policy.png" width="180" style="border-radius:6px;"/> |
| 4 | Policy Configuration & Reviewers | <img src="./Screenshots/04-policy-configuration-reviewers.png" width="180" style="border-radius:6px;"/> 
| 5 | User Blocked – Consent Prompt | <img src="./Screenshots/05-user-blocked-consent-prompt.png" width="180" style="border-radius:6px;"/> |
| 6 | Submit Admin Consent Request | <img src="./Screenshots/06-submit-admin-consent-request.png" width="180" style="border-radius:6px;"/> |
| 7 | Admin Review & Approval | <img src="./Screenshots/07-admin-review-approval.png" width="180" style="border-radius:6px;"/> |
| 8 | Audit Logs – Consent Events | <img src="./Screenshots/08-audit-logs-consent-events.png" width="180" style="border-radius:6px;"/> |





 ## 🧪 Step-by-Step Implementation
## 1️⃣ Block User Consent
Purpose: Eliminate the highest-risk consent vector.
## Actions:

- Entra admin center → Identity → Applications → Enterprise applications
- Consent and permissions → User consent settings
- Set Users can consent to apps accessing company data → Do not allow user consent

**Validation:** Standard users can no longer accept consent prompts.

📸 Screenshot: 01-user-consent-blocked.png

## 2️⃣ Enable Admin Consent Workflow
Purpose: Activate the governance engine for controlled approvals.

## Actions:

- In Consent and permissions → Admin consent workflow
- Toggle Enable admin consent workflow → Yes

**Validation:** New "Admin consent requests" blade appears.
📸 Screenshot: 02-enable-workflow.png

## 3️⃣ Create Workflow Policy for High-Risk Permissions
Purpose: Target only dangerous permissions for approval.
## Actions:

- Admin consent workflow → New workflow
- Name: "Require Approval for .All Application Permissions"
- Permission type: Application permissions
- Classification: All
- Require justification: Yes
- Add reviewers: Select a security group (e.g., "IAM Approvers")
- Enable notifications and reminders

**Validation:** Policy listed as Enabled.

📸 Screenshots: 03-new-workflow.png, 04-policy-details.png

## 4️⃣ Simulate Consent Request (End-User View)
Purpose: Test the blocked experience and request submission.
## Actions:

- As a non-admin user, open Microsoft Graph Explorer
- Attempt a query requiring app permission (e.g., GET /users → needs User.Read.All)
- Click "Consent" → Blocked → Click Request admin approval
- Fill in justification and submit

**Validation:** Request appears in admin portal.

📸 Screenshots: 05-consent-blocked.png, 06-request-submitted.png

## 5️⃣ Review and Approve Request (Admin View)
Purpose: Validate governance review process.
## Actions:

- Go to Admin consent requests → Select pending request
- Review: App publisher, requested permissions, business justification, risk indicators
- Approve (or Deny) with optional comment

**Validation:** Status changes to Approved; user can now run the query.

📸 Screenshot: 07-admin-approval.png

## 6️⃣ Verify Audit Trail
Purpose: Ensure compliance and traceability.
## Actions:

- Identity → Audit logs
- Filter by:
- Category: ApplicationManagement
- Activity: "Consent to application", "Admin consent request updated"


**Validation:** Full log of request creation, review, and decision.

📸 Screenshot: 08-audit-logs.png

## ✅ Expected Results

- Zero unauthorized user consent grants
- All high-risk permission requests routed through approvers
- Complete audit history of consent decisions
- Secure, governed application integration process


## 🔐 License Requirement
Microsoft Entra ID P2 or Microsoft Entra ID Governance license required for Admin Consent Workflow.

## 🧰 Tools & Services Used

- Microsoft Entra admin center
- Enterprise applications → Consent and permissions
- Admin consent requests
- Microsoft Graph Explorer
- Audit logs

This lab provides strong, verifiable evidence of expertise in Entra ID consent governance and the key defense against one of the most common cloud attack techniques.
