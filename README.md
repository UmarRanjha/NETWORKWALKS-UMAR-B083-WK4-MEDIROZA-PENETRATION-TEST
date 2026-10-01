# Penetration Testing Report - Mediroza Hospital

---

## 01. Executive Summary

This security assessment was performed by security researchers on behalf of **Networkwalks** in a strictly controlled educational environment[cite: 1]. The primary target of this engagement was **Mediroza Hospital** (`medirozahospital.com`).

The objective was to evaluate the target's external and application attack surface, identify vulnerabilities, perform proof-of-concept exploitation, and provide actionable remediation strategies.

### Key Findings Summary
* **Critical Unprotected Database Backup Leak**: Directory enumeration via `gobuster` and `robots.txt` disclosed an unlinked path `/old` containing `mediroza_db_backup_2019.sql`, exposing full staff Personally Identifiable Information (PII), monthly salary records, and shareholder structures.
* **Critical SQL Injection (SQLi)**: Identified in the Patient Portal login interface (`/patient/login.php`). This flaw allows authentication bypass and direct access to protected patient medical records.
* **High Weak Document Encryption**: Password-protected PDF lab reports were secured using weak passwords (`123456`), allowing rapid offline cracking.
* **Low Information Disclosure**: The web server exposes specific technical banners (`LiteSpeed`, `x-turbo-charged-by` headers).

---

## 02. Scope and Methodology

### Target Information
* **Target Domain**: `medirozahospital.com`
* **Target IP**: `199.188.201.16`[cite: 3]
* **Testing Environment**: Kali Linux (`umar@kali`)[cite: 1, 2]

### Tools Used
* **Reconnaissance & Enumeration**: `whois`, `whatweb`, `ping`, `gobuster`, `robots.txt` analysis[cite: 1, 2, 3]
* **Web Application Testing**: Google Chrome, SQL Injection manual testing[cite: 5, 6]
* **Password Cracking**: `pdfcrack` with `rockyou.txt` wordlist[cite: 8, 9]

### Methodology
1. **Reconnaissance & Footprinting**: Enumerated domain records, checked host availability, fingerprinted server banners, and performed directory bruteforcing[cite: 1, 2, 3].
2. **Vulnerability Assessment**: Evaluated application input fields for input validation defects and improper access control.
3. **Exploitation**: Demonstrated authentication bypass via SQL Injection to access restricted patient files[cite: 6, 7].
4. **Post-Exploitation Analysis**: Recovered encrypted file contents through offline dictionary attacks and analyzed exposed database dumps[cite: 8, 9].

---

## 03. Findings and Proof of Exploitation

### Step 1: Reconnaissance & Database Backup Discovery

#### 1. Host Connectivity & Technology Fingerprinting
Initial connectivity and environment checks were conducted using `ping google.com` and `whoami`[cite: 1]. A technology scan using `whatweb medirozahospital.com` revealed[cite: 3]:
* **IP Address**: `199.188.201.16`[cite: 3]
* **Web Server**: `LiteSpeed`[cite: 3]
* **Headers**: `x-turbo-charged-by`[cite: 3]

![WhatWeb Output](Screenshot_2026-10-01_04_23_51.png)[cite: 3]

#### 2. Unlinked Directory & Backup File Discovery
Executing directory brute-forcing with `gobuster` alongside manual review of `http://medirozahospital.com/robots.txt` identified an unlinked directory path `/old`. Accessing this path revealed a publicly accessible SQL dump file named `mediroza_db_backup_2019.sql`.

#### 3. Exfiltrated Database Backup Data

