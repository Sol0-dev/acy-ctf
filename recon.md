# Skill: Reconnaissance & Discovery

> **Domain:** Reconnaissance  
> **Difficulty:** All levels  
> **Prerequisites:** Basic networking, nmap, Linux CLI  
> **Invoke Condition:** Phase 1 of every CTF engagement

---

## 1. Philosophy

> *"You can't exploit what you don't know exists."*

Reconnaissance is the foundation of every successful CTF solve. A rushed recon phase leads to missed attack vectors, rabbit holes, and frustration. This skill enforces a systematic, repeatable approach to discovery.

---

## 2. Host Discovery

### 2.1 ICMP Ping Sweep
```bash
# Basic ping sweep
ping -c 1 <target_ip>

# Mass ping sweep for subnet
nmap -sn 10.10.10.0/24

# fping for faster results
fping -a -g 10.10.10.0/24 2>/dev/null
```

### 2.2 TCP Port Scanning
```bash
# Full TCP port scan (fast)
sudo nmap -p- --min-rate 10000 -oA nmap/full_tcp <target_ip>

# Top 1000 ports with service detection
sudo nmap -sCV -p- -oA nmap/all_ports <target_ip>

# SYN scan (stealthy)
sudo nmap -sS -p- <target_ip>

# Connect scan (no root needed)
nmap -sT -p- <target_ip>

# Aggressive scan (OS, version, script, traceroute)
sudo nmap -A -p- <target_ip>
```

**Beginner Note:** The `--min-rate 10000` flag tells nmap to send at least 10,000 packets per second. This speeds up scans significantly on CTF networks where IDS/IPS is typically not a concern.

### 2.3 UDP Port Scanning
```bash
# Top 100 UDP ports
sudo nmap -sU --top-ports 100 <target_ip>

# Full UDP scan (very slow)
sudo nmap -sU -p- <target_ip>

# Common UDP services to check manually
# DNS (53), SNMP (161), TFTP (69), NTP (123)
```

### 2.4 OS Fingerprinting
```bash
# OS detection
sudo nmap -O <target_ip>

# Check TTL in ping response
# Linux: TTL ~64, Windows: TTL ~128, Cisco: TTL ~255
ping -c 1 <target_ip> | grep ttl
```

---

## 3. Web Discovery

### 3.1 Technology Fingerprinting
```bash
# whatweb for tech stack identification
whatweb http://<target_ip>

# Wappalyzer browser extension (manual)
# Check HTTP headers for server info
curl -I http://<target_ip>

# Check for CMS indicators
# WordPress: /wp-admin, /wp-content
# Joomla: /administrator
# Drupal: /user/login
# Craft CMS: /admin, X-Powered-By: Craft CMS
```

### 3.2 Directory & File Bruteforce
```bash
# feroxbuster (recommended)
feroxbuster -u http://<target_ip> -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt

# ffuf for directories
ffuf -u http://<target_ip>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt

# ffuf for files
ffuf -u http://<target_ip>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt -e .php,.txt,.html,.bak

# Gobuster alternative
gobuster dir -u http://<target_ip> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

**Beginner Note:** Always check for backup files (`.bak`, `.old`, `.swp`, `~`) and source code (`.git`, `.svn`, `.hg`). These often contain credentials or reveal the application structure.

### 3.3 Subdomain Enumeration
```bash
# ffuf for virtual hosts
ffuf -u http://<target_ip> -H "Host: FUZZ.<domain>.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Check DNS records
dig axfr @<target_ip> <domain>.htb
dnsrecon -d <domain>.htb -t axfr

# Certificate transparency logs
curl -s "https://crt.sh/?q=%.<domain>.htb&output=json" | jq -r '.[].name_value' | sort -u
```

### 3.4 Parameter Discovery
```bash
# arjun for parameter discovery
arjun -u http://<target_ip>/page.php

# Check common parameters manually
# ?file=, ?page=, ?id=, ?url=, ?path=, ?cmd=
```

### 3.5 API Endpoint Mapping
```bash
# Check for swagger/openapi
/swagger/ui
/api/swagger
/openapi.json
/v2/api-docs

