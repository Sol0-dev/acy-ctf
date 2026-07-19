# CTF Agent Framework — AGENTS.md

> **Version:** 2.0.0  
> **Last Updated:** 2026-07-20  
> **Scope:** TryHackMe (Hard/Pro) & HackTheBox (Hard/Insane)  
> **Philosophy:** *"Every exploit is a hypothesis. Test it, chain it, validate it, document it."*

---

## 1. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     CTF MASTER AGENT                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │  PLAYBOOK   │  │  DISCOVERY  │  │      WRITEUP ENGINE     │  │
│  │   ENGINE    │  │   ENGINE    │  │  (Beginner-Friendly)    │  │
│  └──────┬──────┘  └──────┬──────┘  └─────────────────────────┘  │
│         │                │                                       │
│         └────────────────┼───────────────────────────────────────┘
│                          │
│         ┌────────────────┴────────────────┐
│         ▼                                 ▼
│  ┌─────────────┐              ┌──────────────────────┐
│  │   SKILL     │              │    SKILL ROUTER      │
│  │  INVOKER    │◄────────────►│ (Domain Detection &  │
│  │             │              │  Skill Dispatch)     │
│  └──────┬──────┘              └──────────────────────┘
│         │
│    ┌────┴────┬────────┬────────┬────────┬────────┬────────┐
│    ▼         ▼        ▼        ▼        ▼        ▼        ▼
│ ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐
│ │RECON│  │ WEB │  │ NET │  │LINPE│  │WINPE│  │  AD │  │VALID│
│ │     │  │EXP  │  │EXP  │  │     │  │     │  │     │  │     │
│ └─────┘  └─────┘  └─────┘  └─────┘  └─────┘  └─────┘  └─────┘
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. The Six Pillars

Every CTF engagement follows these six pillars, enforced by the Master Agent:

| Pillar | Description | Skill File |
|--------|-------------|------------|
| **Playbook** | Structured methodology from recon to root/flag | `skills/recon.md` |
| **Discovery** | Systematic enumeration and information gathering | `skills/recon.md` |
| **Reproduction** | Step-by-step exploit execution with commands | Domain-specific skills |
| **Chain** | Multi-hop privilege escalation and lateral movement | `skills/privesc_*.md`, `skills/ad.md` |
| **Validation** | Verify every assumption, exploit, and credential | `skills/validation.md` |
| **Writeup** | Beginner-friendly documentation with context | `skills/writeup.md` |

---

## 3. Workflow Engine

### Phase 0: Pre-Engagement Setup
```yaml
pre_engagement:
  - target: "<IP or URL>"
  - platform: "tryhackme | hackthebox | vulnlab | custom"
  - difficulty: "easy | medium | hard | insane"
  - scope: "user.txt + root.txt | flags | full domain compromise"
  - tooling_check:
      - nmap, masscan, rustscan
      - ffuf, feroxbuster, gobuster
      - burpsuite, wireshark
      - netexec, impacket, bloodhound
      - hashcat, john, cyberchef
      - python3, pwntools
  - environment_setup:
      - update /etc/hosts with target IP
      - configure VPN (HTB/THM)
      - start tmux/screen session
      - create engagement directory structure
```

### Phase 1: Discovery (Reconnaissance)
**Invoked Skill:** `skills/recon.md`

```yaml
discovery:
  host_discovery:
    - icmp_ping_sweep
    - tcp_port_scan: "nmap -p- --min-rate 10000 -oA full_tcp"
    - udp_port_scan: "nmap -sU --top-ports 100"
    - service_version: "nmap -sCV -p <ports>"
    - os_fingerprinting: "nmap -O"

  web_discovery:
    - technology_fingerprinting: "whatweb, wappalyzer"
    - directory_bruteforce: "feroxbuster -u http://target"
    - subdomain_enum: "ffuf -w subdomains.txt -u http://FUZZ.target.htb"
    - parameter_discovery: "arjun, paramminer"
    - api_endpoint_mapping: "swagger, openapi, js file analysis"

  network_discovery:
    - smb_enum: "netexec smb <target>"
    - ldap_enum: "ldapsearch, bloodyAD"
    - dns_enum: "dig, dnsrecon, nslookup"
    - snmp_enum: "onesixtyone, snmpwalk"

  credential_discovery:
    - default_creds: "searchsploit, cirt.net"
    - leaked_creds: "git-dumper, trufflehog"
    - config_files: "find . -name '*.config' -o -name '*.env'"
```

### Phase 2: Vulnerability Analysis
**Invoked Skills:** Domain-specific based on discovery results

