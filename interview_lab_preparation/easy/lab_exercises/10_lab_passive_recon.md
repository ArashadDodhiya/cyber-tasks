# 🔬 Lab Exercise 10: Passive Information Collection

> **Challenge:** Gather information about a target WITHOUT directly touching it
> **Where:** Kali terminal + browser (using safe public targets)
> **Time:** ~30-40 minutes

---

> ⚠️ **Important:** Passive recon means NOT sending any traffic directly to the target.
> You use third-party services and public data. This is 100% legal.
> For practice, use **safe targets** like example.com, scanme.nmap.org, or your own domains.

---

## Exercise 10A: WHOIS Lookup

```bash
# Step 1: WHOIS lookup on a safe target
whois example.com
# 📝 Write down:
#    Registrar: _______________
#    Creation Date: _______________
#    Expiry Date: _______________
#    Name Servers: _______________
#    Registrant Organization: _______________

# Step 2: Try another domain
whois google.com
# 📝 What registrar does Google use?
# 📝 What name servers are listed?

# Step 3: Use online WHOIS
# Open Firefox → https://who.is/
# Search for example.com
# 📝 Does the online tool show different/more info?
```

---

## Exercise 10B: DNS Enumeration

```bash
# Step 1: Basic DNS lookup
nslookup example.com
# 📝 IP address: _______________

# Step 2: Query different record types
dig example.com A         # IPv4 addresses
dig example.com AAAA      # IPv6 addresses
dig example.com MX        # Mail servers
dig example.com TXT       # Text records (SPF, DKIM, etc.)
dig example.com NS        # Name servers
dig example.com SOA       # Start of Authority
dig example.com ANY       # All records

# 📝 For each query, write down the results:
#    A Record(s): _______________
#    MX Record(s): _______________
#    TXT Record(s): _______________
#    NS Record(s): _______________

# Step 3: Reverse DNS lookup
dig -x 93.184.216.34
# 📝 What domain does this IP resolve to?

# Step 4: DNS lookup using different DNS servers
dig @8.8.8.8 example.com      # Google DNS
dig @1.1.1.1 example.com      # Cloudflare DNS
dig @9.9.9.9 example.com      # Quad9 DNS
# 📝 Do different DNS servers give different results?
```

---

## Exercise 10C: Certificate Transparency

```bash
# Certificate transparency logs reveal ALL SSL certificates issued for a domain
# This can expose subdomains!

# Method 1: crt.sh (browser)
# Open Firefox → https://crt.sh/?q=example.com
# 📝 How many certificates were issued?
# 📝 What subdomains do the certificates reveal?

# Method 2: crt.sh via API (from terminal)
curl -s "https://crt.sh/?q=example.com&output=json" | python3 -m json.tool | head -50
# 📝 List unique subdomains found: _______________

# Method 3: Try a different domain (with more subdomains)
curl -s "https://crt.sh/?q=%.github.com&output=json" | python3 -c "
import json, sys
data = json.load(sys.stdin)
domains = set()
for entry in data:
    domains.add(entry.get('common_name', ''))
for d in sorted(domains)[:20]:
    print(d)
"
# 📝 How many subdomains did you find?
```

---

## Exercise 10D: Subdomain Enumeration (Passive)

```bash
# Step 1: Use subfinder (passive subdomain tool)
# Install if not present:
sudo apt install subfinder -y

subfinder -d example.com
# 📝 Subdomains found: _______________

# Step 2: Try a bigger target
subfinder -d github.com | head -20
# 📝 How many subdomains?

# Step 3: Use amass (passive only)
amass enum -passive -d example.com
# 📝 Did amass find additional subdomains that subfinder missed?

# Step 4: Online tools (browser)
# Open: https://dnsdumpster.com/
# Enter: example.com
# 📝 What additional info does DNSDumpster show? (map, MX, NS, etc.)
```

---

## Exercise 10E: Google Dorking

**Task:** Practice these searches in Kali Firefox on google.com:

```
# Exercise 1: Find robots.txt files
# Search: site:example.com robots.txt
# 📝 Results? _______________

# Exercise 2: Find specific file types
# Search: site:github.com filetype:env
# 📝 Did you find any .env files? (These often contain secrets!)

# Exercise 3: Find login pages
# Search: intitle:"login" inurl:admin
# 📝 How many results?

# Exercise 4: Find directory listings
# Search: intitle:"Index of /" "parent directory"
# 📝 Did you find any exposed directories?

# Exercise 5: Find exposed documents
# Search: site:example.com filetype:pdf OR filetype:doc OR filetype:xlsx
# 📝 What sensitive documents did you find?

# Exercise 6: Find error messages
# Search: "Warning: mysql_connect()" site:*.com
# 📝 Any database error pages exposed?
```

### Google Dork Cheat Sheet
```
site:        → limit results to specific domain
filetype:    → search for specific file types
intitle:     → search page titles
inurl:       → search in URLs
intext:      → search in page body
cache:       → view Google's cached version
link:        → find pages linking to a URL
ext:         → same as filetype
```

---

## Exercise 10F: theHarvester

```bash
# theHarvester gathers emails, names, subdomains, IPs from public sources

# Step 1: Basic search
theHarvester -d example.com -b google
# 📝 What emails did it find? _______________
# 📝 What subdomains? _______________

# Step 2: Use multiple sources
theHarvester -d example.com -b google,bing,yahoo,dnsdumpster
# 📝 Did extra sources reveal more information?

# Step 3: Try with a more populated domain
theHarvester -d github.com -b google -l 50
# 📝 How many emails and hosts were found?
```

---

## Exercise 10G: Shodan (Internet Search Engine)

```
Method 1: Browser
1. Open: https://www.shodan.io/ (free account required)
2. Search: apache 2.2
   📝 How many results?
3. Search: "default password" port:80
   📝 What interesting results come up?
4. Search: hostname:example.com
   📝 What ports and services are exposed?

Method 2: CLI (if you have Shodan API key)
shodan init YOUR_API_KEY
shodan search apache 2.2 --limit 5
shodan host 93.184.216.34
```

---

## Exercise 10H: Social Media / OSINT

**Practice finding information from public sources:**

```
1. LinkedIn:
   - Search for "security engineer at [company]"
   - 📝 What employees can you find?
   - 📝 What technologies do they mention in their profiles?

2. GitHub:
   - Search: "password" in public repos
   - Search: org:[company] in:readme
   - 📝 Any secrets exposed in public repos?

3. Check for data breaches:
   - Open: https://haveibeenpwned.com
   - Enter your own email to check
   - 📝 How many breaches is your email in?

4. Check DNS history:
   - Open: https://securitytrails.com (free account)
   - Search: example.com
   - 📝 What DNS changes have happened over time?
```

---

## Exercise 10I: Compile Your OSINT Report

📝 **Fill in your findings:**

```
=== PASSIVE RECONNAISSANCE REPORT ===
Target: example.com (or your chosen safe target)

WHOIS:
  Registrar: _______________
  Created: _______________
  Name Servers: _______________

DNS Records:
  A: _______________
  MX: _______________
  NS: _______________
  TXT: _______________

Subdomains Found:
  - _______________
  - _______________
  - _______________

Technologies Identified:
  - _______________

Emails Found:
  - _______________

SSL Certificates:
  - Issued by: _______________
  - Valid dates: _______________

Shodan Results:
  - Open ports: _______________
  - Services: _______________

Google Dork Findings:
  - _______________

Overall Risk Assessment:
  _______________
```

---

## ✅ Completion Checklist

- [ ] WHOIS lookup with analysis (10A)
- [ ] DNS enumeration — all record types (10B)
- [ ] Certificate transparency search (10C)
- [ ] Passive subdomain enumeration (10D)
- [ ] Google dorking — at least 6 queries (10E)
- [ ] theHarvester scan (10F)
- [ ] Shodan search (10G)
- [ ] Social media / OSINT review (10H)
- [ ] Compiled OSINT report (10I)

---

## 🎉 You've Completed All 10 Easy Challenge Lab Exercises!

**What's next?**
1. Review your notes from each challenge
2. Try doing each challenge again WITHOUT looking at the instructions
3. Move on to the [Moderate Challenges](../../02_moderate_challenges.md)

**Key skills you've practiced:**
- ✅ Encoding/Decoding/Cracking
- ✅ Web reconnaissance
- ✅ SQL injection login bypass
- ✅ Server fingerprinting
- ✅ HTTP form manipulation
- ✅ Client-side validation bypass
- ✅ User enumeration
- ✅ 2FA bypass techniques
- ✅ REST API testing
- ✅ Passive information gathering
