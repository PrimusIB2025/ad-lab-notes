# Active Directory Enumeration & Privilege Escalation

This document outlines the methodology for discovering Active Directory misconfigurations and escalating privileges. It follows the MITRE ATT&CK framework (Discovery & Privilege Escalation) and is intended for authorized lab environments and assessments.

## 1. Initial AD Enumeration & Reconnaissance
Before launching attacks, we must map the domain, trust relationships, and key assets.

**PowerShell (ActiveDirectory Module) / NetExec:**
```powershell
# Get basic domain information
Get-ADDomain
Get-ADForest

# Identify Domain Controllers
Get-ADDomainController -Filter *

# List all privileged groups (Tier 0 assets)
Get-ADGroupMember "Domain Admins"
Get-ADGroupMember "Enterprise Admins"
# Collect data for BloodHound (Run on compromised host)
.\SharpHound.exe -c All

# Analyze paths to Domain Admin, Kerberoastable users, and unconstrained delegation