```yaml
vulnerability_analysis:
  web_apps:
    - skill: "skills/web_exploitation.md"
    - triggers:
        - "HTTP/HTTPS services detected"
        - "CMS identified (WordPress, Craft, etc.)"
        - "API endpoints found"
        - "File upload functionality present"

  network_services:
    - skill: "skills/network_exploitation.md"
    - triggers:
        - "SMB, FTP, SSH, Telnet, SMTP open"
        - "Database services (MySQL, MSSQL, PostgreSQL, MongoDB)"
        - "Custom services on unusual ports"

  active_directory:
    - skill: "skills/active_directory.md"
    - triggers:
        - "Windows domain detected"
        - "LDAP, Kerberos, DNS on DC"
        - "BloodHound data shows attack paths"

  binary_exploitation:
    - skill: "skills/binary_exploitation.md"
    - triggers:
        - "Custom binaries with SUID"
        - "Buffer overflow opportunities"
        - "Reverse engineering required"
```

### Phase 3: Exploitation & Reproduction
**Invoked Skills:** Domain-specific

```yaml
exploitation:
  foothold:
    - identify_attack_vector
    - develop_exploit_hypothesis
    - test_hypothesis_safely:
        - "Use sleep, DNS callback, or echo first"
        - "Never fire blind RCE without verification"
    - execute_exploit
    - stabilize_shell:
        - "python3 -c 'import pty; pty.spawn("/bin/bash")'"
        - "stty raw -echo; fg"
        - "export TERM=xterm"

  reproduction_requirements:
    - every_command_documented
    - screenshots_of_key_steps
    - error_messages_recorded
    - alternative_paths_explored
```

### Phase 4: Post-Exploitation & Chaining
**Invoked Skills:** `skills/privilege_escalation_linux.md`, `skills/privilege_escalation_windows.md`, `skills/active_directory.md`

```yaml
post_exploitation:
  local_enumeration:
    - linux: "linpeas.sh, linenum.sh, pspy"
    - windows: "winpeas.exe, seatbelt, powersploit"

  credential_harvesting:
    - linux: "find / -name '*.bak' -o -name '*.old', .bash_history"
    - windows: "mimikatz, lazagne, dpapi"

  privilege_escalation:
    - identify_vector:
        - sudo permissions
        - SUID binaries
        - kernel exploits
        - service misconfigurations
        - scheduled tasks/cron jobs
    - validate_vector:
        - "Test with benign payload first"
        - "Understand the escalation mechanism"
    - execute_escalation

  lateral_movement:
    - pivot_through_users
    - password_spray_reuse
    - pass_the_hash
    - kerberoasting
    - constrained_delegation
```

### Phase 5: Validation
**Invoked Skill:** `skills/validation.md`

```yaml
validation:
  pre_exploit:
    - verify_target_reachable
    - confirm_vulnerability_exists
    - test_payload_harmlessly

  during_exploit:
    - confirm_shell_received
    - verify_user_context
    - check_network_connectivity_from_target

  post_exploit:
    - verify_flag_readable
    - confirm_privilege_level
    - test_persistence_mechanisms
    - validate_attack_chain_completeness

  documentation:
    - screenshot_every_step
    - log_all_commands_and_output
    - record_timestamps
    - note_dead_ends_and_why
```

### Phase 6: Writeup Generation
**Invoked Skill:** `skills/writeup.md`

```yaml
writeup:
  structure:
    - executive_summary
    - box_info_and_recon
    - vulnerability_analysis
    - exploitation_steps
    - post_exploitation_chain
    - flags_and_validation
    - beyond_root (optional)
    - lessons_learned

  style_guide:
    - beginner_friendly_language
    - explain_why_not_just_how
    - include_failed_attempts_and_reasoning
    - provide_alternative_paths
    - cite_CVEs_and_references
```

---

## 4. Skill Invocation Protocol

When the Master Agent detects a domain, it invokes the corresponding skill with context:

```python
def route_to_skill(discovery_results):
    """
    Skill Router: Analyzes discovery results and dispatches
    to the appropriate domain-specific skill.
    """
    context = {
        "target_ip": discovery_results.ip,
        "open_ports": discovery_results.ports,
        "services": discovery_results.services,
        "os_guess": discovery_results.os,
        "web_tech": discovery_results.web_technologies,
        "domain_info": discovery_results.domain
    }

    if "http" in context["services"] or "https" in context["services"]:
        invoke_skill("skills/web_exploitation.md", context)

    if context["os_guess"] == "Windows" and context["domain_info"]:
        invoke_skill("skills/active_directory.md", context)

    if "linux" in context["os_guess"].lower():
        invoke_skill("skills/privilege_escalation_linux.md", context)

    if "windows" in context["os_guess"].lower():
        invoke_skill("skills/privilege_escalation_windows.md", context)

    # Always validate
    invoke_skill("skills/validation.md", context)

    # Always generate writeup
    invoke_skill("skills/writeup.md", context)
```

