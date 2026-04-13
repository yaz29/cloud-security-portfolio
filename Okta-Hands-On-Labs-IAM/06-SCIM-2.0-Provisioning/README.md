# 06 · SCIM 2.0 Provisioning

![Okta](https://img.shields.io/badge/Okta-Workforce_Identity-007DC1?style=flat-square&logo=okta&logoColor=white)
![Category](https://img.shields.io/badge/Category-Provisioning-0D47A1?style=flat-square)
![Level](https://img.shields.io/badge/Level-Advanced-F44336?style=flat-square)
![Protocol](https://img.shields.io/badge/Protocol-SCIM_2.0-FF6B35?style=flat-square)

---

## Why this matters

Single Sign-On solves authentication. But what about the full lifecycle of a user account creation, updates, and deactivation across every app the company uses? Without automated provisioning, IT manually creates accounts in Salesforce, GitHub, Jira, and a dozen other tools every time someone joins. And when someone leaves, those accounts often aren't removed for weeks.

SCIM (System for Cross-domain Identity Management) is the protocol that solves this. Okta acts as the provisioning engine it pushes creates, updates, and deactivations to any app that speaks SCIM. This lab implements SCIM provisioning end-to-end, including a custom SCIM 2.0 endpoint, so you understand what's happening under the hood when Okta provisions to real apps.

---

## Architecture

```mermaid
flowchart LR
    subgraph Source
        A[Okta Directory\nSource of Truth]
    end
    subgraph SCIM Provisioning
        B[Okta Provisioning Engine]
        B --> C{SCIM Operations}
        C --> D[POST /Users\nCreate]
        C --> E[PUT /Users/:id\nUpdate]
        C --> F[PATCH /Users/:id\nDeactivate]
    end
    subgraph Target Apps
        G[Salesforce]
        H[GitHub Enterprise]
        I[Custom App\nSCIM endpoint]
    end
    A --> B
    D & E & F --> G & H & I
```

---

## SCIM Operations Covered

| Operation | HTTP Method | Okta trigger | What happens |
|---|---|---|---|
| Create user | `POST /Users` | User assigned to app | Account created in target app |
| Update user | `PUT /Users/{id}` | Profile attribute changes | Attributes synced to app |
| Deactivate user | `PATCH /Users/{id}` | User removed from app / deactivated | Account disabled in app |
| Import users | `GET /Users` | Manual import | Existing accounts linked to Okta |

---

## Prerequisites

- Okta org with provisioning features enabled
- A SCIM test server (this lab uses [SCIMulate](https://scimulate.com) or a local mock)
- Postman or similar REST client for debugging
- Familiarity with REST APIs and JSON

---

## Lab Walkthrough

### Step 1 · Set up a SCIM 2.0 test endpoint

For this lab, deploy a simple SCIM server that logs requests and returns valid responses. You can use an existing SCIM mock or the lightweight Node.js SCIM server in this repo's `/scim-server` folder.

![Step 1 — SCIM server running locally with ngrok](./screenshots/01-scim-server-running.png)
*Running your own SCIM server is the fastest way to understand the protocol you see exactly what Okta sends and what your app needs to return.*

---

### Step 2 · Create a custom app integration in Okta

In **Applications → Create App Integration**, choose **SWA** or use the Okta Integration Network search. For a custom SCIM app, go to **Applications → Browse App Catalog → Create New App → SCIM 2.0**.

![Step 2 — Create SCIM app integration in Okta](./screenshots/02-create-scim-app.png)
*Real apps in the Okta Integration Network (Salesforce, GitHub, etc.) have pre-built SCIM connectors but understanding the raw protocol makes debugging much easier.*

---

### Step 3 · Configure the SCIM provisioning connection

Under the app's **Provisioning** tab, enable provisioning and enter your SCIM endpoint URL and authentication token.

![Step 3 — SCIM base URL and auth token configuration](./screenshots/03-scim-connection-config.png)
*Okta supports both Bearer token and OAuth 2.0 authentication for SCIM connections Bearer token is simpler for testing, OAuth 2.0 for production.*

---

### Step 4 · Test the SCIM connection

Click **Test API Credentials** Okta will send a `GET /Users?count=1` request to verify the connection is working.

![Step 4 — SCIM connection test result](./screenshots/04-test-connection.png)
*If this fails, check the ngrok/tunnel URL is active and that your server returns the correct `application/scim+json` content type.*

---

### Step 5 · Enable provisioning features

Under **Provisioning → To App**, enable **Create Users**, **Update User Attributes**, and **Deactivate Users**.

![Step 5 — Provisioning features enabled to app](./screenshots/05-provisioning-features.png)
*Each feature maps to a SCIM operation, you can enable them selectively based on what your target app supports.*

---

### Step 6 · Assign a user and watch the SCIM POST

Assign a test user to the app. Watch your SCIM server's logs Okta will immediately send a `POST /Users` request with the user's profile.

![Step 6 — SCIM POST request received by server](./screenshots/06-scim-post-create.png)
*The SCIM payload includes the user's Okta profile attributes mapped to SCIM standard attributes like `userName`, `name.givenName`, and `emails`.*

---

### Step 7 · Update a user attribute and observe the PATCH

Change the test user's `title` in Okta. Your SCIM server should receive a `PATCH /Users/{id}` request with just the changed attribute.

![Step 7 — SCIM PATCH request for attribute update](./screenshots/07-scim-patch-update.png)
*SCIM uses PATCH for partial updates only changed attributes are sent, not the full profile. This is more efficient than PUT for large user objects.*

---

### Step 8 · Deactivate the user and verify SCIM PATCH

Remove the user from the app or deactivate them in Okta. Confirm a `PATCH /Users/{id}` is sent with `"active": false`.

![Step 8 — SCIM deactivation request payload](./screenshots/08-scim-deactivate.png)
*SCIM deactivation doesn't delete the account in the target app, it sets active=false, which most apps interpret as a disabled state.*

---

## What I Learned

- **SCIM IDs matter.** When Okta creates a user via SCIM `POST`, your server returns an `id` in the response. Okta stores this and uses it for all subsequent `PUT`/`PATCH` operations. If you return a bad ID, updates will fail.
- The `externalId` field is Okta's way of mapping its own user ID to the target app's ID don't ignore it.
- **Attribute mappings are wrong by default** for custom apps. Spend time in the attribute mapping UI to make sure Okta sends what your app actually expects.
- Not all apps implement SCIM correctly. In the real world, you'll hit apps that don't return proper SCIM responses patience and Postman are your tools.

---

## Real-World Applications

- Automating GitHub Enterprise seat management users appear in GitHub the moment they join the Engineering group in Okta
- Ensuring deprovisioned employees can't access Salesforce within seconds of their last day, without any IT manual action
- Synchronizing user profile updates (name changes, department changes) across all SaaS apps automatically

---

## Resources

- [SCIM 2.0 specification (RFC 7644)](https://datatracker.ietf.org/doc/html/rfc7644)
- [Okta SCIM provisioning overview](https://developer.okta.com/docs/concepts/scim/)
- [Building a SCIM server for Okta](https://developer.okta.com/docs/guides/scim-provisioning-integration-overview/)

