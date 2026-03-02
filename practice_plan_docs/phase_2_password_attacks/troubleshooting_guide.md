# 🛠️ Password Attacks — Troubleshooting & Solutions Guide

> **Purpose**: A reference of common errors encountered during password cracking practice, with tested solutions.
> **Last Updated**: March 2, 2026

---

## 📋 Table of Contents

- [1. Hashcat Errors](#1-hashcat-errors)
  - [1.1 rockyou.txt Not Found](#11-rockyoutxt-not-found)
  - [1.2 Not Enough Allocatable Device Memory](#12-not-enough-allocatable-device-memory)
- [2. John the Ripper Errors](#2-john-the-ripper-errors)
  - [2.1 No Password Hashes Left to Crack](#21-no-password-hashes-left-to-crack)
  - [2.2 File Not Found for zip2john / ssh2john](#22-file-not-found-for-zip2john--ssh2john)
- [3. Hydra Errors](#3-hydra-errors)
  - [3.1 Invalid Option -H](#31-invalid-option--h)
  - [3.2 Optional Parameters Format Error](#32-optional-parameters-format-error)
  - [3.3 Unknown Service Error](#33-unknown-service-error)
  - [3.4 small_wordlist.txt Not Found](#34-small_wordlisttxt-not-found)
- [4. Quick Reference — Working Commands](#4-quick-reference--working-commands)
- [5. Practice Setup — Creating Files to Crack](#5-practice-setup--creating-files-to-crack)

---

## 1. Hashcat Errors

### 1.1 rockyou.txt Not Found

**Error:**
```
/usr/share/wordlists/rockyou.txt: No such file or directory
```

**Cause:** Kali Linux ships `rockyou.txt` compressed as `rockyou.txt.gz` to save disk space.

**Solution:**
```bash
# Decompress it (one-time setup)
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Verify it exists (~133 MB)
ls -lh /usr/share/wordlists/rockyou.txt
```

---

### 1.2 Not Enough Allocatable Device Memory

**Error:**
```
* Device #1: Not enough allocatable device memory for this attack.
```

**Cause:** Your VM doesn't have enough GPU/CPU memory (e.g., only 256 MB allocatable) to load the full rockyou.txt wordlist into hashcat's internal structures.

**Solution 1 — Use `-S` flag (slow candidates mode):**
```bash
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt -O --force -S
```
- `-S` processes the wordlist in smaller chunks, using far less memory.

**Solution 2 — Use optimized kernels + CPU only:**
```bash
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt -O --force -D 1
```
- `-O` = optimized kernels (less memory, limits password length to 31 chars)
- `-D 1` = forces CPU-only mode (best for VMs without GPU passthrough)
- `--force` = bypasses hardware warnings

**Solution 3 — Split the wordlist into smaller files:**
```bash
split -l 2000000 /usr/share/wordlists/rockyou.txt /tmp/rockyou_part_

# Then crack each part separately
hashcat -m 0 hash.txt /tmp/rockyou_part_aa -O --force
hashcat -m 0 hash.txt /tmp/rockyou_part_ab -O --force
# ... continue for _ac, _ad, etc.
```

**Solution 4 — Use John the Ripper instead:**
John runs on CPU with minimal memory and works perfectly in low-memory VMs.
```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

> 💡 **Best approach for VMs**: Use **John the Ripper** — it's lightweight and just works. Save hashcat for machines with real GPU access.

---

## 2. John the Ripper Errors

### 2.1 No Password Hashes Left to Crack

**Error:**
```
No password hashes left to crack (see FAQ)
```

**Cause:** John already cracked this hash in a previous run. It stores results in its **potfile** (`~/.john/john.pot`) and won't re-crack the same hash.

**Solution — View the cracked password:**
```bash
john --show --format=raw-md5 hash.txt
```

**To re-crack the same hash (for practice):**
```bash
# Clear the potfile
rm ~/.john/john.pot

# Now re-run the crack
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

---

### 2.2 File Not Found for zip2john / ssh2john

**Error:**
```
! protected.zip : No such file or directory
```

**Cause:** The target file (ZIP, SSH key, etc.) doesn't exist yet. You need to **create practice files** first.

**Solution:** See [Section 5 — Practice Setup](#5-practice-setup--creating-files-to-crack) below for creating practice files.

---

## 3. Hydra Errors

### 3.1 Invalid Option -H

**Error:**
```
hydra: invalid option -- 'H'
```

**Cause:** Hydra does **not** have a `-H` flag for headers. Headers must go **inside the form definition string** using `:H=`.

**Wrong ❌:**
```bash
hydra ... http-get-form "/path:params:error" -H "Cookie: value"
```

**Correct ✅:**
```bash
hydra ... http-get-form '/path:params:error:H=Cookie\: value'
```

---

### 3.2 Optional Parameters Format Error

**Error:**
```
[ERROR] optional parameters must have the format X=value
```
or
```
[ERROR] no valid optional parameter type given: F
```

**Cause:** Hydra is mis-parsing the colon-separated fields in the form string. Usually caused by:
- Spaces in the failure string confusing the parser
- Colons in cookie values not properly escaped
- Incorrect field order

**Solution — Use the correct format with escaped colons:**

The form string has this structure:
```
/path:form_data:failure_string:H=Header
  ①       ②          ③            ④
```

**Working command for DVWA Brute Force (Low Security):**
```bash
hydra -l admin -P ~/small_wordlist.txt 192.168.56.20 http-get-form \
  '/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Username and/or password incorrect.:H=Cookie\: PHPSESSID=YOUR_SESSION_ID; security=low' -V
```

**Key rules:**
- Escape the colon after `Cookie` with backslash: `Cookie\:`
- Put `PHPSESSID` right after `Cookie\:` (with a space)
- Use **single quotes** around the entire form string to prevent shell interpretation
- Add `-V` for verbose output to see what Hydra is doing
- The failure string should match exactly what appears on a failed login

---

### 3.3 Unknown Service Error

**Error:**
```
[ERROR] Unknown service: http-get-form://...
```

**Cause:** Using URL-style syntax `http-get-form://...` which Hydra doesn't accept.

**Wrong ❌:**
```bash
hydra ... "http-get-form://dvwa/path:params:error"
```

**Correct ✅:**
```bash
hydra ... http-get-form "/dvwa/path:params:error"
```

The service (`http-get-form`) and the form definition string are **separate arguments**.

---

### 3.4 small_wordlist.txt Not Found

**Error:**
```
[ERROR] File for passwords not found: /home/kali/small_wordlist.txt
```

**Cause:** The custom wordlist hasn't been created yet.

**Solution:**
```bash
cat > ~/small_wordlist.txt << 'EOF'
admin
password
password123
123456
letmein
welcome
monkey
dragon
master
qwerty
login
admin123
abc123
password1
iloveyou
EOF
```

---

## 4. Quick Reference — Working Commands

### Hash Cracking

```bash
# ─── MD5 with John (best for VMs) ───
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hash.txt
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --show --format=raw-md5 hash.txt

# ─── MD5 with Hashcat (needs GPU or enough memory) ───
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt -O --force -S

# ─── ZIP file password ───
zip2john protected.zip > zip_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
john --show zip_hash.txt

# ─── SSH key passphrase ───
ssh2john id_rsa > ssh_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt ssh_hash.txt
john --show ssh_hash.txt
```

### Online Brute Force

```bash
# ─── SSH brute force ───
hydra -l msfadmin -P ~/small_wordlist.txt 192.168.56.20 ssh -t 4

# ─── FTP brute force ───
hydra -l msfadmin -P ~/small_wordlist.txt 192.168.56.20 ftp -t 4

# ─── DVWA web form brute force (Low Security) ───
hydra -l admin -P ~/small_wordlist.txt 192.168.56.20 http-get-form \
  '/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Username and/or password incorrect.:H=Cookie\: PHPSESSID=YOUR_SESSION; security=low' -V
```

> ⚠️ **Remember**: Replace `YOUR_SESSION` with your actual PHPSESSID from the browser (Developer Tools → Console → type `document.cookie`).

---

## 5. Practice Setup — Creating Files to Crack

### Create an MD5 Hash

```bash
# Create a hash of a known password (for practice)
echo -n "password" | md5sum | cut -d' ' -f1 > hash.txt
cat hash.txt
# Output: 5f4dcc3b5aa765d61d8327deb882cf99
```

### Create a Password-Protected ZIP

```bash
# Step 1: Create a file to zip
echo "This is a secret message" > secret.txt

# Step 2: Create password-protected ZIP (set password like: password123)
zip -e protected.zip secret.txt

# Step 3: Verify it asks for password
unzip -l protected.zip
```

> 💡 **Tip**: Use a password that's in rockyou.txt (e.g., `password123`, `dragon`, `letmein`) so it will actually crack successfully.

### Create a Passphrase-Protected SSH Key

```bash
# Generate SSH key (set passphrase like: butterfly, shadow, letmein)
ssh-keygen -t rsa -b 2048 -f id_rsa
# Enter passphrase when prompted — use something in rockyou.txt
```

---

## 📝 Key Takeaways

| Lesson                                 | Details                                                           |
| -------------------------------------- | ----------------------------------------------------------------- |
| **rockyou.txt needs extraction**       | Run `sudo gunzip /usr/share/wordlists/rockyou.txt.gz` once        |
| **Use John over Hashcat in VMs**       | John uses CPU with minimal memory — ideal for VMs                 |
| **Hashcat `-S` flag**                  | Slow candidates mode — solves memory issues                       |
| **John's potfile**                     | Already cracked hashes are saved in `~/.john/john.pot`            |
| **Hydra headers go inside the string** | Use `:H=Cookie\: value` not `-H "Cookie: value"`                  |
| **Escape colons in Hydra**             | Cookie colon needs `\:` to avoid being treated as field separator |
| **Single quotes for Hydra**            | Prevents shell from interpreting special characters               |
| **Create practice files first**        | Generate test ZIPs and SSH keys before trying to crack them       |

---

> **Related**: [Password Attacks Practice Guide](./password_attacks.md)
