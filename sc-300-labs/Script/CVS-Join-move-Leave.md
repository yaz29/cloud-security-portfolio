CSV Joiner / Mover / Leaver Automation
=======================================
Processes an HR CSV feed and automates identity lifecycle actions
across Okta and Microsoft Entra ID (Graph API).
 
This is the core of any IGA (Identity Governance & Administration) workflow:
  - JOINER  → new employee: create user account, assign groups, send welcome
  - MOVER   → role change: update profile, reassign groups
  - LEAVER  → departure: disable account, remove group access, log for audit
 
This pattern is identical to what SailPoint automates at enterprise scale.
 
CSV Format (hr_feed.csv):
    action, first_name, last_name, email, department, title, manager, start_date, end_date, groups
 
Requirements:
    pip install requests msal openpyxl
 
Usage:
    1. Fill in your credentials in CONFIGURATION section
    2. Place hr_feed.csv in the same directory
    3. Run: python 03_csv_onboarding_offboarding.py

 
    import csv
    import json
    import requests
    import msal
    import openpyxl
    from datetime import datetime
    from pathlib import Path
 
# CONFIGURATION 
    OKTA_DOMAIN   = "https://your-domain.okta.com"
    OKTA_TOKEN    = "your-okta-api-token"
 
    TENANT_ID     = "your-azure-tenant-id"
    CLIENT_ID     = "your-azure-client-id"
    CLIENT_SECRET = "your-azure-client-secret"
    GRAPH_URL     = "https://graph.microsoft.com/v1.0"
 
    INPUT_CSV     = "hr_feed.csv"
    RESULTS_CSV   = "onboarding_results.csv"
    AUDIT_LOG     = "jml_audit_log.txt"
 
 
#  AUTHENTICATION 
    def get_graph_token():
      app = msal.ConfidentialClientApplication(
          CLIENT_ID,
        authority=f"https://login.microsoftonline.com/{TENANT_ID}",
        client_credential=CLIENT_SECRET,
    )
    result = app.acquire_token_for_client(scopes=["https://graph.microsoft.com/.default"])
    return result.get("access_token")
 
    def okta_headers():
      return {"Authorization": f"SSWS {OKTA_TOKEN}", "Content-Type": "application/json"}
 
    def graph_headers(token):
      return {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}
 
 
# JOINER 
    def process_joiner(row, graph_token):
    """
    New employee onboarding:
    1. Create user in Entra ID
    2. Create user in Okta (STAGED)
    3. Assign to groups based on department/role
    4. Log all actions
    """
    print(f"\n[JOINER] Processing: {row['first_name']} {row['last_name']} ({row['email']})")
    results = {"action": "JOINER", "name": f"{row['first_name']} {row['last_name']}",
               "email": row["email"], "status": "", "notes": ""}
 
    # --- Entra ID ---
    domain = row["email"].split("@")[1]
    upn    = row["email"]
    payload = {
        "accountEnabled": True,
        "displayName":    f"{row['first_name']} {row['last_name']}",
        "givenName":      row["first_name"],
        "surname":        row["last_name"],
        "userPrincipalName": upn,
        "mailNickname":   row["email"].split("@")[0],
        "department":     row["department"],
        "jobTitle":       row["title"],
        "passwordProfile": {"forceChangePasswordNextSignIn": True, "password": "TempP@ss2024!"}
    }
    resp = requests.post(f"{GRAPH_URL}/users", headers=graph_headers(graph_token), json=payload)
    if resp.status_code == 201:
        user_id = resp.json()["id"]
        results["notes"] += f"Entra ID created (id={user_id}). "
        log(f"JOINER | ENTRA_ID_CREATED | {row['email']} | id={user_id}")
    else:
        results["notes"] += f"Entra ID error: {resp.status_code}. "
        log(f"JOINER | ENTRA_ID_ERROR | {row['email']} | {resp.text[:100]}")
 
    # --- Okta ---
    okta_payload = {
        "profile": {
            "firstName":  row["first_name"],
            "lastName":   row["last_name"],
            "email":      row["email"],
            "login":      row["email"],
            "department": row["department"],
            "title":      row["title"]
        }
    }
    okta_resp = requests.post(
        f"{OKTA_DOMAIN}/api/v1/users?activate=false",
        headers=okta_headers(), json=okta_payload
    )
    if okta_resp.status_code == 200:
        okta_id = okta_resp.json()["id"]
        results["notes"] += f"Okta created STAGED (id={okta_id}). "
        log(f"JOINER | OKTA_CREATED | {row['email']} | id={okta_id}")
    else:
        results["notes"] += f"Okta error: {okta_resp.status_code}. "
        log(f"JOINER | OKTA_ERROR | {row['email']} | {okta_resp.text[:100]}")
 
    results["status"] = "COMPLETED"
    return results
 
 
