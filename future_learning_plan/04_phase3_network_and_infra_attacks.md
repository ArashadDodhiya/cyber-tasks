# 🏢 Phase 3: Network & Infrastructure Attacks (Months 8-11)

> **"Owning one box is a foothold. Owning the entire network is the mission."**

---

## 🎯 Objective

Master advanced network penetration, Active Directory domination, and multi-subnet pivoting. This is what separates a web hacker from a complete pentester.

---

## 📅 Timeline: 4 Months

| Week | Focus | Deliverable |
|---|---|---|
| 1-2 | Advanced Network Scanning & Enumeration | Custom enum scripts |
| 3-4 | Advanced Exploitation Techniques | Multi-stage exploit chains |
| 5-6 | Pivoting & Tunneling Mastery | Multi-hop pivot lab |
| 7-8 | Active Directory: Kerberos Deep Dive | Kerberoast + AS-REP + delegation attacks |
| 9-10 | AD: Lateral Movement & Persistence | Golden/Silver tickets, DCSync |
| 11-12 | AD: Forest & Trust Attacks | Cross-forest compromise |
| 13-14 | AD: Evasion & OPSEC | Stealth operations in AD |
| 15-16 | Full Network Pentests | 3+ complete network pentests |

---

## 1️⃣ Advanced Network Attacks

### Scanning & Enumeration — Beyond Basics

| Technique | Details |
|---|---|
| **Living off the Land scanning** | Using native OS tools when you can't upload scanners |
| **Firewall evasion scans** | Fragmented scans, decoy scans, source port manipulation |
| **Service fingerprinting** | Version detection, banner grabbing, protocol probing |
| **SNMP exploitation** | Community string brute-forcing, MIB walking |
| **SMB attacks** | Relay attacks, null sessions, share enumeration |
| **LDAP enumeration** | Unauthenticated bind, password spraying via LDAP |
| **NFS exploitation** | Mount exposed shares, UID spoofing |
| **RPC enumeration** | rpcclient, rpcinfo, endpoint mapper |

### Exercises

1. Perform a full network scan with firewall evasion techniques (Nmap: `-f`, `-D`, `--source-port`)
2. Enumerate SMB shares with null session and extract sensitive files
3. Perform SNMP walk with community string brute-force
4. Mount NFS share and exploit UID-based access
5. Use SMB relay attacks with Responder + ntlmrelayx

### Tools to Master

| Tool | Purpose |
|---|---|
| **Nmap** (advanced) | NSE scripts, timing, evasion |
| **CrackMapExec / NetExec** | Multi-protocol pentest tool |
| **Responder** | LLMNR/NBT-NS/mDNS poisoning |
| **ntlmrelayx** | NTLM relay attacks |
| **Impacket** | Python library for Windows protocols |
| **BloodHound** | AD attack path mapping |
| **Ligolo-ng** | Modern tunneling/pivoting |
| **Chisel** | TCP/UDP tunneling |

---

## 2️⃣ Pivoting & Tunneling Mastery

### Techniques to Master

| Technique | Tool | Use Case |
|---|---|---|
| **SSH Local Port Forward** | ssh | Access internal service through compromised host |
| **SSH Dynamic Port Forward** | ssh + proxychains | SOCKS proxy through compromised host |
| **SSH Remote Port Forward** | ssh | Expose internal service to your machine |
| **Ligolo-ng** | ligolo-ng | Modern agent-based tunneling (no SOCKS proxy needed) |
| **Chisel** | chisel | Reverse SOCKS proxy when SSH unavailable |
| **Meterpreter Routing** | Metasploit | Auto-route through meterpreter session |
| **Port forwarding via socat** | socat | Simple port relay on compromised host |
| **DNS Tunneling** | dnscat2, iodine | Exfil and C2 over DNS when only DNS is allowed |
| **ICMP Tunneling** | ptunnel | Tunnel over ICMP when all TCP/UDP is blocked |

### Lab Setup

Build a multi-subnet lab:

```
Attacker (Kali)
    │
    ├── Subnet 1 (DMZ): 10.10.10.0/24
    │   ├── Web Server (dual-homed)
    │   └── FTP Server
    │
    ├── Subnet 2 (Internal): 172.16.1.0/24
    │   ├── DC01 (Domain Controller)
    │   ├── Workstation01
    │   └── File Server
    │
    └── Subnet 3 (Database): 192.168.1.0/24
        ├── DB Server
        └── Backup Server
```

### Exercises

1. Set up Ligolo-ng and route traffic through a compromised DMZ host
2. Chain 3 pivots: Kali → DMZ → Internal → Database subnet
3. Set up Chisel reverse SOCKS and use proxychains for scanning
4. Tunnel through DNS only when TCP/UDP is blocked
5. Perform a full pentest across 3 subnets with proper pivoting

---

## 3️⃣ Active Directory — Deep Mastery

### Kerberos Attack Chain

