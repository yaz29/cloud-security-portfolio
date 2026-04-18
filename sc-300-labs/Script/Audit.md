
IAM Access Report Generator — Audit & Compliance
=================================================
Generates a comprehensive identity access report from Okta and
Microsoft Entra ID, suitable for:
  - Access certification campaigns (SailPoint / manual review)
  - Security audits and compliance reviews
  - Privileged access monitoring
  - Orphaned/inactive account detection
  - Segregation of Duties (SoD) analysis
 
Output: Excel workbook (.xlsx) with multiple audit sheets
 
Requirements:
    pip install requests msal openpyxl
 
Usage:
    python 04_access_report_generator.py
 
    import requests
    import msal
    import openpyxl
      from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
      from openpyxl.utils import get_column_letter
      from datetime import datetime, timezone
      from pathlib import Path
 
# CONFIGURATION 
OKTA_DOMAIN   = "https://your-domain.okta.com"
OKTA_TOKEN    = "your-okta-api-token"
 
TENANT_ID     = "your-azure-tenant-id"
CLIENT_ID     = "your-azure-client-id"
CLIENT_SECRET = "your-azure-client-secret"
GRAPH_URL     = "https://graph.microsoft.com/v1.0"
 
REPORT_DATE   = datetime.now().strftime("%Y-%m-%d")
OUTPUT_FILE   = f"IAM_Access_Report_{REPORT_DATE}.xlsx"
 
 
# AUTHENTICATION 
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
 
 
# DATA COLLECTION 
    def get_all_okta_users():
      """Fetches all users from Okta with profile and status."""
      print("[OKTA] Fetching users...")
      url = f"{OKTA_DOMAIN}/api/v1/users?limit=200"
      users = []
      while url:
        resp = requests.get(url, headers=okta_headers())
        users.extend(resp.json())
        # Handle pagination
        links = resp.headers.get("Link", "")
        url = None
        for link in links.split(","):
            if 'rel="next"' in link:
                url = link.split(";")[0].strip().strip("<>")
    print(f"[OKTA] {len(users)} users retrieved.")
    return users
 
 
    def get_okta_user_groups(user_id):
      """Gets groups for a single Okta user."""
      url = f"{OKTA_DOMAIN}/api/v1/users/{user_id}/groups"
      resp = requests.get(url, headers=okta_headers())
      if resp.status_code == 200:
        return [g["profile"]["name"] for g in resp.json()]
    return []
 
 
    def get_entra_users(token):
      """Fetches all users from Microsoft Entra ID."""
      print("[ENTRA ID] Fetching users...")
      url = (f"{GRAPH_URL}/users?"
           f"$select=id,displayName,userPrincipalName,accountEnabled,"
           f"department,jobTitle,lastPasswordChangeDateTime,signInActivity")
    users = []
    while url:
        resp = requests.get(url, headers=graph_headers(token))
        data = resp.json()
        users.extend(data.get("value", []))
        url = data.get("@odata.nextLink")
    print(f"[ENTRA ID] {len(users)} users retrieved.")
    return users
 
 
    def get_entra_user_groups(token, user_id):
      """Gets group memberships for a single Entra ID user."""
      url = f"{GRAPH_URL}/users/{user_id}/memberOf?$select=displayName"
      resp = requests.get(url, headers=graph_headers(token))
      if resp.status_code == 200:
        return [g["displayName"] for g in resp.json().get("value", [])]
      return []
 
 
    def get_privileged_users(token):
    """
      Identifies users with privileged roles in Entra ID.
      Looks for Global Admin, Security Admin, Privileged Role Admin.
    """
      print("[ENTRA ID] Fetching privileged role assignments...")
      url = f"{GRAPH_URL}/directoryRoles?$expand=members"
      resp = requests.get(url, headers=graph_headers(token))
      privileged = []
      if resp.status_code == 200:
        for role in resp.json().get("value", []):
            role_name = role.get("displayName", "")
            for member in role.get("members", []):
                privileged.append({
                    "role": role_name,
                    "displayName": member.get("displayName", ""),
                    "upn": member.get("userPrincipalName", ""),
                    "id": member.get("id", "")
                })
    print(f"[ENTRA ID] {len(privileged)} privileged role assignments found.")
    return privileged
 
 
