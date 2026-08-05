# Main Repository Index

> **Disclaimer:** All projects and writeups in this repository are conducted in controlled, isolated laboratory environments for educational purposes, defensive research, and incident response training. 

## SOC Analyst & Penetration Testing Portfolio
![Kali accessing Metasploitable](images/image_23d16a.png)
This repository serves as a centralized documentation hub for cybersecurity laboratory exercises. Each directory contains a specific practice detailing the attack lifecycle, root cause analysis, and defensive mitigation strategies.
images/image_23d16a.png 

### Index of Practices

*   **Web Application Security**
    *   [DVWA: SQL Injection (Boolean-Based) & Source Code Analysis](./DVWA_SQL_Injection/README.md)
*   **Network Security**
    *   *(Future practices will be indexed here)*
*   **Incident Response & SIEM**
    *   *(Future practices will be indexed here)*

---
---

# SQL Injection Analysis: From Exploitation to SOC Detection (DVWA)

> **Disclaimer:** This project was conducted in a controlled lab environment for educational and defensive research purposes only. 

## 1. MITRE ATT&CK Mapping
*   **Tactics:** Initial Access, Credential Access, Defense Evasion.
*   **Techniques:**
    *   T1190: Exploit Public-Facing Application
    *   T1212: Exploitation for Credential Access
    *   T1027: Obfuscated Files or Information (Indicator removal via URL Encoding)

## 2. Environment
*   **Attacker System:** Kali Linux
*   **Target System:** Metasploitable (IP: 192.168.56.103)
*   **Target Application:** Damn Vulnerable Web App (DVWA)
*   **Authentication:** Default credentials (admin / password)

## 3. Enumeration
The application was configured to its lowest security setting to simulate a legacy or poorly configured web environment. Initial enumeration consisted of testing valid user IDs (e.g., 1, 2) in the input field to observe the standard database response and establish a baseline of normal application behavior. 

To test for SQL injection vulnerabilities, a classic boolean payload was formulated to manipulate the backend query structure.

## 4. Exploitation
A boolean payload (`' OR '1'='1`) was injected directly into the ID field. 

The application successfully processed the payload and dumped the database records of multiple users. Because the backend database interprets `'1'='1'` as universally true, the injected query successfully bypassed the intended ID restriction and returned all rows within the targeted table.

### Source Code Analysis (Root Cause)
A white-box review of the underlying PHP backend identified the exact implementation flaw:

```php
$getid = "SELECT first_name, last_name FROM users WHERE user_id = '$id'";


images/image_23d16a.png
