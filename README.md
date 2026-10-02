# Penetration Testing Report — Mediroza General Hospital

Client: Mediroza General Hospital  
Target: https://medirozahospital.com  
Type: Black-box Penetration Test  
Duration: 5 Days  
Tester: Bhabani Priyadarshini Panda (Networkwalks B083)  
Authorization: Written permission granted by client

---

## 1. Executive Summary

A black-box penetration test was conducted against medirozahospital.com. The engagement identified a Critical SQL Injection vulnerability in the patient portal login that allowed complete authentication bypass without valid credentials. This initial foothold led to the retrieval of 3 confidential patient pathology reports, which were found to be encrypted with weak, easily-guessable passwords.

Further analysis of document metadata revealed an internal comment referencing an old backup location, which exposed a publicly-accessible directory listing containing a full database backup (`mediroza_db_backup_2019.sql`). This backup exposed sensitive data for 30 staff members (including salaries and National ID numbers) and 10 shareholders (including equity holdings). The combined chain of vulnerabilities represents a severe risk to patient confidentiality, staff PII, and corporate financial data.

## 2. Scope and Methodology

Scope: Full black-box penetration test limited to medirozahospital.com. No social engineering, no denial-of-service testing performed.

Methodology: Reconnaissance → Enumeration → Exploitation → Post-exploitation analysis.

Tools Used:

| Tool | Purpose |
| --- | --- |
| whatweb, wafw00f | Stack and WAF fingerprinting |
| gobuster | Directory/content enumeration |
| Burp Suite | Request interception and manipulation |
| pdf2john, hashcat | PDF password cracking |
| qpdf | PDF decryption |
| exiftool | Metadata extraction |
| curl | Resource retrieval |

## 3. Findings and Proof of Exploitation

### Finding 1: SQL Injection — Authentication Bypass (Critical)

Location: `POST /patient/login.php`  
Payload: `username=admin'-- &password=qwerty`

The injected payload broke out of the SQL query, bypassing authentication entirely. The server returned `302 Found` with `Location: portal.php`, confirming successful bypass, and granted access to the patient portal without valid credentials.

Evidence:

![SQL Injection Request](evidence/01-sqli-request.png)

![SQL Injection Response](evidence/02-sqli-response.png)

### Finding 2: Weak Encryption on Confidential PDFs (High)

Location: 3 retrieved patient lab report PDFs  
Method: Hashes extracted with `pdf2john`, cracked with `hashcat -m 10500` using the rockyou.txt wordlist.

Results:

| File | Patient | Password | Crack Time |
| --- | --- | --- | --- |
| patient_report_1.pdf | S. Dlamini | `123456` | <1 sec |
| patient_report_2.pdf | P. Reddy | `password` | <1 sec |
| patient_report_3.pdf | E. Thompson | `!@#$%^&` | ~2 sec |

All three were trivially crackable, demonstrating inadequate document protection for data classified as confidential patient health information.

Evidence:

![Patient Portal](evidence/03-patient-portal.png)

![PDF Hash Extraction](evidence/04-pdf-hash.png)

![Hashcat Password Recovery](evidence/05-hashcat.png)

Recovered content:

![Recovered Patient Report](evidence/06-recovered-report.png)

### Finding 3: Sensitive Metadata Disclosure Leading to Exposed Database Backup (Critical)

Location: `patient_report_3.pdf` document properties → `https://medirozahospital.com/old/`

Step 1 — Metadata leak: Running `exiftool -a -u -g1` on the decrypted `patient_report_3.pdf` revealed an internal comment: "DB backup moved to /old before site migration, do not delete", with author field `j.malik`. This disclosed an internal server path not intended for public knowledge.

Step 2 — Directory listing open: Navigating to the disclosed path confirmed `/old/` had directory listing enabled, exposing `mediroza_db_backup_2019.sql` (6.3 KB) for unauthenticated download via `curl`.

Evidence:

![PDF Metadata](evidence/07-exiftool.png)

![Old Directory Listing](evidence/08-old-directory.png)

![Database Backup Download](evidence/09-backup-download.png)

Data Exposed in the backup:

* `staff` table: 30 records including full name, job title, department, email, phone, National ID number, and monthly salary (ZAR)
* `shareholders` table: 10 records including shareholder name, share percentage, shares held, share class

This exposes highly sensitive PII (national IDs) and confidential corporate financial/ownership data to any unauthenticated internet user.

Evidence:

![Database Analysis](evidence/10-database-analysis.png)

## 4. Risk Rating

| # | Finding | Risk | Justification |
| --- | --- | --- | --- |
| 1 | SQL Injection — Auth Bypass | Critical | Full authentication bypass, direct access to patient records |
| 2 | Weak PDF Encryption | High | Trivial to crack, exposes patient health data |
| 3 | Metadata Leak → Exposed DB Backup | Critical | Unauthenticated exposure of PII (National IDs) + salary + shareholder equity data |

## 5. Recommendations and Remediation

1. SQL Injection: Use parameterized queries/prepared statements for all database interactions. Implement input validation and a WAF rule set for SQLi patterns.
2. PDF Encryption: Enforce strong, randomly-generated passwords (min. 12 characters) for any confidential document, or move to proper access-controlled delivery instead of password-protected PDFs.
3. Metadata: Strip author/comment metadata from all generated documents before distribution; sanitize internal notes from document properties.
4. Exposed Backup: Immediately remove `/old/` directory from the public web root. Disable directory listing server-wide (`Options -Indexes` on Apache/LiteSpeed equivalent). Store backups outside the web-accessible path, encrypted at rest. Rotate/invalidate any credentials or PII potentially exposed in the leaked backup.

---

This assessment was conducted under written authorization for educational/training purposes (Networkwalks B083).

## About

No description, website, or topics provided.
