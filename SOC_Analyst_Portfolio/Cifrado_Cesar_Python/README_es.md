[ES Español](README_es.md)

# Writeup: Caesar Cipher Decryption & Brute Force (Python)

> **Disclaimer:** All projects and writeups in this repository are conducted in controlled, isolated laboratory environments for educational purposes, defensive research, and incident response training. 

## MITRE ATT&CK Mapping
*   **Tactics:** Credential Access, Defense Evasion.
*   **Techniques:**
    *   [T1140] Deobfuscate/Decode Files or Information
    *   [T1110] Brute Force

---

## 1. Attack Fundamentals

For the Caesar cipher, which uses a substitution algorithm, the optimal attack technique is brute force. This allows testing the entire space of possible keys. The logic involves trying different substitution numbers until the correct key is found. 

For the English alphabet, an attacker can try up to 26 different keys, where the key indicates the number of positions shifted. 
Example for the ciphertext "KROD":
* clave 1 → JQNC
* clave 2 → IPMB
* clave 3 → HOLA (makes sense)
* clave 4 → GNKZ

---

## 2. Environment Setup

First, a text editor (Emacs) is installed on the attacking machine to write the script.

![SOC Analyst - Install Emacs](SOC_Analyst_01_install_emacs.png)

Next, the script named `cifrado_cesar_fb.py` is opened.

![SOC Analyst - Create Script](SOC_Analyst_02_create_script.png)

This is the file where the code will be written.

![SOC Analyst - Emacs Interface](SOC_Analyst_03_emacs_interface.png)

---

## 3. Manual Decryption Algorithm

First, the baseline function for decrypting the text is defined. The `algoritmo_descifrado()` function takes a ciphertext and a key. It initializes an empty string (`texto_plano`) and, using a `for` loop, iterates through each character to apply the reverse shift. The `clave_descifrado` value determines the number of positions to step back.

![SOC Analyst - Define Function](SOC_Analyst_04_def_algorithm.png)

It is necessary to validate that each letter belongs to the English alphabet, which is imported from Python's `string` library.

![SOC Analyst - String Module](SOC_Analyst_05_string_module.png)

The script iterates through each character: if it does not belong to the alphabet, it is kept unchanged; if it is a letter, it retrieves its position, subtracts the key, and appends the resulting letter to `texto_plano`, returning the final message.

![SOC Analyst - Decryption Logic](SOC_Analyst_06_decryption_logic.png)

The program asks the user for the ciphertext and the key, converting the text to lowercase and the key to a numerical format.

![SOC Analyst - User Input](SOC_Analyst_07_user_input.png)

Finally, the `algoritmo_descifrado()` function is called, the result is stored in `texto_plano`, and it is displayed on the screen using `print()`.

![SOC Analyst - Main Execution](SOC_Analyst_08_main_execution.png)

Executing `python3 cifrado_cesar_fb.py` with the text "krod" and key 3 returns the perfectly decrypted text: "hola".

![SOC Analyst - Test Script](SOC_Analyst_09_test_script.png)

---

## 4. Automation and Brute Force (Part 2)

In this second phase, an additional function is programmed to automatically implement the brute force attack. To find the key that returns a meaningful text, language detection is required using the `langdetect` library.

![SOC Analyst - Langdetect](SOC_Analyst_10_langdetect.png)

This library is imported into the script to use the `detect` function.

![SOC Analyst - Import Langdetect](SOC_Analyst_11_import_langdetect.png)

The brute force function is added, which tests all possible keys using `range(len(ALFABETO))`. For each key, the text is decrypted, and `langdetect` checks if it is in Spanish ("es"). Upon detection, it displays the message and the key used, terminating the search with `return`.

![SOC Analyst - Brute Force Logic](SOC_Analyst_12_brute_force_function.png)

The code is updated to invoke this new `fuerza_bruta` function.

![SOC Analyst - Updated Main](SOC_Analyst_13_main_automation.png)

### 4.1. Final Validation

To validate the tool, a ciphertext is generated from `dcode.fr` using a key of 15. The resulting ciphertext is "thid th jc itmid epgp sthrxugpg".

![SOC Analyst - Payload Generation](SOC_Analyst_14_dcode_test_generation.png)

When executing the script against this text, the brute force attack works perfectly.

![SOC Analyst - Successful Automation](SOC_Analyst_15_automated_success.png)