# EXCEL REPORT BUILDER
    HEADER_FILL   = PatternFill("solid", fgColor="2E4057")
    HEADER_FONT   = Font(bold=True, color="FFFFFF", name="Arial", size=10)
    ALERT_FILL    = PatternFill("solid", fgColor="FFE0E0")
    WARNING_FILL  = PatternFill("solid", fgColor="FFF3CD")
    OK_FILL       = PatternFill("solid", fgColor="D4EDDA")
    BORDER        = Border(
      left=Side(style="thin", color="CCCCCC"),
      right=Side(style="thin", color="CCCCCC"),
      top=Side(style="thin", color="CCCCCC"),
      bottom=Side(style="thin", color="CCCCCC")
    )
 
    def style_header_row(ws, row=1):
    for cell in ws[row]:
        cell.fill   = HEADER_FILL
        cell.font   = HEADER_FONT
        cell.alignment = Alignment(horizontal="center", vertical="center", wrap_text=True)
        cell.border = BORDER
 
    def autofit_columns(ws):
    for col in ws.columns:
        max_len = max((len(str(cell.value or "")) for cell in col), default=0)
        ws.column_dimensions[get_column_letter(col[0].column)].width = min(max_len + 4, 45)
 
    def add_sheet_title(ws, title):
    ws.append([title, f"Generated: {datetime.now().strftime('%Y-%m-%d %H:%M')}"])
    ws.merge_cells(start_row=1, start_column=1, end_row=1, end_column=2)
    ws["A1"].font = Font(bold=True, size=13, color="2E4057", name="Arial")
    ws.append([])  # blank row
 
 
