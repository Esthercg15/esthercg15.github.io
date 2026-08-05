[ES Español](README_es.md)

# Writeup: SQL Injection & Source Code Analysis (DVWA)

> **Disclaimer:** All projects and writeups in this repository are conducted in controlled, isolated laboratory environments for educational purposes, defensive research, and incident response training. 

## MITRE ATT&CK Mapping
*   **Tactics:** Initial Access, Credential Access, Defense Evasion.
*   **Techniques:**
    *   [T1190] Exploit Public-Facing Application
    *   [T1212] Exploitation for Credential Access
    *   [T1027] Obfuscated Files or Information (Indicator removal via URL Encoding)

---

## 1. Environment Setup

The exercise begins by checking the network configuration of the victim machine (Metasploitable) to obtain its IP address.

![SOC Analyst - IP Check](SOC_Analyst_01_ifconfig.png)

Once the IP (`192.168.56.103`) is confirmed, web server access is verified from the attacker machine's browser (Kali Linux).

![SOC Analyst - Browser Access](SOC_Analyst_02_acceso_navegador.png)

The Damn Vulnerable Web App (DVWA) portal is accessed by entering the default credentials: `admin` and `password`.

![SOC Analyst - DVWA Login](SOC_Analyst_03_login_dvwa.png)

---

## 2. Enumeration and Exploitation (Low Security)

To analyze the vulnerability in its most critical state, navigate to the "DVWA Security" menu and set the security level to the lowest possible setting ("Low").

![SOC Analyst - Security Menu](SOC_Analyst_04_menu_seguridad.png)
![SOC Analyst - Low Level](SOC_Analyst_05_nivel_low.png)

### 2.1. Baseline Behavior
The first step in enumeration consists of checking how the application behaves when valid user IDs (such as 1, 2, 3...) are entered. The web application returns either an error message or the expected user information.

![SOC Analyst - ID 1 Enumeration](SOC_Analyst_06_enumeracion_id.png)

### 2.2. SQL Injection (Boolean-Based)
Next, a classic boolean statement is injected: `' OR '1'='1`. 
As a result, an error favorable to the attacker is obtained because the application concatenates all inputs without prior sanitization, dumping the information of all users in the database.

![SOC Analyst - Successful Injection](SOC_Analyst_07_inyeccion_exitosa.png)

---

## 3. Source Code Analysis (White-box)

To understand exactly what happened in the backend, the web application's PHP source code is reviewed.

![SOC Analyst - PHP Source Code](SOC_Analyst_08_codigo_php.png)

The vulnerability can be summarized by the interaction of three elements injected by the attacker:

1.  **The Single Quote (`'`):** Tricks the system by prematurely closing the text field, allowing everything written afterwards to be interpreted as executable SQL commands rather than simple strings.
2.  **The Injection (`UNION SELECT`):** Alters the original query, forcing the database to merge the expected result with sensitive information extracted from other tables (such as users and passwords).
3.  **The Patch (`#`):** Converts the rest of the developer's original code into an ignored comment, thereby preventing any loose code remnants from breaking the syntax and causing the attack to fail.

---

## 4. Mitigation and Detection (High Security)

Finally, the defense mechanisms are tested by changing the security mode to "High" and repeating the previous steps.

With the query of the three quotes or `' OR '1'='1`, the application returns nothing. When changing the security level to High and attempting a traditional boolean injection via GET, the application silently mitigates the attack, returning no data and revealing no database structure through errors.

![SOC Analyst - URL Trace](SOC_Analyst_09_rastro_url.png)

### 4.1. SOC Perspective (Artifact Detection)
However, as seen in the address bar, the query was indeed attempted. The URL changes due to the web browser, not because the server accepted the attack. This means that even though the server blocked the attack and the page displays no errors, this request does leave a trace. This trace in the URL (`%27`, `%3D`) is a critical artifact that a SOC analyst can monitor in the web server logs (Access Logs) to detect active exploitation attempts on the network.