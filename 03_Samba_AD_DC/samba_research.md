# Samba AD DC: Security Research & Assessment

Samba Active Directory Domain Controller (AD DC) is an open-source implementation of AD, commonly found in mixed Windows/Linux environments and enterprise infrastructures. This document outlines enumeration, tooling adaptation, and ACL analysis specific to Samba environments.

## 1. Samba Enumeration & Reconnaissance

Unlike Windows AD, Samba AD DC provides specific command-line tools for administration and enumeration.

**Using `samba-tool` (Native Linux):**
```bash
# List domain users
samba-tool user list -H /var/lib/samba/private/sam.ldb

# List domain groups
samba-tool group list -H /var/lib/samba/private/sam.ldb

# View group members (e.g., Domain Admins)
samba-tool group listmembers "Domain Admins" -H /var/lib/samba/private/sam.ldb
# Anonymous/Null binding check
ldapsearch -x -H ldap://<samba_dc_ip> -s base namingcontexts

# User enumeration (if credentials are known)
ldapsearch -x -H ldap://<samba_dc_ip> -D "<user>@<domain>" -w "<password>" -b "DC=domain,DC=local" "(objectClass=user)" sAMAccountName