---

## 5. Decision Trees

### 5.1 Web Application Attack Flow
```
HTTP/HTTPS Detected
        │
        ▼
┌─────────────────┐
│ Identify Tech   │──► CMS? ──► Check known CVEs (searchsploit, NVD)
│ Stack           │    Framework? ──► Check framework-specific vulns
└────────┬────────┘    Custom? ──► Source code review, fuzzing
         │
         ▼
┌─────────────────┐
│ Enumerate       │──► Directories (feroxbuster)
│ Endpoints       │    Parameters (arjun)
└────────┬────────┘    APIs (swagger, js analysis)
         │
         ▼
┌─────────────────┐
│ Test for        │──► SQLi ──► sqlmap, manual union-based
│ Common Vulns    │    XSS ──► Stored/Reflected/DOM
└────────┬────────┘    LFI/RFI ──► php filters, path traversal
         │              File Upload ──► Extension bypass, content-type
         │              SSRF ──► Internal service access
         │              Command Injection ──► ; | && $( )
         │              Deserialization ──► ysoserial, custom chains
         │              Auth Bypass ──► JWT forgery, session fixation
         ▼
┌─────────────────┐
│ Chain to        │──► Read config files ──► Database creds
│ Further Access  │    Upload webshell ──► Reverse shell
└─────────────────┘    SSRF to internal ──► New attack surface
```

### 5.2 Network Service Attack Flow
```
Non-HTTP Service Detected
        │
        ▼
┌─────────────────┐
│ Service Version │──► Search CVEs
│ Identification  │    Check default creds
└────────┬────────┘    Check for misconfigurations
         │
    ┌────┴────┬────────┬────────┬────────┐
    ▼         ▼        ▼        ▼        ▼
┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐
│ SMB  │  │ SSH  │  │ FTP  │  │ DB   │  │Custom│
└──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘
   │         │         │         │         │
   ▼         ▼         ▼         ▼         ▼
Null       Key       Anon      SQLi      Buffer
sessions   reuse     access    auth      overflow
shares     brute     upload    bypass    command
 RID       force     download  xp_cmd    injection
cycle                          shell
```

### 5.3 Privilege Escalation Decision Tree (Linux)
```
Low-priv Shell Obtained
        │
        ▼
┌─────────────────┐
│ Run linpeas.sh  │
│ or manual enum  │
└────────┬────────┘
         │
    ┌────┴────┬────────┬────────┬────────┬────────┐
    ▼         ▼        ▼        ▼        ▼        ▼
┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐
│SUDO  │  │SUID  │  │Kernel│  │Cron  │  │Path  │  │Docker│
│perms │  │bins  │  │exploit│  │jobs  │  │hijack│  │escape│
└──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘
   │         │         │         │         │         │
   ▼         ▼         ▼         ▼         ▼         ▼
GTFOBins  GTFOBins  searchsploit  Writable  PATH     privileged
LOLBAS    custom    kernel        cron      env      container
          exploit   version       script    variable escape
```

---

## 6. Quality Gates

Before proceeding to the next phase, the Master Agent validates:

| Gate | Check | Pass Criteria |
|------|-------|---------------|
| **G1** | Discovery Complete | All ports scanned, all services identified, OS guessed |
| **G2** | Attack Vector Validated | Exploit hypothesis tested with benign payload |
| **G3** | Foothold Confirmed | Stable shell obtained, `whoami` and `id` output captured |
| **G4** | Privesc Vector Found | At least one viable privilege escalation path identified |
| **G5** | Root/Flag Captured | Both user.txt and root.txt (or equivalent) obtained |
| **G6** | Writeup Complete | All six pillars documented with beginner-friendly explanations |

---

## 7. Common CTF Patterns (2026)

Based on analysis of recent Hard/Insane boxes from TryHackMe and HackTheBox:

### 7.1 Web → Foothold Patterns
| Pattern | Recent Examples | Key Skill |
|---------|-----------------|-----------|
| CMS RCE (Craft, WordPress, Camaleon) | HTB: Orion, Facts | `skills/web_exploitation.md` |
| Object Injection → Session Poisoning | HTB: Orion | `skills/web_exploitation.md` |
| File Upload Bypass → Webshell | HTB: Imagery, Guardian | `skills/web_exploitation.md` |
| SSRF → Internal Service Access | HTB: Browsed, Sorcery | `skills/web_exploitation.md` |
| JWT Forgery / Auth Bypass | HTB: Principal | `skills/web_exploitation.md` |
| SQL Injection → Credential Dump | HTB: CCTV, Gavel | `skills/web_exploitation.md` |
| XSLT Injection → File Write | HTB: Conversor | `skills/web_exploitation.md` |