#  REPORT SHEETS 
    def build_okta_users_sheet(wb, users):
      ws = wb.create_sheet("Okta - All Users")
      add_sheet_title(ws, "Okta Identity Report — All Users")
      ws.append(["Full Name", "Login / Email", "Status", "Department",
               "Title", "Created", "Last Updated", "Groups"])
      style_header_row(ws, 3)
 
      disabled_count = 0
      for u in users:
        p = u.get("profile", {})
        status = u.get("status", "")
        if status == "DEPROVISIONED":
            disabled_count += 1
        row = ws.max_row + 1
        ws.append([
            p.get("displayName", f"{p.get('firstName','')} {p.get('lastName','')}").strip(),
            p.get("login", ""),
            status,
            p.get("department", ""),
            p.get("title", ""),
            u.get("created", "")[:10] if u.get("created") else "",
            u.get("lastUpdated", "")[:10] if u.get("lastUpdated") else "",
            ""  # groups populated below if needed
        ])
        # Colour-code by status
        fill = ALERT_FILL if status == "DEPROVISIONED" else (
               WARNING_FILL if status in ("SUSPENDED", "LOCKED_OUT") else OK_FILL)
        for cell in ws[row]:
            cell.fill   = fill
            cell.border = BORDER
            cell.font   = Font(name="Arial", size=9)
 
    autofit_columns(ws)
    ws.freeze_panes = "A4"
    print(f"[REPORT] Okta users sheet: {len(users)} users, {disabled_count} deprovisioned.")
 
 
    def build_entra_users_sheet(wb, users, token):
      ws = wb.create_sheet("Entra ID - All Users")
      add_sheet_title(ws, "Microsoft Entra ID — User Access Report")
      ws.append(["Display Name", "UPN", "Account Status", "Department",
               "Job Title", "Last Pwd Change", "Groups (sample)"])
    style_header_row(ws, 3)
 
    disabled_count = 0
    for u in users:
        enabled = u.get("accountEnabled", True)
        if not enabled:
            disabled_count += 1
        status_text = "ENABLED" if enabled else "DISABLED"
        row = ws.max_row + 1
        ws.append([
            u.get("displayName", ""),
            u.get("userPrincipalName", ""),
            status_text,
            u.get("department", ""),
            u.get("jobTitle", ""),
            (u.get("lastPasswordChangeDateTime") or "")[:10],
            ""
        ])
        fill = ALERT_FILL if not enabled else OK_FILL
        for cell in ws[row]:
            cell.fill   = fill
            cell.border = BORDER
            cell.font   = Font(name="Arial", size=9)
 
    autofit_columns(ws)
    ws.freeze_panes = "A4"
    print(f"[REPORT] Entra ID users sheet: {len(users)} users, {disabled_count} disabled.")
 
 
    def build_privileged_access_sheet(wb, privileged_users):
      ws = wb.create_sheet("Privileged Access")
      add_sheet_title(ws, "Privileged Role Assignments — Entra ID")
      ws.append(["Role Name", "Display Name", "User Principal Name", "Review Status"])
      style_header_row(ws, 3)
 
      for p in privileged_users:
        row = ws.max_row + 1
        ws.append([p["role"], p["displayName"], p["upn"], "PENDING REVIEW"])
        for cell in ws[row]:
            cell.fill   = WARNING_FILL
            cell.border = BORDER
            cell.font   = Font(name="Arial", size=9)
 
    autofit_columns(ws)
    ws.freeze_panes = "A4"
    print(f"[REPORT] Privileged access sheet: {len(privileged_users)} assignments.")
 
 
    def build_summary_sheet(wb, okta_users, entra_users, privileged_users):
      ws = wb.create_sheet("Executive Summary", 0)
      add_sheet_title(ws, f"IAM Access Report — Executive Summary — {REPORT_DATE}")
 
      okta_active      = sum(1 for u in okta_users if u.get("status") == "ACTIVE")
      okta_deprovi     = sum(1 for u in okta_users if u.get("status") == "DEPROVISIONED")
      okta_suspended   = sum(1 for u in okta_users if u.get("status") == "SUSPENDED")
      entra_enabled    = sum(1 for u in entra_users if u.get("accountEnabled"))
      entra_disabled   = sum(1 for u in entra_users if not u.get("accountEnabled"))
 
      rows = [
        ["METRIC", "VALUE", "NOTES"],
        ["── OKTA ──", "", ""],
        ["Total Users (Okta)",         len(okta_users),     ""],
        ["Active Users",               okta_active,         ""],
        ["Deprovisioned / Inactive",   okta_deprovi,        "Review recommended" if okta_deprovi > 0 else "OK"],
        ["Suspended",                  okta_suspended,      "Investigate if unexpected"],
        ["", "", ""],
        ["── MICROSOFT ENTRA ID ──", "", ""],
        ["Total Users (Entra ID)",     len(entra_users),    ""],
        ["Enabled Accounts",           entra_enabled,       ""],
        ["Disabled Accounts",          entra_disabled,      "Verify offboarding complete"],
        ["", "", ""],
        ["── PRIVILEGED ACCESS ──", "", ""],
        ["Privileged Role Assignments", len(privileged_users), "Apply principle of least privilege"],
        ["", "", ""],
        ["── REPORT INFO ──", "", ""],
        ["Report Generated",           datetime.now().strftime("%Y-%m-%d %H:%M"), ""],
        ["Generated By",               "IAM Automation Script (Python)", ""],
    ]
 
    for r in rows:
        row_num = ws.max_row + 1
        ws.append(r)
        if r[0].startswith("──"):
            for cell in ws[row_num]:
                cell.font = Font(bold=True, name="Arial", size=10, color="2E4057")
        elif r[0] == "METRIC":
            for cell in ws[row_num]:
                cell.fill = HEADER_FILL
                cell.font = HEADER_FONT
                cell.border = BORDER
        else:
            for cell in ws[row_num]:
                cell.border = BORDER
                cell.font   = Font(name="Arial", size=9)
            # Highlight concerning values
            if isinstance(r[1], int) and r[1] > 0 and "Review" in str(r[2]):
                ws[row_num][1].fill = WARNING_FILL
 
    autofit_columns(ws)
    print(f"[REPORT] Executive summary sheet built.")
 
 
#  MAIN 
    def generate_report():
      print(f"\n{'='*60}")
      print(f"  IAM ACCESS REPORT GENERATOR")
      print(f"  Date: {REPORT_DATE}")
      print(f"{'='*60}\n")
 
    # Collect data
    graph_token     = get_graph_token()
    okta_users      = get_all_okta_users()
    entra_users     = get_entra_users(graph_token)
    privileged_users = get_privileged_users(graph_token)
 
    # Build workbook
    wb = openpyxl.Workbook()
    wb.remove(wb.active)  # Remove default sheet
 
    build_summary_sheet(wb, okta_users, entra_users, privileged_users)
    build_okta_users_sheet(wb, okta_users)
    build_entra_users_sheet(wb, entra_users, graph_token)
    build_privileged_access_sheet(wb, privileged_users)
 
    wb.save(OUTPUT_FILE)
    print(f"\n[DONE] Report saved: {OUTPUT_FILE}")
    print(f"[DONE] Sheets: Executive Summary | Okta Users | Entra ID Users | Privileged Access")
    return OUTPUT_FILE
 
 
    if __name__ == "__main__":
      generate_report()
