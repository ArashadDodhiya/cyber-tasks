# Rapid-Fire Cheat Sheet — Lab Round Quick Reference

> Memorize this. Speed wins the lab round.

---

## SQL Injection — Speed Reference

```
DETECT:
  Single quote        → does it break?
  Double single quote → does the error go away?
  OR condition true   → boolean true test
  OR condition false  → boolean false test
  SLEEP function      → time-based confirm

COLUMN COUNT:
  ORDER BY 1, then 2, then 3... until error

UNION EXTRACTION:
  Match column count with NULLs
  Replace NULLs with target columns one at a time

DATABASE INFO:
  MySQL:      @@version
  PostgreSQL: version()
  MSSQL:      @@version

TABLE ENUMERATION:
  information_schema.tables  → table_name
  information_schema.columns → column_name, table_name

BLIND TECHNIQUES:
  Boolean: SUBSTRING comparison, check page difference
  Time:    IF + SLEEP (MySQL), pg_sleep (Postgres), WAITFOR DELAY (MSSQL)

COMMENT SYNTAX:
  MySQL:  -- (with space), #
  Others: --
  All:    /* */
```

---

## XSS — Speed Reference

```
CONTEXT DETERMINES PAYLOAD:
  HTML body      → HTML tags with event handlers
  HTML attribute → break out with quote, add event handler
  JavaScript     → break out with quote, inject JS
  URL/href       → javascript: protocol

TAG OPTIONS (if script is blocked):
  img with onerror
  svg with onload
  input with onfocus + autofocus
  details with open + ontoggle
  body with onload

IF alert IS BLOCKED:
  confirm()
  prompt()
  eval with base64 decoded string
  String concatenation with bracket notation

COOKIE EXFILTRATION:
  fetch to attacker server with document.cookie appended
  Image src pointing to attacker server with cookie

CHECK DEFENSES:
  Is HttpOnly set on session cookies?
  Is Content-Security-Policy set?
  What encoding is applied (HTML entities, JS escaping)?
```

---

## IDOR — Speed Reference

```
YOUR ENDPOINT:  /api/resource/YOUR_ID
TEST:           /api/resource/OTHER_ID

WHAT TO CHANGE:
  Adjacent IDs:  ID-1, ID+1
  First user:    ID=1 (often admin)
  Your other account (register 2 accounts, cross-test)

TEST ALL METHODS:
  GET (read) → PUT (modify) → DELETE (remove)

IF UUID:
  Check if leaked in other API responses, comments, listings

IF 403:
  Change HTTP method
  Add extra parameters
  Try older API version
  Change Content-Type
  Path variations
```

---

## Authentication — Speed Reference

```
JWT:
  Decode: base64 decode header and payload (they are not encrypted)
  Algorithm none: change alg to none, remove signature, keep trailing dot
  Weak secret: crack with hashcat mode 16500
  Algorithm confusion: RS256 to HS256, sign with public key

PASSWORD RESET:
  Is token predictable? (sequential, time-based, short)
  Does token expire?
  Is token single-use?
  Is token tied to specific user?
  Does token leak via Referer header?

BRUTE FORCE:
  Test rate limiting: send 20 wrong passwords
  Bypass: X-Forwarded-For header rotation
  Check: IP-based vs account-based limiting
  Try: alternative login endpoints, API versions

2FA/MFA BYPASS:
  Skip 2FA page — go directly to dashboard URL
  Brute force code (check rate limiting)
  Submit null or empty code
  Check if OAuth login path skips 2FA

SESSION:
  Does session ID change after login? (fixation test)
  HttpOnly flag on cookie?
  Secure flag? SameSite attribute?
  Does logout invalidate session server-side?
```

---

## File Upload — Speed Reference

```
EXTENSION BYPASS:
  Double extension, case variation, alternative extensions
  Null byte, semicolon, trailing dot, NTFS stream

CONTENT-TYPE: Change header to image/jpeg

MAGIC BYTES: Prepend GIF89a; to PHP code (polyglot)

WEB SHELL: PHP system function taking GET parameter

POST-UPLOAD:
  Find URL to uploaded file
  Append command parameter
  Establish reverse shell for stability

IF EXTENSION FORCED:
  Upload .htaccess to change handler
  Use LFI if available to include the file
  SVG with JavaScript for XSS
```

---

## Access Control — Speed Reference

```
FORCE BROWSE:
  /admin /dashboard /api-docs /swagger /debug /console

METHOD TESTING:
  Try DELETE, PUT, PATCH on GET-only endpoints
  Method override headers: X-HTTP-Method-Override

PRIVILEGE ESCALATION:
  Add role=admin or is_admin=true to registration/update requests
  Check if server processes extra fields (mass assignment)

PATH BYPASS (if 403):
  Case change: /Admin, /ADMIN
  Trailing: /admin/, /admin/.
  Double slash: //admin
  URL encoding: /admin%20
  Dot segment: /./admin, /admin..;/
  Override headers: X-Original-URL, X-Rewrite-URL
```

---

## HTTP Status Codes That Matter

```
200  OK                 301  Permanent Redirect
302  Temporary Redirect 400  Bad Request
401  Unauthorized       403  Forbidden (EXISTS but no access)
404  Not Found          405  Method Not Allowed
500  Internal Error     503  Service Unavailable
```

**For pentesters:**
- 403 vs 404: 403 means it EXISTS but you lack access → try bypass
- 500 on injection: your input likely broke server-side code → injection possible
- 302 on direct access: redirecting to login → check if data sent before redirect

---

## Encoding Quick Reference

```
URL ENCODING:
  Space → %20    Quote → %27    Double-quote → %22
  Less-than → %3C   Greater-than → %3E   Slash → %2F

DOUBLE ENCODING:
  %27 → %2527    %3C → %253C

BASE64 COMMON VALUES:
  alert(1) → YWxlcnQoMSk=
  admin    → YWRtaW4=
```

---

## Time Management Strategy

```
50 questions, assume 2-3 hours total:

  Easy (approx 30 questions):   2-3 min each
  Medium (approx 15 questions): 5-7 min each
  Hard (approx 5 questions):    10-15 min each

STRATEGY:
  1. First pass: Do ALL easy ones quickly
  2. Second pass: Medium difficulty
  3. Third pass: Hard ones
  4. NEVER spend more than 15 min on one question
  5. Mark and skip if stuck — come back later
  6. You need 40/50 (80%) — you CAN miss 10

PRIORITY ORDER:
  Guaranteed points first (easy)
  Then medium for solid score
  Hard ones are bonus — don't sacrifice easy points for them
```

---

## Burp Suite Keyboard Shortcuts

```
Ctrl+R         Send to Repeater
Ctrl+I         Send to Intruder
Ctrl+Shift+D   Toggle Intercept On/Off
Tab            Switch request/response in Repeater
```

---

> This file is for SPEED during the lab round.
> The scenario files (01-06) are for DEPTH during the manager round.
> Master BOTH to crack this interview.