| Attack | What It Does | Tools |
|---|---|---|
| **AS-REP Roasting** | Crack accounts without pre-authentication | GetNPUsers.py, Rubeus |
| **Kerberoasting** | Crack service account TGS tickets | GetUserSPNs.py, Rubeus |
| **Unconstrained Delegation** | Capture TGTs from connecting users | Rubeus monitor, SpoolSample |
| **Constrained Delegation** | Impersonate any user to specific service | Rubeus s4u |
| **Resource-Based Constrained Delegation** | Set delegation on computer object you control | PowerView, Rubeus |
| **Diamond Tickets** | Modify legitimate TGTs (stealthier than Golden) | Rubeus |
| **Golden Tickets** | Forge TGTs with KRBTGT hash | mimikatz, ticketer.py |
| **Silver Tickets** | Forge TGS for specific services | mimikatz, ticketer.py |
| **Skeleton Key** | Backdoor DC authentication | mimikatz |

### Lateral Movement Techniques

| Technique | Protocol | Detection Level |
|---|---|---|
| **PsExec** | SMB + RPC | 🔴 High (well-known) |
| **WMIExec** | WMI (DCOM) | 🟡 Medium |
| **AtExec** | Task Scheduler | 🟡 Medium |
| **SMBExec** | SMB only | 🟡 Medium |
| **WinRM** | HTTP/HTTPS | 🟢 Low |
| **DCOM** | Various COM objects | 🟢 Low |
| **Pass-the-Hash** | NTLM | 🟡 Medium |
| **Pass-the-Ticket** | Kerberos | 🟢 Low |
| **Overpass-the-Hash** | Kerberos from NTLM | 🟢 Low |

### Persistence Mechanisms

| Method | Where |
|---|---|
| **Golden Ticket** | Kerberos — forge TGTs forever |
| **DCSync** | Extract all domain hashes |
| **AdminSDHolder** | Persistent admin access |
| **SID History** | Hidden admin privileges |
| **Group Policy** | Scheduled tasks, logon scripts |
| **Machine Account Manipulation** | RBCD + computer account abuse |
| **Certificate Abuse (AD CS)** | ESC1-ESC8 for persistent access |

### AD Certificate Services (AD CS) Attacks

This is a HUGE and growing attack surface:

| Technique | Description |
|---|---|
| **ESC1** | Misconfigured certificate template — enrollee supplies SAN |
| **ESC2** | Any purpose / subordinate CA template |
| **ESC3** | Enrollment agent template abuse |
| **ESC4** | Template write permissions abuse |
| **ESC6** | EDITF_ATTRIBUTESUBJECTALTNAME2 flag on CA |
| **ESC7** | CA manager approval bypass |
| **ESC8** | NTLM relay to AD CS HTTP endpoint |

### Exercises

1. Perform full Kerberoasting attack chain: discover → request → crack → access
2. Exploit unconstrained delegation with SpoolSample
3. Achieve Domain Admin via RBCD attack
4. Create and use Golden Ticket
5. Perform DCSync and extract all domain hashes
6. Map entire AD environment with BloodHound — find shortest path to DA
7. Exploit AD CS ESC1 vulnerability
8. Chain: initial foothold → privesc → lateral movement → DA → persistence

### Resources

- 📖 **Attacking and Defending Active Directory** — Nikhil Mittal (Pentester Academy)
- 🎓 **HackTheBox: Pro Labs (Offshore, RastaLabs)**
- 🎓 **TryHackMe: Attacking Kerberos, AD Certificate Templates**
- 🔗 **The Hacker Recipes** — thehacker.recipes (AD section)
- 🔗 **ired.team** — Red teaming and AD notes
- 🔗 **SpecterOps** — BloodHound and AD research

---

## 4️⃣ Post-Exploitation Mastery

### What to Master

| Skill | Details |
|---|---|
| **Credential Harvesting** | SAM dump, LSA secrets, LSASS dump, DPAPI, Credential Manager |
| **Data Exfiltration** | SMB, HTTP, DNS, ICMP, steganography |
| **Situational Awareness** | Network mapping, trust mapping, sensitive data location |
| **Log Evasion** | Event log clearing, Sysmon bypass, timestomping |
| **Cleanup** | Artifact removal, access restoration |

---

## 📝 Phase 3 Completion Checklist

- [ ] Can perform advanced network scanning with firewall evasion
- [ ] Mastered SMB relay attacks with Responder + ntlmrelayx
- [ ] Can chain pivots through 3+ subnets using Ligolo-ng and Chisel
- [ ] Performed Kerberoasting, AS-REP roasting, delegation attacks
- [ ] Achieved Domain Admin through 3+ different attack paths
- [ ] Used BloodHound to map and exploit AD attack paths
- [ ] Created Golden/Silver tickets
- [ ] Performed DCSync
- [ ] Exploited AD CS (at least ESC1 and ESC8)
- [ ] Completed 3+ full network pentests (multi-subnet)

---

## 🔗 Platforms

| Platform | What to Do |
|---|---|
| **HackTheBox Pro Labs** | Offshore, RastaLabs, Zephyr |
| **TryHackMe** | AD attack rooms, pivoting rooms |
| **Proving Grounds** | OSCP-like practice environments |
| **GOAD (Game of Active Directory)** | Multi-forest AD lab (free) |
| **Custom Lab** | Build your own multi-subnet AD lab |

---

**Next: `05_phase4_cloud_offensive_security.md` →**
