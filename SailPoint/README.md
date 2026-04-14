# SailPoint IdentityNow — Hands-On Labs

![SailPoint](https://img.shields.io/badge/SailPoint-IdentityNow-003087?style=flat-square&logo=sailpoint&logoColor=white)
![Labs](https://img.shields.io/badge/Labs-11-4CAF50?style=flat-square)
![Product](https://img.shields.io/badge/Product-Identity_Security_Cloud-0D47A1?style=flat-square)
![Domain](https://img.shields.io/badge/Domain-IGA_%2F_Zero_Trust_%2F_PAM-7B3F9E?style=flat-square)

---

## About this repository

Hands-on SailPoint IdentityNow lab series covering the most relevant topics for IAM Engineer, IGA Analyst, and Identity Security Specialist roles. Each lab includes real business context, architecture diagrams, step-by-step walkthroughs with annotated screenshots, and honest reflections on what was learned and what tripped me up.

The goal is not to follow a checklist without understanding it is to build the mental model of why each capability exists and when you would use it in a real project.

---

## Lab Index

| # | Lab | Key Concepts | Level | Priority |
|---|---|---|---|---|
| 01 | [Access Policies & Risk-based Access](./01-Access-Policies-Risk/) | Risk scoring, Zero Trust, access policies | 🟡 Intermediate | 🟠 High |
| 02 | [Privileged Access Management (PAM)](./02-Privileged-Access-Management/) | JIT access, privileged entitlements, expiration | 🔴 Advanced | 🟠 High |
| 03 | [Application Onboarding & Provisioning](./03-Application-Onboarding-Provisioning/) | SCIM, Web Services, write-back, deprovisioning | 🟡 Intermediate | 🟠 High |
| 04 | [Roles & Access Profiles](./04-Roles-Access-Profiles/) | Business Roles, Access Profiles, RBAC, auto-assignment | 🟡 Intermediate | 🟠 High |
| 05 | [Certification Campaigns](./05-Certification-Campaigns/) | Manager, App Owner campaigns, audit evidence | 🟡 Intermediate | 🔴 Must Have |
| 06 | [Segregation of Duties (SoD)](./06-Segregation-of-Duties/) | SoD policies, violations, preventive detection | 🔴 Advanced | 🟠 High |
| 07 | [Authentication Policies & MFA](./07-Authentication-Policies-MFA/) | MFA, step-up auth, secure sessions | 🟡 Intermediate | 🟠 High |
| 09 | [Source Configuration & Connectors](./09-Source-Configuration-Connectors/) | Incremental aggregation, CSV, troubleshooting | 🟡 Intermediate | 🔴 Must Have |
| 10 | [Advanced Workflows & Event Triggers](./10-Advanced-Workflows-Event-Triggers/) | Event triggers, HTTP steps, ServiceNow integration | 🔴 Advanced | 🟠 High |
| 11 | [Identity Profiles & Transforms](./11-Identity-Profiles-Transforms/) | Transforms, concatenation, lookup, normalization | 🔴 Advanced | 🟠 High |
| 12 | [Access Insights & Reporting](./12-Access-Insights-Reporting/) | Advanced search, SOX reports, dashboards | 🟡 Intermediate | 🟢 Recommended |

---

## Recommended order

```
09 Sources → 11 Identity Profiles → 04 Roles → 03 Provisioning
→ 05 Certifications → 06 SoD → 01 Risk Policies
→ 02 PAM → 07 Auth MFA → 10 Workflows → 12 Insights
```

Start with Sources and Identity Profiles everything else depends on having clean, well-structured data.

---

## Target certification

**SailPoint Certified IdentityNow Engineer** these labs cover approximately 90% of the exam content.

---

## Certification stack

| Platform | Certification | Status |
|---|---|---|
| Microsoft Entra ID | SC-300 ✅ | Completed |
| Okta Workforce Identity | Okta Certified Administrator | In progress |
| SailPoint IdentityNow | SailPoint Certified Engineer | These labs |

With all three platforms covered, the profile fits **IAM Engineer**, **Identity Security Analyst**, and **IGA Specialist** roles the most in-demand positions in cybersecurity today.

---

## Environment used

- **SailPoint Identity Security Cloud** 30-day free trial at [sailpoint.com](https://www.sailpoint.com)
- **Active Directory** Windows Server 2019 on a local VM as the primary Source
- **Virtual Appliance** OVA deployed in VMware Workstation
- **Salesforce Developer Edition**  free, used as a secondary Source with write-back
- **Postman** for exploring the SailPoint REST API during provisioning labs

---

## Related repositories

- [`sc-300-labs`](../sc-300-labs/) — Microsoft Entra ID: Conditional Access, PIM, Identity Protection
- [`Okta-Hands-On-Labs-IAM`](../Okta-Hands-On-Labs-IAM/)  Okta: SSO, MFA, SCIM, API Access Management
