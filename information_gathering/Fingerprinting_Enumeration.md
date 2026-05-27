This module is a BIG step up from the previous ones.

Earlier modules were mainly:

* information disclosure
* errors
* misconfiguration

This module is about:

# “Mapping the target deeply before attacking.”

This is one of the MOST important phases in real penetration testing.

The whole theme here is:

# Fingerprinting + Enumeration

These two words are foundational in cybersecurity.

---

# What Is Fingerprinting?

Fingerprinting means:

> Identifying what technologies a target uses.

Example:

* Which web server?
* Which CMS?
* Which framework?
* Which programming language?
* Which ports?
* Which OS?
* Which plugins?

---

# What Is Enumeration?

Enumeration means:

> Systematically extracting as much information as possible.

Not random guessing.

It is:

* structured discovery
* structured information gathering

---

# Why This Matters

You cannot attack properly unless you know:

* what exists
* what technologies run
* where weaknesses may exist

Real pentesting often spends:

> 60–80% time on enumeration.

---

# Let’s Break Down Every Task

---

# 1. Nmap Full Port Scanning TCP/UDP

Task:

> Find all possible information about all ports

This is HUGE foundational knowledge.

---

# What is a Port?

Think of a computer/server like:

* a building

Ports are:

* doors/services

Example:

* Port 80 → website
* Port 22 → SSH
* Port 3306 → MySQL

---

# What is Nmap?

Nmap

Nmap =

> Network Mapper

One of the most important cybersecurity tools ever created.

Used for:

* scanning
* service detection
* OS fingerprinting
* vulnerability scripts

---

# TCP vs UDP

---

# TCP

Reliable communication.

Examples:

* HTTP
* HTTPS
* SSH

---

# UDP

Faster but unreliable.

Examples:

* DNS
* VoIP
* SNMP

UDP scanning is harder.

---

# Common Ports

| Port | Service |
| ---- | ------- |
| 21   | FTP     |
| 22   | SSH     |
| 23   | Telnet  |
| 25   | SMTP    |
| 53   | DNS     |
| 80   | HTTP    |
| 110  | POP3    |
| 139  | NetBIOS |
| 143  | IMAP    |
| 443  | HTTPS   |
| 445  | SMB     |
| 3306 | MySQL   |
| 3389 | RDP     |

---

# Important Nmap Concepts

---

# Service Detection

Example:

```bash id="rk2cok"
nmap -sV target.com
```

Finds:

* Apache version
* SSH version
* FTP version

---

# OS Detection

```bash id="74tthp"
nmap -O target.com
```

Attempts:

* Linux?
* Windows?
* FreeBSD?

---

# Aggressive Scan

```bash id="0dy4u5"
nmap -A target.com
```

Does:

* version detection
* scripts
* traceroute
* OS detection

---

# Full Port Scan

```bash id="4i1m5j"
-p-
```

Means:

> scan ALL 65535 ports

---

# UDP Scan

```bash id="x9bl3k"
-sU
```

Very important but slower.

---

# NSE Scripts

Your screenshot mentions:

> service vulnerability enumeration using NSE

This is important.

---

# What is NSE?

NSE =

> Nmap Scripting Engine

Prebuilt scripts for:

* vulnerability detection
* brute force
* enumeration

---

# Example NSE Script

```bash id="xjlwmn"
nmap --script vuln
```

Checks for known vulnerabilities.

---

# Important Lesson

Open port ≠ vulnerability

But:

* it reveals attack surface

---

# 2. Netcraft, WhatWeb, Wappalyzer

Task:

> Fingerprint technologies

This is web technology identification.

---

# What is Wappalyzer?

Wappalyzer

Browser extension.

Detects:

* WordPress
* React
* PHP
* Nginx
* Analytics
* CDN

---

# WhatWeb

WhatWeb

CLI fingerprinting tool.

Identifies:

* CMS
* frameworks
* libraries
* server tech

---

# Netcraft

Netcraft

Provides:

* hosting info
* server data
* historical data

---

# Why Fingerprinting Matters

Suppose you detect:

```txt id="qj5d0j"
WordPress 5.2
```

Now attacker searches:

```txt id="qmgc9v"
WordPress 5.2 vulnerabilities
```

Same with:

* Apache versions
* plugins
* frameworks

---

# What You Learn Here

How websites reveal:

* frameworks
* JavaScript libraries
* backend tech
* CDN
* analytics
* security headers

---

# 3. APP1 to APP63 — Directory Enumeration

