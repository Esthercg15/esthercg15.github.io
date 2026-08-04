# SQL Injection Analysis: From Exploitation to SOC Detection (DVWA)

## 1. Executive Summary
This practice demonstrates the lifecycle of a SQL Injection attack in a controlled environment. The exercise began by exploiting a lack of input sanitization to bypass authentication logic in a deliberately vulnerable web application. Following the exploitation, a white-box analysis of the source code was conducted to understand the root cause of the vulnerability. Finally, the application was tested under a hardened security level to observe how mitigation techniques affect the attacker's visibility and what artifacts are left behind for a Security Operations Center (SOC) to analyze.

## 2. Infrastructure Setup
* **Attacker System:** Kali Linux
* **Target System:** Metasploitable (IP: 192.168.56.103)
* **Target Application:** Damn Vulnerable Web App (DVWA)

---

## 3. Phase 1: Vulnerability Exploitation (Low Security)

### The Approach
The application was configured to its lowest security setting to simulate a legacy or poorly configured web environment. Initial enumeration consisted of testing valid user IDs (e.g., 1, 2) to observe the standard database response and establish a baseline of normal application behavior.

To test for SQL injection vulnerabilities, a classic boolean payload was injected into the ID field: `' OR '1'='1`. 

### The Result
The application successfully processed the payload and dumped the database records of multiple users. Because the backend database interprets `'1'='1'` as universally true, the injected query successfully bypassed the intended ID restriction and returned all rows within the targeted table.

### Source Code Analysis (Root Cause)
A review of the underlying PHP backend identified the exact implementation flaw:

```php
$getid = "SELECT first_name, last_name FROM users WHERE user_id = '$id'";

