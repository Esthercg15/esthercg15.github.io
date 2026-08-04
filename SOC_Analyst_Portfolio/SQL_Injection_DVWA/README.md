# 💉 SQL Injection (Boolean-Based) & Source Code Analysis in DVWA

> ⚠️ **Disclaimer:** This project was conducted in a controlled lab environment for educational and defensive research purposes only. 

## 🖥️ Lab Environment & Initial Setup
* **Target Machine:** Metasploitable (IP: `192.168.56.103`)[cite: 1].
* **Attacker Machine:** Kali Linux[cite: 1].
* **Application:** DVWA (Damn Vulnerable Web App) accessed via the Kali web browser[cite: 1].
* **Authentication:** Logged into the application using default credentials (`admin` / `password`)[cite: 1].

---

## 🔓 Phase 1: Exploitation (Low Security Level)

### Enumeration
* The DVWA security level was set to "low" to test the application in its most vulnerable state[cite: 1].
* Initial tests were conducted by providing standard user IDs (e.g., 1, 2, 3)[cite: 1].
* The web application returned standard error messages or user information, demonstrating how it behaves under normal conditions[cite: 1].

### Injection & Execution
* The boolean payload `' OR '1'='1` was injected into the ID field[cite: 1].
* The application concatenated this input without prior sanitization, resulting in an error[cite: 1].
* This lack of input validation allowed the query to bypass intended logic and dump user information[cite: 1].

### 🔍 Source Code Analysis (Whitebox)
Reviewing the underlying PHP code reveals exactly why the vulnerability exists and how the payload manipulates the database[cite: 1]:
* **The Escape (`'`):** Tricks the system by prematurely closing the text field, allowing everything written afterward to be interpreted as executable SQL commands rather than simple text[cite: 1].
* **The Injection (`UNION SELECT` / `OR`):** Alters the original query, forcing the database to merge the expected result with confidential information extracted from other tables (like users and passwords)[cite: 1].
* **The Patch (`#`):** Converts the remainder of the developer's original code into an ignored comment, preventing residual code from breaking the syntax and causing the attack to fail[cite: 1].

---

## 🛡️ Phase 2: Mitigation & SOC Detection (High Security Level)

To understand defensive mechanisms, the DVWA security mode was changed to "High" and the previous steps were repeated[cite: 1].

### Observations
* The same boolean payload (`' OR '1'='1`) was submitted[cite: 1].
* The application mitigated the attack silently[cite: 1].
* No data was returned, and the application did not reveal the database structure through error messages[cite: 1].
* The URL changed in the address bar, but this was due to the web browser encoding the URL, not because the server accepted the malicious attack[cite: 1].

### SOC Analyst Takeaways
* **Log Traces:** Even though the server successfully blocked the attack and the web page displayed no errors, this specific request leaves a trace in the server logs[cite: 1]. 
* **Detection Opportunities:** Blue teams can monitor these URL encoded payloads (`%3D`, `+OR+`) in HTTP access logs to detect and alert on active enumeration attempts against the infrastructure[cite: 1].
