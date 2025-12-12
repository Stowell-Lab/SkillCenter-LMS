# SkillsCenter LMS (Power Platform) - README

A solution‑aware Power Apps + Power Automate implementation for the SkillsCenter LMS. This package contains the installable Solution ZIP file for deployment.

## Contents
* [Overview](#overview)
* [Architecture](#architecture)
* [Prerequisites](#prerequisites)
* [SharePoint Provisioning](#sharepoint-provisioning)
* [Environment Variables](#environment-variables)
* [Connection References](#connection-references)
* [Importing the Solution](#importing-the-solution)
* [Post‑Import Checklist](#post-import-checklist)
* [Smoke Tests](#smoke-tests)
* [Troubleshooting](#troubleshooting)

---

## Overview
The SkillsCenter solution tracks module submissions, grading, student summaries, certificates, milestones, and enrollment sync. All apps and flows are packaged as a Power Platform Solution; SharePoint lists and libraries act as the data layer.

> **Security Note:** This solution is designed to host data on a secure, grader-only SharePoint site. Students interact with the data solely through the Canvas App and should not have direct access to navigate the backend SharePoint site.

## Architecture
* **Canvas Apps:** Student App, Proctor/Grader App
* **Flows:** Grading, archive + certificate generation, student summary recalculation, enrollment sync, milestone logic, etc.
* **SharePoint:** Lists for configuration and data; libraries for files.
* **Environment Variables:** Site URLs, list/library names, document paths.
* **Connection References:** SharePoint, Teams, Forms, OneDrive, Excel.

## Prerequisites
* **Licensing:** Microsoft 365 Licensing (e.g., Office 365 E3) covering Power Apps and Power Automate use rights.
* **Power Platform:** Environment access (Dev/Test/Prod).
* **SharePoint:** Site Owner on the target Grader-Only site (e.g., *Skills Center – Proctors*).
    > **Critical:** This site must be restricted to Proctors/Staff. Students should not be members of this site.

---

## SharePoint Provisioning

### Security Architecture: Two-Site Model
To maintain academic integrity and data security, we use a two-site structure:

1.  **Parent Site (Student Facing):** A general site accessible to all students. This may house the link to the App, static SOPs, or general announcements.
2.  **Grader Site (Backend):** The "target site" for the lists/libraries below. Access must be restricted to Proctors/Graders only.
    * *Note: Students should never navigate directly to this site.*

Create the following lists and libraries on the **Grader Site**. Display names can match below; ensure the URL segment matches what you set in Environment Variables.

### Lists

#### Grades
* **Title** (Single line of text)
* **ModuleID** (Lookup → Modules)
* **FirstSubmissionDate** (Date and Time)
* **Status** (Choice)
* **Attempts** (Number)
* **Grader** (Single line of text)
* **EarnedHours** (Number)
* **ProctorComments** (Multiple lines of text)
* **LatestSubmissionLink** (Hyperlink or Picture)
* **RecentSubmissionDate** (Date and Time)
* **StudentID** (Lookup → StudentSummary: ID; include additional fields: Student Name, Class)
    * *Student ID: Student Name* (Lookup – additional field)
    * *Student ID: Class* (Lookup – additional field)
* **SubmissionID** (Single line of text)
* **PlanningScore** (Number)
* **MethodsScore** (Number)
* **AnalysisScore** (Number)
* **ConclusionScore** (Number)
* **CitationsScore** (Number)
* **FormatScore** (Number)
* **MasteryScore** (Number)
* **StudentRecordID** (Single line of text)
* **CertificateLink** (Hyperlink or Picture)
* **RejectionHistory** (Multiple lines of text)
* **ModuleID: ModuleHours** (Lookup – additional field from Modules)
* **Semester** (Single line of text)
* **Override** (Yes/No)

#### Modules
* **Title** (Single line of text)
* **ModuleName** (Single line of text)
* **ModuleHours** (Number)
* **Active** (Choice)
* **Description** (Multiple lines of text)
* **SOPPDF** (Single line of text)
* **VirtualLab** (Single line of text)
* **Podcast** (Single line of text)
* **Prerequisites** (Multiple lines of text)
* **X** (Number)
* **Y** (Number)
* **LabelPosition** (Choice: Left / Right / Top / Bottom)
* **Difficulty** (Choice: Beginner / Intermediate / Advanced)

#### StudentSummary (one row per student)
* **Title** (Single line of text)
* **TotalModuleHours** (Number)
* **Student Name** (Single line of text)
* **Student Email** (Single line of text)
* **Class** (Single line of text)
* **Semester** (Single line of text)
* **Status** (Single line of text)
* **CreditHours** (Number)
* **ModulePct** (Number)
* **BiweeklyReq** (Number)
* **ParticipationPct** (Number)
* **ExtraCreditHours** (Number)
* **ExtraCreditPct** (Number)
* **FinalPct** (Number)
* **Completed** (Multiple lines of text)
* **ExpectedHours** (Number)
* **InProgress** (Multiple lines of text)
* **SubmissionDates** (Multiple lines of text)
* **ModulesBy** (Single line of text)
* **NextSubmissionDate** (Single line of text)
* **TotalModules** (Number)
* **ConversationID** (Single line of text)
* **TotalSeminars** (Number)
* **SeminarPct** (Number)
* **Multiplier** (Number)
* **LabMeeting1** (Yes/No)
* **LabMeeting2** (Yes/No)
* **LabMeeting3** (Yes/No)
* **LabMeeting4** (Yes/No)
* **RolloverHours** (Number)
* **AlreadyCompleted** (Multiple lines of text)

#### Admin (single row: Title = "Default")
* **Title** (Single line of text)
* **StartDate** (Date and Time)
* **EndDate** (Date and Time)
* **Milestone1Date … Milestone7Date** (Date and Time; blanks allowed)
* **ProctorEmails** (Multiple lines of text)
* **AppVersion** (Number)

#### ModuleLinks
* **Title** (Single line of text)
* **SourceId** (Lookup -> Modules: ID)
* **TargetId** (Lookup -> Modules: ID)

### Libraries

#### Student Records (Document Library)
* **Title** (Single line of text)
* **Description** (Multiple lines of text)
* **StudentID** (Single line of text)
* **ModuleID** (Single line of text)
* **AttemptNumber** (Number)
* **SubmissionDate** (Date and Time)
* **DocumentType** (Choice)
* **GradedDate** (Date and Time)
* **ProctorComments** (Multiple lines of text)
* **Columns Alternative:** StudentID (Person/Text), ModuleID (Lookup or Text), AttemptNumber (Number), SubmissionDate (Date/Time), DocumentType (Choice: Submission / ArchiveSnapshot / Certificate), Semester (Text)
* **Turn Versioning on**

#### Certifications (Document Library)
* Store generated PDFs; add columns if needed (e.g., StudentID, ModuleID, CertificateDate)
* **Turn Versioning on**

---

## Environment Variables

### Lookup Configuration (Grades → StudentSummary)
After creating **StudentSummary** and **Grades** lists:
1.  In **Grades** → List settings → Create column, choose **Lookup**.
2.  Name it `StudentID`.
3.  Get information from: **StudentSummary**.
4.  In this column: **ID**.
5.  Add additional columns: check **Student Name** and **Class** (or your exact column display names).
6.  Save.
    * *Note: If you later rename fields in StudentSummary, re-open this lookup and re-select the additional fields.*

### Variable Table
Set these during solution import (or after, in Solution → Environment Variables):

| EV Name | Example Value | Notes |
| :--- | :--- | :--- |
| **CertificateFile** | `/Certifications/Certificate.docx` | Template path inside the site. Place the certificate Word template here. |
| **CertificatePath** | `/Certifications` | Library (URL segment) |
| **StudentRecordPath** | `/Student Records` | Library (URL segment; match actual URL). Often `/Module Submissions`. |
| **SyncTable** | `/Documents/ActiveStudents.xlsx` | Excel with table `Enrollments`. |
| **Enrollments** | Site connection reference | Pick your Grader/Backend site connection (e.g., Skills Center – Proctors) |
| **SkillsCenter** | Site connection reference | Site-level CR if used. |
| **StudentSummary** | `StudentSummary` | List name bound via CR. |
| **Grades** | `Grades` | List name bound via CR. |
| **Modules** | `Modules` | List name bound via CR. |
| **MMTList** | `MMTList` | If used. |
| **ModuleLinks** | `ModuleLinks` | If used. |
| **TemplateList** | `TemplateList` | If used. |
| **Admin** | `Admin` | Settings list. |
| **Certifications** | `Certifications` | Library name bound via CR. |

> **Important:** Verify the library/list URL in Library/List Settings → URL. Use that in EVs (spaces often appear as `%20`).

---

## Connection References
Map these to connections in the target environment (prefer a service account):

* **SharePoint** (site-level)
* **Microsoft Teams**
* **Microsoft Forms**
* **Excel Online (Business)**
* **OneDrive for Business**

All flows/apps reference these CRs—no personal connections should remain in Prod.

---

## Importing the Solution

### Install from ZIP
**Prerequisites:**
* You are Environment Maker (or Admin) in the target Power Platform environment.
* You are Owner on the target Grader-Only SharePoint site.
* Required lists & libraries are created (see SharePoint Provisioning).

**Steps:**
1.  Locate the solution zip file included in this package. [Download v1.0.0 Release](https://github.com/Stowell-Lab/SkillCenter-LMS/releases/tag/v1.0.0)
2.  Go to **make.powerapps.com** → **Solutions** → **Import**.
3.  Select the solution zip file.
4.  On **Connections**, sign in / select existing until you see green checks.
5.  **Set Environment Variables** (exact values):
    * `CertificateFile` → `/Certifications/Certificate.docx`
    * `CertificatePath` → `/Certifications`
    * `StudentRecordPath` → `/Student Records` (match the URL segment of your library; spaces may be `%20`)
    * `SyncTable` → `/Documents/ActiveStudents.xlsx` (Excel with table `Enrollments`)
    * `Site connection reference` → select your Skills Center Grader SharePoint site
    * `Lists/Libraries connection references` → StudentSummary, Grades, Modules, MMTList, ModuleLinks, TemplateList, Admin, Certifications
6.  Click **Import** and wait for success.

### Post-Import Actions
* **Flows:** Turn **On**; open each → Edit → Save (rebinds EVs/CRs).
* **Apps:** Open, verify data sources, Save and Publish.
* **Permissions:** Ensure the service account has **Contribute** on Student Records and Certifications.

---

## Post‑Import Checklist
* [ ] **Environment Variables:** confirm values are correct.
* [ ] **Connection References:** all green, bound to target connections.
* [ ] **Cloud Flows:** set to **On**. Open each → Edit → Save to rebind to EVs/CRs.
* [ ] **Canvas Apps:** open, confirm data sources resolve, Save and Publish.
* [ ] **Permissions Verification:**
    * **Service Account:** Has Contribute (or higher) on the Grader SharePoint site.
    * **Proctors:** Have Edit access to the Grader SharePoint site.
    * **Students:** **NO ACCESS** to the Grader SharePoint site. They should only access data via the Canvas App.

---

## Smoke Tests
* **Modules:** Create a module with `ModuleHours`.
* **StudentSummary:** Seed a student row (or run the enrollment sync to create it).
* **Grades:** Create a Completed grade linked to the module; ensure recalculation updates StudentSummary.
* **File flow:** Upload a submission DOCX to Student Records; verify archive/certificate flows and metadata updates.
* **Milestones:** Set a future date in Admin; check Student App displays “You should have X modules by DATE.”

---

## Troubleshooting
* **List/Library not in dropdown during import:** Wrong site CR or the list/library wasn’t created yet. Fix the site CR, refresh EVs.
* **404/Path errors in flows:** EV path must match the library URL segment (e.g., `/Student%20Records`).
* **Flows Off after import:** Turn On, open, Edit→Save to rebind.
* **Excel sync fails:** Workbook must have table `Enrollments` with exact headers (`StudentEmail`, `StudentName`, `Class`, `Semester`, `CreditHours`).
* **Permissions issues:** Flows use the connection’s identity; ensure that account has the required SharePoint rights.

---

## Conventions
* One row per student in `StudentSummary`; archive prior terms in a history list if needed.
* Rollover stores hours as a number; do not move past‑term files.
* Milestones are read from `Admin`; blanks are ignored.
