The title:

> “Information is not only text. It is everything, everywhere.”

This is a very important cybersecurity mindset.

Most beginners think:

* hacking = SQL injection only
* vulnerabilities = login bypass only

But real-world security issues come from:

* comments
* metadata
* headers
* source code
* old files
* backup files
* server banners
* robots.txt
* JS files
* cookies
* directory listings
* IP cameras
* search engines
* exposed admin panels

This lab is teaching you:

> “Everything leaks information.”

---

# Main Topics Covered in This Lab

The checklist reveals the actual learning objectives.

---

# 1. Dynamic vs Static Pages

Task:

> Identify the number of dynamic and static pages

---

## What is a Static Page?

A static page is:

* fixed content
* same for every user
* usually simple HTML/CSS

Example:

```html
about.html
contact.html
logo.png
style.css
```

Server just sends the file directly.

---

## What is a Dynamic Page?

Dynamic page changes based on:

* user
* database
* input
* session
* API

Examples:

```php
login.php
profile.jsp?id=1
dashboard.aspx
search?q=test
```

These pages usually:

* interact with databases
* process user input
* contain vulnerabilities

---

## Why This Matters in Security

Dynamic pages are usually attack surfaces.

Because they:

* accept input
* query databases
* process sessions
* execute backend logic

Static pages usually have fewer risks.

---

## What You Need to Learn

### Extensions

Static:

* .html
* .css
* .js
* .png
* .jpg

Dynamic:

* .php
* .jsp
* .asp
* .aspx
* .cgi

---

## Tools Used

* Burp Suite
* Browser DevTools
* Wappalyzer
* Dirsearch
* FFUF
* Gobuster

---

# 2. Session Cookies

Task:

> Find a session cookie

---

## What is a Session?

When you login:

* server remembers you
* creates a session ID

Example:

```http
Set-Cookie: PHPSESSID=abc123
```

That cookie proves:

> “This browser is authenticated.”

---

## Why Session Cookies Matter

If attacker steals session cookie:

* they can impersonate user
* sometimes no password needed

This is called:

* Session Hijacking

---

## Things You Need to Learn

### Cookie Attributes

Important flags:

* HttpOnly
* Secure
* SameSite

---

## Example

```http
Set-Cookie: sessionid=12345; HttpOnly; Secure
```

Meaning:

* JavaScript cannot access it
* only HTTPS allowed

---

## Vulnerabilities Related

* Session fixation
* Session hijacking
* Predictable sessions
* Missing HttpOnly
* Missing Secure

---

# 3. Find SQL Files with Directory Listing

Task:

> Find SQL file with directory listing enabled

---

This is a VERY common misconfiguration.

---

## What is Directory Listing?

Normally:

```txt
/files/
```

should not show file contents.

But if directory listing enabled:
you see:

```txt
backup.sql
users.sql
config.php
```

Like browsing folders in Windows Explorer.

---

## Why Dangerous?

SQL files may contain:

* database dumps
* usernames
* passwords
* email addresses

Huge data leaks happen this way.

---

# What You Need to Learn

## Common Sensitive Files

* backup.sql
* db.sql
* users.sql
* config.php.bak
* .env
* backup.zip

---

## Tools

* Dirsearch
* Gobuster
* FFUF

---

## Concept

This is called:

* Sensitive Information Disclosure
* Misconfiguration

---

# 4. Find IP Cameras Using Shodan

Task:

> find 5 live IP camera using shodan

This teaches:

* OSINT
* Internet exposure
* IoT insecurity

---

# What is Shodan?

Shodan

Shodan is like Google for:

* servers
* routers
* cameras
* IoT devices

Instead of webpages, it indexes:

* open ports
* banners
* devices

---

# Why IP Cameras Matter

Many cameras:

* use default passwords
* expose RTSP streams
* expose admin panels

Sometimes public cameras are accidentally exposed.

---

# What You Learn Here

## Search Filters

Examples:

```txt
webcamXP
port:554
hikvision
country:IN
city:Mumbai
```

---

## Important Ethics

You are supposed to:

* identify exposure
* not access illegally
* not change settings

Recon only.

---

# 5. Find Vulnerable Nginx/IIS/Apache Versions

Task:

> find nginx, IIS and apache older vulnerable versions in Mumbai using shodan

