# Challenge 7: User ID Enumeration

> **Difficulty:** 🟢 EASY | **Success Ratio:** 100%
> **Goal:** Discover valid usernames or user IDs through differences in application responses.

---

## What They Test

Can you identify valid usernames/email addresses/user IDs by analyzing differences in the application's responses (error messages, response times, status codes)?

---

## Key Concept

**Username enumeration** occurs when an application reveals whether a specific username exists. This is a security issue because attackers can build a list of valid users before attempting password attacks.

---

## Where to Find Enumeration Points

### Login Page
```
Valid user + wrong password → "Invalid password"
Invalid user → "User does not exist"
                  ↑ Different messages = ENUMERATION!

Correct response should be:
"Invalid username or password" (same for both cases)
```

### Registration Page
```
Try registering with existing username:
→ "Username already taken" = confirms user exists

Try registering with existing email:
→ "Email already registered" = confirms email exists
```

### Password Reset / Forgot Password
```
Existing email → "Password reset link sent"
Non-existing email → "Email not found"
                         ↑ ENUMERATION!

Sometimes even:
Existing email → 200 OK (with success message)
Non-existing email → 200 OK (with error message)
The response LENGTH differs = enumeration via response size
```

### API Endpoints
```bash
GET /api/user/admin     → 200 OK (user exists)
GET /api/user/nonexist  → 404 Not Found
GET /api/users/1        → 200 OK with user data
GET /api/users/999      → 404 Not Found

# User profile pages
/profile/admin     → 200 OK
/profile/nonexist  → 404 Not Found
```

---

## Enumeration Techniques

### Technique 1: Error Message Analysis

```bash
# Manual testing
# Login with known username (like admin), wrong password
# Compare error message with completely random username
# Different messages = enumeration

# Automate with curl
curl -s -X POST https://target.com/login \
  -d "username=admin&password=wrong" | grep -i "error\|invalid\|incorrect"

curl -s -X POST https://target.com/login \
  -d "username=randomuser12345&password=wrong" | grep -i "error\|invalid\|incorrect"

# Compare outputs - if different, username enumeration exists
```

### Technique 2: Response Size/Length Analysis

```bash
# Even if error messages look identical, the response SIZE may differ

# Check response sizes
curl -s -o /dev/null -w "%{size_download}" -X POST https://target.com/login \
  -d "username=admin&password=wrong"
# Output: 4521

curl -s -o /dev/null -w "%{size_download}" -X POST https://target.com/login \
  -d "username=fakefakeuser&password=wrong"
# Output: 4489
# Different sizes = enumeration via response length!
```

### Technique 3: Response Time Analysis

```bash
# Valid users may trigger password hash comparison (takes time)
# Invalid users skip hash comparison (faster response)

# Measure response times
curl -s -o /dev/null -w "%{time_total}" -X POST https://target.com/login \
  -d "username=admin&password=wrong"
# Output: 0.512

curl -s -o /dev/null -w "%{time_total}" -X POST https://target.com/login \
  -d "username=fakefakeuser&password=wrong"
# Output: 0.103
# Significantly different times = timing-based enumeration!
```

### Technique 4: HTTP Status Code Differences

```
Valid user: HTTP 200 (with "wrong password" message)
Invalid user: HTTP 401 ("unauthorized")

Or:
Valid user: HTTP 302 Redirect (to password retry)
Invalid user: HTTP 200 (stay on same page with error)
```

---

## Burp Suite Intruder Enumeration

### Step-by-Step
```
1. Capture login/register/reset request in Burp Proxy
2. Right-click → Send to Intruder (Ctrl+I)
3. Go to Intruder tab → Positions
4. Clear all positions (Clear §)
5. Highlight the username value → click "Add §"
   Example: username=§admin§&password=wrong
6. Go to Payloads tab
7. Load a username wordlist:
   - /usr/share/seclists/Usernames/top-usernames-shortlist.txt
   - /usr/share/seclists/Usernames/Names/names.txt
   - /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
8. Start Attack
9. Sort results by:
   - Response Length (different length = valid user)
   - Status Code (different code = valid user)
   - Response content (use grep match to highlight keywords)
```

### Grep Match Setup
```
In Intruder → Options → Grep - Match:
Add: "Invalid password"      (indicates valid username)
Add: "already taken"         (registration enumeration)
Add: "reset link sent"       (password reset enumeration)

Results will show a checkmark for responses containing these strings
```

---

## Common Username Wordlists

```bash
# SecLists (must-have)
/usr/share/seclists/Usernames/top-usernames-shortlist.txt    # 17 names
/usr/share/seclists/Usernames/Names/names.txt                 # ~10K names
/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt

# Common default usernames to always try
admin
administrator
root
user
test
guest
info
mysql
postgres
operator
support
webmaster
```

---

## Enumeration via Different Application Areas

### Login Form
| Indicator       | Valid User       | Invalid User     |
| --------------- | ---------------- | ---------------- |
| Error message   | "Wrong password" | "User not found" |
| Response length | 4521 bytes       | 4489 bytes       |
| Response time   | 500ms            | 100ms            |
| Status code     | 200              | 401              |

### Registration Form
| Indicator     | Existing User              | New User                  |
| ------------- | -------------------------- | ------------------------- |
| Error message | "Username taken"           | "Registration successful" |
| Email check   | "Email already registered" | Form proceeds             |

### Password Reset
| Indicator     | Existing Email       | Non-existing Email |
| ------------- | -------------------- | ------------------ |
| Message       | "Reset link sent"    | "Email not found"  |
| Response time | Slower (sends email) | Faster (no email)  |

### API Endpoints
| Indicator     | Valid ID       | Invalid ID    |
| ------------- | -------------- | ------------- |
| Status code   | 200 OK         | 404 Not Found |
| Response body | User data JSON | Error JSON    |

---

## Automating Enumeration with Scripts

### Python Script
```python
import requests
import time

url = "https://target.com/login"
usernames = open("/usr/share/seclists/Usernames/top-usernames-shortlist.txt").read().splitlines()

for username in usernames:
    start = time.time()
    resp = requests.post(url, data={
        "username": username,
        "password": "wrongpassword123"
    })
    elapsed = time.time() - start
    
    # Check multiple enumeration indicators
    print(f"User: {username:20s} | Status: {resp.status_code} | "
          f"Length: {len(resp.text):5d} | Time: {elapsed:.3f}s | "
          f"{'VALID?' if 'wrong password' in resp.text.lower() else ''}")
```

### ffuf (Fast Fuzzer)
```bash
# Enumerate via response size difference
ffuf -u https://target.com/login -X POST \
  -d "username=FUZZ&password=wrong" \
  -w /usr/share/seclists/Usernames/top-usernames-shortlist.txt \
  -fs 4489   # Filter out responses of size 4489 (invalid user size)

# Enumerate via keyword match
ffuf -u https://target.com/login -X POST \
  -d "username=FUZZ&password=wrong" \
  -w /usr/share/seclists/Usernames/top-usernames-shortlist.txt \
  -mr "wrong password"   # Match responses containing "wrong password"
```

---

## Tips for the Interview

1. **Check login, registration, AND password reset** — enumeration often exists in one but not others
2. **Compare response length** even if messages look identical — length differences are subtle
3. **Use Burp Intruder's length column** — sort by length to spot outliers quickly
4. **Check response time** — timing attacks are often overlooked
5. **Always report all valid users found** — the challenge expects a list
6. **This often leads to password brute-force** — after finding valid usernames, try common passwords
