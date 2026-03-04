# Challenge 4: Server Fingerprinting

> **Difficulty:** 🟢 EASY | **Success Ratio:** 100%
> **Goal:** Identify the web server, technology stack, version numbers, and running services.

---

## What They Test

Can you determine what software a server is running, including the web server, programming language, framework, CMS, database, and their versions?

---

## Methodology

### Step 1: HTTP Header Analysis

```bash
# Get response headers
curl -I https://target.com
curl -v https://target.com 2>&1 | head -30

# Key headers to look for:
# Server: Apache/2.4.41 (Ubuntu)
# Server: nginx/1.18.0
# Server: Microsoft-IIS/10.0
# X-Powered-By: PHP/7.4.3
# X-Powered-By: ASP.NET
# X-AspNet-Version: 4.0.30319
# X-Generator: WordPress 5.8
# X-Drupal-Cache: HIT
# X-Magento-Vary: ...
# Set-Cookie: PHPSESSID=... → PHP
# Set-Cookie: JSESSIONID=... → Java
# Set-Cookie: ASP.NET_SessionId=... → ASP.NET
# Set-Cookie: connect.sid=... → Node.js Express

# Filter for specific headers
curl -s -I https://target.com | grep -iE "server|x-powered|x-aspnet|x-generator|x-drupal|set-cookie|x-frame|x-xss|content-security"
```

### Step 2: Nmap Service Detection

```bash
# Basic service and version detection
nmap -sV target.com

# Aggressive scan (OS detection + scripts + version + traceroute)
nmap -A target.com

# Full port scan with service detection
nmap -p- -sV --min-rate 5000 target.com

# Specific ports for web servers
nmap -p 80,443,8080,8443,8000,3000,5000 -sV target.com

# Use NSE scripts for more info
nmap -sV --script=http-headers target.com
nmap -sV --script=http-title target.com
nmap -sV --script=http-server-header target.com
nmap -sV --script=http-enum target.com
nmap -sV --script=http-methods target.com
nmap --script=ssl-enum-ciphers -p 443 target.com

# OS detection
nmap -O target.com
nmap -O --osscan-guess target.com
```

### Step 3: Banner Grabbing

```bash
# Netcat banner grab
nc -v target.com 80
# Then type: HEAD / HTTP/1.1
# Host: target.com
# (press Enter twice)

# OpenSSL for HTTPS
openssl s_client -connect target.com:443

# Telnet
telnet target.com 80

# Grab specific service banners
nc -v target.com 21    # FTP
nc -v target.com 22    # SSH
nc -v target.com 25    # SMTP
nc -v target.com 3306  # MySQL
nc -v target.com 5432  # PostgreSQL
```

### Step 4: Web Technology Identification Tools

```bash
# whatweb (pre-installed on Kali)
whatweb target.com
whatweb -a 3 target.com    # Aggressive mode
whatweb -v target.com      # Verbose output

# Wappalyzer (browser extension)
# Install from Chrome/Firefox store
# Visit target → see technology breakdown

# Builtwith (online)
https://builtwith.com/target.com

# Netcraft (online)
https://sitereport.netcraft.com/?url=target.com

# WhatRuns (browser extension)
# Alternative to Wappalyzer

# httpx (fast HTTP toolkit)
echo "target.com" | httpx -tech-detect -status-code -title

# Nikto (vulnerability scanner with fingerprinting)
nikto -h target.com
nikto -h https://target.com
```

### Step 5: CMS Detection

```bash
# CMSMap
cmsmap https://target.com

# WPScan (WordPress specific)
wpscan --url https://target.com
wpscan --url https://target.com --enumerate u,vp,vt
# u = users, vp = vulnerable plugins, vt = vulnerable themes

# Joomscan (Joomla specific)
joomscan -u https://target.com

# Droopescan (Drupal, WordPress, Joomla, SilverStripe)
droopescan scan drupal -u https://target.com

# Manual CMS identification
curl https://target.com/wp-login.php         # WordPress
curl https://target.com/administrator/        # Joomla
curl https://target.com/user/login            # Drupal
curl https://target.com/admin/login           # Many CMS
curl https://target.com/wp-content/           # WordPress
curl https://target.com/components/           # Joomla
curl https://target.com/sites/default/files/  # Drupal
```

### Step 6: SSL/TLS Analysis

```bash
# sslscan
sslscan target.com

# sslyze
sslyze target.com

# testssl.sh
testssl.sh target.com

# OpenSSL
openssl s_client -connect target.com:443 -showcerts

# Check certificate details
echo | openssl s_client -connect target.com:443 2>/dev/null | openssl x509 -text -noout
# Look for: Organization, Common Name, Subject Alt Names, Validity
```

### Step 7: Error Page Analysis

```bash
# Trigger error pages to reveal technology
curl https://target.com/nonexistent_page_12345
curl https://target.com/test.php       # PHP error?
curl https://target.com/test.asp       # ASP error?
curl https://target.com/test.jsp       # JSP error?

# Common error pages reveal tech:
# Apache: "Apache/2.4.41 (Ubuntu) Server at..."
# Nginx: "nginx" default error page
# IIS: Detailed ASP.NET error (yellow screen of death)
# Tomcat: "Apache Tomcat/9.0" error page
# Django: Debug page with full stack trace
# Rails: "Routing Error" page
# Laravel: Whoops error page
# Spring Boot: "Whitelabel Error Page"
```

---

## Technology Identification Cheat Sheet

| Indicator                            | Technology           |
| ------------------------------------ | -------------------- |
| `PHPSESSID` cookie                   | PHP                  |
| `JSESSIONID` cookie                  | Java (Tomcat/Spring) |
| `ASP.NET_SessionId` cookie           | ASP.NET              |
| `connect.sid` cookie                 | Node.js Express      |
| `csrftoken` cookie + Django template | Django (Python)      |
| `laravel_session` cookie             | Laravel (PHP)        |
| `_rails_session` cookie              | Ruby on Rails        |
| `/wp-content/` or `/wp-includes/`    | WordPress            |
| `/administrator/`                    | Joomla               |
| `Drupal.settings` in JS              | Drupal               |
| `X-Magento-Vary` header              | Magento              |
| `.aspx` extensions                   | ASP.NET              |
| `.jsp` extensions                    | Java Server Pages    |
| `.php` extensions                    | PHP                  |
| `__viewstate` hidden field           | ASP.NET Web Forms    |

---

## What to Report

When you've completed fingerprinting, document:

```
Server Information:
- Web Server: Apache 2.4.41
- OS: Ubuntu Linux  
- Programming Language: PHP 7.4.3
- CMS: WordPress 5.8
- Database: MySQL 5.7 (inferred from PHP + WordPress)
- Open Ports: 22 (SSH), 80 (HTTP), 443 (HTTPS), 3306 (MySQL)
- SSL/TLS: TLS 1.2 and 1.3 supported
- Notable Headers: X-Powered-By exposed, no security headers
```

---

## Tips for the Interview

1. **Start with `curl -I`** — fastest way to get initial info
2. **Run `whatweb`** — it identifies most technologies in one command
3. **Check cookies** — session cookie names immediately reveal the backend language
4. **Trigger error pages** — intentional 404s/500s often reveal technology versions
5. **Check all open ports** — not just 80/443, services on other ports give clues
6. **Don't forget SSL info** — certificate details reveal domains and organization info
