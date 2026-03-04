# Challenge 26: Passive Information Collection

> **Difficulty:** 🟢 EASY | **Success Ratio:** 100%
> **Goal:** Gather information about a target without directly interacting with their systems.

---

## What They Test

Can you collect valuable reconnaissance data about a target using only **passive** techniques — methods that do NOT directly touch or send requests to the target's infrastructure?

---

## Key Concept

### Passive vs Active Reconnaissance

| Type        | Description                      | Example                    | Detectable? |
| ----------- | -------------------------------- | -------------------------- | ----------- |
| **Passive** | Using third-party data/services  | WHOIS, Google, Shodan      | ❌ No        |
| **Active**  | Directly interacting with target | Nmap, Nikto, brute forcing | ✅ Yes       |

In passive recon, the target **never knows you're investigating them**.

---

## Methodology

### Step 1: WHOIS Lookup

```bash
# Domain registration info
whois target.com

# What to extract:
# - Registrant name, email, phone
# - DNS servers
# - Registration/expiration dates
# - Registrar info

# If WHOIS is protected (privacy guard), try:
# - Historical WHOIS records
# - https://whoisology.com
# - https://domainbigdata.com
```

---

### Step 2: DNS Enumeration

```bash
# All DNS records
dig target.com ANY
dig target.com ANY +short

# Specific record types
dig target.com A          # IPv4 address
dig target.com AAAA       # IPv6 address
dig target.com MX         # Mail servers
dig target.com NS         # Name servers
dig target.com TXT        # TXT records (SPF, DKIM, verification)
dig target.com CNAME      # Aliases
dig target.com SOA        # Start of Authority

# nslookup alternative
nslookup -type=any target.com
nslookup -type=mx target.com
nslookup -type=txt target.com

# Reverse DNS lookup (IP → hostname)
dig -x 1.2.3.4
nslookup 1.2.3.4

# DNS zone transfer (sometimes misconfigured)
dig axfr target.com @ns1.target.com
# If successful → dumps ALL DNS records!
```

#### TXT Records Can Reveal
```
- SPF records → email infrastructure
- DKIM → email security
- Google/Microsoft verification → services used
- API keys accidentally left in TXT records
- Cloud service verification tokens
```

---

### Step 3: Subdomain Enumeration (Passive Only)

```bash
# subfinder (fast, passive)
subfinder -d target.com -silent
subfinder -d target.com -o subdomains.txt

# amass (comprehensive, passive)
amass enum -passive -d target.com
amass enum -passive -d target.com -o amass_results.txt

# assetfinder
assetfinder target.com
assetfinder --subs-only target.com

# Online sources (browser):
# Certificate Transparency
https://crt.sh/?q=%.target.com

# VirusTotal
https://www.virustotal.com/gui/domain/target.com/relations

# Sublist3r
sublist3r -d target.com

# DNSDumpster
https://dnsdumpster.com/

# SecurityTrails
https://securitytrails.com/domain/target.com/dns
```

---

### Step 4: Google Dorking (Google Hacking)

```
# Basic searches
site:target.com                           # All indexed pages
site:target.com inurl:admin               # Admin pages
site:target.com inurl:login               # Login pages
site:target.com intitle:"index of"        # Directory listings
site:target.com filetype:pdf              # PDF documents
site:target.com filetype:doc              # Word documents
site:target.com filetype:xls              # Excel spreadsheets
site:target.com filetype:sql              # SQL dumps
site:target.com filetype:log              # Log files
site:target.com filetype:bak              # Backup files
site:target.com filetype:xml              # XML files
site:target.com filetype:conf             # Configuration files
site:target.com filetype:env              # Environment files

# Finding sensitive information
site:target.com "password"
site:target.com "username" "password"
site:target.com "api_key" OR "apikey"
site:target.com "secret_key"
site:target.com "BEGIN RSA PRIVATE KEY"
site:target.com "CONNECT TO" filetype:sql
site:target.com ext:php intitle:phpinfo()
site:target.com inurl:wp-config.php

# Finding exposed services
site:target.com inurl:jenkins
site:target.com inurl:gitlab
site:target.com inurl:jira
site:target.com inurl:confluence
site:target.com intitle:"Dashboard [Jenkins]"

# Finding error pages (information disclosure)
site:target.com "error" | "exception" | "stack trace"
site:target.com "Fatal error" | "Warning:" | "mysql_"

# Combine operators
site:target.com -www              # Subdomains only (exclude www)
site:*.target.com                 # All subdomains
```

