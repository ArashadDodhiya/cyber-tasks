# 🔬 Lab Exercise 4: Server Fingerprinting

> **Challenge:** Identify server technology, versions, OS, and running services
> **Where:** Kali → Metasploitable (`192.168.56.20`)
> **Time:** ~30-40 minutes

---

## Exercise 4A: HTTP Header Analysis

```bash
# Step 1: Get HTTP response headers
curl -I http://192.168.56.20
# 📝 Write down:
#    Server: _______________
#    X-Powered-By: _______________
#    Other interesting headers: _______________

# Step 2: Verbose output for more details
curl -v http://192.168.56.20 2>&1 | head -30
# 📝 What additional info do you see in the connection handshake?

# Step 3: Filter for technology headers specifically
curl -s -I http://192.168.56.20 | grep -iE "server|x-powered|x-aspnet|x-generator|set-cookie"
# ✅ EXPECTED: Server header reveals Apache version
# ✅ Set-Cookie with PHPSESSID tells you it's PHP

# Step 4: Check other web services
curl -I http://192.168.56.20:8180
# ✅ EXPECTED: Apache Tomcat/Coyote — different web server on different port
# 📝 What version of Tomcat?
```

---

## Exercise 4B: Nmap Service Detection

```bash
# Step 1: Quick scan — find open ports
nmap 192.168.56.20
# 📝 How many open ports did you find? List them all.
# ✅ Found 18 open ports: 21(ftp), 22(ssh), 23(telnet), 80(http), 111(rpcbind), 512(exec), 513(login), 514(shell), 1099(rmiregistry), 1524(ingreslock), 2121(ccproxy-ftp), 3306(mysql), 5432(postgresql), 5900(vnc), 6000(X11), 6667(irc), 8009(ajp13), 8180(unknown).

# Step 2: Service version detection
nmap -sV 192.168.56.20
# ✅ EXPECTED: Each port shows service name AND version
# 📝 Document every service and version:
#    Port 21:  _______________
#    Port 22:  _______________
#    Port 23:  _______________
#    Port 25:  _______________
#    Port 80:  _______________
#    Port 139: _______________
#    Port 445: _______________
#    Port 3306: _______________
#    Port 5432: _______________
#    Port 8180: _______________
#    Others:    _______________

# Step 3: OS detection
sudo nmap -O 192.168.56.20
# 📝 What OS did nmap detect? _______________
# 📝 How confident was the guess? _______________

# Step 4: Aggressive scan (combines everything)
nmap -A 192.168.56.20
# ✅ This does: -sV (versions) + -sC (scripts) + -O (OS) + --traceroute
# 📝 What additional info do NSE scripts reveal?

# Step 5: Full port scan (find everything)
nmap -p- --min-rate 5000 192.168.56.20
# 📝 Did you find any ports you missed in the quick scan?

# Step 6: Save scan results for your notes
nmap -sV -sC -A -oA /tmp/metasploitable_scan 192.168.56.20
# This saves: .nmap (text), .xml, .gnmap files
cat /tmp/metasploitable_scan.nmap
# 📝 Keep this file — it's your complete fingerprint!
```

---

## Exercise 4C: Banner Grabbing

```bash
# Step 1: FTP banner grab (port 21)
nc -v 192.168.56.20 21
# ✅ EXPECTED: "220 (vsFTPd 2.3.4)" — exact version!
# Type: QUIT (to exit)
# 📝 FTP version: _______________

# Step 2: SSH banner grab (port 22)
nc -v 192.168.56.20 22
# ✅ EXPECTED: "SSH-2.0-OpenSSH_4.7p1 Debian-8ubuntu1"
# Press Ctrl+C to exit
# 📝 SSH version: _______________

# Step 3: SMTP banner grab (port 25)
nc -v 192.168.56.20 25
# ✅ EXPECTED: "220 metasploitable.localdomain ESMTP Postfix..."
# Type: QUIT
# 📝 SMTP server: _______________

# Step 4: HTTP banner grab (port 80)
nc -v 192.168.56.20 80
# Then type exactly:
HEAD / HTTP/1.1
Host: 192.168.56.20

# (press Enter twice after Host line)
# ✅ EXPECTED: Server header with Apache version
# 📝 Web server: _______________

# Step 5: MySQL banner grab (port 3306)
nc -v 192.168.56.20 3306
# ✅ EXPECTED: MySQL version string in the banner
# Press Ctrl+C to exit
# 📝 MySQL version: _______________

# Step 6: IRC banner grab (port 6667)
nc -v 192.168.56.20 6667
# ✅ EXPECTED: IRC daemon info (UnrealIRCd)
# 📝 IRC daemon: _______________
```

