# 🔬 Lab Exercise 8: 2FA Bypass Using Brute Force Attack

> **Challenge:** Bypass two-factor authentication by brute-forcing the OTP
> **Where:** OWASP Juice Shop on Kali (Docker) + Burp Suite
> **Time:** ~40-50 minutes

---

## Setup: Install OWASP Juice Shop

> DVWA doesn't have 2FA, so we use Juice Shop — a modern vulnerable web app.

```bash
# Step 1: Install Docker (if not already)
sudo apt install docker.io -y
sudo systemctl start docker

# Step 2: Run Juice Shop
sudo docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop

# Step 3: Verify it's running
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000
# ✅ EXPECTED: 200

# Step 4: Open in Kali Firefox
# Go to: http://localhost:3000
```

> If port 3000 is already in use:
> `sudo docker run -d -p 3001:3000 --name juice-shop2 bkimminich/juice-shop`
> Access at `http://localhost:3001`

---

## Exercise 8A: Understand 2FA Basics (Concept Exercise)

Before practicing, understand the concept:

```
2FA Flow:
1. User enters username + password → Step 1 authentication
2. Server generates a short OTP (4-6 digit code)
3. OTP is sent to user's phone/email
4. User enters the OTP → Step 2 authentication
5. If correct → access granted

Attack Surface:
├── Is the OTP short? (4 digits = only 10,000 possibilities)
├── Is there rate limiting? (Can you try many codes fast?)
├── Does the OTP expire? (Long expiry = more time to brute force)
├── Is the same OTP reused? (No regeneration = easier to crack)
└── Can you skip Step 2 entirely? (Navigate directly to dashboard)
```

---

## Exercise 8B: Register and Set Up 2FA on Juice Shop

```
Step 1: Create an account on Juice Shop
  - Go to http://localhost:3000/#/register
  - Email: test@test.com
  - Password: test1234!
  - Security Question: pick any, answer: test
  - Click Register

Step 2: Login
  - Go to http://localhost:3000/#/login
  - Email: test@test.com
  - Password: test1234!

Step 3: Enable 2FA (if available)
  - Go to http://localhost:3000/profile or Account settings
  - Look for Two-Factor Authentication / 2FA / TOTP
  - If available, enable it and note the setup key/QR code
  
  📝 Did you find 2FA settings?
  📝 What type of 2FA does Juice Shop use? (TOTP, SMS, etc.)
```

---

## Exercise 8C: Brute Force OTP with Burp Suite (Simulation)

> This exercise teaches the **technique** even if the exact Juice Shop 2FA may vary.

### Practice Scenario: Brute-Force a 4-Digit PIN
```
Step 1: Set up Burp for Juice Shop
  - Configure Firefox proxy → 127.0.0.1:8080
  - Browse to http://localhost:3000

Step 2: Find a page that accepts numeric input
  - The Juice Shop "Forgot Password" or Security Answer field works
  - Or find any challenge that requires a numeric/short code

Step 3: Capture the request in Burp
  - Turn Intercept ON
  - Submit a test value
  - Right-click the intercepted request → "Send to Intruder"

Step 4: Configure Intruder for brute force
  - Positions tab:
    - Clear existing positions
    - Highlight the OTP/code value → Add §
  - Payloads tab:
    - Payload type: Numbers
    - From: 0000
    - To: 9999
    - Step: 1
    - Min integer digits: 4
    - Max integer digits: 4
  - Click "Start Attack"

Step 5: Analyze results
  - Sort by Response Length — different length = correct code!
  - Sort by Status Code — different code = correct code!
  - 📝 Did you find the correct value?
```

---

## Exercise 8D: Practice OTP Brute Force Against DVWA Brute Force Page

> Since DVWA doesn't have real 2FA, we simulate the OTP concept using its Brute Force page.

```
Step 1: Go to http://192.168.56.20/dvwa/vulnerabilities/brute/
  (Security: Low)

Step 2: Capture a login request in Burp
  - Username: admin
  - Password: test
  - Intercept → Send to Intruder

Step 3: Brute force the password field (simulating OTP brute force)
  - Positions: Mark only the password value → §test§
  - Payloads: Simple list → add:
    admin
    password
    password123
    admin123
    letmein
    12345
    test
    changeme
    welcome
    root

Step 4: Start Attack → check results
  - Which password gave a different response length?
  ✅ EXPECTED: "password" gives a different (larger) response = successful login!
  📝 This is exactly the same technique used for OTP brute forcing
```

---

## Exercise 8E: Other 2FA Bypass Techniques (Practice on Juice Shop)

### Technique 1: Skip the 2FA Page Entirely
```bash
# After Step 1 authentication, try navigating directly to authenticated pages
# Instead of entering the OTP, try:
curl http://localhost:3000/api/Products -b "YOUR_SESSION_COOKIE"
curl http://localhost:3000/rest/user/whoami -b "YOUR_SESSION_COOKIE"
# 📝 Can you access authenticated resources without completing 2FA?
```

### Technique 2: Response Manipulation
```
1. Login with correct credentials → 2FA page appears
2. Enter wrong OTP → Intercept the RESPONSE in Burp
3. Look for:
   - "success": false → change to "success": true
   - HTTP 401 → change to 200
   - Error message → change to success message
4. Forward the modified response
5. 📝 Did you bypass 2FA?
```

### Technique 3: Null/Empty OTP
```bash
# Try submitting empty or null OTP values:
# In Burp, change the OTP parameter to:
#   otp=
#   otp=null
#   otp=0
#   otp=000000
#   (or remove the parameter entirely)
# 📝 Does any of these bypass the check?
```

### Technique 4: Reuse Previous OTP
```
1. Complete 2FA normally with a valid OTP
2. Log out
3. Log in again
4. Try the SAME OTP from step 1
📝 Does the old OTP still work? (If yes → OTP isn't properly invalidated)
```

---

## Exercise 8F: Try Juice Shop Security Challenges

Juice Shop has many built-in challenges. Try these related ones:

```
1. Open http://localhost:3000/#/score-board
   (or navigate to it from the sidebar)

2. Look for challenges related to:
   - "Brute Force" category
   - "Authentication" category
   - "Broken Authentication" category

3. Try to solve at least 2 challenges from these categories

📝 Which challenges did you solve?
📝 What technique did you use for each?
```

---

## ✅ Completion Checklist

- [ ] Installed and ran OWASP Juice Shop (Setup)
- [ ] Understood 2FA concepts and attack surfaces (8A)
- [ ] Registered and explored 2FA on Juice Shop (8B)
- [ ] Practiced OTP brute force with Burp Intruder (8C)
- [ ] Simulated OTP brute force on DVWA Brute Force page (8D)
- [ ] Tried 2FA bypass techniques: skip, response manipulation, null OTP (8E)
- [ ] Attempted Juice Shop security challenges (8F)

---

**Next:** [Challenge 9: REST API HTTP Methods →](./09_lab_api_methods.md)