# Analyze JavaScript files for API endpoints
curl -s http://<target_ip>/assets/js/main.js | grep -oE '"/api/[a-zA-Z0-9/_-]*"'
```

---

## 4. Network Service Discovery

### 4.1 SMB Enumeration
```bash
# List shares
netexec smb <target_ip>
smbclient -L //<target_ip> -N

# Enumerate users
netexec smb <target_ip> --users
rpcclient -U "" <target_ip> -c "enumdomusers"

# Check for null sessions
smbmap -H <target_ip>

# Enumerate shares with creds
netexec smb <target_ip> -u <user> -p <pass> --shares
```

### 4.2 LDAP Enumeration
```bash
# Anonymous LDAP search
ldapsearch -x -H ldap://<target_ip> -b "dc=<domain>,dc=local"

# Enumerate domain info
ldapsearch -x -H ldap://<target_ip> -b "" -s base "(objectclass=*)"

# BloodHound data collection
bloodhound-python -u <user> -p <pass> -ns <target_ip> -d <domain>.local -c all
```

### 4.3 DNS Enumeration
```bash
# Zone transfer attempt
dig @<target_ip> <domain>.htb axfr

# DNS enumeration
dnsrecon -d <domain>.htb -t std,brt -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Reverse DNS lookup
dig -x <target_ip>
```

### 4.4 SNMP Enumeration
```bash
# Check for default community strings
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt <target_ip>

# SNMP walk
snmpwalk -v 2c -c public <target_ip>
snmpwalk -v 2c -c public <target_ip> 1.3.6.1.2.1.25.1.6.0  # System processes
snmpwalk -v 2c -c public <target_ip> 1.3.6.1.2.1.25.4.2.1.2  # Running programs
```

### 4.5 Database Service Detection
```bash
# MySQL
nmap -p 3306 --script mysql-info <target_ip>

# MSSQL
nmap -p 1433 --script ms-sql-info <target_ip>

# PostgreSQL
nmap -p 5432 --script pgsql-brute <target_ip>

# MongoDB
nmap -p 27017 --script mongodb-info <target_ip>
```

---

## 5. Credential Discovery

### 5.1 Source Code Leaks
```bash
# Git repository exposure
git-dumper http://<target_ip>/.git/ ./source_code

# Check for common config files
curl http://<target_ip>/.env
curl http://<target_ip>/config.php
curl http://<target_ip>/web.config

# Search for hardcoded credentials in source
grep -rni "password\|passwd\|pwd\|secret\|key" ./source_code/
```

### 5.2 Default Credentials
```bash
# Search for default creds
searchsploit <service_name> default credentials
# Check cirt.net/passwords
# Check https://github.com/ihebski/DefaultCreds-cheat-sheet
```

### 5.3 Credential Reuse Patterns
```bash
# Common patterns to look for:
# - Database config files (config.php, .env, database.yml)
# - Backup files (.bak, .old, .zip, .tar.gz)
# - Log files (.log, access.log, error.log)
# - History files (.bash_history, .mysql_history)
```

---

## 6. Recon Checklist

```
□ Host is reachable (ping)
□ All TCP ports scanned (nmap -p-)
□ Service versions identified (nmap -sCV)
□ OS fingerprinted (nmap -O)
□ UDP top ports checked
□ Web technology identified (whatweb)
□ Directories bruteforced (feroxbuster)
□ Subdomains enumerated (ffuf vhosts)
□ API endpoints mapped
□ SMB shares enumerated
□ LDAP queried
□ DNS records checked
□ SNMP community strings tested
□ Source code leaks checked (.git, .env)
□ Default credentials tested
□ Backup/old files checked
□ robots.txt and sitemap.xml checked
```

---

## 7. Common Rabbit Holes to Avoid

1. **Over-focusing on one port**: If HTTP seems like a dead end, check other services.
2. **Ignoring UDP**: SNMP and TFTP are common foothold vectors.
3. **Not checking version numbers**: Always match versions to known CVEs.
4. **Missing virtual hosts**: A single IP can host multiple applications.
5. **Not checking for backups**: `.git`, `.zip`, `.sql` files often contain the entire application.

---

*"Reconnaissance is not a phase — it's a continuous process that happens throughout the engagement."*
