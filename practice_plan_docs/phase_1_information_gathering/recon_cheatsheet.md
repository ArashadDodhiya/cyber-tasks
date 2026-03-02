# ⚡ Information Gathering — Quick Reference Cheatsheet

> Print this. Pin it to your wall. Refer to it during every engagement.

---

## 🔍 Passive Recon Commands

### WHOIS & DNS
```bash
whois <domain>
host <domain>
host -t mx <domain>
host -t ns <domain>
host -t txt <domain>
dig any <domain>
dig axfr @<ns_server> <domain>        # Zone transfer attempt
nslookup -type=any <domain>
```

### Subdomain Discovery
```bash
# Certificate Transparency
curl -s "https://crt.sh/?q=%25.<domain>&output=json" | jq -r '.[].name_value' | sort -u

# theHarvester
theHarvester -d <domain> -b google,bing,crtsh -l 500

# Subfinder
subfinder -d <domain> -all -o subs.txt

# Amass
amass enum -passive -d <domain> -o subs.txt
```

### Google Dorking
```
site:<domain> filetype:pdf|doc|xls|ppt
site:<domain> inurl:login|admin|dashboard
site:<domain> intitle:"index of"
site:<domain> filetype:env|log|conf|bak|sql
site:github.com "<domain>" password|secret|key
site:pastebin.com "<domain>"
```

### Shodan
```
hostname:<domain>
org:"<org_name>"
ssl.cert.subject.cn:<domain>
port:3389 country:US
vuln:CVE-XXXX-XXXXX
```

### OSINT
```bash
sherlock <username>                    # Username search across platforms
```

---

## 🎯 Active Scanning Commands

### Host Discovery
```bash
nmap -sn <network/cidr>               # Ping sweep
nmap -PR <network/cidr>               # ARP scan (local only)
nmap -PS22,80,443,445 <network/cidr>  # TCP SYN ping
arp-scan -l                           # ARP scan (local)
```

### Port Scanning
```bash
sudo nmap -sS <target>                # SYN scan (stealth)
nmap -sT <target>                     # TCP connect
sudo nmap -sU --top-ports 100 <target> # UDP scan
nmap -p- <target>                     # All TCP ports
nmap -p- --min-rate 5000 <target>     # Fast full scan
```

### Service & OS Detection
```bash
nmap -sV <target>                     # Version detection
nmap -sV -sC <target>                 # Version + default scripts
sudo nmap -O <target>                 # OS detection
nmap -A <target>                      # Aggressive (all-in-one)
```

### NSE Scripts
```bash
nmap --script vuln <target>           # Vulnerability scripts
nmap --script smb-enum-shares -p 445 <target>
nmap --script http-enum -p 80 <target>
nmap --script ftp-anon -p 21 <target>
nmap --script smtp-enum-users -p 25 <target>
nmap --script "smb-vuln*" -p 445 <target>
nmap --script ms-sql-info -p 1433 <target>
```

### Output
```bash
nmap <options> -oA <basename>         # All formats (XML + gnmap + txt)
nmap <options> -oN output.txt         # Normal text only
nmap <options> -oX output.xml         # XML only
```

---

## 🌐 Web Enumeration Commands

### Directory Bruteforcing
```bash
gobuster dir -u <url> -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak
feroxbuster -u <url> -w /usr/share/wordlists/dirb/common.txt
dirb <url>
```

### Technology Detection
```bash
whatweb <url> -v
curl -sI <url>                        # Response headers
```

### Vulnerability Scanning
```bash
nikto -h <url>
```

### Manual Checks
```bash
curl <url>/robots.txt
curl <url>/sitemap.xml
curl <url>/.git/config
curl <url>/.env
curl <url>/server-status
```

---

## 📡 Service Enumeration Commands

### SMB (Ports 139/445)
```bash
smbclient -L //<target> -N             # List shares (anon)
smbclient //<target>/<share> -N         # Connect to share
enum4linux -a <target>                  # Full SMB enum
crackmapexec smb <target> --shares      # List shares
crackmapexec smb <target> --users       # List users
crackmapexec smb <target> --pass-pol    # Password policy
rpcclient -U "" -N <target>             # RPC (then: enumdomusers)
```

### SMTP (Port 25)
```bash
nc <target> 25                          # Banner grab, VRFY <user>
smtp-user-enum -M VRFY -U users.txt -t <target>
```

### SNMP (Port 161/UDP)
```bash
snmpwalk -v 2c -c public <target>
snmp-check <target>
onesixtyone -c community_strings.txt <target>
```

### DNS (Port 53)
```bash
dnsenum <domain>
dnsrecon -d <domain> -t std
dnsrecon -d <domain> -t axfr            # Zone transfer
gobuster dns -d <domain> -w <wordlist>
```

### LDAP (Port 389)
```bash
ldapsearch -x -H ldap://<target> -b "DC=domain,DC=local" "(objectClass=user)" sAMAccountName
```

### MSSQL (Port 1433)
```bash
crackmapexec mssql <target>
nmap --script ms-sql-info -p 1433 <target>
```

---

## 🔑 Vulnerability Research

```bash
searchsploit <service> <version>        # Local exploit database
searchsploit -m <path>                  # Copy exploit locally
searchsploit --nmap output.xml          # Match against Nmap XML
```

---

## 📁 Key Wordlist Locations (Kali)

```
/usr/share/wordlists/
├── dirb/common.txt                    # ~4,600 entries (fast)
├── dirb/big.txt                       # ~20,000 entries (thorough)
├── dirbuster/directory-list-2.3-medium.txt  # ~220,000 (comprehensive)
├── rockyou.txt.gz                     # Passwords (14M+)
└── SecLists/                          # apt install seclists
    ├── Discovery/Web-Content/         # Web directories & files
    ├── Discovery/DNS/                 # Subdomain wordlists
    ├── Usernames/                     # Username lists
    └── Passwords/                     # Password lists
```

---

## ⚡ One-Liner Combos

```bash
# Full recon pipeline in one command
nmap -sS -sV -sC -O -p- --min-rate 5000 <target> -oA full_recon

# Quick web recon
whatweb <url> && nikto -h <url> && gobuster dir -u <url> -w /usr/share/wordlists/dirb/common.txt

# Subdomain → resolve → probe
subfinder -d <domain> -silent | httpx -silent -status-code -title

# Find all SMB shares on a subnet
crackmapexec smb <network/cidr> --shares 2>/dev/null | grep READ

# Check entire subnet for EternalBlue
nmap --script smb-vuln-ms17-010 -p 445 <network/cidr> --open
```

---

> 💡 Replace `<target>`, `<domain>`, `<url>`, `<network/cidr>` with your actual values.
> Always use `-oA` with Nmap to save results. Always have written permission before scanning.