This is VERY important.

---

# What is Directory Enumeration?

Trying to discover:

* hidden pages
* admin panels
* backups
* APIs
* uploads
* configuration files

---

# Example

Trying:

```txt id="v4yh1m"
/admin
/backup
/login
/uploads
/config
```

---

# Why Important?

Many sensitive resources:

* are not linked publicly
* but still accessible

---

# Tools Mentioned

---

# Gobuster

Gobuster

Fast directory brute-forcer.

---

# Dirb

DIRB

Classic content discovery tool.

---

# Dirbuster

DirBuster

GUI-based enumeration tool.

---

# FFUF

ffuf

Modern high-speed fuzzing tool.

---

# Burp Content Discovery

Using:
Burp Suite

to discover hidden content.

---

# What Is Fuzzing?

Supplying many possible inputs automatically.

Example:

```txt id="ksc3zd"
/FUZZ
```

Wordlist tries:

* admin
* login
* backup

---

# Wordlists

VERY important concept.

Common wordlists:

* SecLists
* raft
* common.txt

---

# Important Discovery Targets

---

# Admin Panels

```txt id="uw14bf"
/admin
/admin.php
/wp-admin
```

---

# Backup Files

```txt id="vvbq9y"
backup.zip
site.bak
db.sql
```

---

# Uploads

```txt id="v4xh5u"
/uploads
/files
```

---

# Config Files

```txt id="tnrk0m"
.env
config.php
```

---

# 4. Check and Download robots.txt

This is VERY important beginner recon.

---

# What is robots.txt?

Example:

```txt id="qgd5z8"
https://site.com/robots.txt
```

Used for:

* search engine instructions

---

# Example

```txt id="1m9t2x"
Disallow: /admin
Disallow: /backup
```

This accidentally reveals hidden paths.

---

# Why Attackers Check It

Developers often expose:

* admin paths
* sensitive directories
* staging panels

---

# Important Concept

robots.txt is:

> advisory, not security

Anyone can access it.

---

# 5. Identify Client-Side Validation in JavaScript

VERY important concept.

---

# What is Client-Side Validation?

Validation happening in browser using JavaScript.

Example:

```javascript id="k5g91z"
if(password.length < 8)
```

---

# Why Important?

Client-side validation can be bypassed.

Because:

* attacker controls browser

---

# Important Security Principle

Never trust client-side validation alone.

Server MUST validate too.

---

# Example Vulnerability

Browser blocks:

```txt id="r3qqqa"
password too short
```

But attacker intercepts request with Burp and sends anyway.

---

# What You Learn

* JS analysis
* validation bypass
* client/server trust issues

---

# 6. BURP Crawl and Record Results

VERY important real-world workflow.

---

# What is Crawling?

Automatically exploring application.

Like:

* clicking links automatically
* mapping application

---

# Burp Spider/Crawler

Finds:

* endpoints
* parameters
* APIs
* hidden paths

---

# Why Important?

You cannot secure/test:
what you haven't discovered.

---

# Burp Suite Skills Learned

* proxying
* crawling
* intercepting
* mapping app structure

---

# 7. WPScan, Joomscan, Droopescan

This is CMS enumeration.

---

# CMS = Content Management System

Examples:

* WordPress
* Joomla
* Drupal

VERY common targets.

---

# WPScan

WPScan

Scans:

* WordPress version
* plugins
* themes
* vulnerabilities

---

# Joomscan

JoomScan

For Joomla sites.

---

# Droopescan

Droopescan

For Drupal and related CMS.

---

# Why CMS Enumeration Matters

Suppose scanner finds:

```txt id="mknl5d"
Plugin: vulnerable-slider
```

Attacker searches:

```txt id="1a6n16"
vulnerable-slider exploit
```

---

# Bigger Concepts Hidden in This Module

---

# A. Attack Surface Mapping

Understanding:

* what exists
* what can be attacked

---

# B. Technology Fingerprinting

Identifying:

* framework
* language
* CMS
* server
* plugins

---

# C. Recon Automation

Using tools to:

* automate discovery
* speed up enumeration

---

# D. Security Through Obscurity Failure

Hidden pages are not truly hidden.

Enumeration finds them.

---

# E. Passive vs Active Recon

---

# Passive Recon

No direct interaction.

Examples:

* Netcraft
* Wappalyzer

---

# Active Recon

Direct scanning.

Examples:

* Nmap
* Gobuster
* Burp

---