# 🛡️ Penetration Testing Report — Mediroza General Hospital

> **NetworkWalks Cybersecurity & Ethical Hacking Internship — Batch B083**  
> **Week 4 Capstone — Authorized Black-Box Penetration Test**

---

## 📋 Project Information

| Field | Details |
|---|---|
| **Client** | Mediroza General Hospital |
| **Target** | `https://medirozahospital.com` |
| **Testing Type** | Black-box Web Application Penetration Test |
| **Assessment Duration** | 5 Days |
| **Tester** | Bhabani Priyadarshini Panda |
| **Program** | NetworkWalks Cybersecurity & Ethical Hacking Internship |
| **Batch** | B083 |
| **Authorization** | Written permission granted by the client |
| **Scope** | `medirozahospital.com` |
| **Methodology** | Reconnaissance → Enumeration → Exploitation → Post-exploitation Analysis |

---

## ⚖️ Authorization & Ethical Disclaimer

This project was performed as part of an authorized cybersecurity training engagement for the NetworkWalks B083 internship.

All testing activities were restricted to the designated training target and were performed under the defined rules of engagement.

- No social engineering or phishing was performed.
- No denial-of-service testing was performed.
- Testing was limited to the authorized target.
- Sensitive information shown in screenshots should be redacted before public publication.

> **Disclaimer:** The techniques documented here are intended for authorized security testing and controlled cybersecurity training environments only.

---

# 📑 Table of Contents

