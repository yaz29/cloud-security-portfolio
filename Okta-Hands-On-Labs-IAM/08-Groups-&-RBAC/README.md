# 08 · Groups & Role-Based Access Control (RBAC)

![Okta](https://img.shields.io/badge/Okta-Workforce_Identity-007DC1?style=flat-square&logo=okta&logoColor=white)
![Category](https://img.shields.io/badge/Category-Authorization-4A148C?style=flat-square)
![Level](https://img.shields.io/badge/Level-Intermediate-FF9800?style=flat-square)
![Feature](https://img.shields.io/badge/Feature-Group_Rules-FF6B35?style=flat-square)

---

## Why this matters

Access control is the beating heart of any identity program. Who can see what, and who decides? Without a structured approach, you end up with ad-hoc permissions granted by managers via email, service desk tickets, and tribal knowledge. Security audits become nightmares. New starters wait days for access. When someone leaves, access lingers.

RBAC with Okta Groups brings structure: roles are modeled as groups, groups are assigned to apps and resources, and users get the right access simply by being in the right groups. This lab builds a real RBAC model defining roles, creating group rules that auto-assign users based on attributes, and verifying that access follows the model consistently.

---

## Architecture

```mermaid
flowchart TD
    subgraph Users
        U1[Alice\ndept=Engineering]
        U2[Bob\ndept=Sales]
        U3[Carol\ntitle=Manager]
    end
    subgraph Groups
        G1[Engineering]
        G2[Sales]
        G3[Managers]
        G4[All-Staff]
    end
    subgraph Apps & Resources
        A1[GitHub Enterprise\nEngineering only]
        A2[Salesforce\nSales only]
        A3[Workday\nManagers only]
        A4[Slack\nAll Staff]
    end
    U1 -->|Auto-assigned via rule| G1
    U2 -->|Auto-assigned via rule| G2
    U3 -->|Auto-assigned via rule| G3
    U1 & U2 & U3 -->|All users| G4
    G1 --> A1
    G2 --> A2
    G3 --> A3
    G4 --> A4
```

---

## Prerequisites

- Okta org with multiple test users
- Test users with different `department` and `title` profile attributes
- At least two app integrations from previous labs

---

## Lab Walkthrough

### Step 1 · Create the base group structure

Navigate to **Directory → Groups** and create the groups that represent your RBAC roles: `Engineering`, `Sales`, `Managers`, `All-Staff`.

![Step 1 — Group list in Okta directory](./screenshots/01-groups-list.png)
*Keep group names clear and consistent a naming convention like `APP-ROLE` (e.g., `GitHub-ReadOnly`) scales better than generic names as the org grows.*

---

### Step 2 · Create group rules for automatic assignment

Go to **Directory → Groups → Rules** and create a rule: *If user's `department` equals `Engineering`, add to group `Engineering`.*

![Step 2 — Group rule conditions editor](./screenshots/02-group-rule-editor.png)
*Group rules run on a schedule and evaluate when users are created or updated attributes from Okta's Universal Directory drive automatic group membership.*

---

### Step 3 · Verify rule evaluation and membership

After activating the rule, click **Preview** to see which existing users would be added. Then activate and let Okta evaluate.

![Step 3 — Group rule preview showing affected users](./screenshots/03-group-rule-preview.png)
*The preview mode is invaluable always preview before activating a rule that could affect thousands of users.*

---

### Step 4 · Assign apps to groups

Open each app's **Assignments** tab and assign the corresponding group. The Engineering group gets GitHub; Sales gets Salesforce; Managers get Workday; All-Staff gets Slack.

![Step 4 — Assign group to application](./screenshots/04-app-group-assignment.png)
*Group-based app assignment means adding a new employee to the right group is all it takes app provisioning cascades from that single action.*

---

### Step 5 · Test with a newly-created user

Create a test user with `department = Engineering`. Confirm they are automatically added to the Engineering group (via rule) and provisioned in GitHub.

![Step 5 — New user automatically in Engineering group](./screenshots/05-auto-group-membership.png)
*This is the payoff no manual action from IT, no service desk ticket. The user has exactly the right access based on their profile.*

---

### Step 6 · Test access revocation

Change the test user's `department` to `Sales`. Within the rule evaluation window, confirm they are removed from Engineering and added to Sales, with app assignments updating accordingly.

![Step 6 — Group membership change after department update](./screenshots/06-access-revocation.png)
*Access updates trigger automatically on attribute change this is how RBAC prevents stale permissions when people change roles.*

---

### Step 7 · Configure group push to downstream apps

For apps that support group push (e.g., Salesforce via SCIM), configure **Push Groups** to sync Okta groups to the app's internal group/role system.

![Step 7 — Group push configuration to Salesforce](./screenshots/07-group-push.png)
*Group push extends RBAC into the apps themselves a Salesforce user's profile and role in Salesforce can be controlled entirely from Okta.*

---

### Step 8 · Review admin roles and separation of duties

Under **Security → Administrators**, review Okta's built-in admin roles (Super Admin, App Admin, Read-Only Admin). Assign the App Admin role to a test admin scoped to specific apps only.

![Step 8 — Scoped admin role assignment in Okta](./screenshots/08-admin-roles.png)
*Least-privilege applies to admins too an app owner shouldn't need Super Admin rights just to manage their own app's assignments.*

---

## What I Learned

- **Group rules have a processing delay** up to a few minutes in practice. Don't expect instant membership change after activating a rule; build testing with that lag in mind.
- Nesting groups (groups within groups) isn't natively supported in Okta the way it is in Active Directory. Plan your group model flat or use app-level group push instead.
- **Static vs. dynamic groups**: static groups are manually managed; dynamic groups are rule-driven. For scale, you want rule-driven but some edge cases (like contractors) need manual management.
- Removing a user from an app via group removal doesn't instantly deprovision them in every app deprovision timing depends on the app's SCIM or provisioning connector settings.

---

## Real-World Applications

- Automatically provisioning new engineers into GitHub and Jira on their first day, just by setting their department in the HRIS
- Granting time-limited elevated group membership for a contractor project, then having it expire automatically via a Workflow
- Auditing who has access to sensitive apps by exporting group membership reports a clean RBAC model makes audits fast

---

## Resources

- [Okta Groups overview](https://help.okta.com/en-us/content/topics/users-and-groups.htm)
- [Group rules](https://help.okta.com/en-us/content/topics/users-and-groups/group-rules.htm)
- [Role-Based Access Control best practices](https://developer.okta.com/blog/2021/09/08/secure-api-access-control)