##### A. Staff Records (`staff` Table)
| ID | Full Name | Job Title | Department | Email | Phone | National ID | Monthly Salary (ZAR) | Date Joined |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: |
| 1 | Dr. Rajesh Naidoo | Chief Pathologist | Diagnostics Lab | `r.naidoo@medirozahospital.com` | +27 82 101 2007 | 85021013011081 | R 138,000 | 2009-03-16 |
| 2 | Sarah Botha | Chief Financial Officer | Finance | `s.botha@medirozahospital.com` | +27 82 102 2014 | 85031023022082 | R 152,000 | 2011-07-01 |
| 3 | Dr. Johan van der Merwe | Medical Director | Management | `j.merwe@medirozahospital.com` | +27 82 103 2021 | 85041033033083 | R 160,000 | 2007-01-22 |
| 4 | Dr. Anita Naicker | Consultant Cardiologist | Cardiology | `a.naicker@medirozahospital.com` | +27 82 104 2028 | 85051043044084 | R 132,000 | 2012-09-10 |
| 5 | Dr. Ahmed Kara | Consultant Physician | Internal Medicine | `a.kara@medirozahospital.com` | +27 82 105 2035 | 85061053055085 | R 128,000 | 2013-02-18 |
| 6 | Dr. Yusuf Cassim | Senior Registrar | Emergency & Trauma | `y.cassim@medirozahospital.com` | +27 82 106 2042 | 85071063066086 | R 74,000 | 2018-05-04 |
| 7 | Michael Roberts | HR Director | Human Resources | `m.roberts@medirozahospital.com` | +27 82 107 2049 | 85081073077087 | R 96,000 | 2010-11-15 |
| 8 | Susan Pretorius | HR Officer | Human Resources | `s.pretorius@medirozahospital.com` | +27 82 108 2056 | 85091083088088 | R 32,000 | 2016-08-23 |
| 9 | Jameel Malik | IT Systems Administrator | IT | `j.malik@medirozahospital.com` | +27 82 109 2063 | 85101093099080 | R 58,000 | 2015-04-12 |
| 10 | Thabo Molefe | Network Engineer | IT | `t.molefe@medirozahospital.com` | +27 82 110 2070 | 85111103110081 | R 46,000 | 2017-10-02 |
| 11 | Nomvula Khumalo | Registered Nurse | Emergency & Trauma | `n.khumalo@medirozahospital.com` | +27 82 111 2077 | 85121113121082 | R 34,000 | 2016-01-19 |
| 12 | Lerato Mokoena | Registered Nurse | Pediatrics | `l.mokoena@medirozahospital.com` | +27 82 112 2084 | 85011123132083 | R 33,000 | 2017-06-07 |
| 13 | Bongani Ndlovu | Registered Nurse | Cardiology | `b.ndlovu@medirozahospital.com` | +27 82 113 2091 | 85021133143084 | R 35,000 | 2015-12-01 |
| 14 | Zanele Mahlangu | Nursing Sister | Theatre | `z.mahlangu@medirozahospital.com` | +27 82 114 2098 | 85031143154085 | R 42,000 | 2013-03-25 |
| 15 | Kagiso Sithole | Pharmacist | Pharmacy | `k.sithole@medirozahospital.com` | +27 82 115 2105 | 85041153165086 | R 61,000 | 2014-09-08 |
| 16 | Naledi Zulu | Pharmacy Assistant | Pharmacy | `n.zulu@medirozahospital.com` | +27 82 116 2112 | 85051163176087 | R 26,000 | 2019-02-14 |
| 17 | Themba Nkosi | Radiographer | Radiology | `t.nkosi@medirozahospital.com` | +27 82 117 2119 | 85061173187088 | R 44,000 | 2016-07-30 |
| 18 | Palesa Radebe | Radiographer | Radiology | `p.radebe@medirozahospital.com` | +27 82 118 2126 | 85071183198080 | R 43,000 | 2017-04-11 |
| 19 | Deepak Pillay | Lab Technologist | Diagnostics Lab | `d.pillay@medirozahospital.com` | +27 82 119 2133 | 85081193209081 | R 41,000 | 2015-05-20 |
| 20 | Kavitha Govender | Lab Technician | Diagnostics Lab | `k.govender@medirozahospital.com` | +27 82 120 2140 | 85091203220082 | R 35,000 | 2018-08-06 |
| 21 | Dr. Suresh Moodley | Consultant Radiologist | Radiology | `s.moodley@medirozahospital.com` | +27 82 121 2147 | 85101213231083 | R 130,000 | 2012-02-28 |
| 22 | Dr. Fatima Patel | Pediatrician | Pediatrics | `f.patel@medirozahospital.com` | +27 82 122 2154 | 85111223242084 | R 118,000 | 2013-10-17 |
| 23 | Nisha Singh | Physiotherapist | Rehabilitation | `n.singh@medirozahospital.com` | +27 82 123 2161 | 85121233253085 | R 48,000 | 2016-11-09 |
| 24 | Dr. Vikram Chetty | Anaesthetist | Theatre | `v.chetty@medirozahospital.com` | +27 82 124 2168 | 85011243264086 | R 135,000 | 2011-06-13 |
| 25 | David Smith | Facilities Manager | Operations | `d.smith@medirozahospital.com` | +27 82 125 2175 | 85021253275087 | R 52,000 | 2014-01-27 |
| 26 | Karen O'Connor | Billing Administrator | Finance | `k.oconnor@medirozahospital.com` | +27 82 126 2182 | 85031263286088 | R 29,000 | 2018-03-19 |
| 27 | James Wilson | Security Supervisor | Operations | `j.wilson@medirozahospital.com` | +27 82 127 2189 | 85041273297080 | R 27,000 | 2019-09-02 |
| 28 | Linda Fourie | Receptionist | Front Office | `l.fourie@medirozahospital.com` | +27 82 128 2196 | 85051283308081 | R 19,000 | 2020-02-10 |
| 29 | Peter van Wyk | Procurement Officer | Supply Chain | `p.wyk@medirozahospital.com` | +27 82 129 2203 | 85061293319082 | R 38,000 | 2015-08-24 |
| 30 | Andile Mbeki | Ward Clerk | Administration | `a.mbeki@medirozahospital.com` | +27 82 130 2210 | 85071303330083 | R 21,000 | 2019-11-05 |