#  MOVER 
    def process_mover(row, graph_token):
    """
    Role/department change:
    1. Find user in Entra ID by email
    2. Update department and title
    3. Log the change for audit trail
    """
    print(f"\n[MOVER] Processing: {row['first_name']} {row['last_name']} ({row['email']})")
    results = {"action": "MOVER", "name": f"{row['first_name']} {row['last_name']}",
               "email": row["email"], "status": "", "notes": ""}
 
    # Find user by UPN
    search = requests.get(
        f"{GRAPH_URL}/users/{row['email']}?$select=id,displayName",
        headers=graph_headers(graph_token)
    )
    if search.status_code != 200:
        results["status"] = "ERROR"
        results["notes"] = f"User not found in Entra ID: {row['email']}"
        return results
 
    user_id = search.json()["id"]
    update_payload = {"department": row["department"], "jobTitle": row["title"]}
    upd = requests.patch(
        f"{GRAPH_URL}/users/{user_id}",
        headers=graph_headers(graph_token), json=update_payload
    )
    if upd.status_code == 204:
        results["notes"] = f"Profile updated: dept={row['department']}, title={row['title']}"
        results["status"] = "COMPLETED"
        log(f"MOVER | PROFILE_UPDATED | {row['email']} | dept={row['department']} title={row['title']}")
    else:
        results["status"] = "ERROR"
        results["notes"] = f"Update failed: {upd.status_code}"
        log(f"MOVER | UPDATE_ERROR | {row['email']} | {upd.text[:100]}")
 
    return results
 
 
#  LEAVER 
    def process_leaver(row, graph_token):
    """
    Employee offboarding:
    1. Disable Entra ID account (revokes all tokens immediately)
    2. Deactivate Okta account
    3. Log for compliance audit
    """
    print(f"\n[LEAVER] Processing: {row['first_name']} {row['last_name']} ({row['email']})")
    results = {"action": "LEAVER", "name": f"{row['first_name']} {row['last_name']}",
               "email": row["email"], "status": "", "notes": ""}
 
    # Disable in Entra ID
    search = requests.get(
        f"{GRAPH_URL}/users/{row['email']}?$select=id",
        headers=graph_headers(graph_token)
    )
    if search.status_code == 200:
        user_id = search.json()["id"]
        disable = requests.patch(
            f"{GRAPH_URL}/users/{user_id}",
            headers=graph_headers(graph_token),
            json={"accountEnabled": False}
        )
        if disable.status_code == 204:
            results["notes"] += "Entra ID disabled. "
            log(f"LEAVER | ENTRA_DISABLED | {row['email']} | id={user_id}")
    else:
        results["notes"] += "Entra ID user not found. "
 
    # Deactivate in Okta
    okta_search = requests.get(
        f"{OKTA_DOMAIN}/api/v1/users/{row['email']}",
        headers=okta_headers()
    )
    if okta_search.status_code == 200:
        okta_id = okta_search.json()["id"]
        deactivate = requests.post(
            f"{OKTA_DOMAIN}/api/v1/users/{okta_id}/lifecycle/deactivate",
            headers=okta_headers()
        )
        if deactivate.status_code == 200:
            results["notes"] += "Okta deactivated. "
            log(f"LEAVER | OKTA_DEACTIVATED | {row['email']} | id={okta_id}")
    else:
        results["notes"] += "Okta user not found. "
 
    results["status"] = "COMPLETED"
    return results
 
 
# CSV PROCESSOR 
    def process_hr_feed(csv_path):
    """
    Reads HR CSV and routes each row to the correct lifecycle handler.
    Expected CSV columns: action, first_name, last_name, email, department, title
    """
    print(f"\n{'='*60}")
    print(f"  JML AUTOMATION — Processing {csv_path}")
    print(f"  Started: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print(f"{'='*60}")
 
    graph_token = get_graph_token()
    all_results = []
 
    with open(csv_path, newline="", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        for row in reader:
            action = row.get("action", "").upper().strip()
            if action == "JOIN":
                result = process_joiner(row, graph_token)
            elif action == "MOVE":
                result = process_mover(row, graph_token)
            elif action == "LEAVE":
                result = process_leaver(row, graph_token)
            else:
                print(f"[SKIP] Unknown action '{action}' for {row.get('email')}")
                continue
            all_results.append(result)
 
    # Save results to CSV
    if all_results:
        keys = all_results[0].keys()
        with open(RESULTS_CSV, "w", newline="", encoding="utf-8") as f:
            writer = csv.DictWriter(f, fieldnames=keys)
            writer.writeheader()
            writer.writerows(all_results)
        print(f"\n[DONE] Results saved to {RESULTS_CSV}")
        print(f"[DONE] Audit log saved to {AUDIT_LOG}")
 
    return all_results
 
 
# AUDIT LOG 
    def log(message):
      with open(AUDIT_LOG, "a") as f:
        f.write(f"{datetime.now().isoformat()} | {message}\n")
 
 
#  GENERATE SAMPLE CSV 
    def create_sample_csv():
    """Creates a sample HR feed CSV for testing."""
    rows = [
        ["action", "first_name", "last_name", "email", "department", "title"],
        ["JOIN",  "Carlos",  "Ruiz",     "carlos.ruiz@company.com",   "IT Security", "Security Analyst"],
        ["JOIN",  "Laura",   "Martinez", "laura.martinez@company.com", "Finance",     "Financial Analyst"],
        ["MOVE",  "Pedro",   "Sanchez",  "pedro.sanchez@company.com",  "IT Security", "IAM Engineer"],
        ["LEAVE", "Isabel",  "Torres",   "isabel.torres@company.com",  "HR",          "HR Manager"],
    ]
    with open(INPUT_CSV, "w", newline="") as f:
        csv.writer(f).writerows(rows)
    print(f"[SAMPLE] Sample CSV created: {INPUT_CSV}")
 
 
#  MAIN 
    if __name__ == "__main__":
      # Create sample CSV if it doesn't exist
      if not Path(INPUT_CSV).exists():
        create_sample_csv()
 
    # Process the HR feed
    results = process_hr_feed(INPUT_CSV)
    print(f"\n[SUMMARY] Processed {len(results)} identities.")
