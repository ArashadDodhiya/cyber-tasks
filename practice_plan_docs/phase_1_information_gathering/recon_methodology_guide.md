# 📋 Information Gathering — Recon Methodology & Process Guide

> This guide explains **HOW** professional pentesters approach reconnaissance systematically.
> It's the "thinking framework" behind the tools — use it with any scenario.

---

## 🧠 The Recon Mindset

### The 3 Rules of Professional Recon

1. **Document everything** — If you didn't write it down, it didn't happen
2. **Passive before active** — Gather maximum intel before touching the target
3. **Breadth before depth** — Map the full attack surface before diving deep into one area

### Recon is NOT Random Scanning

❌ Beginner approach: *"Let me nmap everything and see what happens"*
✅ Professional approach: *"Let me understand the target, plan my approach, then systematically enumerate"*

---

## 🔄 The 5-Stage Recon Framework

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   Stage 1          Stage 2          Stage 3             │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐          │
│  │ SCOPE &  │───▶│ PASSIVE  │───▶│ ACTIVE   │          │
│  │ PLANNING │    │ RECON    │    │ SCANNING │          │
│  └──────────┘    └──────────┘    └──────────┘          │
│                                       │                 │
│                                       ▼                 │
│                  Stage 5          Stage 4               │
│                ┌──────────┐    ┌──────────┐            │
│                │ REPORT & │◀───│ SERVICE  │            │
│                │ DOCUMENT │    │  ENUM    │            │
│                └──────────┘    └──────────┘            │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### Stage 1: Scope & Planning (15 min)

Before touching ANY tool, answer these questions:

| Question                                  | Why It Matters                                     |
| ----------------------------------------- | -------------------------------------------------- |
| What is in scope?                         | Avoid testing assets you don't have permission for |
| What type of test? (black/gray/white box) | Determines how much info you start with            |
| External or internal?                     | Completely different toolsets and approach         |
| Are there any restrictions?               | Some clients ban certain scan types or times       |
| What is the goal?                         | Find vulns? Prove access? Compliance check?        |
| What is the timeline?                     | Determines how thorough vs fast you need to be     |

**Create your notes structure**:
```
recon_notes/
├── scope.txt              # What you're allowed to test
├── passive_findings.md    # All passive recon results
├── active_findings.md     # All active scan results
├── subdomains.txt         # Discovered subdomains
├── live_hosts.txt         # Confirmed alive hosts
├── services.csv           # IP | Port | Service | Version
├── credentials.txt        # Any discovered credentials
├── screenshots/           # Web app screenshots
└── raw_scans/             # Nmap XML, Gobuster output, etc.
```

---

### Stage 2: Passive Reconnaissance (1-3 hours)

**Goal**: Learn everything possible without the target knowing.

#### Checklist:
```
☐ WHOIS lookup → registrant, registrar, nameservers
☐ DNS records → A, AAAA, MX, NS, TXT, CNAME, SOA
☐ Subdomain enumeration → crt.sh, subfinder, amass, theHarvester
☐ Google Dorking → files, login pages, directory listings
☐ Shodan/Censys → internet-facing services & versions
☐ Social media OSINT → employee names, tech stack hints
☐ GitHub/GitLab search → leaked code, config files, credentials
☐ Wayback Machine → historical pages, removed content
☐ Job postings → tech stack revealed in requirements
☐ Paste sites → leaked data mentioning target
```

#### Decision Tree After Passive Recon:
```
Found subdomains?
  ├── Yes → Resolve them, add to active scan targets
  └── No  → Try different wordlists, check Wayback

Found employee emails?
  ├── Yes → Note email format, check for credential leaks
  └── No  → Try LinkedIn OSINT, theHarvester with more sources

Found tech stack info?
  ├── Yes → Research known CVEs for those versions
  └── No  → Will discover during active scanning phase
```

---

### Stage 3: Active Scanning (1-2 hours)

**Goal**: Discover every open port and running service.

#### The Scanning Sequence:
```bash
# Step 1: Host discovery (find what's alive)
sudo nmap -sn <network_range> -oA 01_host_discovery

# Step 2: Quick port scan (top 1000 ports, fast results)
sudo nmap -sS --top-ports 1000 -iL live_hosts.txt -oA 02_quick_scan

# Step 3: Full port scan (all 65535 TCP ports)
sudo nmap -sS -p- --min-rate 5000 -iL live_hosts.txt -oA 03_full_tcp

# Step 4: Service/version detection (only on open ports)
sudo nmap -sV -sC -p <open_ports> -iL live_hosts.txt -oA 04_services

# Step 5: UDP scan (top 100 — because UDP is too slow for all)
sudo nmap -sU --top-ports 100 -iL live_hosts.txt -oA 05_udp

# Step 6: OS detection
sudo nmap -O -iL live_hosts.txt -oA 06_os_detect
```

> 💡 **Pro Tip**: Don't run `-A` (aggressive) on large networks. Break it into stages so you get fast results while comprehensive scans run in the background.

#### Organizing Scan Results:

