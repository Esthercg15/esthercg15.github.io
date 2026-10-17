[🇪🇸 Español](README.es.md)

# FTP vs SFTP: Traffic Analysis with Wireshark and Detection with Wazuh

Lab exercise demonstrating why FTP is an insecure protocol, how SFTP solves the problem, and how a SIEM detects brute-force access attempts.

---

## MITRE ATT&CK Mapping

- **Tactics:** Credential Access, Discovery.
- **Techniques:**
  - [T1040] Network Sniffing (capture of plaintext credentials)
  - [T1110.001] Brute Force: Password Guessing
  - [T1078] Valid Accounts (use of system credentials on exposed services)

---

## 1. Environment Setup

The lab consists of three hosts on the same local network:

| Host | Role | IP |
|---|---|---|
| Windows 11 PC | FTP/SFTP client and Wireshark capture | `192.168.18.193` |
| Ubuntu (VirtualBox, bridged adapter) | FTP server (`vsftpd`) and SSH/SFTP, with Wazuh agent | `192.168.18.54` |
| Ubuntu Server | Wazuh 4.14.7 SIEM (manager, indexer and dashboard) | `192.168.18.113` |

The exercise begins by installing the FTP server on the virtual machine and checking that it is listening on port 21:

```bash
sudo apt install vsftpd
sudo ss -tlnp | grep :21
```

From the Windows PC, a Wireshark capture is started and the server is accessed with the command-line FTP client, since modern browsers no longer support `ftp://`:

```
ftp 192.168.18.54
ls
bye
```

---

## 2. FTP Credential Sniffing

Applying the `ftp` filter, Wireshark shows the full session between client and server, from the initial greeting to the disconnection.

![FTP session in Wireshark](img/01-ftp-paquetes.png)

Analysis of the control connection (port 21) reveals:

- **Server banner:** `220 (vsFTPd 3.0.5)` discloses the software and its version, useful information for looking up known vulnerabilities.
- **Plaintext credentials:** the `USER` and `PASS` commands expose the username and password without any encryption (the password is hidden in the image).
- **Login confirmation:** the `230 Login successful.` response confirms that the captured credentials are valid.
- **Client exposure:** in active mode, the `PORT 192,168,18,193,196,44` command reveals the client's IP address and the port where it waits for data (196 × 256 + 44 = `50220`).

---

## 3. Data Channel Analysis

In active mode, it is the server that opens the data connection to the client from its port 20. By following the TCP stream of the `FTP-DATA` packet, the file listing of the user's home directory is fully readable.

![Plaintext file listing](img/02-ftp-datos.png)

---

## 4. Mitigation: SFTP

To test the secure alternative, the exercise is repeated with SFTP, which runs over SSH on port 22 and requires no additional software: it is included in `openssh-server` and in the Windows OpenSSH client.

```
sftp esther@192.168.18.54
```

Applying the `ssh` filter and following the TCP stream, only the initial phase of the connection is readable:

![Encrypted SFTP session](img/03-sftp-flujo.png)

- **Version banners** from both endpoints (`OpenSSH_for_Windows_9.5` and `OpenSSH_10.2p1 Ubuntu`).
- **Algorithm negotiation:** key exchange (including the post-quantum hybrid `mlkem768x25519-sha256`), server host key (`ssh-ed25519`) and encryption (`chacha20-poly1305`, `aes256-gcm`).

After the key exchange, all traffic is encrypted: the username, password, commands and files are no longer readable.

| | FTP | SFTP |
|---|---|---|
| Credentials | Visible | Encrypted |
| Commands and files | Visible | Encrypted |
| Plaintext information | Entire session | Banner and negotiated algorithms |
| Ports | 21 and 20 | 22 |

---

## 5. Detection with Wazuh

For the detection phase, the Wazuh agent is installed on the virtual machine, pointing to the SIEM server:

```bash
sudo WAZUH_MANAGER='192.168.18.113' WAZUH_AGENT_NAME='Ubuntu-lab' dpkg -i ./wazuh-agent_4.14.7-1_amd64.deb
sudo systemctl enable --now wazuh-agent
```

Next, several SFTP connection attempts with incorrect passwords are made from the Windows PC in less than a minute.

### 5.1 Overview

Filtering by `agent.name:Ubuntu-lab`, the Threat Hunting dashboard shows 16 authentication failures and maps them to the MITRE ATT&CK techniques *Password Guessing*, *Brute Force* and *Valid Accounts*.

![Wazuh dashboard](img/04-wazuh-dashboard.png)

### 5.2 Event Analysis

The query `agent.name:Ubuntu-lab and rule.groups:authentication_failed` returns the details of each alert. The `data.srcip` field identifies the source of every attempt: `192.168.18.193`.

![Failed authentication events](img/05-wazuh-eventos.png)

Each attempt is logged by several system components, which produces several alerts per attempt:

| Rule | Level | Description | Log source |
|---|---|---|---|
| 5760 | 5 | `sshd: authentication failed` | SSH service |
| 5557 | 5 | `unix_chkpwd: Password check failed` | Local password check (no source IP) |
| 5503 | 5 | `PAM: User login failed` | PAM authentication system |
| **2502** | **10** | `User missed the password more than one time` | **Correlation of repeated failures** |

### 5.3 Correlation: Rule 2502

Wazuh does not just log isolated failures: when it detects repeated failures, it correlates them into a higher-severity alert (from level 5 to level 10), mapped to the *Brute Force* technique and the *Credential Access* tactic, and relevant for PCI DSS, GDPR, HIPAA and NIST 800-53 compliance.

![Rule 2502](img/06-wazuh-regla-2502.png)

---

## 6. Remediation

- Removal of the FTP server, which was not needed: `sudo apt purge vsftpd`.
- Change of the password exposed during testing.
- Removal of sensitive files from the home directory.

If a file transfer service had to be kept, the recommendations would be:

- Use **SFTP** or **FTPS** instead of FTP.
- Create a dedicated user without shell access, without reusing system credentials.
- Restrict each user to their own directory (`chroot_local_user=YES`) and disable anonymous access.
- Hide the version banner and restrict access by IP with `ufw`.
- Monitor access with a SIEM.

**Next step:** configure Wazuh active response to automatically block the source IP when a brute-force attack is detected.

---

## 7. Conclusions

- Analyzing a TCP session packet by packet (control and data connections, FTP active mode) turns the theory about insecure protocols into something tangible.
- SFTP encryption protects credentials, commands and data, although the software version and the negotiated algorithms remain visible.
- A SIEM adds value by correlating low-level events, identifying the source of an attack and putting it in context with MITRE ATT&CK.