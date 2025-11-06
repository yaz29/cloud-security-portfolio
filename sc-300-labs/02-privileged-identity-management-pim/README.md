# 🔐 Lab 02 – Advanced Privileged Identity Management (PIM) & Identity Governance 

**Objective:**  
Implement Just-in-Time (JIT) access for Azure AD admin roles using approval workflows and MFA.

---

## 🧭 Overview
Privileged Identity Management (PIM) enhances security by limiting standing admin privileges.  
In this lab, we configured eligible roles, activation approval, and alerts for privileged access.

---

## Full Flow Demo (MFA Prompt → Access Granted)


----

## Step-by-Step Lab (15 Actions)

| Step | Action | Portal Path | Screenshot |
|------|-------|-------------|------------|
| 1 | **Enable PIM for Azure AD roles** | `Azure AD` → `Privileged Identity Management` → `Azure AD roles` → **Manage** → **Enable PIM** | `PIM_Enable.png` |
| 2 | **Discover and enable PIM for Azure resources** (optional) | `PIM` → `Azure resources` → **Discover resources** → Select subscription → **Manage resource** | `PIM_Azure_Resources.png` |
| 3 | **Assign eligible roles** (Global Admin, User Admin, Billing Admin) | `PIM` → `Azure AD roles` → **+ Add assignments** → **Eligible** → Select user → Assign | `PIM_Eligible_Assignment.png` |
| 4 | **Set maximum activation duration (4 hours)** | `PIM` → `Azure AD roles` → **Role settings** → Edit → **Maximum activation duration: 4 hours** | `PIM_Max_Duration.png` |
| 5 | **Require MFA on activation** | Role settings → **Require multifactor authentication on activation** ✓ | `PIM_Require_MFA.png` |
| 6 | **Require justification on activation** | Role settings → **Require justification on activation** ✓ | `PIM_Require_Justification.png` |
| 7 | **Enable approval workflow** | Role settings → **Require approval** → Select approver (`approver@...`) | `PIM_Approval_Workflow.png` |
| 8 | **Create Access Review (monthly)** | `PIM` → `Azure AD roles` → **Access reviews** → **+ Create review** → Scope: Eligible members → Frequency: Monthly | `Access_Review_Created.png` |
| 9 | **Enable email notifications** | `PIM` → `Settings` → **Email notifications** → Enable for approvers & reviewers | `PIM_Notifications.png` |
| 10 | **Activate role as eligible user** | Portal → **My roles** → Eligible → **Activate** → Enter justification → Complete MFA | `PIM_Activation_Success.png` |
| 11 | **Validate activation in Audit Logs** | `Azure AD` → `Audit logs` → Filter: `Activity: Manage PIM` → View activation event | `Audit_Log_Activation.png` |
| 12 | **Export activation history (CSV)** | `PIM` → `Azure AD roles` → **Role activation history** → **Export** | `PIM_Activation_Export.csv` |
| 13 | **Apply Conditional Access to PIM-activated users** | `Conditional Access` → New policy → Users: Include `All users`, Exclude `admin-lab` → Cloud apps: `Microsoft Azure Management` → Grant: **Require device compliance** | Enforces secure endpoints |
| 14 | **Simulate activation denial (no MFA)** | Attempt activation without MFA → Capture error | Proves enforcement works |
| 15 | **Generate Sign-in Logs report** | `Azure AD` → `Sign-ins` → Filter: `PIM` → Export | Full traceability |

---

## 🖼️  Evidence Summary (Attached)

Screenshots stored in [`/screenshots`](./screenshots/)  
- `PIM_Enable.png` – PIM enabled  
- `PIM_Eligible_Assignment.png` – Role assigned  
- `PIM_Activation_Success.png` – MFA + justification  
- `Access_Review_Created.png` – Monthly review  
- `Audit_Log_Activation.png` – Audit trail  
- `PIM_Activation_Export.csv` – Exported report  
- `CA_Policy_PIM.png` – Conditional Access (bonus)

---

## Conclusion
> **Zero Standing Access (ZSA)** achieved:  
> - No permanent privileged roles  
> - Just-in-time activation with MFA, justification, and approval  
> - Monthly access reviews + full audit export  
> - Conditional Access enforcement on management plane  

---

## Prerequisites
- Microsoft 365 Developer Tenant (free, auto-renew) or Azure Free Trial
- 3 test users: `admin-lab@contoso-lab.onmicrosoft.com`, `approver@...`, `reviewer@...`
- Global Admin access (initial setup only)

---

## 🎯 Outcome
✅ Implemented least-privilege admin model with JIT access.  
✅ Reduced standing Global Administrator assignments to zero.  
✅ Enabled automated alerts and review reminders.

---

📘 *Certification Reference:*  
[Microsoft SC-300: Manage Azure AD roles with Privileged Identity Management](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/privileged-identity-management)

---

👩‍💻 *Author:* [Yazmina G.](https://www.linkedin.com/in/yasmina-g-p-227576a)

