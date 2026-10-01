# NETWORKWALKS-UMAR-B083-WK4-MEDIROZA-PENETRATION-TEST

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Penetration Testing Report - Mediroza Hospital</title>
    <style>
        :root {
            --bg-color: #0d1117;
            --card-bg: #161b22;
            --border-color: #30363d;
            --text-color: #c9d1d9;
            --text-heading: #f0f6fc;
            --accent-blue: #58a6ff;
            --accent-red: #f85149;
            --accent-orange: #d29922;
            --accent-green: #3fb950;
            --code-bg: #161b22;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            line-height: 1.6;
            margin: 0;
            padding: 40px 20px;
        }

        .container {
            max-width: 900px;
            margin: 0 auto;
            background-color: var(--card-bg);
            border: 1px solid var(--border-color);
            border-radius: 6px;
            padding: 40px;
        }

        h1, h2, h3 {
            color: var(--text-heading);
            border-bottom: 1px solid var(--border-color);
            padding-bottom: 0.3em;
        }

        h1 { font-size: 2em; margin-top: 0; }
        h2 { font-size: 1.5em; margin-top: 24px; }
        h3 { font-size: 1.25em; margin-top: 18px; border-bottom: none; }

        a { color: var(--accent-blue); text-decoration: none; }
        a:hover { text-decoration: underline; }

        code {
            font-family: ui-monospace, SFMono-Regular, SF Mono, Menlo, Consolas, Liberation Mono, monospace;
            background-color: rgba(110, 118, 129, 0.4);
            padding: 0.2em 0.4em;
            border-radius: 6px;
            font-size: 85%;
        }

        pre {
            background-color: #0d1117;
            border: 1px solid var(--border-color);
            padding: 16px;
            border-radius: 6px;
            overflow: auto;
        }

        pre code {
            background-color: transparent;
            padding: 0;
            font-size: 100%;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin: 16px 0;
        }

        table th, table td {
            padding: 10px 13px;
            border: 1px solid var(--border-color);
            text-align: left;
        }

        table th {
            background-color: #21262d;
            color: var(--text-heading);
        }

        blockquote {
            margin: 0;
            padding: 0 1em;
            color: #8b949e;
            border-left: 0.25em solid var(--border-color);
        }

        .badge {
            display: inline-block;
            padding: 2px 8px;
            border-radius: 12px;
            font-size: 12px;
            font-weight: bold;
            color: #ffffff;
        }

        .badge-critical { background-color: var(--accent-red); }
        .badge-high { background-color: #db6d28; }
        .badge-medium { background-color: var(--accent-orange); }
        .badge-low { background-color: var(--accent-green); }

        img {
            max-width: 100%;
            height: auto;
            border: 1px solid var(--border-color);
            border-radius: 6px;
            margin: 12px 0;
        }

        hr {
            height: 0.25em;
            padding: 0;
            margin: 24px 0;
            background-color: var(--border-color);
            border: 0;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>Penetration Testing Report</h1>
    
    <hr>

    <h2>01. Executive Summary</h2>
    <p>This security assessment was performed by security researchers on behalf of <strong>Networkwalks</strong> in a strictly controlled educational environment[cite: 1]. The primary target of this engagement was <strong>Mediroza Hospital</strong> (<code>medirozahospital.com</code>)[cite: 3].</p>
    <p>The objective was to evaluate the target's external and application attack surface, identify vulnerabilities, perform proof-of-concept exploitation, and provide remediation strategies.</p>

    <h3>Key Findings Summary</h3>
    <ul>
        <li><strong>Critical SQL Injection (SQLi)</strong>: Found in the Patient Portal login interface (<code>/patient/login.php</code>)[cite: 5]. This allows authentication bypass and direct access to protected patient data[cite: 6, 7].</li>
        <li><strong>High Weak Password Protection</strong>: Encrypted patient PDF reports were protected using extremely weak passwords (<code>123456</code>)[cite: 9], enabling rapid offline brute-force cracking[cite: 9].</li>
        <li><strong>Low Information Disclosure &amp; Server Banners</strong>: Web servers reveal exact banner details (<code>LiteSpeed</code>, <code>x-turbo-charged-by</code> headers)[cite: 3].</li>
    </ul>

    <hr>

    <h2>02. Scope and Methodology</h2>

    <h3>Target Information</h3>
    <ul>
        <li><strong>Target Domain</strong>: <code>medirozahospital.com</code>[cite: 3]</li>
        <li><strong>Target IP</strong>: <code>199.188.201.16</code>[cite: 3]</li>
        <li><strong>Testing Environment</strong>: Kali Linux (<code>umar@kali</code>)[cite: 1, 2]</li>
    </ul>

    <h3>Tools Used</h3>
    <ul>
        <li><strong>Network &amp; Information Gathering</strong>: <code>whois</code>, <code>whatweb</code>, <code>ping</code>[cite: 1, 2, 3]</li>
        <li><strong>Web Exploitation</strong>: Google Chrome Browser, Manual SQL Injection payloads[cite: 5, 6]</li>
        <li><strong>Password Cracking</strong>: <code>pdfcrack</code> with <code>rockyou.txt</code> wordlist[cite: 8, 9]</li>
    </ul>

    <h3>Methodology</h3>
    <ol>
        <li><strong>Reconnaissance &amp; Footprinting</strong>: Domain enumeration using <code>whois</code>, host connectivity checks via ICMP (<code>ping</code>), and server fingerprinting using <code>whatweb</code>[cite: 1, 2, 3].</li>
        <li><strong>Vulnerability Assessment</strong>: Identifying web application input fields, specifically testing authentication mechanics for injection flaws[cite: 5].</li>
        <li><strong>Exploitation</strong>: Exploiting SQL Injection to bypass authentication and retrieve restricted medical lab reports[cite: 6, 7].</li>
        <li><strong>Post-Exploitation &amp; Cracking</strong>: Downloading protected PDF assets and executing offline dictionary attacks using <code>pdfcrack</code> to recover document passwords[cite: 8, 9].</li>
    </ol>

    <hr>

    <h2>03. Findings and Proof of Exploitation</h2>

    <h3>Step 1: Initial Reconnaissance and Infrastructure Fingerprinting</h3>

    <h4>1. Host Connectivity and Environment Verification</h4>
    <p>Initial connectivity testing was performed using <code>ping google.com</code> to verify internet access and <code>whoami</code> to verify session privileges on Kali Linux[cite: 1].</p>
    <p><img src="Screenshot_2026-10-01_04_23_03.png" alt="Terminal Environment Setup">[cite: 2]</p>

    <h4>2. Domain Registration Reconnaissance (<code>whois</code>)</h4>
    <p>Executing <code>whois medirozahospital.com</code> returned no domain registration matches in the registry database[cite: 2].</p>
    <pre><code>whois medirozahospital.com</code></pre>
    <p><img src="Screenshot_2026-10-01_04_23_03.png" alt="WHOIS Query Result">[cite: 2]</p>

    <h4>3. Web Technology Fingerprinting (<code>whatweb</code>)</h4>
    <p>Executing <code>whatweb medirozahospital.com</code> revealed key web server architecture:</p>
    <ul>
        <li><strong>Target IP</strong>: <code>199.188.201.16</code>[cite: 3]</li>
        <li><strong>Web Server</strong>: <code>LiteSpeed</code>[cite: 3]</li>
        <li><strong>HTTP Status</strong>: <code>301 Moved Permanently</code> redirecting to HTTPS, followed by <code>403 Forbidden</code> on direct HTTP root requests[cite: 3].</li>
        <li><strong>Headers</strong>: <code>x-turbo-charged-by</code>[cite: 3]</li>
    </ul>
    <pre><code>whatweb medirozahospital.com</code></pre>
    <p><img src="Screenshot_2026-10-01_04_23_51.png" alt="WhatWeb Output">[cite: 3]</p>

    <hr>

    <h3>Step 2: Web Application Vulnerability Analysis &amp; Exploitation</h3>

    <h4>1. SQL Injection Error Detection</h4>
    <p>Navigating to <code>http://medirozahospital.com/patient/login.php</code> presented the <strong>Patient Portal</strong>[cite: 5]. Inputting a single quote <code>'</code> into the <strong>Username</strong> field triggered a raw MySQL error message[cite: 5]:</p>
    <blockquote>
        Warning: mysqli_query(): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''' at line 1[cite: 5]
    </blockquote>
    <p>This raw database exception confirms <strong>SQL Injection (SQLi)</strong>[cite: 5].</p>
    <p><img src="Screenshot_2026-10-01_06_52_34.png" alt="SQL Error Message Exposed">[cite: 5]</p>

    <h4>2. Authentication Bypass Exploitation</h4>
    <p>By supplying an authentication bypass payload into the <code>Username</code> field, the backend query condition was modified:</p>
    <ul>
        <li><strong>Username Payload</strong>: <code>admin' -- </code>[cite: 6]</li>
        <li><strong>Password</strong>: Any arbitrary string (e.g., <code>23232w2ds</code>)[cite: 6]</li>
    </ul>
    <p><img src="Screenshot_2026-10-01_06_52_51.png" alt="Authentication Bypass Payload">[cite: 6]</p>

    <p>Submitting this payload bypassed the login check without valid credentials, successfully loading the restricted page <code>/patient/portal.php</code> ("My lab reports")[cite: 7].</p>
    <p><img src="Screenshot_2026-10-01_06_53_01.png" alt="Successful Patient Portal Access">[cite: 7]</p>

    <p>The portal exposes downloadable, password-protected PDF patient files[cite: 7]:</p>
    <ol>
        <li><code>Pathology Report - S. Dlamini</code> (<code>Lab Ref LR-2024-1187</code>)[cite: 7]</li>
        <li><code>Pathology Report - P. Reddy</code> (<code>Lab Ref LR-2024-1192</code>)[cite: 7]</li>
        <li><code>Pathology Report - E. Thompson</code> (<code>Lab Ref LR-2024-1205</code>)[cite: 7]</li>
    </ol>

    <hr>

    <h3>Step 3: Document Security &amp; Offline Password Cracking</h3>

    <h4>1. Password Prompt on PDF Open</h4>
    <p>Downloading and attempting to view <code>patient_report_1.pdf</code> prompted a password requirement dialog[cite: 10].</p>
    <p><img src="Screenshot_2026-10-01_07_10_37.png" alt="PDF Password Prompt">[cite: 10]</p>

    <h4>2. Offline Dictionary Attack with <code>pdfcrack</code></h4>
    <p><code>pdfcrack</code> was installed and executed against <code>patient_report_1.pdf</code> using the <code>/usr/share/wordlists/rockyou.txt</code> wordlist[cite: 8, 9]:</p>
    <pre><code>sudo apt update &amp;&amp; sudo apt install pdfcrack
pdfcrack -f patient_report_1.pdf -w /usr/share/wordlists/rockyou.txt</code></pre>
    <p><img src="Screenshot_2026-10-01_07_10_20.png" alt="PDFCrack Tool Installation">[cite: 8]</p>

    <p>The tool successfully cracked the document encryption key within seconds[cite: 9]:</p>
    <pre><code>found user-password: '123456'</code></pre>
    <p><img src="Screenshot_2026-10-01_07_10_27.png" alt="PDF Password Cracked Successfully">[cite: 9]</p>

    <hr>

    <h2>04. Risk Rating</h2>

    <table>
        <thead>
            <tr>
                <th>Vulnerability Title</th>
                <th>Affected Component</th>
                <th>CVSS v3.1 Base Score</th>
                <th>Risk Rating</th>
                <th>Justification</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td><strong>SQL Injection (Authentication Bypass)</strong></td>
                <td><code>/patient/login.php</code>[cite: 5]</td>
                <td><strong>9.8</strong> (Critical)</td>
                <td><span class="badge badge-critical">CRITICAL</span></td>
                <td>Allows unauthenticated attackers to bypass security mechanisms[cite: 6], gain administrative or patient access[cite: 7], and read/modify database content.</td>
            </tr>
            <tr>
                <td><strong>Weak PDF File Encryption / Default Passwords</strong></td>
                <td>Patient Lab Reports[cite: 7, 9]</td>
                <td><strong>7.5</strong> (High)</td>
                <td><span class="badge badge-high">HIGH</span></td>
                <td>Protected medical files use predictable passwords (<code>123456</code>)[cite: 9], allowing unauthorized individuals to breach Protected Health Information (PHI) effortlessly[cite: 9].</td>
            </tr>
            <tr>
                <td><strong>Verbose Database Error Disclosure</strong></td>
                <td><code>/patient/login.php</code>[cite: 5]</td>
                <td><strong>5.3</strong> (Medium)</td>
                <td><span class="badge badge-medium">MEDIUM</span></td>
                <td>Raw MySQL error messages reveal backend database structure and driver details[cite: 5], significantly easing attack payload construction[cite: 6].</td>
            </tr>
            <tr>
                <td><strong>Server Technology Information Leakage</strong></td>
                <td>HTTP Headers / Web Server[cite: 3]</td>
                <td><strong>5.3</strong> (Low)</td>
                <td><span class="badge badge-low">LOW</span></td>
                <td>Exposing exact web server signatures (<code>LiteSpeed</code>)[cite: 3] assists attackers during initial fingerprinting[cite: 3].</td>
            </tr>
        </tbody>
    </table>

    <hr>

    <h2>05. Recommendations and Remediation</h2>

    <h3>1. Fix SQL Injection (Critical)</h3>
    <p><strong>Parameterized Queries</strong>: Replace raw dynamic SQL concatenations (<code>mysqli_query</code>)[cite: 5] with <strong>Prepared Statements</strong> using parameterized PDO or MySQLi queries.</p>
    <p><em>Remediation Example (PHP MySQLi)</em>:</p>
    <pre><code>$stmt = $conn-&gt;prepare("SELECT id, username FROM patients WHERE username = ? AND password = ?");
$stmt-&gt;bind_param("ss", $username, $password);
$stmt-&gt;execute();
$result = $stmt-&gt;get_result();</code></pre>

    <h3>2. Standardize Error Handling (Medium)</h3>
    <p><strong>Disable Verbose Errors</strong>: Disable display of raw SQL database errors in production (<code>display_errors = Off</code> in <code>php.ini</code>). Catch exceptions gracefully and show generic error messages to end-users[cite: 5].</p>

    <h3>3. Enforce Strong File Encryption Policies (High)</h3>
    <p><strong>Strong Password Generation</strong>: Abandon basic or sequential password schemes (e.g., <code>123456</code>) for PDF encryption[cite: 9].</p>
    <p><strong>Secure Access Controls</strong>: Implement high-entropy, dynamically generated passwords (or multi-factor verification) for patient PDF files sent via digital channels.</p>

    <h3>4. Hardening Web Server Headers (Low)</h3>
    <p><strong>Suppress Banners</strong>: Configure <code>LiteSpeed</code> to remove or obscure response headers like <code>Server</code> and <code>x-turbo-charged-by</code>[cite: 3].</p>
</div>

</body>
</html>
