Microsoft Graph API - IAM User Management
==========================================
Automates identity lifecycle operations in Microsoft Entra ID (Azure AD)
using the Microsoft Graph REST API and MSAL authentication.
 
Covers: SC-300 / Microsoft Entra ID skills
Operations: create user, disable user, enable user,
            get user groups, list all users, reset password
 
Requirements:
    pip install msal requests
 
Setup (Azure Portal):
    1. App registrations → New registration
    2. API permissions → Microsoft Graph → Application:
         User.ReadWrite.All, Group.Read.All, Directory.ReadWrite.All
    3. Certificates & secrets → New client secret
    4. Copy Tenant ID, Client ID, Client Secret below


 
import requests
import msal
import json
from datetime import datetime

 
# ── CONFIGURATION ─────────────────────────────────────────────────────────────
TENANT_ID     = "your-tenant-id"          # Azure Portal → Entra ID → Tenant ID
CLIENT_ID     = "your-client-id"          # App Registration → Application (client) ID
CLIENT_SECRET = "your-client-secret"      # App Registration → Certificates & secrets
GRAPH_URL     = "https://graph.microsoft.com/v1.0"
SCOPES        = ["https://graph.microsoft.com/.default"]
 
 
# ── AUTHENTICATION ─────────────────────────────────────────────────────────────
 """ 
    def get_access_token():
    """Authenticates using client credentials flow (app-only)."""
    app = msal.ConfidentialClientApplication(
        CLIENT_ID,
        authority=f"https://login.microsoftonline.com/{TENANT_ID}",
        client_credential=CLIENT_SECRET,
    )
    result = app.acquire_token_silent(SCOPES, account=None)
    if not result:
        result = app.acquire_token_for_client(scopes=SCOPES)
    if "access_token" in result:
        print("[AUTH] Token acquired successfully.")
        return result["access_token"]
    raise Exception(f"[AUTH ERROR] {result.get('error_description')}")
 
 
def get_headers(token):
    return {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}

 """
# ── USER OPERATIONS ────────────────────────────────────────────────────────────
def list_users(token):
    """Returns all users in the tenant with key attributes."""
    url = f"{GRAPH_URL}/users?$select=id,displayName,userPrincipalName,accountEnabled,jobTitle,department"
    response = requests.get(url, headers=get_headers(token))
    users = response.json().get("value", [])
    print(f"\n[LIST USERS] Found {len(users)} users:")
    for u in users:
        status = "ENABLED" if u.get("accountEnabled") else "DISABLED"
        print(f"  - {u['displayName']} | {u['userPrincipalName']} | {status}")
    return users
 
 
def create_user(token, first_name, last_name, department, job_title, domain):
    Provisions a new user in Entra ID.
    Example: create_user(token, "Maria", "Garcia", "IT Security", "IAM Analyst", "company.com")
    
    """
    upn = f"{first_name.lower()}.{last_name.lower()}@{domain}"
    payload = {
        "accountEnabled": True,
        "displayName": f"{first_name} {last_name}",
        "givenName": first_name,
        "surname": last_name,
        "userPrincipalName": upn,
        "mailNickname": f"{first_name.lower()}.{last_name.lower()}",
        "jobTitle": job_title,
        "department": department,
        "passwordProfile": {
            "forceChangePasswordNextSignIn": True,
            "password": "TempP@ssw0rd2024!"
        }
    }
    response = requests.post(f"{GRAPH_URL}/users", headers=get_headers(token), json=payload)
    if response.status_code == 201:
        user = response.json()
        print(f"[CREATE] User created: {user['displayName']} ({upn})")
        return user
    print(f"[CREATE ERROR] {response.status_code} - {response.text}")
    return None
 
 
def disable_user(token, user_id, reason="Account deactivated via IAM automation"):
    """
    Disables a user account (offboarding / leaver process).
    Equivalent to revoking access without deleting the account.
    """
    payload = {"accountEnabled": False}
    response = requests.patch(f"{GRAPH_URL}/users/{user_id}", headers=get_headers(token), json=payload)
    if response.status_code == 204:
        print(f"[DISABLE] User {user_id} disabled. Reason: {reason}")
        log_action("DISABLE", user_id, reason)
    else:
        print(f"[DISABLE ERROR] {response.status_code} - {response.text}")
 
 
def enable_user(token, user_id):
    """Re-enables a previously disabled user account."""
    payload = {"accountEnabled": True}
    response = requests.patch(f"{GRAPH_URL}/users/{user_id}", headers=get_headers(token), json=payload)
    if response.status_code == 204:
        print(f"[ENABLE] User {user_id} re-enabled.")
    else:
        print(f"[ENABLE ERROR] {response.status_code} - {response.text}")
 
 
def get_user_groups(token, user_id):
    """Returns all groups and roles assigned to a user — useful for access review."""
    url = f"{GRAPH_URL}/users/{user_id}/memberOf?$select=displayName,id"
    response = requests.get(url, headers=get_headers(token))
    groups = response.json().get("value", [])
    print(f"\n[USER GROUPS] Groups for user {user_id}:")
    for g in groups:
        print(f"  - {g['displayName']} (id: {g['id']})")
    return groups
 
 
def reset_password(token, user_id, temp_password="TempP@ssw0rd2024!"):
    """Forces a password reset on next sign-in — used during onboarding or security incidents."""
    payload = {
        "passwordProfile": {
            "forceChangePasswordNextSignIn": True,
            "password": temp_password
        }
    }
    response = requests.patch(f"{GRAPH_URL}/users/{user_id}", headers=get_headers(token), json=payload)
    if response.status_code == 204:
        print(f"[RESET PWD] Password reset for user {user_id}.")
    else:
        print(f"[RESET PWD ERROR] {response.status_code} - {response.text}")
 
 '''
# ── AUDIT LOG ──────────────────────────────────────────────────────────────────
def log_action(action, user_id, notes=""):
    """Appends an entry to a local audit log file."""
    with open("graph_audit_log.txt", "a") as f:
        entry = f"{datetime.now().isoformat()} | {action} | user_id={user_id} | {notes}\n"
        f.write(entry)
        print(f"[LOG] {entry.strip()}")
 
 
# ── MAIN ───────────────────────────────────────────────────────────────────────
if __name__ == "__main__":
    token = get_access_token()
 
    # 1. List all users
    users = list_users(token)
 
    # 2. Create a test user (joiner)
    new_user = create_user(token, "Test", "User", "IT Security", "IAM Analyst", "yourdomain.com")
 
    # 3. Get groups for the first user found
    if users:
        get_user_groups(token, users[0]["id"])
 
    # 4. Disable a user (leaver) — replace with a real user ID from your tenant
    # disable_user(token, "user-object-id-here", reason="Employee offboarding")