---

### Step 5: Shodan / Censys (Internet-Wide Scanning Data)

```bash
# Shodan (search for target's devices/servers)
# https://www.shodan.io

# Shodan CLI
shodan search hostname:target.com
shodan search "target.com"
shodan host 1.2.3.4              # Info about specific IP

# What Shodan reveals:
# - Open ports and services
# - SSL certificate details
# - Software versions
# - Vulnerabilities (CVEs)
# - Screenshots of web services
# - IoT devices

# Censys (alternative)
# https://search.censys.io
# Search for domain or IP → see certificates, open ports

# FOFA (Chinese alternative)
# https://fofa.info
```

---

### Step 6: Email & People OSINT

```bash
# theHarvester (emails, names, subdomains)
theHarvester -d target.com -b google
theHarvester -d target.com -b linkedin
theHarvester -d target.com -b google,bing,linkedin,twitter,yahoo

# hunter.io (email addresses)
# https://hunter.io/search/target.com
# Find email patterns: firstname.lastname@target.com

# Phonebook.cz
# https://phonebook.cz/
# Search for emails, domains, URLs

# LinkedIn reconnaissance
# Search for: "target.com" employees
# Look for: job titles, technologies mentioned, team structure

# Check for data breaches
# https://haveibeenpwned.com
# Check if target employees' emails appear in breaches

# Dehashed (paid)
# https://dehashed.com
# Search for leaked credentials by domain
```

---

### Step 7: Technology & Infrastructure

```bash
# Builtwith (technology profiler)
# https://builtwith.com/target.com
# Reveals: CMS, frameworks, analytics, hosting, CDN

# Netcraft (site report)
# https://sitereport.netcraft.com/?url=target.com
# Reveals: hosting, history, risk rating

# Wappalyzer
# Browser extension → visit site → see tech stack

# DNS History
# https://securitytrails.com/domain/target.com/history/a
# Shows historical IP addresses → previous hosting

# IP ranges
# Check if company owns IP ranges:
whois 1.2.3.4
# Look for organization's ASN

# BGP/ASN info
# https://bgp.he.net/
# Search for company name → find their IP ranges
```

---

### Step 8: Social Media & Code Repositories

```bash
# GitHub (source code, secrets)
# Search: "target.com" password
# Search: "target.com" api_key
# Search: org:targetcompany password
# Search: org:targetcompany secret
# Search: org:targetcompany token

# GitLeaks / TruffleHog (automated secret detection)
trufflehog github --org=targetcompany
gitleaks detect --source=repofolder

# Pastebin / paste sites
# Search on Google: site:pastebin.com "target.com"
# Check: ghostbin, paste.ee, dpaste, etc.

# Social media
# Twitter: @targetcompany → tech announcements, employee handles
# Facebook: company page
# Reddit: r/target or discussions about the company
```

---

## Passive Recon Report Template

```
Target: target.com
Date: YYYY-MM-DD

DOMAIN INFORMATION:
- Registrar: [registrar]
- Registered: [date]
- Expires: [date]
- Name Servers: [ns1, ns2]

DNS RECORDS:
- A: [IP addresses]
- MX: [mail servers]
- TXT: [SPF, DKIM, verification records]
- NS: [nameservers]

SUBDOMAINS FOUND:
- www.target.com
- mail.target.com
- dev.target.com
- staging.target.com
[...]

TECHNOLOGY STACK:
- Web Server: [Apache/Nginx/IIS]
- CMS: [WordPress/Drupal/Custom]
- Language: [PHP/Python/Java]
- CDN: [Cloudflare/Akamai]

EMAILS FOUND:
- admin@target.com
- john.doe@target.com
[...]

OPEN PORTS (Shodan):
- 80 (HTTP)
- 443 (HTTPS)
- 22 (SSH)
[...]

GOOGLE DORKING FINDINGS:
- Exposed admin panel at /admin
- Directory listing at /files/
[...]
```

---

## Tips for the Interview

1. **Start with WHOIS and DNS** — quickest passive info
2. **Google Dorking is extremely powerful** — memorize the key operators
3. **Always check Certificate Transparency (crt.sh)** — best subdomain source
4. **Shodan reveals infrastructure** without touching the target
5. **theHarvester combines multiple sources** → run it early
6. **Check GitHub for leaked secrets** — employees accidentally push credentials
7. **Document everything** — interviewers value organized findings
8. **Never send requests to the target** — that makes it active recon!
