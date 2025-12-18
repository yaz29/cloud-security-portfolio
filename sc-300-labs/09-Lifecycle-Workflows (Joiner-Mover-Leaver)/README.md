## 🔐 Lab 09 – Microsoft Entra ID Lifecycle Workflows (Joiner-Mover-Leaver Automation)
## 📌 Overview
This lab demonstrates hands-on configuration of Microsoft Entra ID Governance Lifecycle Workflows to automate the full Joiner-Mover-Leaver (JML) identity lifecycle.
- The implementation automates onboarding,
- role changes, and secure offboarding using HR-driven attributes, built-in tasks, and triggers
- reducing manual effort, enforcing least-privilege access, and ensuring compliance in enterprise environments.
  
This showcases production-level IAM automation expertise, highly valued in roles such as Identity Engineer, Cloud Security Specialist, Governance Administrator, and Zero Trust Architect.

## 🎯 Lab Objectives

- Understand the Joiner-Mover-Leaver (JML) model and Lifecycle Workflows
- Create automated workflows for Joiner (onboarding), Mover (role changes), and Leaver (offboarding)
- Configure triggers, scopes, and tasks (e.g., TAP generation, group management, account disable)
- Test on-demand execution and monitor runs/audits
- Validate automated provisioning and de-provisioning


## ⚠️ Real-World Risk
- Manual JML processes lead to delays,
- orphaned accounts, and over-privileged access
- contributing to 80% of breaches involving compromised identities (per industry reports).
  
Lifecycle Workflows enable proactive, automated governance aligned with:

- Zero Trust (timely access revocation)
- NIST IR 7628 (identity lifecycle management)
- GDPR/SOX compliance (audit-ready trails)
- Microsoft Secure Future Initiative

## This lab mitigates:

- Delayed onboarding → reduced productivity
- Forgotten access on role changes → privilege creep
- Orphaned accounts after departure → insider/exfiltration risk

## 🛠 What I Built

- Joiner workflow: Pre-hire TAP generation + welcome email + group addition
- Mover workflow: Automatic group/license updates on department change
- Leaver workflow: Disable account + remove all access on termination date
- On-demand testing and full monitoring via Insights & Audit logs
- Cleanup: Workflows disabled/deleted after validation


## 🎥 Suggested Demo Animation
Create a short screen recording GIF (e.g., lifecycle-workflow-demo.gif) showing:

Creating a workflow
Running on-demand on a test user
Verifying user changes (e.g., groups added, account disabled)

Example placeholder (replace with your own):
./Screenshots/lifecycle-workflow-demo.gif

## 📐 Architecture & Flow Diagrams (Mermaid)
Diagram 01 – High-Level JML Lifecycle in Microsoft Entra ID
Description: Overview of how HR attributes trigger automated workflows across Joiner, Mover, and Leaver phases.



flowchart LR

    subgraph HR [HR System]
        Attr[Attributes<br/>employeeHireDate<br/>department<br/>employeeLeaveDateTime]
    end
    subgraph EntraID [Microsoft Entra ID Governance]
        direction TB
        LW[Lifecycle Workflows<br/>Engine]
        Trigger[Triggers<br/>Time-based / Attribute Change / On-Demand]
        Scope[Execution Conditions<br/>Scope Rules]
        Tasks[Tasks<br/>TAP, Email, Groups, Disable, etc.]
    end
    subgraph User [User Object]
        New[New Hire]
        Active[Active Employee]
        Former[Former Employee]
    end
    HR -->|Sync Attributes| EntraID
    Trigger --> LW
    Scope --> LW
    LW --> Tasks
    Tasks --> User
    style HR fill:#e3f2fd,stroke:#90caf9
    style EntraID fill:#e8f5e8,stroke:#81c784
    style User fill:#fff3e0,stroke:#ffb74d

## Diagram 02 – Workflow Execution Flow
Description: Step-by-step automation pipeline from trigger to task completion.  

flowchart TD

    Start[Trigger Fired] --> ScopeCheck{Scope Matches?}
    ScopeCheck -->|No| End[Skip]
    ScopeCheck -->|Yes| Tasks[Execute Tasks Sequentially]
    Tasks --> Status[Update Run Status]
    Status --> Audit[Log to Audit]
    Status --> End[Complete]
    style ScopeCheck fill:#fff3cd,stroke:#ffc107
    style Tasks fill:#e3f2fd,stroke:#2196f3
    style End fill:#d4edda,stroke:#28a745



## Diagram 03 – Joiner vs. Leaver Task Comparison
Description: Contrasting actions for onboarding and offboarding.


flowchart LR

    subgraph Joiner [Joiner Phase]
        J1[Generate TAP]
        J2[Send Welcome Email]
        J3[Add to Groups]
        J4[Assign Licenses]
    end
    subgraph Leaver [Leaver Phase]
        L1[Disable Account]
        L2[Remove Groups]
        L3[Remove Licenses]
        L4[Send Notification]
    end
    style Joiner fill:#d4edda,stroke:#28a745
    style Leaver fill:#f8d7da,stroke:#dc3545
----

## 📊 Evidence Summary (Screenshots)

