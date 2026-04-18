Okta API - Identity Lifecycle Automation
=========================================
Automates the full identity lifecycle (joiner / mover / leaver)
using the Okta REST API.
 
Covers: Okta Administrator skills
Operations: create user, activate user, deactivate user,
            assign/remove group, list users, get user profile,
            reset MFA factors, update user profile
 
Requirements:
    pip install requests
 
Setup (Okta Admin Console):
    1. Security → API → Tokens → Create Token
    2. Copy your Okta domain and API token below
 
Okta Free Developer Tenant: developer.okta.com/signup
"""
 
    import requests
    import json
    from datetime import datetime
 
# CONFIGURATION
OKTA_DOMAIN = "https://your-domain.okta.com"   # e.g. https://dev-12345678.okta.com
API_TOKEN   = "your-okta-api-token"             # Security → API → Tokens
 
 
    def get_headers():
        return {
        "Authorization": f"SSWS {API_TOKEN}",
        "Content-Type": "application/json",
        "Accept": "application/json"
      }
 
 
# USER OPERATIONS 
    def list_users(status=None):
    """
    Lists users in Okta. Filter by status: ACTIVE, DEPROVISIONED,
    SUSPENDED, LOCKED_OUT, PASSWORD_EXPIRED, PROVISIONED, STAGED.
    """
    url = f"{OKTA_DOMAIN}/api/v1/users"
    params = {"filter": f'status eq "{status}"'} if status else {"limit": 50}
    response = requests.get(url, headers=get_headers(), params=params)
    users = response.json()
    print(f"\n[LIST USERS] Found {len(users)} users (status={status or 'ALL'}):")
    for u in users:
        profile = u.get("profile", {})
        print(f"  - {profile.get('displayName')} | {profile.get('email')} | {u.get('status')}")
    return users
 
 
    def create_user(first_name, last_name, email, login, department, title, activate=True):
    """
    Creates a new user in Okta (JOINER process).
    activate=True → sends activation email immediately.
    activate=False → user remains in STAGED state until manually activated.
    """
    payload = {
        "profile": {
            "firstName":   first_name,
            "lastName":    last_name,
            "email":       email,
            "login":       login,
            "department":  department,
            "title":       title,
            "displayName": f"{first_name} {last_name}"
        },
        "credentials": {
            "password": {"value": "TempP@ss2024!"}
        }
    }
    url = f"{OKTA_DOMAIN}/api/v1/users?activate={str(activate).lower()}"
    response = requests.post(url, headers=get_headers(), json=payload)
    if response.status_code == 200:
        user = response.json()
        print(f"[CREATE] User created: {first_name} {last_name} ({login}) | ID: {user['id']}")
        log_action("CREATE_USER", user["id"], f"{first_name} {last_name} | {email}")
        return user
    print(f"[CREATE ERROR] {response.status_code} - {response.text}")
    return None
 
 
    def deactivate_user(user_id, reason="Offboarding"):
    """
    Deactivates a user (LEAVER process).
    Deactivated users cannot log in. All sessions are immediately revoked.
    """
    url = f"{OKTA_DOMAIN}/api/v1/users/{user_id}/lifecycle/deactivate"
    response = requests.post(url, headers=get_headers())
    if response.status_code == 200:
        print(f"[DEACTIVATE] User {user_id} deactivated. Reason: {reason}")
        log_action("DEACTIVATE_USER", user_id, reason)
    else:
        print(f"[DEACTIVATE ERROR] {response.status_code} - {response.text}")
 
 
    def suspend_user(user_id, reason="Temporary suspension"):
    """Suspends a user temporarily — they can be unsuspended later."""
    url = f"{OKTA_DOMAIN}/api/v1/users/{user_id}/lifecycle/suspend"
    response = requests.post(url, headers=get_headers())
    if response.status_code == 200:
        print(f"[SUSPEND] User {user_id} suspended. Reason: {reason}")
        log_action("SUSPEND_USER", user_id, reason)
    else:
        print(f"[SUSPEND ERROR] {response.status_code} - {response.text}")
 
 
    def assign_group(user_id, group_id):
    """
    Assigns a user to an Okta group (MOVER process / role change).
    Groups in Okta control app access and MFA policies.
    """
    url = f"{OKTA_DOMAIN}/api/v1/groups/{group_id}/users/{user_id}"
    response = requests.put(url, headers=get_headers())
    if response.status_code == 204:
        print(f"[ASSIGN GROUP] User {user_id} added to group {group_id}.")
        log_action("ASSIGN_GROUP", user_id, f"group_id={group_id}")
    else:
        print(f"[ASSIGN GROUP ERROR] {response.status_code} - {response.text}")
 
 
    def remove_from_group(user_id, group_id):
    """Removes a user from a group — used during role changes or offboarding."""
    url = f"{OKTA_DOMAIN}/api/v1/groups/{group_id}/users/{user_id}"
    response = requests.delete(url, headers=get_headers())
    if response.status_code == 204:
        print(f"[REMOVE GROUP] User {user_id} removed from group {group_id}.")
        log_action("REMOVE_GROUP", user_id, f"group_id={group_id}")
    else:
        print(f"[REMOVE GROUP ERROR] {response.status_code} - {response.text}")
 
 
    def get_user_groups(user_id):
    """Returns all groups a user belongs to — useful for access review and auditing."""
    url = f"{OKTA_DOMAIN}/api/v1/users/{user_id}/groups"
    response = requests.get(url, headers=get_headers())
    groups = response.json()
    print(f"\n[USER GROUPS] Groups for user {user_id}:")
    for g in groups:
        print(f"  - {g['profile']['name']} (id: {g['id']})")
    return groups
 
 
    def reset_mfa_factors(user_id):
    """
    Resets all MFA factors for a user.
    Used during security incidents, lost devices, or new device onboarding.
    """
    url = f"{OKTA_DOMAIN}/api/v1/users/{user_id}/factors"
    factors = requests.get(url, headers=get_headers()).json()
    for factor in factors:
        factor_id = factor["id"]
        del_url = f"{OKTA_DOMAIN}/api/v1/users/{user_id}/factors/{factor_id}"
        requests.delete(del_url, headers=get_headers())
        print(f"[RESET MFA] Factor {factor.get('factorType')} deleted for user {user_id}.")
    log_action("RESET_MFA", user_id, f"{len(factors)} factors removed")
 
 
    def update_user_profile(user_id, updates: dict):
    """
    Updates user profile attributes (MOVER process / data correction).
    Example: update_user_profile(id, {"department": "Finance", "title": "Manager"})
    """
    url = f"{OKTA_DOMAIN}/api/v1/users/{user_id}"
    payload = {"profile": updates}
    response = requests.post(url, headers=get_headers(), json=payload)
    if response.status_code == 200:
        print(f"[UPDATE PROFILE] User {user_id} updated: {updates}")
        log_action("UPDATE_PROFILE", user_id, str(updates))
    else:
        print(f"[UPDATE ERROR] {response.status_code} - {response.text}")
 
 
    def list_groups():
    """Lists all groups in the Okta org."""
    url = f"{OKTA_DOMAIN}/api/v1/groups"
    response = requests.get(url, headers=get_headers())
    groups = response.json()
    print(f"\n[GROUPS] {len(groups)} groups found:")
    for g in groups:
        print(f"  - {g['profile']['name']} (id: {g['id']})")
    return groups
 
 
# AUDIT LOG 
    def log_action(action, user_id, notes=""):
        with open("okta_audit_log.txt", "a") as f:
        entry = f"{datetime.now().isoformat()} | {action} | user_id={user_id} | {notes}\n"
        f.write(entry)
        print(f"[LOG] {entry.strip()}")
 
 
# MAIN 
    if __name__ == "__main__":
 
    # 1. List all active users
    active_users = list_users(status="ACTIVE")
 
    # 2. Create a new user (joiner)
    new_user = create_user(
        first_name  = "Ana",
        last_name   = "Lopez",
        email       = "ana.lopez@yourdomain.com",
        login       = "ana.lopez@yourdomain.com",
        department  = "IT Security",
        title       = "IAM Analyst",
        activate    = False    # STAGED — activate manually after review
    )
 
    # 3. List all groups
    groups = list_groups()
 
    # 4. Assign new user to a group (replace IDs with real ones from your org)
    # if new_user and groups:
    #     assign_group(new_user["id"], groups[0]["id"])
 
    # 5. Deactivate a user (leaver)
    # deactivate_user("user-id-here", reason="Contract ended 2024-01-31")
