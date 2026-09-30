# MEDIROZA GENERAL HOSPITAL

## Web Application Penetration Testing Report

| **Assessment Information** | **Details** |
|---|---|
| **Prepared by** | Kabo Sekoto |
| **Programme** | NetworkWalks Internship Programme |
| **Assessment** | Black-Box Web Application Penetration Test |
| **Target** | `medirozahospital.com` |
| **Date** | September 2026 |
| **Classification** | **Confidential — Authorised Personnel Only** |

---

# 01 — EXECUTIVE SUMMARY

A black box penetration test was conducted against the authorised Mediroza General Hospital web application.

The assessment identified multiple vulnerabilities affecting authentication, file exposure, information disclosure and application security. Several weaknesses could be chained together to gain access to sensitive application data.

Testing was performed within the authorised scope using standard penetration testing tools and techniques.

### Overall Risk

**🔴 CRITICAL**

---

# 02 — FINDINGS SUMMARY

| ID | Finding                                 | Severity    | CVSS |
| -- | --------------------------------------- | ----------- | ---: |
| 01 | SQL Injection / Authentication Bypass   | 🔴 Critical |  9.8 |
| 02 | Exposed Database Backup                 | 🔴 Critical |  9.1 |
| 03 | Weak PDF Password Protection            | 🟠 High     |  7.5 |
| 04 | Sensitive Paths Disclosed by robots.txt | 🟡 Medium   |  5.3 |
| 05 | Directory Listing Enabled               | 🟡 Medium   |  5.3 |
| 06 | Username Enumeration                    | 🟡 Medium   |  5.3 |
| 07 | Verbose SQL Errors                      | 🟢 Low      |  3.1 |

---

# 03 — SCOPE & METHODOLOGY

| Category               | Details              |
| ---------------------- | -------------------- |
| **Target**             | medirozahospital.com |
| **Test Type**          | Black-Box            |
| **Scope**              | Target domain        |
| **Duration**           | 3 Days               |
| **Authorisation**      | Granted              |
| **DoS Testing**        | Not performed        |
| **Social Engineering** | Not performed        |

### Methodology

**Reconnaissance → Enumeration → Vulnerability Identification → Controlled Exploitation → Evidence Collection → Reporting**

---

# 04 — TOOLS USED

| Tool            | Purpose                  |
| --------------- | ------------------------ |
| Nmap            | Port & service discovery |
| Gobuster        | Directory enumeration    |
| cURL            | HTTP testing             |
| Firefox         | Manual testing           |
| sqlmap          | SQL injection testing    |
| John the Ripper | Password testing         |
| pdf2john        | PDF hash extraction      |
| ExifTool        | Metadata analysis        |
| wget            | File retrieval           |
| Kali Linux      | Testing platform         |

---

# 05 — RECONNAISSANCE

### Nmap Service Discovery

```text
nmap -Pn -sV -sC medirozahospital.com
```

### Results

| Service     | Result             |
| ----------- | ------------------ |
| IP Address  | 199.188.201.16     |
| HTTP/HTTPS  | 80 / 443           |
| Web Server  | OpenResty 1.31.1.1 |
| SMTP        | 587                |
| SMTP Server | Exim 4.99.5        |

**Evidence:** `NMAP_SCAN`

> **[INSERT YOUR NMAP SCREENSHOT]**

---

# 06 — DIRECTORY ENUMERATION

Review of `robots.txt` revealed the following application paths:

```text
/patient/
/staff/
/old/
```

Directory indexing was enabled on `/patient/`, exposing application files including:

```text
login.php
portal.php
download.php
error_log
```

**Risk:** Attackers can identify application components and potentially sensitive resources.

**Evidence:** `DIRECTORY_ENUMERATION`

> **[INSERT SCREENSHOT]**

---

# 07 — FINDING 01

## SQL INJECTION — AUTHENTICATION BYPASS

**Severity:** 🔴 Critical
**CVSS:** 9.8

### Description

The patient portal login functionality was vulnerable to SQL injection.

A single quotation mark caused the application to return a MySQL syntax error, confirming improper input handling.

### Test

```text
Username: admin'--
Password: anything
```

The application subsequently granted access to the protected patient portal.

### Impact

Successful exploitation allowed unauthorised access to patient documents.

**Evidence:** `SQL_ERROR` / `SQL_BYPASS`

> **[INSERT SQL ERROR SCREENSHOT]**

> **[INSERT AUTHENTICATION BYPASS SCREENSHOT]**

---

# 08 — PATIENT DATA EXPOSURE

Following the authentication bypass, pathology reports were accessible through the patient portal.

| Patient        | Reference    | Date        |
| -------------- | ------------ | ----------- |
| Sipho Dlamini  | LR-2024-1187 | 04 Nov 2024 |
| Priya Reddy    | LR-2024-1192 | 05 Nov 2024 |
| Emily Thompson | LR-2024-1205 | 06 Nov 2024 |

**Impact:** Unauthorised access to confidential medical documentation.

> **[INSERT PORTAL EVIDENCE]**

---

# 09 — FINDING 02

## WEAK PDF PASSWORD PROTECTION

**Severity:** 🟠 High
**CVSS:** 7.5

PDF password hashes were extracted for authorised security testing:

