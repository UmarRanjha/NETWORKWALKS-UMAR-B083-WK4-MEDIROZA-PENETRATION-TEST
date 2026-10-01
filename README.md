# Penetration Testing Report: Mediroza Hospital

> **Disclaimer**: This penetration testing engagement was conducted in a controlled environment for educational purposes. All security testing on `medirozahospital.com` was authorized by Networkwalks. Unauthorized application of these security testing techniques against systems without explicit written consent is strictly prohibited.

---

## 📋 Table of Contents
- [01. Executive Summary](#01-executive-summary)
- [02. Scope and Methodology](#02-scope-and-methodology)
- [03. Findings and Proof of Exploitation](#03-findings-and-proof-of-exploitation)
- [04. Risk Rating](#04-risk-rating)
- [05. Recommendations and Remediation](#05-recommendations-and-remediation)

---

## 01. Executive Summary

This security assessment was performed by security researchers on behalf of **Networkwalks** in a strictly controlled educational environment. The primary target of this engagement was **Mediroza Hospital** (`medirozahospital.com`).

The objective was to evaluate the target's external and application attack surface, identify vulnerabilities, perform proof-of-concept exploitation, and provide remediation strategies.

### Key Findings Summary
* **Critical SQL Injection (SQLi)**: Identified in the Patient Portal login interface (`/patient/login.php`), enabling authentication bypass and unauthorized access to protected patient portal accounts.
* **High Weak Password Protection**: Encrypted patient PDF reports utilized extremely weak passwords (`123456`), allowing rapid offline brute-force recovery.
* **Low Information Disclosure**: Web server response headers reveal exact software details (`LiteSpeed`, `x-turbo-charged-by`).

---

## 02. Scope and Methodology

### Target Information
| Attribute | Details |
| :--- | :--- |
| **Target Domain** | `medirozahospital.com` |
| **Target IP** | `199.188.201.16` |
| **Testing OS** | Kali Linux (`umar@kali`) |

### Tools Used
* **Network & Footprinting**: `whois`, `whatweb`, `ping`
* **Web Exploitation**: Web Browser, Manual SQL Injection
* **Password Cracking**: `pdfcrack` with `rockyou.txt` wordlist

### Methodology
1. **Reconnaissance & Footprinting**: Domain discovery via `whois`, host availability checks using ICMP (`ping`), and server technology identification via `whatweb`.
2. **Vulnerability Assessment**: Input field analysis on public forms to check for input sanitization and logic flaws.
3. **Exploitation**: Authentication bypass via SQL Injection to access restricted medical records.
4. **Post-Exploitation Analysis**: Verification of file security mechanisms on downloaded patient artifacts.

---

## 03. Findings and Proof of Exploitation

### Step 1: Initial Reconnaissance and Infrastructure Fingerprinting

#### 1. Host Connectivity and Environment Verification
Initial connectivity testing verified active network routing and privilege levels on the assessment machine.

![Terminal Environment Setup](Screenshot_2026-10-01_04_23_03.png)

#### 2. Domain Registration Reconnaissance (`whois`)
Querying the WHOIS database for `medirozahospital.com` returned no active registry match.

```bash
whois medirozahospital.com
```

![WHOIS Query Result](Screenshot_2026-10-01_04_23_03.png)

#### 3. Web Technology Fingerprinting (`whatweb`)
Executing `whatweb` identified the primary web infrastructure:
* **Target IP**: `199.188.201.16`
* **Web Server**: `LiteSpeed`
* **HTTP Status**: `301 Moved Permanently` (Redirecting to HTTPS), `403 Forbidden` on root directory requests.
* **Custom Headers**: `x-turbo-charged-by`

```bash
whatweb medirozahospital.com
```

![WhatWeb Output](Screenshot_2026-10-01_04_23_51.png)

---

### Step 2: Web Application Vulnerability Analysis & Exploitation

#### 1. SQL Injection Error Detection
Navigating to `http://medirozahospital.com/patient/login.php` opened the **Patient Portal**. Submitting a single quote (`'`) into the **Username** field generated an unhandled database exception:

> `Warning: mysqli_query(): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''' at line 1`

This raw database error confirms the presence of **SQL Injection (SQLi)**.

![SQL Error Message Exposed](Screenshot_2026-10-01_06_52_34.png)

#### 2. Authentication Bypass Exploitation
By submitting a boolean SQL payload into the `Username` field, the backend authentication logic was manipulated:

* **Username Payload**: `admin' -- `
* **Password**: `23232w2ds` (Arbitrary input)

![Authentication Bypass Payload](Screenshot_2026-10-01_06_52_51.png)

Submitting this payload bypassed authentication completely, providing access to `/patient/portal.php` ("My lab reports").

![Successful Patient Portal Access](Screenshot_2026-10-01_06_53_01.png)

The portal contained multiple downloadable patient files:
1. `Pathology Report - S. Dlamini` (`Lab Ref LR-2024-1187`)
2. `Pathology Report - P. Reddy` (`Lab Ref LR-2024-1192`)
3. `Pathology Report - E. Thompson` (`Lab Ref LR-2024-1205`)

---

### Step 3: Document Security & Offline Password Cracking

#### 1. Protected PDF Access Analysis
Opening the downloaded file `patient_report_1.pdf` prompted a password requirement dialog.

![PDF Password Prompt](Screenshot_2026-10-01_07_10_37.png)

#### 2. Offline Password Recovery (`pdfcrack`)
`pdfcrack` was executed against `patient_report_1.pdf` using the standard `rockyou.txt` dictionary file:

```bash
sudo apt update && sudo apt install pdfcrack
pdfcrack -f patient_report_1.pdf -w /usr/share/wordlists/rockyou.txt
```

![PDFCrack Tool Installation](Screenshot_2026-10-01_07_10_20.png)

The tool recovered the document password in seconds:

```text
found user-password: '123456'
```

![PDF Password Cracked Successfully](Screenshot_2026-10-01_07_10_27.png)

---

## 04. Risk Rating

| Vulnerability Title | Affected Component | CVSS v3.1 Score | Risk Rating | Justification |
| :--- | :--- | :--- | :--- | :--- |
| **SQL Injection (Auth Bypass)** | `/patient/login.php` | **9.8** | `CRITICAL` | Allows unauthenticated attackers to bypass login forms and access restricted application data. |
| **Weak PDF Encryption Passwords** | Patient Lab Reports | **7.5** | `HIGH` | Protected health records use simple passwords (`123456`), allowing trivial offline cracking. |
| **Verbose Database Error Output** | `/patient/login.php` | **5.3** | `MEDIUM` | MySQL exceptions reveal backend database structure, assisting in payload crafting. |
| **Server Banner Disclosure** | Web Server Headers | **5.3** | `LOW` | Server headers expose underlying software (`LiteSpeed`), aiding technical fingerprinting. |

---

## 05. Recommendations and Remediation

### 1. Implement Parameterized Queries (Critical)
* Replace dynamic string concatenation in database calls with **Prepared Statements** using parameterized PDO or MySQLi queries.

```php
// Remediation Example (PHP MySQLi)
$stmt = $conn->prepare("SELECT id, username FROM patients WHERE username = ? AND password = ?");
$stmt->bind_param("ss", $username, $password);
$stmt->execute();
$result = $stmt->get_result();
```

### 2. Disable Public Database Errors (Medium)
* Set `display_errors = Off` in `php.ini` for production environments. Implement custom error pages to prevent backend detail leakage.

### 3. Enforce Strong File Security Standards (High)
* Require robust, pseudo-random password generation for exported confidential documents.
* Transition to token-based or multi-factor verified download mechanisms rather than static PDF password protection.

### 4. Obfuscate Server Headers (Low)
* Reconfigure the web server (`LiteSpeed`) to strip `Server` and custom descriptive response headers.
