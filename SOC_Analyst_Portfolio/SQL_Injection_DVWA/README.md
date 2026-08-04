# SQL Injection Analysis: From Exploitation to SOC Detection (DVWA)

**Author:** [Tu Nombre]
**Date:** [Fecha]
**Objective:** To manually exploit a boolean-based SQL injection, analyze the underlying vulnerable PHP code, and document the detection artifacts left in the server logs from a Blue Team perspective.

---

## 1. Executive Summary
This practice demonstrates the lifecycle of a SQL Injection attack in a controlled environment (Metasploitable 2 / DVWA). I started by exploiting a lack of input sanitization to bypass authentication logic. Following the exploitation, I conducted a white-box analysis of the source code to understand the root cause. Finally, I tested the application under a hardened security level to observe how mitigation techniques affect the attacker's visibility and what traces are left for a Security Operations Center (SOC) to detect.

## 2. Infrastructure Setup
* **Attacker System:** Kali Linux
* **Target System:** Metasploitable (IP: 192.168.56.103)
* **Target Application:** Damn Vulnerable Web App (DVWA)

---

## 3. Phase 1: Vulnerability Exploitation (Low Security)

### The Approach
I configured the application to its lowest security setting to simulate a legacy or poorly configured web application. My initial enumeration consisted of testing valid user IDs (e.g., 1, 2) to observe the standard database response.

To test for SQL injection vulnerabilities, I injected a classic boolean payload: `' OR '1'='1`. 

### The Result
The application successfully processed the payload and dumped the information of multiple users. Because the database interprets `'1'='1'` as universally true, the query bypassed the intended ID restriction and returned all rows.

### Source Code Analysis (Root Cause)
By reviewing the PHP backend, I identified the exact flaw:
`$getid = "SELECT first_name, last_name FROM users WHERE user_id = '$id'";`

The variable `$id` is passed directly into the SQL query without any sanitization (like `mysqli_real_escape_string`) or parameterization. 
* The injected single quote (`'`) breaks out of the expected string.
* The `OR` operator alters the logic.
* The `#` symbol (if used) comments out the rest of the query, preventing syntax errors.

---

## 4. Phase 2: Mitigation and Detection (High Security)

### The Approach
To understand how security controls affect an attacker, I increased the DVWA security level to "High" and attempted the exact same boolean payload.

### The Result
The application mitigated the attack silently. No data was returned, and no SQL syntax errors were displayed on the screen. The application failed securely.

### SOC & Blue Team Observations
Although the attack failed on the front end, my analysis of the browser's behavior revealed critical information for defensive monitoring:

1. **URL Encoding:** The browser automatically encoded the payload in the address bar (e.g., converting spaces to `+` and `=` to `%3D`). The resulting parameter was `?id='+OR+'1'%3D'1`.
2. **Log Artifacts:** Even though the server dropped the malicious request, this encoded string is inevitably recorded in the web server's access logs (e.g., Apache `access.log`).
3. **Detection Strategy:** As a SOC Analyst, I would implement SIEM detection rules targeting specific URL-encoded SQLi indicators (like `%27` for quotes, `%3D` for equals, and `UNION`) within HTTP GET requests. Even failed attempts serve as high-fidelity alerts of active reconnaissance or exploitation attempts against our infrastructure.

---

## 5. Analyst Notes & Lessons Learned
*(Nota: Escribe aquí 2 o 3 líneas con tus propias palabras sobre lo que te ha parecido más interesante de la práctica. Por ejemplo: "This lab helped me bridge the gap between executing an attack and understanding how a SOC actually sees it in the logs...")*