```text
pdf2john patient_report_1.pdf > hash1.txt
pdf2john patient_report_2.pdf > hash2.txt
pdf2john patient_report_3.pdf > hash3.txt
```

### Results

| Document | Recovered Password | Assessment     |
| -------- | ------------------ | -------------- |
| Report 1 | `123456`           | Extremely Weak |
| Report 2 | `password`         | Extremely Weak |
| Report 3 | `!@#$%^&`          | Weak           |

**Impact:** Sensitive documents could be accessed using easily guessable passwords.

**Evidence:** `PDF_PASSWORD_TEST`

---

# 10 — FINDING 03

## INFORMATION DISCLOSURE VIA PDF METADATA

**Severity:** 🟡 Medium

Metadata analysis was performed using ExifTool:

```text
exiftool patient_report_3.pdf
```

The document contained an internal comment referencing an old database backup location.

```text
Author: j.malik
Comment: DB backup moved to /old before site migration
```

**Impact:** Internal information assisted further enumeration of the target.

**Evidence:** `PDF_METADATA`

> **[INSERT EXIFTOOL SCREENSHOT]**

---

# 11 — FINDING 04

## PUBLICLY ACCESSIBLE DATABASE BACKUP

**Severity:** 🔴 Critical
**CVSS:** 9.1

The `/old/` directory contained a publicly accessible database backup:

```text
mediroza_db_backup_2019.sql
```

The file could be retrieved without authentication.

```text
wget https://medirozahospital.com/old/mediroza_db_backup_2019.sql
```

### Exposed Information

* Employee records
* Salary information
* Shareholder information
* Database structure
* Other organisational data

**Impact:** Direct exposure of sensitive organisational information.

**Evidence:** `OLD_DIRECTORY`

> **[INSERT DIRECTORY/BACKUP SCREENSHOT]**

---

# 12 — DATABASE EXPOSURE

### Staff Records

**30 employee records identified.**

| Employee               | Position          |   Salary |
| ---------------------- | ----------------- | -------: |
| Dr Johan van der Merwe | Medical Director  | R160,000 |
| Sarah Botha            | CFO               | R152,000 |
| Dr Rajesh Naidoo       | Chief Pathologist | R138,000 |
| Dr Vikram Chetty       | Anaesthetist      | R135,000 |
| Dr Suresh Moodley      | Radiologist       | R130,000 |

### Shareholder Records

**10 shareholder records identified.**

| Shareholder            | Ownership |
| ---------------------- | --------: |
| Dr Rajesh Naidoo       |       18% |
| Cedar Health Holdings  |       15% |
| Dr Johan van der Merwe |       12% |
| Reddy Family Trust     |       11% |
| Thabo Molefe           |       10% |
| Dr Ahmed Kara          |        8% |

**Evidence:** `DATABASE_EXPOSURE`

> **[INSERT DATABASE EVIDENCE]**

---

# 13 — ATTACK CHAIN

```text
Public Web Application
        ↓
Directory Enumeration
        ↓
Patient Portal Discovery
        ↓
SQL Injection
        ↓
Authentication Bypass
        ↓
Patient Document Access
        ↓
PDF Metadata Disclosure
        ↓
/old/ Directory Discovery
        ↓
Database Backup Exposure
        ↓
Sensitive Data Exposure
```

---

# 14 — REMEDIATION

## 🔴 IMMEDIATE

* Fix SQL injection using prepared statements.
* Remove the database backup from the public web directory.
* Disable detailed SQL/database errors.
* Review all exposed sensitive files.

## 🟠 SHORT TERM

* Enforce strong document-password requirements.
* Disable directory indexing.
* Review `robots.txt` disclosures.
* Implement generic authentication error messages.

## 🟡 MEDIUM TERM

* Implement MFA for sensitive portals.
* Deploy appropriate WAF controls.
* Review web-server permissions.
* Remove obsolete files and backups.

## 🟢 LONG TERM

* Conduct regular penetration testing.
* Implement a Secure Development Lifecycle.
* Establish an incident-response process.
* Provide security awareness training.

---

# 15 — CONCLUSION

The assessment identified critical weaknesses within the Mediroza General Hospital web environment.

The primary risks were associated with **SQL injection, authentication bypass, exposed backups and insufficient protection of sensitive documents**.

The assessment also demonstrated how multiple lower-level weaknesses can be combined into a broader attack path.

Priority should be given to correcting the authentication and database-exposure vulnerabilities, followed by improvements to access control, server configuration and secure development practices.

---

# 16 — EVIDENCE REGISTER

| Evidence ID             | Evidence                 |
| ----------------------- | ------------------------ |
| `NMAP_SCAN`             | Service discovery        |
| `DIRECTORY_ENUMERATION` | Web directory discovery  |
| `SQL_ERROR`             | SQL injection evidence   |
| `SQL_BYPASS`            | Authentication bypass    |
| `PDF_PASSWORD_TEST`     | PDF password testing     |
| `PDF_METADATA`          | Metadata disclosure      |
| `OLD_DIRECTORY`         | Database backup exposure |
| `DATABASE_EXPOSURE`     | Database findings        |

---

## CONFIDENTIAL

**Prepared by:** Kabo Sekoto
**NetworkWalks Internship Programme**
**September 2026**
