[EN English](README.md)

# Writeup: SQL Injection & Source Code Analysis (DVWA)

> **Disclaimer:** Todos los proyectos y writeups en este repositorio se realizan en entornos de laboratorio controlados y aislados con fines educativos, de investigación defensiva y entrenamiento en respuesta a incidentes.

## Mapeo MITRE ATT&CK
*   **Tácticas:** Initial Access, Credential Access, Defense Evasion.
*   **Técnicas:**
    *   [T1190] Exploit Public-Facing Application
    *   [T1212] Exploitation for Credential Access
    *   [T1027] Obfuscated Files or Information (Indicator removal via URL Encoding)

---

## 1. Entorno y Preparación (Environment Setup)

El ejercicio comienza comprobando la configuración de red de la máquina víctima (Metasploitable) para obtener su dirección IP.

![SOC Analyst - Comprobación IP](SOC_Analyst_01_ifconfig.png)

Una vez confirmada la IP (`192.168.56.103`), se verifica el acceso al servidor web desde el navegador de la máquina atacante (Kali Linux).

![SOC Analyst - Acceso Navegador](SOC_Analyst_02_acceso_navegador.png)

Se accede al portal de Damn Vulnerable Web App (DVWA) introduciendo las credenciales por defecto: `admin` y `password`.

![SOC Analyst - Login DVWA](SOC_Analyst_03_login_dvwa.png)

---

## 2. Enumeración y Explotación (Low Security)

Para analizar la vulnerabilidad en su estado más crítico, se navega al menú "DVWA Security" y se configura el nivel de seguridad al nivel más bajo posible ("Low").

![SOC Analyst - Menú Seguridad](SOC_Analyst_04_menu_seguridad.png)
![SOC Analyst - Nivel Low](SOC_Analyst_05_nivel_low.png)

### 2.1. Comportamiento Base
El primer paso de la enumeración consiste en comprobar cómo se comporta el aplicativo al introducir IDs de usuarios válidos (como 1, 2, 3...). La aplicación web devuelve un mensaje de error o la información del usuario esperado.

![SOC Analyst - Enumeración ID 1](SOC_Analyst_06_enumeracion_id.png)

### 2.2. Inyección SQL (Boolean-Based)
A continuación, se prueba a inyectar una sentencia booleana clásica: `' OR '1'='1`. 
Como resultado, se obtiene un error favorable para el atacante porque la aplicación concatena todos los fallos sin sanitizar la entrada previamente, volcando la información de todos los usuarios de la base de datos.

![SOC Analyst - Inyección Exitosa](SOC_Analyst_07_inyeccion_exitosa.png)

---

## 3. Análisis de Código (White-box Analysis)

Para entender exactamente qué ha sucedido en el backend, se revisa el código fuente PHP del aplicativo web.

![SOC Analyst - Código Fuente PHP](SOC_Analyst_08_codigo_php.png)

La vulnerabilidad se resume en la interacción de tres elementos inyectados por el atacante:

1.  **La comilla (`'`):** Engaña al sistema cerrando prematuramente el campo de texto, permitiendo que todo lo que se escriba después sea interpretado como comandos SQL ejecutables y no como simples palabras.
2.  **La inyección (`UNION SELECT`):** Altera la consulta original obligando a la base de datos a fusionar el resultado esperado con información confidencial extraída de otras tablas (como usuarios y contraseñas).
3.  **El parche (`#`):** Convierte el resto del código original del desarrollador en un comentario ignorado, evitando así que queden restos de código suelto que rompan la sintaxis y hagan fallar el ataque.

---

## 4. Mitigación y Detección (High Security)

Finalmente, se procede a probar los mecanismos de defensa cambiando el modo de seguridad a "High" y volviendo a lanzar los pasos anteriores.

Con la consulta de las tres comillas o `' OR '1'='1`, la aplicación no arroja nada. Al cambiar el nivel de seguridad a High e intentar una inyección booleana tradicional por GET, la aplicación mitiga el ataque de forma silenciosa, no devolviendo datos ni revelando la estructura de la base de datos mediante errores.

![SOC Analyst - Rastro en URL](SOC_Analyst_09_rastro_url.png)

### 4.1. Perspectiva SOC (Detección de artefactos)
Aunque, según se observa en la barra de direcciones, sí se ha intentado la consulta. La URL cambia por culpa del navegador web, no porque el servidor haya aceptado el ataque. Esto significa que aunque el servidor haya bloqueado el ataque y la página no muestre ningún error, esta petición sí deja un rastro. Este rastro en la URL (`%27`, `%3D`) es un artefacto crítico que un analista SOC puede monitorizar en los registros del servidor web (Access Logs) para detectar intentos activos de explotación en la red.