---

## Exercise 4D: Web Technology Identification Tools

```bash
# Step 1: whatweb (pre-installed on Kali)
whatweb http://192.168.56.20
# ✅ EXPECTED: Shows server, language, framework all at once
# 📝 What did whatweb identify?

# Step 2: whatweb aggressive mode
whatweb -a 3 http://192.168.56.20
# 📝 Did aggressive mode find more info?

# Step 3: whatweb on different ports/services
whatweb http://192.168.56.20:8180
# 📝 What technology is on port 8180?

whatweb http://192.168.56.20/dvwa/
# 📝 What does it reveal about DVWA?

whatweb http://192.168.56.20/phpMyAdmin/
# 📝 What phpMyAdmin version?

# Step 4: Nikto vulnerability scanner (also fingerprints)
nikto -h 192.168.56.20
# ✅ EXPECTED: Lots of findings — server version, outdated software, misconfigs
# 📝 List the top 5 most important findings from Nikto:
#    1. _______________
#    2. _______________
#    3. _______________
#    4. _______________
#    5. _______________
```

---

## Exercise 4E: Specific NSE Script Scans

```bash
# Step 1: HTTP methods detection
nmap --script=http-methods -p 80 192.168.56.20
# 📝 What HTTP methods are allowed?

# Step 2: HTTP headers
nmap --script=http-headers -p 80 192.168.56.20
# 📝 Any new header info?

# Step 3: HTTP title
nmap --script=http-title -p 80,8180 192.168.56.20
# 📝 What are the page titles?

# Step 4: SMB OS discovery
nmap --script=smb-os-discovery -p 445 192.168.56.20
# ✅ EXPECTED: Reveals OS version, computer name, domain, workgroup

# Step 5: Vulnerability scan
nmap --script=vuln -p 21,22,80,445 192.168.56.20
# 📝 What known vulnerabilities did nmap find?
```

---

## Exercise 4F: Error Page Analysis

```bash
# Step 1: Trigger a 404 error on Apache
curl http://192.168.56.20/nonexistent_page_12345
# ✅ EXPECTED: Apache error page reveals version

# Step 2: Trigger errors for different technologies
curl http://192.168.56.20/test.php
curl http://192.168.56.20/test.asp
curl http://192.168.56.20/test.jsp
# 📝 Which extensions return different error types?
# 📝 Do any error pages reveal additional server info?

# Step 3: Trigger Tomcat error
curl http://192.168.56.20:8180/nonexistent
# 📝 What does the Tomcat error page reveal?
```

---

## Exercise 4G: Compile Your Fingerprint Report

📝 **Fill this out as your final report:**

```
=== SERVER FINGERPRINT REPORT ===
Target: 192.168.56.20

Operating System: _______________
Kernel Version: _______________

Web Server (Port 80): _______________
Web Server (Port 8180): _______________

Programming Language: _______________
Database: _______________

Open Ports & Services:
  21/tcp: _______________
  22/tcp: _______________
  23/tcp: _______________
  25/tcp: _______________
  80/tcp: _______________
  139/tcp: _______________
  445/tcp: _______________
  3306/tcp: _______________
  5432/tcp: _______________
  8180/tcp: _______________

Web Applications Found:
  - DVWA at /dvwa/
  - phpMyAdmin at /phpMyAdmin/
  - _______________
  - _______________

Known Vulnerabilities:
  - _______________
  - _______________
  - _______________
```

---

## ✅ Completion Checklist

- [ ] Analyzed HTTP headers with curl (4A)
- [ ] Performed full nmap scan with version/OS detection (4B)
- [ ] Banner grabbed 6+ services with netcat (4C)
- [ ] Used whatweb and nikto for web fingerprinting (4D)
- [ ] Ran specific NSE scripts (4E)
- [ ] Analyzed error pages (4F)
- [ ] Compiled a complete fingerprint report (4G)

---

**Next:** [Challenge 5: HTTP Form Manipulation →](./05_lab_form_manipulation.md)
