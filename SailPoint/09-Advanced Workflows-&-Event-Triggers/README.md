# 09 · Advanced Workflows & Event Triggers

![SailPoint](https://img.shields.io/badge/SailPoint-IdentityNow-003087?style=flat-square&logo=sailpoint&logoColor=white)
![Category](https://img.shields.io/badge/Category-Automation-6A1B9A?style=flat-square)
![Level](https://img.shields.io/badge/Level-Advanced-F44336?style=flat-square)
![Priority](https://img.shields.io/badge/Priority-High_Priority-FF6F00?style=flat-square)

---

## Why this matters

Basic SailPoint workflows approving a request, sending a notification cover 80% of use cases. The remaining 20% multi-step processes with conditional logic, integrations with external systems, automatic responses to security events require the advanced automation layer.

Event Triggers and advanced Workflows are what transform SailPoint from a passive governance tool into an active orchestration engine. When a user violates an SoD policy, the workflow can notify the manager, create a ticket in ServiceNow, request approval, and revoke the access all automatically. This lab is the most technical in the series and the one that most differentiates a junior profile from a senior one.

---

## Architecture

```mermaid
flowchart TD
    subgraph Event Triggers
        T1[Identity Created]
        T2[Access Request Submitted]
        T3[Policy Violation Detected]
        T4[Certification Completed]
        T5[Account Aggregated]
    end
    subgraph Workflow Engine
        W[Workflow Steps]
        W --> S1[HTTP Step\nCall external API]
        W --> S2[Send Email\nNotification]
        W --> S3[Wait and Approval\nAwait decision]
        W --> S4[Conditional Branch\nif else logic]
        W --> S5[Transform\nManipulate data]
    end
    subgraph Output
        O1[Ticket in ServiceNow]
        O2[Slack message]
        O3[SailPoint action\nrevoke or assign]
        O4[Webhook to SIEM]
    end
    T1 & T2 & T3 & T4 & T5 --> W
    S1 --> O1 & O4
    S2 --> O2
    S3 & S4 & S5 --> O3
```

---

## Prerequisites

- Labs 01-09 completed data and configurations from previous labs available to connect in workflows
- A Make (formerly Integromat) or Zapier account to simulate external webhooks (free tier)
- Basic familiarity with JSON and REST APIs

---

## Lab Walkthrough

### Step 1 · Explore the Workflow Designer

Go to **Admin → Workflow** and open the Workflow Designer. Explore the available step types: Start/End, HTTP Request, Approval, Email, Branch, Transform.

![Step 1 — Workflow Designer with available step types](./screenshots/01-workflow-designer.png)
*The Workflow Designer is visual and block-based similar to Okta Workflows or Power Automate. However, for complex logic, the Transform steps require basic JSON knowledge.*

---

### Step 2 · Create a Policy Violation notification workflow

Create a workflow triggered when an SoD violation is detected (Event Trigger: `POLICY_VIOLATION`) that sends an email notification to the user's manager with the violation details.

![Step 2 — Workflow with Policy Violation trigger and email step](./screenshots/02-policy-violation-workflow.png)
*The `POLICY_VIOLATION` trigger includes in its payload the affected user, the conflicting entitlements, and the violated policy all the information needed for a meaningful notification email.*

---

### Step 3 · Add an approval step to the workflow

After the notification, add an Approval step: the manager must decide whether to revoke the access or request an exception. The workflow waits for the response before continuing.

![Step 3 — Approval step with decision options](./screenshots/03-approval-step.png)
*The approval step makes the workflow interactive the system does not act automatically but waits for a decision from a human with context.*

---

### Step 4 · Add conditional logic with a Branch step

Add a Branch step after the approval: if the manager approves the exception → document and continue; if they decide to revoke → proceed to automatic revocation.

![Step 4 — Conditional branch based on approval decision](./screenshots/04-conditional-branch.png)
*Conditional logic is what makes a workflow genuinely useful different responses to different decisions, without additional manual intervention.*

---

### Step 5 · Configure an HTTP Step to integrate with ServiceNow

Add an HTTP step that calls the ServiceNow API (or your test webhook) to create an incident ticket with the SoD violation details.

![Step 5 — HTTP Step configured with ServiceNow API call](./screenshots/05-http-step-servicenow.png)
*ITSM integration via HTTP Step is the most requested use case in enterprise projects SailPoint detects the event, ServiceNow manages the remediation workflow.*

---

### Step 6 · Use a Transform Step to prepare the payload

Before the HTTP Step, add a Transform Step that formats the trigger data into the JSON the ServiceNow API expects: map fields, concatenate strings, format dates.

![Step 6 — Transform Step preparing the JSON payload for ServiceNow](./screenshots/06-transform-step.png)
*The Transform Step uses VTL (Velocity Template Language) or JSONPath to manipulate data this is the most technical part of workflows and where most time is spent in real projects.*

---

### Step 7 · Create an Event Trigger for Identity Created

Create a second workflow triggered by the `IDENTITY_CREATED` event. When a new identity is created, send a webhook to Slack with the new employee's basic details.

![Step 7 — Identity Created workflow with Slack notification](./screenshots/07-identity-created-trigger.png)
*The `IDENTITY_CREATED` trigger is at the heart of any automated Joiner process any welcome action, notification, or additional onboarding step starts here.*

---

### Step 8 · Monitor workflow executions

Go to **Admin → Workflow → Executions** and review the execution history of your workflows: successes, failures, duration per step, and the payload processed at each step.

![Step 8 — Workflow execution history with per-step detail](./screenshots/08-workflow-executions.png)
*The execution history is essential for debugging when a workflow fails, the execution view shows exactly which step failed and what data it held at that moment.*

---

## What I Learned

- **VTL (Velocity Template Language)** in Transform Steps is the steepest learning curve in the Workflow Designer. It is worth investing time to learn it without it, the most complex workflows are simply not possible.
- **Event Triggers have different payloads** depending on the event type the `POLICY_VIOLATION` payload has different fields than `ACCESS_REQUEST_SUBMITTED`. Always check the documentation for the specific trigger before designing the workflow.
- I learned that **approval step timeouts must be configured explicitly** a workflow waiting indefinitely for an approval blocks the entire process. Always define a timeout and an escalation action.
- The **ServiceNow integration via HTTP** works well for creating tickets, but managing the ticket lifecycle (updates, closure) requires a reverse webhook from ServiceNow back to SailPoint more complex but fully implementable.

---

## Real-World Applications

- Automating SoD violation response: detect → notify manager → wait for decision → execute action → close ticket in ServiceNow, all without any manual IT intervention
- Building an express offboarding process that, on a Terminated status trigger, simultaneously notifies HR, IT, and Security and revokes access in the correct order
- Sending real-time alerts to Slack whenever privileged access to a critical system is granted, allowing the security team to react within minutes

---

## Resources

- [Workflows in SailPoint ISC](https://documentation.sailpoint.com/saas/help/workflows/workflow_overview.html)
- [Event Triggers](https://documentation.sailpoint.com/saas/help/workflows/event_triggers.html)
- [Workflow steps reference](https://documentation.sailpoint.com/saas/help/workflows/workflow_steps.html)
