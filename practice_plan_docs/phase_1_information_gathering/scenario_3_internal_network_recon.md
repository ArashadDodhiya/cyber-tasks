# 🎯 Scenario 3: Internal Network Reconnaissance — Post-Breach Enumeration

> **Real-World Context**: You've gained initial access to a corporate network (via phishing, VPN creds, etc.).
> You're on a workstation with a 192.168.1.0/24 IP. Time to map the entire internal network.
> Goal: Find domain controllers, file shares, databases, and other high-value targets.

---

## 📖 The Storyline

You've compromised an employee workstation at "FinanceFirst Corp" during a penetration test.
You have a low-privilege shell. The client wants to know: *"How far can an attacker get from a single compromised workstation?"*

This is an **assumed breach** scenario — very common in modern pentests.

---

## 🔄 The Process (Step-by-Step)

### Phase A — Situational Awareness (Where Am I?)

#### Step 1: Local Machine Enumeration

```bash
# === On Linux ===
whoami                          # What user am I?
id                              # Groups and UID
hostname                        # Machine name
uname -a                       # OS and kernel version
ip addr show                    # Network interfaces and IPs
ip route                        # Default gateway and routes
cat /etc/resolv.conf            # DNS servers (often the DC!)
cat /etc/hosts                  # Static host entries
arp -a                          # Recently contacted hosts

# === On Windows ===
whoami /all                     # User, groups, privileges
hostname                        # Machine name
ipconfig /all                   # Full network config
route print                     # Routing table
arp -a                          # ARP cache
net config workstation          # Domain info
systeminfo                      # Full system information
```

**Critical info to extract**:
| Info Found                   | Why It Matters                                    |
| ---------------------------- | ------------------------------------------------- |
| Domain: `financefirst.local` | Now you know the AD domain name                   |
| DNS: `192.168.1.10`          | DNS server is almost always the Domain Controller |
| Gateway: `192.168.1.1`       | Network boundary device                           |
| Subnet: `255.255.255.0`      | Network size — 254 potential hosts                |
| Groups: `Domain Users`       | Your current access level                         |

---

### Phase B — Network Discovery (What's Around Me?)

#### Step 2: Host Discovery

```bash
# ARP scan (fastest for local subnet, very stealthy)
arp-scan -l
# or
nmap -sn -PR 192.168.1.0/24 -oA arp_discovery

# Ping sweep (may be blocked by firewalls)
nmap -sn 192.168.1.0/24 -oA ping_sweep

# If ICMP is blocked, try TCP discovery
nmap -sn -PS22,80,443,445 192.168.1.0/24 -oA tcp_discovery

# Discover multiple subnets (if routing allows)
nmap -sn 10.0.0.0/24 -oA other_subnet
nmap -sn 172.16.0.0/16 --min-rate 5000 -oA large_subnet
```

**Expected Discovery**:
```
192.168.1.1   — Gateway/Firewall
192.168.1.10  — Domain Controller (DC01)
192.168.1.11  — Second DC (DC02)
192.168.1.20  — File Server
192.168.1.30  — Database Server
192.168.1.50  — IT Admin Workstation
192.168.1.100-200 — Employee workstations
192.168.1.250 — Printer/MFP
```

#### Step 3: Port Scanning Critical Targets

```bash
# Quick scan on the probable Domain Controller
sudo nmap -sS -sV --top-ports 1000 192.168.1.10 -oA dc_scan

# Common ports on a Domain Controller:
# 53  — DNS
# 88  — Kerberos (CONFIRMS it's a DC!)
# 135 — RPC
# 139 — NetBIOS
# 389 — LDAP
# 445 — SMB
# 636 — LDAPS
# 3268 — Global Catalog
# 3389 — RDP

# Full port scan on high-value targets
sudo nmap -sS -p- --min-rate 5000 192.168.1.10 192.168.1.20 192.168.1.30 -oA hv_targets

# Service detection on open ports
sudo nmap -sV -sC -p 53,88,135,139,389,445,636,3268 192.168.1.10 -oA dc_services
```

---

### Phase C — Active Directory Enumeration

#### Step 4: Domain Enumeration (No Special Tools)

```bash
# From a Windows machine (even as low-priv user)
net user /domain                 # List all domain users
net group /domain                # List all domain groups
net group "Domain Admins" /domain  # Who are the admins?
net group "Enterprise Admins" /domain
net group "Domain Controllers" /domain
nltest /dclist:financefirst.local  # List all DCs
nslookup -type=srv _ldap._tcp.dc._msdcs.financefirst.local  # Find DCs via DNS
```

#### Step 5: SMB Enumeration (Goldmine for Info)

```bash
# Enumerate shares on file server
smbclient -L //192.168.1.20 -N           # Anonymous listing
smbclient -L //192.168.1.20 -U 'user%password'  # Authenticated

# enum4linux — automated SMB/RPC enumeration
enum4linux -a 192.168.1.10                # Full enum on DC

# CrackMapExec — modern Swiss Army knife
crackmapexec smb 192.168.1.0/24 --shares  # Enum shares across entire subnet
crackmapexec smb 192.168.1.10 --users      # Enum users
crackmapexec smb 192.168.1.10 --groups     # Enum groups
crackmapexec smb 192.168.1.10 --pass-pol   # Password policy (important!)

# Look for readable shares with sensitive data
smbclient //192.168.1.20/SharedDocs -U 'user%password'
# Then: ls, cd, get <filename>
```

