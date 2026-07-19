# Skill: CTF Writeup Generation

> **Domain:** Documentation & Knowledge Sharing  
> **Difficulty:** All levels  
> **Prerequisites:** Markdown, clear communication, empathy for beginners  
> **Invoke Condition:** After flags captured and chain validated

---

## 1. Philosophy

> *"A writeup is not a log of commands — it's a story of discovery. Teach the reader to think, not just to copy."*

---

## 2. Writeup Structure

### 2.1 Executive Summary
```markdown
# [Box Name] - [Platform] ([Difficulty])

## TL;DR
- **Foothold:** [One-line summary of initial access]
- **User:** [One-line summary of user privilege]
- **Root:** [One-line summary of root privilege]
- **Key Techniques:** [List of 3-5 key skills used]

> **Time Taken:** X hours  
> **Rating:** X/10  
> **Recommended Prerequisites:** [List of boxes/rooms to do first]
```

### 2.2 Box Information
```markdown
## Box Info
| Property | Value |
|----------|-------|
| Name | [Box Name] |
| Platform | TryHackMe / HackTheBox |
| Difficulty | Easy / Medium / Hard / Insane |
| OS | Linux / Windows |
| IP | 10.10.10.X |
| Creator | [Creator Name] |
```

### 2.3 Reconnaissance
```markdown
## Reconnaissance

### Port Scan
```bash
# Show your nmap command and output
sudo nmap -p- --min-rate 10000 -oA nmap/full_tcp <target_ip>
```

### Key Findings
- Port 80: Nginx running Craft CMS
- Port 22: OpenSSH 8.9p1
- Virtual host: orion.htb

> **Beginner Note:** Explain WHY you chose these scan options. 
> "I used --min-rate 10000 because CTF networks typically don't have IDS/IPS, 
> so we can scan aggressively without worrying about detection."
```

### 2.4 Vulnerability Analysis
```markdown
## Vulnerability Analysis

### Finding 1: [Vulnerability Name]
- **Location:** [Where you found it]
- **Description:** [What it is]
- **Impact:** [What it allows]
- **CVE:** [If applicable]

### Discovery Process
> Show your THOUGHT PROCESS, not just commands.
> "I noticed the site was running Craft CMS. I checked the footer and saw 
> 'Powered by CraftCMS', which led me to research known vulnerabilities..."
```

### 2.5 Exploitation Steps
```markdown
## Exploitation

### Step 1: [Action]
```bash
# Show the command
# Show the output
```

> **Why this works:** Explain the underlying mechanism.
> "This payload works because Craft CMS's image transform endpoint 
> deserializes user input without proper validation, allowing us to 
> inject arbitrary PHP objects into the session file."

### Step 2: [Action]
```bash
# Continue with numbered steps
```

> **Failed Attempts:** Document what DIDN'T work and why.
> "I initially tried direct file upload to /admin, but the upload 
> endpoint required authentication. This led me to look for 
> unauthenticated entry points instead."
```

### 2.6 Post-Exploitation Chain
```markdown
## Post-Exploitation

### User Shell
```bash
whoami
# www-data
```

### Privilege Escalation
```bash
# Show your enumeration
# Show the vector you found
# Show the exploitation
```

> **Chain Visualization:**
```
[Web User] --> [Read Config] --> [Database Creds] --> [Crack Hash] --> [SSH as user]
    |
    +--> [SUID Binary] --> [Root Shell]
```
```

### 2.7 Flags
```markdown
## Flags

```
user.txt: [hash/flag]
root.txt: [hash/flag]
```
```

### 2.8 Beyond Root (Optional)
```markdown
## Beyond Root

### Unintended Paths
- [Alternative exploitation route you found]

### Lessons Learned
- [What this box taught you]
- [Techniques to remember for future boxes]

### Similar Boxes
- [Boxes with similar techniques]
```

---

## 3. Beginner-Friendly Guidelines

