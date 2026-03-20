# 🔬 Lab Exercise 1: Cookie Tampering

> **Challenge:** Manipulate cookies to escalate privileges or bypass authentication
> **Where:** DVWA on Metasploitable + Juice Shop on Kali
> **Time:** ~40-50 minutes

---

## Exercise 1A: Inspect DVWA Cookies

**Target:** DVWA at `http://192.168.56.20/dvwa/`

```
Step 1: Login to DVWA (admin / password)

Step 2: Open DevTools (F12) → Application → Cookies
  - Click on http://192.168.56.20
  - 📝 List all cookies you see:
    - Cookie Name: ______________ Value: ______________
    - Cookie Name: ______________ Value: ______________

Step 3: Examine the "security" cookie
  - 📝 What is its current value? ______________
  - 📝 What are the cookie attributes? (Path, Domain, HttpOnly, Secure)

Step 4: Check cookie in Console tab
  - Press F12 → Console → type: document.cookie
  - 📝 Which cookies are visible via JavaScript?
  - 📝 Which cookies are NOT visible? (These have HttpOnly flag)
```

---

## Exercise 1B: Modify Cookies to Change DVWA Security Level

```
Step 1: In DevTools → Application → Cookies
  - Find the "security" cookie (currently set to "low")
  - Double-click the value and change it to: medium
  - Refresh the page (F5)
  - Go to: http://192.168.56.20/dvwa/security.php
  ✅ EXPECTED: Security level now shows "Medium"!
  📝 The server trusts the cookie value without server-side validation

Step 2: Try changing security to "high"
  - Change cookie value to: high
  - Refresh and check security page
  ✅ EXPECTED: Security level changed to "High"

Step 3: Try changing security to "impossible"
  - Change cookie value to: impossible
  - Refresh and check
  📝 Does it accept "impossible" level?

Step 4: Try an invalid value
  - Change cookie value to: hacked
  - Refresh the page
  📝 What happens with an invalid cookie value?
```

---

## Exercise 1C: Base64 Cookie Decode and Modify

**Task:** Practice encoding/decoding cookies as you'd see in real apps.

```bash
# Scenario: You find a cookie with value "dXNlcj1ndWVzdDtyb2xlPXVzZXI="

# Step 1: Decode it
echo "dXNlcj1ndWVzdDtyb2xlPXVzZXI=" | base64 -d
# ✅ EXPECTED: user=guest;role=user

# Step 2: Craft an escalated version
echo -n "user=admin;role=admin" | base64
# ✅ EXPECTED: dXNlcj1hZG1pbjtyb2xlPWFkbWlu
# 📝 This is how you'd replace the cookie value

# Step 3: Try another encoded cookie
echo "aXNBZG1pbj1mYWxzZQ==" | base64 -d
# ✅ EXPECTED: isAdmin=false

# Step 4: Craft the escalated version
echo -n "isAdmin=true" | base64
# 📝 Write down the encoded value: ______________
```

---

## Exercise 1D: Cookie Tampering via Burp Suite

**Target:** DVWA at `http://192.168.56.20/dvwa/`

```
Step 1: Configure Burp proxy (127.0.0.1:8080)
  - Turn Intercept ON

Step 2: Navigate to any DVWA page in Firefox

Step 3: In Burp, find the Cookie header in the intercepted request
  - It should look like:
    Cookie: PHPSESSID=abc123; security=low

Step 4: Modify the cookie in the intercepted request
  - Change: security=low → security=impossible
  - Click Forward

Step 5: Observe the page behavior
  📝 Did the security level change?
  📝 Are the challenges now harder?

Step 6: Try modifying the PHPSESSID
  - Intercept another request
  - Change the PHPSESSID to a random value: PHPSESSID=hacked123
  - Forward the request
  📝 What happened? (You should be logged out — session invalidated)
  ✅ This shows that modifying session cookies can break your session
```

---

## Exercise 1E: JWT Token Inspection on Juice Shop

**Target:** Juice Shop at `http://localhost:3000`

```
Step 1: Register and login to Juice Shop
  - Go to http://localhost:3000/#/register
  - Email: test@juice.com
  - Password: testtest1!
  - Security Question: pick any, answer: test
  - Login with your new account

Step 2: Find the JWT token
  - F12 → Application → Local Storage → http://localhost:3000
  - Find the "token" key
  - 📝 Copy the token value (it starts with "eyJ...")

Step 3: Decode the JWT
  - Go to https://jwt.io in another tab (or use command line)
  - Paste the token

  # Or decode on Kali terminal:
  TOKEN="<paste your token here>"
  echo "$TOKEN" | cut -d '.' -f 1 | base64 -d 2>/dev/null; echo
  echo "$TOKEN" | cut -d '.' -f 2 | base64 -d 2>/dev/null; echo

  📝 Header — what algorithm is used? ______________
  📝 Payload — what is your email? ______________
  📝 Payload — what is your role? ______________
  📝 Payload — what is your user ID? ______________

Step 4: Try to tamper with the JWT
  - On jwt.io, change the role to "admin" in the payload
  - Copy the modified token
  - In DevTools → Local Storage → replace the token value
  - Refresh the page
  📝 Did the role change take effect?
  📝 Why or why not? (Think about JWT signature verification)
```

---

## Exercise 1F: Session Cookie Prediction

**Task:** Test if session IDs are predictable.

```bash
# Step 1: Get multiple session cookies from DVWA
# Open 3 separate private/incognito windows, login to DVWA each time
# F12 → Console → document.cookie in each window

# 📝 Session 1 PHPSESSID: ______________
# 📝 Session 2 PHPSESSID: ______________
# 📝 Session 3 PHPSESSID: ______________

# Step 2: Analyze the session IDs
# 📝 Are they sequential (incrementing numbers)?
# 📝 Are they random-looking (long hex strings)?
# 📝 Do they follow a pattern?

# Step 3: Try to use one session from another browser
# Copy Session 1's PHPSESSID
# In a different browser, set cookie:
# F12 → Console →
document.cookie = "PHPSESSID=<session1_value>; path=/dvwa/"
# Navigate to http://192.168.56.20/dvwa/
# 📝 Are you logged in as the other session's user?
# ✅ This demonstrates session hijacking via cookie theft
```

---

## ✅ Completion Checklist

- [ ] Inspected and documented DVWA cookies (1A)
- [ ] Modified security cookie to change DVWA security level (1B)
- [ ] Decoded/encoded Base64 cookies for privilege escalation (1C)
- [ ] Tampered cookies in transit using Burp Suite (1D)
- [ ] Inspected and decoded JWT token from Juice Shop (1E)
- [ ] Analyzed session cookie predictability (1F)

---

**Next:** [Challenge 10: XSS Basics →](./02_lab_xss_basics.md)