| # | Action | Screenshot |
| - | ------ | ---------- |
| 1 | Lifecycle Workflows Dashboard | <img src="./Screenshots/01-lifecycle-workflows-dashboard.png" width="180" style="border-radius:6px;"/> |
| 2 | Template Selection | <img src="./Screenshots/02-template-selection.png" width="180" style="border-radius:6px;"/> |
| 3 | Joiner Workflow Configuration | <img src="./Screenshots/03-joiner-workflow-configuration.png" width="180" style="border-radius:6px;"/> |
| 4 | Mover Workflow Tasks | <img src="./Screenshots/04-mover-workflow-tasks.png" width="180" style="border-radius:6px;"/> |
| 5 | Leaver Workflow (Disable Account) | <img src="./Screenshots/05-leaver-workflow-disable-account.png" width="180" style="border-radius:6px;"/> |
| 6 | On-Demand Run & Success | <img src="./Screenshots/06-on-demand-run-success.png" width="180" style="border-radius:6px;"/> |
| 7 | Audit Logs Verification | <img src="./Screenshots/07-audit-logs-verification.png" width="180" style="border-radius:6px;"/> |

----

## 🧪 Step-by-Step Implementation
## 1️⃣ Access Lifecycle Workflows Dashboard
Purpose (IAM reasoning): Gain visibility into automated lifecycle processes and execution insights.

**Actions:**

Sign in to Microsoft Entra admin center
Navigate to Identity Governance > Lifecycle workflows > Overview/Insights

**Validation:** Dashboard shows workflow runs, success rates, and trends.

📸 Screenshot: dashboard.png

## 2️⃣ Create Joiner Workflow
Purpose (IAM reasoning): Automate secure, efficient onboarding with pre-hire preparation.

**Actions:**

- Click Create workflow > Browse templates > Select Onboard pre-hire employee or Onboard new hire
- Set trigger: employeeHireDate with offset (-7 or 0 days)
- Add scope: e.g., department equals "Sales"
- Add tasks: Generate Temporary Access Pass, Send welcome email, Add to groups
- Review > Create > Enable schedule

**Validation:** Workflow appears in list; test on-demand.

📸 Screenshots: templates.png, joiner-config.png


## 3️⃣ Create Mover Workflow
Purpose (IAM reasoning): Prevent privilege creep by dynamically adjusting access on role changes.

**Actions:**

- Create workflow > Select attribute change template
- Trigger: Changes to department or jobTitle
- Tasks: Remove from old groups > Add to new groups > Update licenses
- Enable

**Validation:** Attribute update on test user triggers changes.

📸 Screenshot: mover-tasks.png

## 4️⃣ Create Leaver Workflow
Purpose (IAM reasoning): Rapid de-provisioning to minimize ex-employee risk.

**Actions:**

- Create workflow > Select Post-offboarding template
- Trigger: employeeLeaveDateTime +1 day (or on-demand)
- Tasks: Disable account, Remove all groups/licenses
- Create and enable (or test on-demand)

**Validation:** Test user account disabled, access revoked.

📸 Screenshots: leaver-disable.png, run-success.png

## 5️⃣ Test & Monitor Execution
Purpose (IAM reasoning): Verify automation reliability and maintain audit trail.

**Actions:**

- Select workflow > Run on demand > Choose test users
- Monitor Runs tab and Insights
- Check user object changes and Audit logs

**Validation:** Successful runs, user modifications confirmed.

📸 Screenshots: run-success.png, audit-logs.png

## 6️⃣ Run Workflows On-Demand & Verify Execution Success
 **Purpose (IAM reasoning):** Confirm that workflows execute correctly in a controlled manner and produce the expected changes on user objects before enabling scheduled runs.
 
## Actions:

- Select one of your created workflows (Joiner, Mover, or Leaver)
- Click Run on demand
- Choose 1–2 test users (ensure they meet scope conditions)
- Start the run
- Navigate to the Runs tab of the workflow
- Wait for processing (usually a few minutes) and refresh until status shows Succeeded

## Validation:

- Run history shows "Succeeded" with no failed tasks
- Check the test user(s): e.g., groups added (Joiner), groups updated (Mover), or account disabled (Leaver)

📸 Screenshot: run-success.png (capture the "Runs" tab showing successful completion)

## 7️⃣ Verify Changes & Review Audit Logs
**Purpose (IAM reasoning):** Ensure full traceability and compliance by confirming automated actions are logged for governance and auditing purposes.

## Actions:

- Open the affected test user(s) in Users > All users
- Verify changes: e.g., new group memberships, license assignments, or sign-in blocked status
- Go to Identity Governance > Audit logs
- Filter by Activity: "Lifecycle Workflows" or specific task names
- Review entries for workflow execution, task completion, and user attribute modifications

**Validation:** Audit logs show detailed entries for each task executed (e.g., "User added to group", "User account disabled")

📸 Screenshot: audit-logs.png (capture filtered audit log entries related to your workflow runs)

## ✅ Expected Results

- Automated JML processes with zero manual intervention
- Secure onboarding (TAP + groups)
- Timely offboarding (account disabled)
- Complete audit logging for compliance


## 🔐 License Requirement
Microsoft Entra ID Governance or Microsoft Entra Suite (P2-level) required for Lifecycle Workflows.
Licenses assigned temporarily for lab and removed during cleanup.

## 🧰 Tools & Services Used

- Microsoft Entra admin center
- Identity Governance > Lifecycle Workflows
- Built-in tasks and templates
- Audit & Sign-in logs for verification

This lab provides strong, verifiable proof of advanced IAM automation skills with Microsoft Entra ID, 
perfect for demonstrating expertise in identity lifecycle governance.