1. [Executive Summary](#-executive-summary)
2. [Scope & Methodology](#-scope--methodology)
3. [Tools Used](#-tools-used)
4. [Phase 1 — Reconnaissance](#-phase-1--reconnaissance)
5. [Phase 2 — Enumeration](#-phase-2--enumeration)
6. [Phase 3 — SQL Injection Authentication Bypass](#-phase-3--sql-injection-authentication-bypass)
7. [Phase 4 — Patient Portal Enumeration](#-phase-4--patient-portal-enumeration)
8. [Phase 5 — PDF Extraction & Password Cracking](#-phase-5--pdf-extraction--password-cracking)
9. [Phase 6 — PDF Decryption](#-phase-6--pdf-decryption)
10. [Phase 7 — Metadata Analysis](#-phase-7--metadata-analysis)
11. [Phase 8 — Backup Exposure](#-phase-8--backup-exposure)
12. [Findings & Risk Ratings](#-findings--risk-ratings)
13. [Attack Chain](#-attack-chain)
14. [Recommendations](#-recommendations)
15. [Evidence](#-evidence)
16. [Conclusion](#-conclusion)
17. [Author](#-author)

---

# 🎯 Executive Summary

A black-box penetration test was conducted against the Mediroza General Hospital web application as part of the NetworkWalks B083 Week 4 capstone exercise.

The assessment identified a chain of security weaknesses beginning with a SQL injection vulnerability in the patient portal login. The vulnerability allowed authentication to be bypassed without knowing a valid password.

After obtaining access to the patient portal, three password-protected pathology reports were identified. The PDFs used weak passwords that could be recovered through password-cracking techniques.

Further analysis of the recovered document revealed metadata containing an internal reference to an old backup directory. The exposed directory contained a database backup that was accessible without authentication.

The resulting attack chain demonstrated how individually simple weaknesses can combine to expose sensitive patient, employee, and corporate information.

### 🔴 Key Findings

| ID | Finding | Risk |
|---|---|---|
| MED-01 | SQL Injection — Authentication Bypass | 🔴 Critical |
| MED-02 | Weak Password Protection on Confidential PDFs | 🟠 High |
| MED-03 | Sensitive Metadata Disclosure | 🟠 High |
| MED-04 | Publicly Accessible Database Backup | 🔴 Critical |

---

# 🔍 Scope & Methodology

## Scope

Testing was limited to:

```text
https://medirozahospital.com
```

The assessment focused on:

- Web application reconnaissance
- Endpoint discovery
- Authentication testing
- SQL injection testing
- Patient portal enumeration
- Password-protected PDF analysis
- PDF password recovery
- Document metadata analysis
- Backup exposure analysis

## Methodology

```text
Reconnaissance
      ↓
Enumeration
      ↓
Authentication Testing
      ↓
SQL Injection
      ↓
Patient Portal Access
      ↓
PDF Discovery
      ↓
PDF Hash Extraction
      ↓
Password Recovery
      ↓
PDF Decryption
      ↓
Metadata Analysis
      ↓
Backup Discovery
      ↓
Database Exposure
```

---

# 🧰 Tools Used

| Tool | Purpose |
|---|---|
| `whatweb` | Web technology fingerprinting |
| `wafw00f` | WAF detection |
| `gobuster` | Directory/content enumeration |
| **Burp Suite** | HTTP interception and request manipulation |
| `pdf2john` | Extracting password hashes from PDFs |
| **Hashcat** | PDF password recovery |
| `qpdf` | PDF decryption |
| `exiftool` | Metadata extraction |
| `curl` | Resource retrieval |
| Kali Linux | Testing environment |

---

# 🔎 Phase 1 — Reconnaissance

Initial reconnaissance was performed to identify the technologies, web server configuration, and exposed application surface.

### Web Fingerprinting

```bash
whatweb https://medirozahospital.com
```

### WAF Detection

```bash
wafw00f https://medirozahospital.com
```

### Evidence

> 📸 **Figure 01 — WhatWeb fingerprinting result**
>
> `[INSERT IMAGE HERE]`

> 📸 **Figure 02 — WAF detection result**
>
> `[INSERT IMAGE HERE]`

---

# 🗂️ Phase 2 — Enumeration

Directory and endpoint enumeration was performed to identify accessible application paths.

Example:

```bash
gobuster dir -u https://medirozahospital.com \
-w /usr/share/wordlists/dirb/common.txt
```

The reconnaissance phase identified areas including:

```text
/patient/
/staff/
/old/
```

The patient portal became the primary focus for authentication testing.

### Evidence

> 📸 **Figure 03 — Directory enumeration**
>
> `[INSERT IMAGE HERE]`

---

# 💉 Phase 3 — SQL Injection Authentication Bypass

## Target

```text
POST /patient/login.php
```

The login endpoint accepted username and password parameters.

Normal authentication testing was first performed using invalid credentials to establish the application's baseline response.

A SQL injection test was then performed against the username parameter.

### Payload

```text
username=admin'-- -&password=test
```

The payload attempts to terminate the original SQL string and comment out the remainder of the SQL statement.

### Burp Suite Request

> 📸 **Figure 04 — Burp Suite request showing SQL injection payload**
>
> `[INSERT IMAGE HERE]`

### Successful Response

The vulnerable endpoint returned:

```text
HTTP/2 302 Found
Location: portal.php
```

The redirect to `portal.php` demonstrated that authentication had been bypassed.

### Impact

An attacker who can exploit this vulnerability may gain access to functionality intended only for authenticated patients without possessing valid credentials.

---

# 🏥 Phase 4 — Patient Portal Enumeration

After authentication bypass, the patient portal was accessible.

The portal exposed multiple pathology report entries.

Example structure:

```text
My lab reports

Pathology Report - S. Dlamini
Lab Ref LR-2024-1187
2024-11-04
PDF (encrypted)
Download → download.php?id=1

Pathology Report - P. Reddy
Lab Ref LR-2024-1192
2024-11-05
PDF (encrypted)
Download → download.php?id=2

Pathology Report - E. Thompson
Lab Ref LR-2024-1205
2024-11-06
PDF (encrypted)
Download → download.php?id=3
```

### Evidence

> 📸 **Figure 05 — Authenticated patient portal**
>
> `[INSERT IMAGE HERE]`

> 📸 **Figure 06 — Patient report listing**
>
> `[INSERT IMAGE HERE]`

---

# 🔐 Phase 5 — PDF Extraction & Password Cracking

The discovered pathology reports were password protected.

The PDF files were first downloaded for offline analysis.

### PDF Verification

```bash
file patient3.pdf
```

The file was identified as a PDF document requiring a password.

### Hash Extraction

The `pdf2john` utility was used to extract a crackable representation of the PDF password.

```bash
pdf2john patient3.pdf > patient3.hash
```

The resulting hash was then processed with Hashcat using the appropriate PDF hash mode.

### Hashcat

```bash
hashcat -m 10500 -a 0 patient3.hash /path/to/wordlist.txt
```

The assessment recovered the following passwords:

| PDF | Recovered Password |
|---|---|
| `patient_report_1.pdf` | `123456` |
| `patient_report_2.pdf` | `password` |
| `patient_report_3.pdf` | `!@#$%^&` |

> 📸 **Figure 07 — PDF hash extraction**
>
> `[INSERT IMAGE HERE]`

> 📸 **Figure 08 — Hashcat password recovery**
>
> `[INSERT IMAGE HERE]`

### Security Impact

The documents were classified as confidential patient reports, yet their passwords were sufficiently weak to be recovered rapidly.

This demonstrates that password-protected documents should not rely on short, predictable, or commonly used passwords as their primary security control.

---

# 🔓 Phase 6 — PDF Decryption

After recovering the password, `qpdf` was used to create a decrypted copy of the protected PDF.

Example:

```bash
qpdf --password='RECOVERED_PASSWORD' \
--decrypt patient3.pdf patient3_decrypted.pdf
```

The decrypted document could then be inspected offline.

### Evidence

> 📸 **Figure 09 — Successful PDF decryption**
>
> `[INSERT IMAGE HERE]`

> 📸 **Figure 10 — Recovered report content**
>
> `[INSERT IMAGE HERE]`

> ⚠️ **Privacy:** Redact patient names, identifiers, medical information, and other sensitive content before committing screenshots to GitHub.

---

# 🧾 Phase 7 — Metadata Analysis

The decrypted PDF was examined for document metadata.

```bash
exiftool -a -u -g1 patient3_decrypted.pdf
```

The metadata contained an internal comment indicating that a database backup had previously been moved to an `/old/` directory.

The metadata also contained an author reference:

```text
j.malik
```

The comment disclosed an internal server path that was not intended to be exposed to unauthenticated users.

### Evidence

> 📸 **Figure 11 — ExifTool metadata output**
>
> `[INSERT IMAGE HERE]`

---

# 📂 Phase 8 — Backup Exposure

The disclosed `/old/` path was then investigated within the authorized scope.

Directory listing was enabled and exposed a database backup:

```text
mediroza_db_backup_2019.sql
```

The backup could be retrieved without authentication.

Example:

```bash
curl -O https://medirozahospital.com/old/mediroza_db_backup_2019.sql
```

### Database Contents

The exposed backup contained:

### `staff`

Approximately 30 records containing fields including:

- Full name
- Job title
- Department
- Email
- Phone number
- National ID number
- Monthly salary

### `shareholders`

Approximately 10 records containing:

- Shareholder name
- Share percentage
- Shares held
- Share class

### Evidence

> 📸 **Figure 12 — `/old/` directory listing**
>
> `[INSERT IMAGE HERE]`

> 📸 **Figure 13 — Database backup retrieval**
>
> `[INSERT IMAGE HERE]`

> 📸 **Figure 14 — Database structure/content**
>
> `[INSERT IMAGE HERE]`

> ⚠️ **Important:** Do not publish real National ID numbers, phone numbers, email addresses, salaries, or other personal information. Redact or replace sensitive values in screenshots.

---

# 🚨 Findings & Risk Ratings

## MED-01 — SQL Injection Authentication Bypass

**Severity:** 🔴 Critical

**Location:**

```text
POST /patient/login.php
```

**Issue:**

The login functionality was vulnerable to SQL injection, allowing authentication to be bypassed.

**Impact:**

- Authentication bypass
- Unauthorized access to patient portal functionality
- Access to confidential reports

**Recommendation:**

Use parameterized queries / prepared statements for all database operations.

Additional controls should include:

- Server-side input validation
- Least-privilege database accounts
- Secure error handling
- Appropriate WAF protections
- Authentication monitoring

---

## MED-02 — Weak PDF Password Protection

**Severity:** 🟠 High

**Issue:**

Confidential patient reports were protected using weak passwords that could be recovered through password-cracking techniques.

**Impact:**

An attacker who obtains the encrypted PDF files may recover their passwords and access confidential information.

**Recommendation:**

- Use strong randomly generated passwords.
- Avoid common passwords and predictable patterns.
- Prefer authenticated document-delivery mechanisms where practical.
- Apply appropriate encryption and access-control policies.

---

## MED-03 — Sensitive Metadata Disclosure

**Severity:** 🟠 High

**Issue:**

PDF metadata contained internal information referencing a backup directory.

**Impact:**

Metadata can unintentionally disclose:

- Internal usernames
- File paths
- Server structure
- Backup locations
- Development information

**Recommendation:**

Strip unnecessary metadata before distributing documents externally.

---

## MED-04 — Publicly Accessible Database Backup

**Severity:** 🔴 Critical

**Issue:**

A database backup was accessible from a web-exposed directory without authentication.

**Impact:**

The backup exposed sensitive employee and shareholder information.

**Recommendation:**

- Remove backups from the public web root.
- Store backups outside web-accessible directories.
- Disable directory listing.
- Encrypt backups at rest.
- Apply strict access controls.
- Rotate credentials associated with potentially exposed systems.
- Review logs for unauthorized access.

---

# 🔗 Attack Chain

The complete attack chain can be represented as:

```text
Public Web Application
        │
        ▼
Reconnaissance
        │
        ▼
Patient Login Endpoint
        │
        ▼
SQL Injection
        │
        ▼
Authentication Bypass
        │
        ▼
Patient Portal
        │
        ▼
Encrypted Pathology Reports
        │
        ▼
PDF Password Recovery
        │
        ▼
Decrypted Patient Report
        │
        ▼
Metadata Analysis
        │
        ▼
Internal /old/ Path Disclosed
        │
        ▼
Directory Listing Enabled
        │
        ▼
Database Backup Exposed
        │
        ▼
Sensitive Staff & Shareholder Data
```

---

# 🛠️ Recommendations & Remediation

| Vulnerability | Recommended Remediation |
|---|---|
| SQL Injection | Use prepared statements / parameterized queries |
| Authentication bypass | Implement secure authentication and session handling |
| Weak PDF passwords | Use strong randomly generated credentials or controlled document delivery |
| Metadata disclosure | Strip unnecessary metadata before publication |
| Directory listing | Disable directory indexing |
| Public backup | Move backups outside the web root |
| Backup protection | Encrypt backups at rest |
| Sensitive data exposure | Apply least-privilege access controls |
| Credential exposure | Rotate affected credentials |
| Monitoring | Implement logging and alerting for suspicious access |

---

# 📸 Evidence

All screenshots for this project will be added to the `evidence/` directory.

Suggested structure:

```text
evidence/
├── 01-whatweb.png
├── 02-wafw00f.png
├── 03-gobuster.png
├── 04-sqli-request.png
├── 05-sqli-response.png
├── 06-patient-portal.png
├── 07-pdf-hash.png
├── 08-hashcat.png
├── 09-pdf-decryption.png
├── 10-report.png
├── 11-exiftool.png
├── 12-old-directory.png
├── 13-backup-download.png
└── 14-database-analysis.png
```

Then replace the placeholders in this README with:

```markdown
![Description](evidence/01-whatweb.png)
```

For example:

```markdown
### Evidence — SQL Injection

![Burp Suite SQL Injection Request](evidence/04-sqli-request.png)
```

---

# 📁 Repository Structure

```text
NETWORKWALKS-BHABANI-B083-WK4-MEDIROZA-PENTEST/
│
├── README.md
│
└── evidence/
    ├── 01-whatweb.png
    ├── 02-wafw00f.png
    ├── 03-gobuster.png
    ├── 04-sqli-request.png
    ├── 05-sqli-response.png
    ├── 06-patient-portal.png
    ├── 07-pdf-hash.png
    ├── 08-hashcat.png
    ├── 09-pdf-decryption.png
    ├── 10-report.png
    ├── 11-exiftool.png
    ├── 12-old-directory.png
    ├── 13-backup-download.png
    └── 14-database-analysis.png
```

> **Do not commit recovered patient reports, database dumps, passwords, National ID numbers, phone numbers, or other sensitive information to a public repository.**

---

# 📝 Conclusion

The assessment demonstrated a multi-stage attack chain in which several weaknesses compounded one another.

The initial SQL injection provided unauthorized access to the patient portal. Weak protection of confidential PDF reports allowed their passwords to be recovered. Metadata within one of the recovered documents then disclosed an internal backup location, ultimately leading to an exposed database backup containing sensitive organizational information.

The exercise demonstrates the importance of defense-in-depth: fixing only one weakness would not necessarily eliminate the entire attack chain.

---

# 👤 Author

**Bhabani Priyadarshini Panda**

Cybersecurity Intern — NetworkWalks Academy  
Batch B083

### Project

**NETWORKWALKS-BHABANI-B083-WK4-MEDIROZA-PENTEST**

### Focus Areas

- Web Application Security
- SQL Injection
- Authentication Testing
- PDF Security
- Password Cracking
- Metadata Forensics
- Sensitive Data Exposure
- Penetration Testing

---

## ⚠️ Responsible Disclosure

This repository documents an authorized cybersecurity training exercise. The techniques and procedures described here must only be applied to systems for which explicit authorization has been obtained.