---

This teaches:

* Server fingerprinting
* Banner grabbing
* Version enumeration

---

# What Are These?

## Apache

Apache HTTP Server

## Nginx

Nginx

## IIS

Microsoft IIS

These are web servers.

---

# Why Versions Matter

Old versions may contain:

* RCE vulnerabilities
* directory traversal
* buffer overflow
* authentication bypass

---

## Example

```txt
Apache/2.2.8
```

Very old.

You then:

* search CVEs
* identify risks

---

# Important Security Concepts Here

## Banner Grabbing

Server reveals:

```http
Server: Apache/2.4.49
```

This leaks version info.

---

## CVE

Common Vulnerabilities and Exposures.

Example:

```txt
CVE-2021-41773
```

---

# 6. Find Admin Login Interface

Task:

> find admin login interface

---

## What is This Teaching?

Attack surface discovery.

Admin panels are high-value targets.

Examples:

```txt
/admin
/admin.php
/login
/wp-admin
/dashboard
/panel
```

---

# Why Important?

Admin pages:

* may use weak passwords
* may expose software versions
* may contain vulnerabilities

---

## Tools

* Dirsearch
* FFUF
* Burp
* Manual recon

---

# 7. Sensitive Information Through HTML Comments

Task:

> Sensitive information through source code HTML comments

---

This is VERY common.

---

## Example

```html
<!-- TODO: remove test password admin123 -->
```

Or:

```html
<!-- API KEY HERE -->
```

---

# What You Learn

Developers accidentally expose:

* passwords
* internal paths
* API keys
* usernames
* debug info

---

# Important Beginner Skill

Always:

* View Source
* Inspect JS files
* Read comments

---

# Bigger Concepts This Lab Is Secretly Teaching

---

# A. Reconnaissance

Before attacking:

* gather information
* map architecture
* identify technologies

Recon is often:

> 70% of real penetration testing.

---

# B. Misconfiguration

Most tasks here are actually:

* weak configuration
* exposed files
* information disclosure

Not “hacking” in movie style.

---

# C. OSINT

Open Source Intelligence:

* public information gathering

Using:

* Shodan
* Google dorks
* headers
* DNS info

---

# D. Enumeration

Systematically discovering:

* files
* pages
* versions
* technologies
* services

---

# E. Attack Surface Mapping

Finding:

* where application can be attacked

---

# Technologies Mentioned in Your Screenshot

You can see URLs like:

* `.php`
* `.jsp`
* `install.php`
* `signup`
* `register`

This means the lab includes:

* PHP applications
* Java/JSP applications
* Authentication systems
* Session handling

---

# What You Should Learn Practically

---

# Browser Skills

Learn:

* View Source
* DevTools
* Network tab
* Cookies
* Headers

---

# Burp Suite Basics

Learn:

* Proxy
* Repeater
* Intercept
* HTTP requests

---

# HTTP Basics

Understand:

* GET
* POST
* Cookies
* Headers
* Sessions
* Status codes

---

# Directory Enumeration

Learn:

* FFUF
* Dirsearch
* Gobuster

---

# Web Servers

Understand:

* Apache
* Nginx
* IIS

---

# Basic Shodan Queries

Understand:

* filters
* ports
* banners

---

# Most Important Mental Shift

This lab is teaching you:

> Security is often not about “breaking in”.
> It’s about noticing what others accidentally exposed.

That is one of the biggest truths in cybersecurity.

---

# Recommended Learning Order For You

## Phase 1

Learn:

* HTTP
* Cookies
* Sessions
* Status codes

---

## Phase 2

Learn:

* HTML source
* JS files
* DevTools
* Directory listing

---

## Phase 3

Learn:

* Burp Suite
* Intercepting requests
* Cookies
* Headers

---

## Phase 4

Learn:

* Shodan
* OSINT
* Banner grabbing
* Server fingerprinting

---

# Important Ethical Note

These labs are designed for learning.

Outside labs:

* never access systems without permission
* never exploit real targets illegally
* never modify exposed systems

Recon and learning are okay in authorized environments.

---

# What This Entire Module Is Really About

If I summarize the whole module in one sentence:

> “Learn how web applications accidentally expose information, and how attackers discover it.”

That’s the core theme here.