**What to look for in shares**:
- `IT` or `Admin` folders — scripts with hardcoded creds
- `HR` folders — employee lists, org charts
- `Finance` folders — sensitive documents
- `Backup` folders — database dumps, config backups
- Any `.bat`, `.ps1`, `.vbs` files — often contain passwords

#### Step 6: LDAP Enumeration

```bash
# Query LDAP for all users
ldapsearch -x -H ldap://192.168.1.10 -b "DC=financefirst,DC=local" "(objectClass=user)" \
  sAMAccountName description memberOf

# Look for:
# - Users with "password" in description field (yes, this happens!)
# - Service accounts (often have weak/known passwords)
# - Users in high-privilege groups
# - Disabled accounts that still exist

# Using ldapdomaindump for pretty output
ldapdomaindump -u 'financefirst\user' -p 'password' 192.168.1.10 -o ldap_dump/
```

---

### Phase D — Service Enumeration

#### Step 7: Database Discovery

```bash
# Look for MSSQL
nmap -p 1433 --script ms-sql-info 192.168.1.0/24

# Look for MySQL
nmap -p 3306 --script mysql-info 192.168.1.0/24

# Look for PostgreSQL
nmap -p 5432 192.168.1.0/24

# CrackMapExec for MSSQL
crackmapexec mssql 192.168.1.0/24
```

#### Step 8: Look for Quick Wins

```bash
# Check for hosts vulnerable to EternalBlue (MS17-010)
nmap --script smb-vuln-ms17-010 -p 445 192.168.1.0/24

# Check for SMB signing disabled (relay attacks possible)
crackmapexec smb 192.168.1.0/24 --gen-relay-list nosigning.txt

# Check for LLMNR/NBT-NS (credentials via Responder)
# Just run and wait — systems will broadcast credentials
sudo responder -I eth0 -wrf

# Check for default credentials on web services
# Visit any discovered web portals (printers, admin panels, etc.)
```

---

## 📊 Network Map Template

After enumeration, build a map:

```
┌─────────────────────────────────────────────────────┐
│                  192.168.1.0/24                     │
│                                                     │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐         │
│  │   DC01   │   │   DC02   │   │ FileServ │         │
│  │ .10      │   │ .11      │   │ .20      │         │
│  │ Win2019  │   │ Win2019  │   │ Win2016  │         │
│  │ Kerberos │   │ Kerberos │   │ SMB/NFS  │         │
│  │ LDAP,DNS │   │ LDAP,DNS │   │ 5 shares │         │
│  └──────────┘   └──────────┘   └──────────┘         │
│                                                     │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐         │
│  │  DB01    │   │ WebApp01 │   │ ITAdmin  │         │
│  │ .30      │   │ .40      │   │ .50      │         │
│  │ Win2016  │   │ Linux    │   │ Win10    │         │
│  │ MSSQL    │   │ Apache   │   │ RDP open │         │
│  │ 1433     │   │ 80,443   │   │ Priv user│         │
│  └──────────┘   └──────────┘   └──────────┘         │
│                                                     │
│  ┌─────────────────────────────────────┐            │
│  │  Workstations (.100 - .200)         │            │
│  │  ~50 Windows 10 machines            │            │
│  │  SMB signing: DISABLED on most      │            │
│  └─────────────────────────────────────┘            │
└─────────────────────────────────────────────────────┘
```

---

## 🏠 Practice This Scenario At Home

| Real-World Step           | Practice Equivalent                               |
| ------------------------- | ------------------------------------------------- |
| Local machine enumeration | Run system info commands on your Kali VM          |
| Host discovery            | ARP scan your home lab subnet                     |
| Port scanning DCs         | Full Nmap scan on Metasploitable                  |
| SMB enumeration           | `enum4linux -a` and `smbclient` on Metasploitable |
| AD enumeration            | Set up a free Windows Server trial as a DC        |
| Service discovery         | Scan Metasploitable for all services on all ports |
| Full internal recon       | Treat your entire lab as a "corporate network"    |

### Recommended Lab Setup for This Scenario

```
Kali Linux (Attacker)          → 192.168.56.10
Metasploitable 2 (Linux srv)   → 192.168.56.20
Windows Server (DC - trial)    → 192.168.56.30  [Optional but valuable]
Windows 10 (Workstation)       → 192.168.56.40  [Optional]
```

> 💡 **Free Windows Server**: Microsoft offers 180-day evaluation ISOs at
> `https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server`

---

> **Key Takeaway**: Internal recon is about speed and stealth. You need to quickly identify the "crown jewels" (Domain Controllers, databases, file servers) while staying under the radar. In real engagements, you may only have minutes before the SOC detects you — know your commands by heart.