### 3.1 Explain the "Why"
```markdown
❌ BAD:
```bash
nmap -p- --min-rate 10000 10.10.10.10
```

✅ GOOD:
```bash
# Scan all 65535 TCP ports at high speed
# --min-rate 10000 ensures we send at least 10k packets/sec
# This is safe in CTF environments without IDS/IPS
sudo nmap -p- --min-rate 10000 10.10.10.10
```
```

### 3.2 Show Failed Attempts
```markdown
## Rabbit Holes

### Attempt 1: SQL Injection in Contact Form
I initially tried SQL injection in the contact form, but:
- The form submitted to /api/contact which returned 404
- No database interaction was actually happening
- This was a dead end

### Attempt 2: Brute Force Admin Login
I tried common credentials on /admin/login:
- admin:admin, admin:password, etc.
- All failed — the admin password was strong
- This confirmed I needed an unauthenticated vector
```

### 3.3 Build Intuition
```markdown
> **Pattern Recognition:** This box uses a similar technique to HTB: Surveillance 
> (Craft CMS exploitation), but with a different vulnerability vector. The key 
> difference is that Surveillance required authentication, while Orion uses an 
> unauthenticated image transform endpoint.
```

### 3.4 Visual Aids
```markdown
## Attack Flow

```
    [Attacker]
        |
        | HTTP Request to /index.php?p=admin/actions/assets/generate-transform
        |
        v
    [Craft CMS] --- Deserializes user input --> [Yii Framework]
        |                                        |
        | Session file poisoned                  | Object Injection
        v                                        v
    [PHP Session] <--- Include via LFI ------- [RCE Payload]
        |
        v
    [Reverse Shell as www-data]
        |
        v
    [Read /var/www/html/craft/.env] --> [Database Creds]
        |
        v
    [Dump Users Table] --> [Crack Bcrypt Hash]
        |
        v
    [SSH as orion]
        |
        v
    [inetd telnet auth bypass via USER env]
        |
        v
    [Root Shell]
```
```

### 3.5 Cite Sources
```markdown
## References
- [CVE-2023-41892](https://nvd.nist.gov/vuln/detail/CVE-2023-41892) - Craft CMS RCE
- [Yii Framework Object Injection](https://www.yiiframework.com/doc/guide/2.0/en/security-best-practices)
- [HTB: Surveillance Writeup](https://0xdf.gitlab.io/2024/01/13/htb-surveillance.html) - Similar Craft CMS box
```

---

## 4. Writeup Templates

### 4.1 Quick Template (Speed Run)
```markdown
# [Box] - [Difficulty]

## Recon
```bash
[nmap output]
```

## Foothold
[2-3 sentences + key command]

## User
[2-3 sentences + key command]

## Root
[2-3 sentences + key command]

## Flags
- user: [flag]
- root: [flag]
```

### 4.2 Detailed Template (Learning)
```markdown
# [Box] - [Difficulty] - Detailed Walkthrough

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [Reconnaissance](#reconnaissance)
3. [Vulnerability Analysis](#vulnerability-analysis)
4. [Exploitation](#exploitation)
5. [Post-Exploitation](#post-exploitation)
6. [Flags](#flags)
7. [Lessons Learned](#lessons-learned)

## Executive Summary
...

## Reconnaissance
### Host Discovery
### Web Enumeration
### Service Enumeration

## Vulnerability Analysis
### Finding 1
### Finding 2

## Exploitation
### Step-by-Step
### Failed Attempts

## Post-Exploitation
### User Shell
### Privilege Escalation
### Root Shell

## Flags

## Lessons Learned
```

---

## 5. Quality Checklist

```
□ All six pillars documented (Playbook, Discovery, Reproduction, Chain, Validation, Writeup)
□ Commands include explanations
□ Failed attempts documented with reasoning
□ Visual aids included (diagrams, screenshots)
□ CVEs and references cited
□ Alternative paths mentioned
□ Beginner-friendly language used
□ No copy-paste without context
□ Proof of concept is reproducible
□ Screenshots show key moments
```

---

*"The best writeup doesn't just show how to root a box — it teaches the reader how to root the next one."*
