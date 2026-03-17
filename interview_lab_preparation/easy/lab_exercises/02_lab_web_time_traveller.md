# 🔬 Lab Exercise 2: Web Time Traveller

> **Challenge:** Find historical/hidden information about websites
> **Where:** Kali terminal + Kali Firefox browser + Metasploitable
> **Time:** ~25-30 minutes

---

## Exercise 2A: Check Robots.txt and Sitemap on Metasploitable

**Target:** Metasploitable web server at `192.168.56.20`

```bash
# Step 1: Check robots.txt
curl http://192.168.56.20/robots.txt
# 📝 Does it exist? What paths are disallowed?

# Step 2: Check sitemap
curl http://192.168.56.20/sitemap.xml
# 📝 Does it exist? What URLs are listed?

# Step 3: Check for DVWA's robots.txt
curl http://192.168.56.20/dvwa/robots.txt
# 📝 Any hidden paths?

# Step 4: Try common hidden files
curl -s -o /dev/null -w "%{http_code}" http://192.168.56.20/.htaccess
curl -s -o /dev/null -w "%{http_code}" http://192.168.56.20/.git/config
curl -s -o /dev/null -w "%{http_code}" http://192.168.56.20/backup.zip
curl -s -o /dev/null -w "%{http_code}" http://192.168.56.20/phpinfo.php
# ✅ Look for 200 status codes — those files exist!
# Metasploitable likely has phpinfo.php accessible
```

```bash
# Step 5: If phpinfo.php returns 200, open it!
curl http://192.168.56.20/phpinfo.php | head -50
# Or open in Firefox: http://192.168.56.20/phpinfo.php
# ✅ This reveals PHP version, server config, loaded modules — goldmine!
```

---

## Exercise 2B: Directory Discovery on Metasploitable

```bash
# Step 1: Use gobuster to find hidden directories
gobuster dir -u http://192.168.56.20 -w /usr/share/wordlists/dirb/common.txt
# ✅ EXPECTED: You'll find directories like /dvwa, /phpMyAdmin, /tikiwiki, etc.

# Step 2: Explore what you found
# For each discovered directory, check it in the browser:
# http://192.168.56.20/phpMyAdmin/
# http://192.168.56.20/tikiwiki/
# http://192.168.56.20/twiki/
# http://192.168.56.20/dav/
# 📝 Document all accessible paths and what they contain
```

---

## Exercise 2C: Wayback Machine (Public Website Practice)

**Target:** Use safe public targets — NOT your Metasploitable!

```bash
# Open Kali Firefox and go to:
# https://web.archive.org/web/*/example.com

# Try these safe targets:
# https://web.archive.org/web/*/tesla.com
# https://web.archive.org/web/*/google.com

# ✅ Task: Find the OLDEST snapshot of tesla.com
# 📝 What year was the oldest snapshot?
# 📝 How did the website look back then?
# 📝 Are there any exposed pages in old versions that don't exist now?
```

### Wayback Machine CLI (waybackurls)
```bash
# Install waybackurls if not available
go install github.com/tomnomnom/waybackurls@latest
# Or download directly:
# https://github.com/tomnomnom/waybackurls/releases

# Use it to find all archived URLs for a domain
echo "example.com" | waybackurls | head -30
# 📝 Look for interesting paths: /admin, /backup, /api, /config, etc.
```

---

## Exercise 2D: Google Dorking (Passive Discovery)

**Task:** Practice Google dork syntax using safe queries.

Open Kali Firefox → go to google.com and try:

```
# Find robots.txt files
site:example.com filetype:txt

# Find exposed config files (general practice)
intitle:"index of" "backup"

# Find login pages
inurl:admin intitle:login

# Find exposed directories
intitle:"Index of /" "parent directory"

# Find specific file types
site:github.com filetype:sql "password"
```

> ⚠️ **Important:** Only use Google dorking for learning. Never use it to access systems you don't own.

---

## Exercise 2E: Check HTTP Headers for Historical Clues

```bash
# Step 1: Check Metasploitable HTTP headers
curl -I http://192.168.56.20
# 📝 What web server and version is revealed?
# 📝 Is there an X-Powered-By header?

# Step 2: Check for HTTP response headers on DVWA
curl -I http://192.168.56.20/dvwa/
# 📝 Any version info leaked?

# Step 3: Check different services
curl -I http://192.168.56.20:8180
# ✅ EXPECTED: This is Apache Tomcat — version info in headers

# Step 4: Check for backup/old file extensions
for ext in bak old orig save swp txt~; do
    code=$(curl -s -o /dev/null -w "%{http_code}" http://192.168.56.20/index.$ext)
    echo "index.$ext → $code"
done
# ✅ Any 200 responses = backup files found!
```

---

## Exercise 2F: View Page Source for Hidden Information

**Task:** Open DVWA in Kali Firefox and inspect the source.

```
1. Go to http://192.168.56.20/dvwa/
2. Right-click → View Page Source (Ctrl+U)
3. Look for:
   - HTML comments: <!-- ... -->
   - Hardcoded credentials
   - JavaScript files with interesting paths
   - Hidden form fields
   - API endpoints

4. Now check the DVWA login page source:
   http://192.168.56.20/dvwa/login.php (Ctrl+U)
   📝 What hidden fields do you see?
   📝 Any JavaScript validation logic?

5. Check JavaScript files linked in the page
   📝 Do any contain version numbers or developer comments?
```

---

## ✅ Completion Checklist

- [ ] Checked robots.txt and sitemap on Metasploitable (2A)
- [ ] Discovered hidden directories with gobuster (2B)
- [ ] Used Wayback Machine to find old snapshots (2C)
- [ ] Practiced Google dorking syntax (2D)
- [ ] Analyzed HTTP headers for version info (2E)
- [ ] Inspected page source for hidden information (2F)

---

**Next:** [Challenge 3: User Login Bypass →](./03_lab_login_bypass.md)