Parse Nmap XML output into a usable spreadsheet:

```bash
# Extract open ports into a CSV
grep -E "open" 03_full_tcp.gnmap | awk '{print $2, $0}' | sort > open_ports_summary.txt

# Or use grep on the .nmap file
grep -E "^[0-9]+/(tcp|udp)" 04_services.nmap
```

---

### Stage 4: Service Enumeration (2-4 hours)

**Goal**: Deep-dive into each discovered service.

#### Service-Specific Checklist:

| Port(s)   | Service    | What to Enumerate                               |
| --------- | ---------- | ----------------------------------------------- |
| 21        | FTP        | Anonymous login? Version? Writable directories? |
| 22        | SSH        | Version? Auth methods? Banner?                  |
| 25/587    | SMTP       | VRFY users? Open relay? Version?                |
| 53        | DNS        | Zone transfer? Subdomain brute? Recursive?      |
| 80/443    | HTTP/S     | Directories, tech stack, vulns, APIs, headers   |
| 88        | Kerberos   | Domain Controller confirmed. AS-REP roast?      |
| 110/143   | POP3/IMAP  | Version? Default creds?                         |
| 135/445   | RPC/SMB    | Shares, users, groups, password policy, vulns   |
| 161       | SNMP       | Community strings? System info? User enum?      |
| 389/636   | LDAP/S     | Domain users, groups, OUs, SPNs                 |
| 1433      | MSSQL      | Default SA creds? xp_cmdshell?                  |
| 3306      | MySQL      | Anonymous login? Version?                       |
| 3389      | RDP        | NLA enabled? BlueKeep vulnerable?               |
| 5432      | PostgreSQL | Default creds? Version?                         |
| 5985      | WinRM      | Can you get a shell with valid creds?           |
| 8080/8443 | Web Proxy  | Alt web services, admin panels, Jenkins, Tomcat |

---

### Stage 5: Report & Documentation (30 min - 1 hour)

**Goal**: Organize findings into actionable intelligence.

#### The Output:
```markdown
# Reconnaissance Summary

## Scope
- Target: [name]
- Type: [external/internal]
- Date: [date]

## Attack Surface Summary
- Subdomains discovered: XX
- Live hosts: XX
- Open ports: XX
- Web applications: XX
- Users identified: XX

## High-Value Targets
| Host       | IP           | Services                | Priority | Notes              |
| ---------- | ------------ | ----------------------- | -------- | ------------------ |
| DC01       | 192.168.1.10 | DNS,LDAP,Kerberos,SMB   | CRITICAL | Domain Controller  |
| FileServer | 192.168.1.20 | SMB (5 shares readable) | HIGH     | Sensitive data     |
| WebApp     | 192.168.1.40 | HTTP (Apache 2.2.15)    | HIGH     | Outdated version   |
| DB01       | 192.168.1.30 | MSSQL 2014              | HIGH     | Check for SA creds |

## Next Steps (For Exploitation Phase)
1. Attempt credential spray against discovered users
2. Test Apache 2.2.15 for known CVEs
3. Enumerate readable SMB shares for credentials
4. Run Responder for LLMNR/NBT-NS poisoning
```

---

## ⏱️ Time Management Template

| Phase                 | External Test | Internal Test | Bug Bounty  |
| --------------------- | ------------- | ------------- | ----------- |
| Scope & Planning      | 15 min        | 15 min        | 10 min      |
| Passive Recon         | 2-3 hrs       | 30 min        | 2-4 hrs     |
| Active Scanning       | 1-2 hrs       | 1-2 hrs       | 1-2 hrs     |
| Service Enumeration   | 2-3 hrs       | 2-4 hrs       | 2-3 hrs     |
| Reporting/Documenting | 1 hr          | 1 hr          | 30 min      |
| **Total**             | **6-9 hrs**   | **4-7 hrs**   | **5-9 hrs** |

---

## 🛑 Common Mistakes to Avoid

| Mistake                         | Why It's Bad                                       | Fix                                  |
| ------------------------------- | -------------------------------------------------- | ------------------------------------ |
| Scanning without permission     | Illegal. You will go to jail.                      | Always have written authorization    |
| Running `nmap -A` on everything | Too slow, too noisy, wastes time                   | Break into staged scans              |
| Not saving scan output          | Can't reproduce, can't include in report           | Always use `-oA` flag                |
| Ignoring UDP                    | Misses SNMP, DNS, TFTP, NTP                        | Always scan top UDP ports            |
| Only scanning top 1000 ports    | Misses services on high ports (8080, 8443, etc.)   | Always do a full `-p-` scan          |
| Not checking SSL certs          | Misses alternative hostnames, expiry, weak ciphers | Always check certificates            |
| Skipping passive recon          | Miss subdomains, employees, leaked creds           | Always passive before active         |
| Not organizing notes            | Can't find what you found, report takes forever    | Use structured folder from the start |

---

> **Key Takeaway**: Methodology beats tools. A junior with a solid methodology will outperform a senior randomly running tools. Follow the 5-stage framework, adapt it to your scenario, and always document as you go.
