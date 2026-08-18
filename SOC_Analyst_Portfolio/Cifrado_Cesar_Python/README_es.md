[EN English](README.md)

# Writeup: Descifrado César y Fuerza Bruta (Python)

> **Disclaimer:** Todos los proyectos y writeups en este repositorio se realizan en entornos de laboratorio controlados y aislados con fines educativos, de investigación defensiva y entrenamiento en respuesta a incidentes.

## Mapeo MITRE ATT&CK
*   **Tácticas:** Credential Access, Defense Evasion.
*   **Técnicas:**
    *   [T1140] Deobfuscate/Decode Files or Information
    *   [T1110] Brute Force

---

## 1. Fundamentos del Ataque

Para el cifrado César, que utiliza un algoritmo de sustitución, la técnica de ataque óptima es la fuerza bruta. Esto permite probar todo el espacio de claves posibles. La secuencia lógica consiste en probar distintos números de sustitución hasta dar con la clave correcta.

Para el alfabeto inglés, se pueden probar hasta 26 claves distintas, donde la clave indica el número de posiciones desplazadas. 
Ejemplo para el texto cifrado "KROD":
* clave 1 → JQNC
* clave 2 → IPMB
* clave 3 → HOLA (tiene sentido)
* clave 4 → GNKZ

---

## 2. Preparación del Entorno

Se procede a instalar un editor de texto (Emacs) para programar el script en la máquina atacante.

![SOC Analyst - Instalar Emacs](SOC_Analyst_01_install_emacs.png)

A continuación, se abre el script al que se ha llamado `cifrado_cesar_fb.py`.

![SOC Analyst - Crear Script](SOC_Analyst_02_create_script.png)

Este es el fichero donde se va a programar el código.

![SOC Analyst - Interfaz Emacs](SOC_Analyst_03_emacs_interface.png)

---

## 3. Algoritmo de Descifrado Manual

En primer lugar, se definen las líneas base de la función que va a descifrar el texto. La función `algoritmo_descifrado()` recibe un texto cifrado y una clave. Inicializa una cadena vacía (`texto_plano`) y, mediante un bucle `for`, recorre cada carácter para aplicar el desplazamiento inverso correspondiente. El valor de `clave_descifrado` determinará el número de posiciones a retroceder.

![SOC Analyst - Definir Función](SOC_Analyst_04_def_algorithm.png)

Se debe validar que cada letra pertenece al alfabeto inglés, el cual se importará desde la librería `string` de Python.

![SOC Analyst - Módulo String](SOC_Analyst_05_string_module.png)

El script recorre cada carácter: si no pertenece al alfabeto, lo conserva; si es una letra, obtiene su posición, resta la clave y añade la letra resultante a `texto_plano`, devolviendo el mensaje final.

![SOC Analyst - Lógica de Descifrado](SOC_Analyst_06_decryption_logic.png)

El programa pide al usuario el mensaje cifrado y la clave, convirtiendo el texto a minúsculas y la clave a formato numérico.

![SOC Analyst - Entrada de Usuario](SOC_Analyst_07_user_input.png)

Finalmente, se llama a la función `algoritmo_descifrado()`, el resultado se guarda en `texto_plano` y se muestra en pantalla con `print()`.

![SOC Analyst - Ejecución Principal](SOC_Analyst_08_main_execution.png)

Al ejecutar `python3 cifrado_cesar_fb.py` con el texto "krod" y la clave 3, se obtiene el texto perfectamente descifrado: "hola".

![SOC Analyst - Prueba de Script](SOC_Analyst_09_test_script.png)

---

## 4. Automatización y Fuerza Bruta (2ª Parte)

En esta segunda fase, se programa una función adicional para implementar el ataque de fuerza bruta automáticamente. Para encontrar la clave que devuelva un texto con sentido, se requiere detectar el idioma usando la librería `langdetect`.

![SOC Analyst - Langdetect](SOC_Analyst_10_langdetect.png)

Se exporta esta librería en el script para importar la función `detect`.

![SOC Analyst - Importar Langdetect](SOC_Analyst_11_import_langdetect.png)

Se añade la función de fuerza bruta que prueba todas las claves posibles usando `range(len(ALFABETO))`. Para cada clave, se descifra el texto y `langdetect` comprueba si está en español ("es"). Al detectarlo, muestra el mensaje, la clave utilizada y termina la búsqueda con `return`.

![SOC Analyst - Lógica de Fuerza Bruta](SOC_Analyst_12_brute_force_function.png)

Se actualiza el código para invocar esta nueva función `fuerza_bruta`.

![SOC Analyst - Main Actualizado](SOC_Analyst_13_main_automation.png)

### 4.1. Validación Final

Para validar la herramienta, se crea un texto cifrado desde `dcode.fr` con una clave de 15. El texto cifrado resultante es "thid th jc itmid epgp sthrxugpg".

![SOC Analyst - Generación de Payload](SOC_Analyst_14_dcode_test_generation.png)

Al ejecutar el script contra este texto, el ataque de fuerza bruta funciona correctamente.

![SOC Analyst - Éxito de Automatización](SOC_Analyst_15_automated_success.png)