##### B. Shareholder Records (`shareholders` Table)
| ID | Shareholder Name | Share Percent (%) | Shares Held | Share Class |
| :---: | :--- | :---: | :---: | :--- |
| 1 | Dr. Rajesh Naidoo | 18.0% | 180,000 | Ordinary |
| 2 | Cedar Health Holdings (Pty) Ltd | 15.0% | 150,000 | Ordinary |
| 3 | Dr. Johan van der Merwe | 12.0% | 120,000 | Ordinary |
| 4 | Reddy Family Trust | 11.0% | 110,000 | Ordinary |
| 5 | Thabo Molefe | 10.0% | 100,000 | Ordinary |
| 6 | Sarah Botha | 9.0% | 90,000 | Ordinary |
| 7 | Dr. Ahmed Kara | 8.0% | 80,000 | Preferential |
| 8 | Naledi Zulu | 7.0% | 70,000 | Ordinary |
| 9 | Michael Roberts | 6.0% | 60,000 | Ordinary |
| 10 | Dr. Vikram Chetty | 4.0% | 40,000 | Preferential |

---

### Step 2: Web Application Vulnerability Analysis & Exploitation

#### 1. SQL Injection Identification
Testing input parameters on `/patient/login.php` triggered a detailed database syntax error upon submitting a single quote (`'`):

> `Warning: mysqli_query(): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''' at line 1`

![SQL Error Message Exposed](Screenshot_2026-10-01_06_52_34.png)[cite: 5]

#### 2. Authentication Bypass Exploitation
Inputting an injection payload altered the backend authentication query logic:
* **Username Field**: `admin' -- `[cite: 6]
* **Password Field**: Arbitrary text[cite: 6]

![Authentication Bypass Payload](Screenshot_2026-10-01_06_52_51.png)[cite: 6]

Submitting this payload bypassed credential validation entirely and provided access to `/patient/portal.php`[cite: 7].

![Successful Patient Portal Access](Screenshot_2026-10-01_06_53_01.png)[cite: 7]

The portal contained direct links to downloadable encrypted PDF reports[cite: 7]:
1. `Pathology Report - S. Dlamini` (`Lab Ref LR-2024-1187`)[cite: 7]
2. `Pathology Report - P. Reddy` (`Lab Ref LR-2024-1192`)[cite: 7]
3. `Pathology Report - E. Thompson` (`Lab Ref LR-2024-1205`)[cite: 7]

---

### Step 3: Document Encryption Analysis & Cracking

#### 1. Password Recovery via `pdfcrack`
Attempting to open `patient_report_1.pdf` prompted a password challenge[cite: 10]. Using `pdfcrack` with the `rockyou.txt` wordlist recovered the key[cite: 8, 9]:

```bash
sudo apt update && sudo apt install pdfcrack
pdfcrack -f patient_report_1.pdf -w /usr/share/wordlists/rockyou.txt
