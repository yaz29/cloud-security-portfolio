## 05-Access-Review-Automation 

## 📆 Scheduling and Automating Periodic Membership Reviews for Compliance"

## 📝 Description"
  This repository demonstrates the configuration and automation of Access Reviews 
  in Microsoft Entra ID (Azure Active Directory). It serves as a hands-on portfolio 
  project proving SC-300 Identity and Access Administrator capabilities.

## 🎯 Objectives

  - "Enforce least privilege and Zero Trust principles"
  - "Maintain compliant and secure identity lifecycles"
  - "Identify and remove unnecessary or risky access"
  - "Automate scheduled access reviews across the organization"

## ⚙️ Features Implemented

  **group_membership_reviews:**
  ### 👥 Group Membership Reviews
      - "Automated recurring Access Reviews for M365 and Security groups"
      - "Optional guest user inclusion/exclusion"
      - "Sign-in activity–based recommendations"

  **application_assignment_reviews:**
   ### 🧩 Application Assignment Reviews
      - "Periodic certification of user assignments to enterprise apps"
      - "Reviews completed by application owners or administrators"

  **automatic_access_removal:**
   ### 🗑️ Automatic Access Removal
      - "Auto-apply reviewer decisions to remove denied users"
      - "Remove access when reviewers do not respond"
      - "Optional justification requirement"

 ### 📅 Scheduling & Notifications"
      - "Weekly, monthly, quarterly, or annual review cycles"
      - "Automated reviewer email reminders"
      - "Custom review duration"

 ### 🔑 PIM Integration"

      - "Privileged role Access Reviews"
      - "Review of permanent vs. eligible role assignments"
      - "PIM activation history analysis"

## 🧠 SC-300 Skills Demonstrated
 ### 🛡️ Identity Governance"
      - "Access Review lifecycle configuration"
      - "Governance for groups, applications, and privileged roles"
      - "External/guest access management"

 ### 🔧 Automation & Scripting"
      - "Azure Portal configuration"
      - "Microsoft Graph API automation"
      - "PowerShell scripting"

 ### 🏛️ Compliance & Zero Trust"

      - "ISO 27001 / SOC2 / SOX alignment"
      - "Audit-ready access certifications"
      - "Least privilege enforcement"

  ### 📊 Monitoring & Reporting"
 
      - "Exporting Access Review results"
      - "Governance reporting"
      - "Remediation documentation"

## 🧰 Technologies Used
technologies:
  - "Microsoft Entra ID (Azure AD)"
  - "Access Reviews"
  - "Privileged Identity Management (PIM)"
  - "Microsoft Graph API"
  - "PowerShell"
  - "Azure Portal"

## 🚀 Step-by-Step Process
  - ### 🔐 Access Microsoft Entra Admin Center
      - Go to https://entra.microsoft.com.
      - Log in with Identity Governance / Global Admin / PIM Admin.
      - Navigate to Identity Governance → Access Reviews.
      -  screenshot: "screenshots/step1_dashboard.png"

  - ### 📍 Navigate to Access Reviews
      Manage:
      - Groups
      - Applications
      - Guest Users
      - Privileged Roles (via PIM)
      -  screenshot: "screenshots/step1_dashboard.png"

  - ### 🆕 Create a New Access Review
      - Click New Access Review.
      - Select Teams + Groups.
      - Choose review type (All users or Guests only).
      -  screenshot: "screenshots/step2_creation.png"

  - ### 📁 Select the Group to Review
      - Click Select a Group.
      - Search and choose a Security or M365 group.
      - Proceed to configuration.
      - screenshot: "screenshots/step3_group_selection.png"

  - ### ⚙️ Configure Review Settings
      Settings:
      - Reviewers
      - Recurrence settings
      - Review duration
      - Auto-apply decisions
      - Activity-based recommendations
      - screenshot: "screenshots/step4_schedule.png"

  - ### ✉️ Configure Reviewer Notifications
      - Enable reminder emails.
      - Configure notification frequency.
      - Review settings and create the review.
      -  screenshot: "screenshots/step5_reviewers.png"

  - ### 📊 Monitor the Review
      Reviewers choose:
      - Approve
      - Deny
      - Not Reviewed
      - screenshot: "screenshots/step6_in_progress.png"

  - ### 📉 Review Activity Recommendations
      - Identify inactive or risky accounts.
      - Use sign-in data for decision support.
      - screenshot: "screenshots/step7_activity.png"

  - ### 🗂️ Apply the Review Results
      - Auto-apply will enforce changes automatically.
      - Manual apply is available.
      - Export results for compliance.
      - screenshot: "screenshots/step8_results.png"

## 📸 Evidences

  | Step | Description | Screenshot |
  |------|-------------|------------|
  | 1 | Access Reviews Dashboard | ![](screenshots/step1_dashboard.png) |
  | 2 | Create New Review | ![](screenshots/step2_creation.png) |
  | 3 | Select Group | ![](screenshots/step3_group_selection.png) |
  | 4 | Configure Recurrence | ![](screenshots/step4_schedule.png) |
  | 5 | Reviewer Assignment | ![](screenshots/step5_reviewers.png) |
  | 6 | Review In Progress | ![](screenshots/step6_in_progress.png) |
  | 7 | Activity Recommendations | ![](screenshots/step7_activity.png) |
  | 8 | Apply Results | ![](screenshots/step8_results.png) |

## 🎞️ Demo GIF
  description: "End-to-end demonstration of the Access Review workflow."
  gif: "demo/demo.gif"

## 📈 Project Outcomes
  - "20–40% reduction in unnecessary access"
  - "Automatic cleanup of inactive or unverified accounts"
  - "Zero Trust governance improvements"
  - "Audit-ready documentation"

## 🏁 Conclusion
  This project demonstrates hands-on SC-300 knowledge in identity governance,
  access review automation, compliance alignment, and privileged identity management.
