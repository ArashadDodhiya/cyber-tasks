# 🔐 Authentication & Session Flaws — Interview Scenarios

> *"Authentication bugs are the gateway to account takeover. Know every angle."*

---

## Scenario 1: Broken Password Reset (Medium)

**🎯 Interviewer:** "You're testing a password reset flow. What do you look for?"

### ✅ Model Answer:

"I analyze the entire flow systematically:

**Step 1 — Request phase:**
- Does the reset form tell me if a username/email exists? ('User not found' = enumeration)
- Can I request resets for any email?
- Is there rate limiting? Can I spam reset requests?

**Step 2 — Token analysis:**
```
Reset link: https://app.com/reset?token=abc123def456
```
- How long is the token? (short = brute-forceable)
- Is it predictable? (request 5 tokens, look for patterns)
- Is it tied to a specific user? (can I use User A's token for User B?)
- Does it expire? (use token after 24 hours — does it still work?)
- Is it single-use? (use same token twice)

**Step 3 — Reset execution:**
- Can I change the email in the reset request?
  ```
  POST /reset-password
  token=abc123&email=victim@test.com&new_password=hacked
  ```
  If I change the email, whose password changes?
- Is there a `user_id` parameter I can manipulate?

**Step 4 — Token leakage:**
- Is the token in the URL? → Leaks via Referer header to third-party resources on the page
- Is the token logged server-side? → Could be in log files
- Does the page load external resources (analytics, ads)? → Token sent as Referer

**Red flags I've seen:**
- Token = MD5(email + timestamp) → predictable if I know when they requested
- Token = Base64(user_id) → decode it to see
- Token doesn't expire → valid forever
- Token not invalidated after password change → can be reused"

### 🔥 Follow-ups:

**Q: "You found the token is `base64(user_id:timestamp)`. How do you exploit?"**

> "If I know the target's user_id (from IDOR or enumeration) and the approximate time they requested a reset:
> 1. Generate tokens for that user_id with timestamps ±5 minutes from the request time
> 2. That's ~600 tokens (one per second for 10 minutes)
> 3. Automate trying each token against the reset endpoint
> 4. If there's no rate limiting on the reset endpoint, this takes seconds
> 
> This is why tokens MUST be cryptographically random (e.g., 32+ bytes from a CSPRNG), not derived from guessable data."

---

## Scenario 2: JWT Vulnerabilities (Hard)

**🎯 Interviewer:** "The application uses JWTs for authentication. How do you attack it?"

### ✅ Model Answer:

"First, I decode the JWT (it's just base64):

```
Header:  {"alg": "HS256", "typ": "JWT"}
Payload: {"sub": "1042", "role": "user", "exp": 1720000000}
Signature: HMACSHA256(header.payload, secret)
```

**Attack 1 — Algorithm None:**
```json
{"alg": "none", "typ": "JWT"}
```
Change algorithm to `none`, remove the signature. If the server accepts it, I can forge any JWT.

**Attack 2 — Algorithm Confusion (RS256 → HS256):**
If the server uses RS256 (asymmetric), I change to HS256 (symmetric) and sign with the PUBLIC key. The server might verify using the public key as an HMAC secret.

**Attack 3 — Weak Secret:**
Brute-force the HMAC secret using hashcat:
```
hashcat -a 0 -m 16500 jwt.txt wordlist.txt
```
If the secret is weak (e.g., 'secret', 'password'), I crack it and can forge any token.

**Attack 4 — Parameter Manipulation:**
```json
{"sub": "1042", "role": "admin", "exp": 99999999999}
```
Change `role` to `admin`, extend expiration. Re-sign if I have the secret.

**Attack 5 — JWK Header Injection:**
```json
{"alg": "RS256", "jwk": {"kty": "RSA", "n": "...", "e": "..."}}
```
Embed my own public key in the header. Sign with my private key. If the server trusts the embedded key, it verifies against MY key.

**Attack 6 — KID Parameter Injection:**
```json
{"alg": "HS256", "kid": "/dev/null"}
```
If `kid` points to a file, and I can make it point to a known file (like `/dev/null` which is empty), I sign with an empty secret."

### 🔥 Follow-ups:

**Q: "Explain why Algorithm Confusion works technically."**

> "The server is configured for RS256:
> - It has a private key (signs tokens) and a public key (verifies tokens)
> - Verification: `RSA_VERIFY(token, public_key)`
> 
> When I change the header to HS256:
> - HMAC verification uses: `HMAC_VERIFY(token, secret_key)`
> - If the server's code naively uses the same key variable for both algorithms, it uses the PUBLIC key as the HMAC secret
> - I know the public key (it's public!), so I sign: `HMAC(header.payload, public_key)`
> - The server verifies against the same public key and it matches
> 
> Fix: Always enforce the expected algorithm server-side, never trust the `alg` header."

---

## Scenario 3: Session Fixation & Hijacking (Medium-Hard)

**🎯 Interviewer:** "Explain session fixation vs session hijacking and how you'd test for each."

### ✅ Model Answer:

"**Session Fixation** — I force the victim to use a session ID I already know.

Attack flow:
```
1. Attacker gets a valid session ID: SESSIONID=abc123
2. Attacker sends victim a link: https://app.com/login?SESSIONID=abc123
3. Victim logs in using this session ID
4. Attacker now uses SESSIONID=abc123 → authenticated as victim
```

**Testing for fixation:**
- Before login, note my session ID
- Log in successfully
- Is the session ID the SAME after login? → Vulnerable! (should be regenerated)
- Can I set the session ID via URL parameter? Cookie? HTTP header?

**Session Hijacking** — I steal or predict an existing valid session.

Methods:
1. **XSS** → `document.cookie` (if no HttpOnly)
2. **Network sniffing** → Session in HTTP traffic (if no HTTPS/HSTS)
3. **Predictable sessions** → Sequential or time-based session IDs
4. **Session in URL** → Leaks via Referer, logs, browser history
5. **Cross-site attacks** → CSRF to perform actions, no need to steal session

**Session security checklist:**
```
[ ] Session ID regenerated after login
[ ] HttpOnly flag on session cookie
[ ] Secure flag on session cookie (HTTPS only)
[ ] SameSite attribute set
[ ] Session expires after reasonable timeout
[ ] Logout actually invalidates the session server-side
[ ] Session ID has sufficient entropy (128+ bits)
```"

### 🔥 Follow-ups:

**Q: "You click 'Logout' and your session cookie is deleted from the browser. Are you really logged out?"**

> "Not necessarily! The browser deleted the cookie, but the session might still be valid server-side. 
> 
> Test: Copy your session token BEFORE logging out. Log out. Then make a request with the old session token manually (using curl or Burp). If the server still accepts it, the logout only cleared the client-side cookie — the session is NOT invalidated server-side.
> 
> This means if an attacker stole your session token before you logged out, they can still use it even after you 'logged out.'
> 
> Fix: Logout must destroy the session on the server, not just delete the cookie."

---

## Scenario 4: Brute Force & Rate Limiting (Medium)

**🎯 Interviewer:** "How do you test if a login is vulnerable to brute force, and how do you bypass rate limiting?"

### ✅ Model Answer:

"**Testing for brute force protection:**
1. Send 10-20 login attempts with wrong passwords
2. Observe: Am I blocked? CAPTCHA? Delayed? Same response?
3. If nothing changes after 20 attempts → no rate limiting → vulnerable

**Rate limiting bypass techniques:**

| Bypass | How |
|--------|-----|
| **IP rotation** | `X-Forwarded-For: 1.2.3.4` — change each request (if server trusts this header) |
| **Header variation** | `X-Real-IP`, `X-Originating-IP`, `X-Client-IP` |
| **Account locking test** | Does it lock the ACCOUNT or the IP? If account, attacker causes denial of service |
| **Case variation** | `admin@test.com` vs `Admin@test.com` vs `ADMIN@TEST.COM` — treated as different? |
| **Whitespace** | `admin@test.com` vs `admin@test.com ` (trailing space) |
| **Null byte** | `admin@test.com%00` |
| **JSON array** | `{\"password\": [\"pass1\", \"pass2\", \"pass3\"]}` — some parsers try all |
| **Distributed** | Slow brute force from multiple IPs — 1 attempt per minute per IP |

**My approach in practice:**
1. First, test if rate limiting exists at all
2. If yes, test if it's per-IP or per-account
3. Try header-based IP spoofing
4. Test if the limit resets after a successful login
5. Check if there's a different login endpoint (mobile API, old API version) without rate limiting"

---

## Scenario 5: Multi-Factor Authentication Bypass (Hard)

**🎯 Interviewer:** "The app has 2FA. How would you attempt to bypass it?"

### ✅ Model Answer:

"MFA bypass techniques, from most common to advanced:

**1. Direct page access — skip the 2FA step entirely:**
```
Normal flow: /login → /verify-2fa → /dashboard
Test: /login → /dashboard (skip verify-2fa)
If /dashboard loads without 2FA verification → broken flow
```

**2. Response manipulation:**
```
POST /verify-2fa
code=123456
Response: {"success": false, "error": "Invalid code"}

Modify response in Burp to: {"success": true}
Does the app trust the client-side state?
```

**3. Brute force the code:**
- 6-digit OTP = 1,000,000 possibilities
- If no rate limiting → brute-forceable in minutes
- If rate limited per code but not per session → create new sessions

**4. Code reuse:**
- Use a valid code, log out, log in again — does the same code work?
- Use a code after its time window — is expiration enforced?

**5. Backup codes:**
- Are backup codes rate-limited separately?
- Are they shorter/simpler than OTP? → Easier to brute force

**6. Token/cookie manipulation:**
```
After entering username/password, check cookies:
is_verified=false → change to is_verified=true
mfa_pending=1 → change to mfa_pending=0
```

**7. Null/empty code:**
```
POST /verify-2fa
code=          (empty)
code=000000    (zeros)
code=null      
```
Sometimes null handling bypasses the check.

**8. OAuth/SSO bypass:**
If the app supports OAuth login AND traditional login, does the OAuth path require 2FA? Often it doesn't."

---

## Quick Recall — Auth & Session Cheat Sheet

```
PASSWORD RESET:
  Check: Token predictability, expiration, user-binding, single-use
  Leak:  Referer header, URL logging, browser history
  
JWT ATTACKS:
  alg:none → remove signature
  RS256→HS256 → sign with public key  
  Weak secret → hashcat -m 16500
  jwk injection → embed your key
  kid injection → point to known file

SESSION:
  Fixation: Does session ID change after login?
  Hijacking: HttpOnly? Secure? SameSite? HTTPS?
  Logout: Is session destroyed server-side?
  
BRUTE FORCE BYPASS:
  X-Forwarded-For header rotation
  IP-based vs account-based limiting
  Alternative login endpoints
  JSON array payloads
  
MFA BYPASS:
  Skip the 2FA page entirely
  Response manipulation  
  Code brute force (no rate limit?)
  Null/empty code submission
  OAuth path without MFA
```