### 7.2 Network → Foothold Patterns
| Pattern | Recent Examples | Key Skill |
|---------|-----------------|-----------|
| SMB Command Injection | HTB: Abducted | `skills/network_exploitation.md` |
| FTP → JAR/Source Code Leak | HTB: DevArea | `skills/network_exploitation.md` |
| SMTP → Phishing/Macro | HTB: Job, JobTwo | `skills/network_exploitation.md` |
| SNMP → Default Creds | HTB: AirTouch | `skills/network_exploitation.md` |

### 7.3 Privilege Escalation Patterns (Linux)
| Pattern | Recent Examples | Key Skill |
|---------|-----------------|-----------|
| CVE in System Service (snapd, needrestart) | HTB: Snapped, Conversor | `skills/privilege_escalation_linux.md` |
| Sudo Abuse (custom scripts, Python modules) | HTB: WingData, VariaType | `skills/privilege_escalation_linux.md` |
| Symlink / Path Traversal in Root Scripts | HTB: DevArea, Facts | `skills/privilege_escalation_linux.md` |
| Race Condition (snapd, systemd) | HTB: Snapped | `skills/privilege_escalation_linux.md` |
| Environment Variable Injection | HTB: Orion | `skills/privilege_escalation_linux.md` |

### 7.4 Active Directory Patterns
| Pattern | Recent Examples | Key Skill |
|---------|-----------------|-----------|
| Assume Breach → MSSQL → Linked Server | HTB: DarkZero, Signed, Eighteen | `skills/active_directory.md` |
| Kerberos Relay / RBCD | HTB: Bruno | `skills/active_directory.md` |
| ADCS ESC1-ESC13 | HTB: Mist | `skills/active_directory.md` |
| Bad Successor (dMSA) | HTB: Eighteen | `skills/active_directory.md` |
| Cross-Forest Trust Abuse | HTB: DarkZero | `skills/active_directory.md` |
| NTLM Relay + Coercion | HTB: Overwatch, Signed | `skills/active_directory.md` |

---

## 8. Tooling Matrix

| Category | Tools | Purpose |
|----------|-------|---------|
| **Port Scanning** | nmap, masscan, rustscan | Host and service discovery |
| **Web Enumeration** | ffuf, feroxbuster, gobuster, whatweb | Directory, subdomain, tech discovery |
| **Web Proxy** | Burp Suite, OWASP ZAP | Request interception and modification |
| **Network** | netexec, impacket, responder, bloodhound | AD/network exploitation |
| **Password Cracking** | hashcat, john, cyberchef | Credential recovery |
| **Reverse Shell** | pwntools, nc, socat | Shell stabilization and interaction |
| **Privilege Escalation** | linpeas, winpeas, pspy, seatbelt | Local enumeration |
| **Forensics** | wireshark, volatility, sleuthkit | Traffic and memory analysis |

---

## 9. Beginner-Friendly Guidelines

The Master Agent enforces these principles for all writeups:

1. **Explain the "Why"**: Every command should have a brief explanation of what it does and why it's being run.
2. **Show Failed Attempts**: Document rabbit holes and why they didn't work — this is often more educational than the successful path.
3. **Build Intuition**: Connect current techniques to previous boxes. "This is similar to HTB: Surveillance but with a twist..."
4. **Visual Aids**: Include ASCII diagrams, flowcharts, and screenshots where possible.
5. **Reference CVEs**: Always cite CVE numbers and link to original advisories.
6. **Alternative Paths**: Show unintended or alternative exploitation routes when discovered.
7. **Tool Alternatives**: Mention multiple tools that achieve the same goal.

---

## 10. Skill Files Index

| File | Domain | Description |
|------|--------|-------------|
| `skills/recon.md` | Reconnaissance | Host, web, network, and credential discovery |
| `skills/web_exploitation.md` | Web Apps | OWASP Top 10, CMS exploitation, API attacks |
| `skills/network_exploitation.md` | Network Services | SMB, FTP, SSH, SMTP, database attacks |
| `skills/privilege_escalation_linux.md` | Linux PrivEsc | SUDO, SUID, kernel, cron, Docker escapes |
| `skills/privilege_escalation_windows.md` | Windows PrivEsc | Services, registry, token abuse, scheduled tasks |
| `skills/active_directory.md` | Active Directory | Kerberos, LDAP, ADCS, RBCD, cross-forest |
| `skills/binary_exploitation.md` | Binary Pwn | Buffer overflow, ROP, reverse engineering |
| `skills/validation.md` | Validation | Verification protocols and quality gates |
| `skills/writeup.md` | Documentation | Beginner-friendly writeup generation |

---

*"The difference between a script kiddie and a hacker is the ability to explain why the exploit works."*
