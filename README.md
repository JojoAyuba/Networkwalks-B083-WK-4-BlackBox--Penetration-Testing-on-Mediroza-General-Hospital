# Networkwalks-B083-WK-4-BlackBox--Penetration-Testing-on-Mediroza-General-Hospital

# 🛡️ Mediroza General Hospital — Black-Box Penetration Test

<p align="center">
  <img src="https://img.shields.io/badge/Assessment-Black--Box%20Pentest-blue?style=for-the-badge">
  <img src="https://img.shields.io/badge/Scope-Authorized%20Educational%20Lab-orange?style=for-the-badge">
  <img src="https://img.shields.io/badge/
</p>

<p align="left">
Your text goes here.
</p>

---

## 📌 Overview

This repository documents an **authorized black-box penetration testing assessment** of the Mediroza General Hospital web application.

### Target

```text
https://medirozahospital.com

The assessment was conducted as part of an authorized educational penetration-testing exercise and focused on externally accessible web application functionality, reconnaissance, authentication, authorization, sensitive-file exposure, document security, and information disclosure.

⚠️ Disclaimer: This repository is intended for authorized security testing and educational purposes only. Do not apply these techniques to systems without explicit permission from the owner.

🎯 Assessment Objectives

The assessment focused on:

🔎 External reconnaissance and attack-surface discovery
🌐 Web application and directory enumeration
🔐 Authentication and authorization testing
💉 Identification of input-validation weaknesses
📄 Patient report access-control testing
🔑 PDF password-strength analysis
🧾 Sensitive PDF metadata analysis
💾 Discovery of exposed database backups
📊 Analysis of sensitive information disclosure
📝 Documentation of findings and remediation recommendations
🧪 Methodology

The assessment followed a black-box penetration-testing workflow.

┌──────────────────────────┐
│ 1. Reconnaissance        │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│ 2. Enumeration           │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│ 3. Authentication Testing│
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│ 4. Access-Control Testing│
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│ 5. Document Analysis     │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│ 6. Data Exposure Review  │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│ 7. Reporting & Remediation│
└──────────────────────────┘

🔎 Reconnaissance

Initial reconnaissance identified the following application paths:

/patient/
/staff/
/old/
/patient/login.php
/patient/portal.php
/patient/download.php
/patient/reports/
Network Services Observed
21    FTP
25    SMTP proxy
26    SMTP / Exim
53    DNS / BIND
80    HTTP
110   POP3
143   IMAP
443   HTTPS
465   SMTPS
587   SMTP
993   IMAPS
995   POP3S

Note: The presence of these services alone does not establish that each service is vulnerable. They represent externally observable attack surface.

🤖 robots.txt

The following directories were disclosed through robots.txt:

/patient/
/staff/
/old/

This information was useful during reconnaissance because it revealed application areas that were not necessarily linked from the public website.

🚨 Findings Summary
ID	Finding	Severity
🔴 F-02	Confidential Internal SQL Database Backup Exposure	Critical
🔴 F-04	SQL Injection Login Bypass	Critical
🟠 F-05	Confidential PDFs Accessible After Login Bypass	High
🟠 F-06	Weak PDF Passwords	High
🟡 F-01	Unauthenticated Directory Listing	Medium
🟡 F-03	Username Enumeration	Medium
🟡 F-07	Sensitive PDF Metadata	Medium

---

🔴 F-01 — Unauthenticated Directory Listing
Severity

Medium

Affected Resource
https://medirozahospital.com/patient/
Description

The /patient/ directory was accessible without authentication and returned a directory listing exposing application resources.

The listing included:

reports/
download.php
error_log
login.php
logout.php
portal.php
Impact

The directory listing exposes the internal structure of the patient application and identifies potentially sensitive functionality to an unauthenticated user.

The listing itself does not prove unauthorized access to patient records, but it provides useful reconnaissance information and exposes the locations of potentially sensitive resources.

Recommendation
Disable directory indexing.
Restrict access to sensitive application directories.
Protect error_log.
Review authorization controls in download.php.
Prevent predictable direct access to report files.
Review authentication and session controls in portal.php.

---

🔴 F-02 — Confidential Internal SQL Database Backup Exposure
Severity

Critical

Affected Artifact
mediroza_db_backup_2019.sql
Database
mediroza_hr
Application
Mediroza CMS 1.4.2
Description

An internal SQL database backup was exposed to an unauthorized party.

The backup contained populated:

staff
shareholders

tables.

👨‍💼 Staff Information

The staff table contained:

Field	Description
full_name	Employee name
job_title	Job position
department	Department
email	Employee email
phone	Telephone number
national_id	National identification information
monthly_salary_zar	Monthly salary
date_joined	Employment date

Records identified: 30 employee records.

🏢 Shareholder Information

The shareholders table contained:

Field	Description
shareholder_name	Shareholder identity
share_percent	Ownership percentage
shares_held	Number of shares
share_class	Share classification

Records identified: 10 shareholder records.

Impact

This represents a significant confidentiality breach because the exposed backup contains employee personal and financial information as well as confidential corporate ownership information.

Potential consequences include:

Privacy violations
Identity-related fraud
Targeted social engineering
Exposure of employee salaries
Exposure of corporate ownership information
Further compromise if credentials or other sensitive information are present
Recommendation
Remove exposed database backups from public storage.
Store backups outside the web root.
Apply restrictive filesystem permissions.
Prevent serving of .sql, .bak, .dump, and similar files.
Review access logs.
Search for additional exposed backups.
Rotate potentially exposed credentials.

---

Implement automated sensitive-file scanning.
🟡 F-03 — Username Enumeration
Severity

Medium

Affected Resource
/patient/login.php
Description

The supplied assessment evidence identified username enumeration on the patient login page.

Differences in application responses can allow an unauthenticated user to determine whether particular usernames exist.

Impact

This can make targeted authentication attacks more efficient by allowing an attacker to identify valid accounts.

Recommendation
Use generic authentication error messages.
Maintain consistent responses for valid and invalid usernames.
Implement rate limiting.
Monitor repeated authentication attempts.
Consider multi-factor authentication for sensitive accounts.

---

🔴 F-04 — SQL Injection Login Bypass
Severity

Critical

Affected Resource
/patient/login.php
Description

The supplied assessment evidence identified SQL injection resulting in authentication bypass.

If reproduced as documented, an unauthenticated attacker can circumvent the intended login control and reach functionality intended for authenticated users.

Impact

Authentication bypass may provide unauthorized access to patient-portal functionality and potentially sensitive information.

Recommendation
Use parameterized queries/prepared statements.
Avoid dynamic SQL string concatenation.
Validate input server-side.
Apply least-privilege database permissions.
Implement security logging.
Perform dedicated SQL injection testing during remediation.

---

🟠 F-05 — Confidential PDFs Accessible After Login Bypass
Severity

High

Affected Resource
/patient/reports/
Description

The supplied assessment evidence indicates that confidential patient PDF reports became accessible after the login-bypass condition.

Impact

Patient reports may contain sensitive medical and personal information, creating a significant confidentiality risk.

Recommendation

Authorization should be enforced on every individual report request, rather than relying solely on the login mechanism.

Recommended controls:

Server-side authorization checks
Per-user report ownership validation
Non-predictable report identifiers
Protection of report storage directories
Access logging and monitoring

---

🟠 F-06 — Weak PDF Passwords
Severity

High

Affected Files
patient_report_*.pdf
Description

The supplied assessment evidence identified patient PDF passwords as sufficiently weak or predictable to be recovered using a wordlist.

Impact

If an attacker obtains a protected PDF, weak password protection may provide little practical resistance against unauthorized disclosure.

Recommendation
Use long, random and unique document passwords.
Avoid predictable patient or organizational password patterns.
Use stronger document encryption where appropriate.
Keep document passwords/keys separate from protected files.

---

🟡 F-07 — Sensitive PDF Metadata
Severity

Medium

Affected File
patient_report_3.pdf
Description

Sensitive metadata was identified within the patient PDF.

Depending on the document-generation process, metadata may reveal:

Author
Creator/Application
Creation Date
Modification Date
Internal Paths
Usernames
Document-generation details
Impact

Metadata can provide additional reconnaissance information or unintentionally disclose internal information.

Recommendation
Sanitize PDF metadata before external delivery.
Remove unnecessary author information.
Remove internal filesystem paths.
Review document-generation workflows.
Establish a secure document-publication process.
🔗 Attack Chain

The findings indicate a potential relationship between several weaknesses in the patient portal:

          ┌────────────────────────────┐
          │ Directory Listing          │
          │ /patient/                  │
          └─────────────┬──────────────┘
                        │
                        ▼
          ┌────────────────────────────┐
          │ Application Discovery      │
          │ login.php / reports/       │
          └─────────────┬──────────────┘
                        │
                        ▼
          ┌────────────────────────────┐
          │ Username Enumeration       │
          └─────────────┬──────────────┘
                        │
                        ▼
          ┌────────────────────────────┐
          │ SQL Injection              │
          │ Authentication Bypass      │
          └─────────────┬──────────────┘
                        │
                        ▼
          ┌────────────────────────────┐
          │ Patient Report Access      │
          └─────────────┬──────────────┘
                        │
                ┌───────┴────────┐
                ▼                ▼
       ┌────────────────┐ ┌────────────────┐
       │ Weak PDF       │ │ PDF Metadata   │
       │ Passwords      │ │ Disclosure     │
       └────────────────┘ └────────────────┘
Separate Data-Exposure Path
Exposed SQL Backup
       │
       ▼
mediroza_hr Database
       │
 ┌─────┴─────────┐
 ▼               ▼
staff        shareholders
 │               │
 ▼               ▼
Employee       Ownership
& Salary       Information
Data
📊 Risk Matrix
Severity	Count
🔴 Critical	2
🟠 High	2
🟡 Medium	3
🟢 Low	0

Total Findings - 7

🛠️ Key Remediation Priorities
Priority 1 — Remove Sensitive Exposures
Remove exposed database backups.
Disable directory indexing.
Protect error logs.
Move confidential reports outside directly browsable web directories.
Priority 2 — Fix Authentication
Remediate SQL injection.
Implement parameterized queries.
Prevent username enumeration.
Implement rate limiting.
Consider MFA.
Priority 3 — Fix Authorization

Every patient report request should independently verify:

Authenticated User
        +
User Authorization
        +
Report Ownership
        =
Access Granted
Priority 4 — Secure Patient Documents
Use strong document passwords.
Use appropriate encryption.
Sanitize PDF metadata.
Protect report storage.
Monitor report access.
Priority 5 — Secure Backups

Backups should follow:

Backup
  │
  ├── Outside Web Root
  │
  ├── Access Controlled
  │
  ├── Encrypted
  │
  ├── Least Privilege
  │
  └── Regularly Audited
📁 Suggested Repository Structure
mediroza-pentest/
│
├── README.md
│
├── report/
│   └── Mediroza_Penetration_Testing_Report.docx
│
├── evidence/
│   ├── figure-01-patient-directory-listing.png
│   ├── staff-table.png
│   ├── shareholders-table.png
│   └── database-exposure-summary.png
│
├── screenshots/
│
└── notes/



📋 Final Assessment Summary

The assessment identified weaknesses across multiple security layers:

Security Area	Finding
Information Disclosure	
Authentication	
Authorization	
Document Security	
Metadata Security	

The most significant issues identified were:

🔴 F-02 — Confidential Internal SQL Database Backup Exposure

and

🔴 F-04 — SQL Injection Login Bypass

Both were rated Critical within the assessment.

The combined findings demonstrate weaknesses across the application's directory exposure, authentication, authorization, patient-document protection, and backup-security controls.

<p align="center">

---

👤 Author - JOSIAH AYUBA

LinkedIn - https://www.linkedin.com/in/josiahayuba/

Mediroza General Hospital — Black-Box Penetration Testing Project

Assessment Type: External Black-Box Web Application Security Assessment

