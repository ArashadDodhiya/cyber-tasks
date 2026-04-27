# File Upload Vulnerabilities — Interview Scenarios

> File upload is one of the most dangerous features. One successful bypass can mean Remote Code Execution.

---

## Scenario 1: Basic File Upload Bypass (Medium)

**Interviewer:** "An app allows uploading profile pictures. Only .jpg and .png are allowed. How do you upload a web shell?"

### Model Answer:

"I test what type of validation is in place — client-side, server-side, or both.

**Test 1 — Client-side only?**
Disable JavaScript or intercept the request in Burp. If the restriction is only in the browser, bypassing is trivial.

**Test 2 — Extension bypass attempts:**

| Attempt | Technique |
|---------|-----------|
| Double extension | shell.php.jpg |
| Case variation | shell.pHp |
| Alternative extensions | .php5, .phtml, .phar |
| Null byte | shell.php%00.jpg |
| Semicolon | shell.php;.jpg |
| Trailing dot (Windows) | shell.php. |
| NTFS stream (Windows) | shell.php::$$DATA |

**Test 3 — Content-Type bypass:**
Change the Content-Type header from application/x-php to image/jpeg.
If server checks Content-Type header instead of actual content, this works.

**Test 4 — Magic bytes bypass:**
Prepend real JPEG magic bytes (FF D8 FF E0) to your PHP file.
Server checks magic bytes and sees JPEG. Web server checks extension and executes PHP.

**Test 5 — After successful upload:**
Find where the file is stored and try to access it directly via URL to trigger execution."

### Follow-ups:

**Q: "Explain what magic bytes are and why this bypass works."**

> "Magic bytes (file signatures) are the first few bytes that identify a file type:
> - JPEG: FF D8 FF E0
> - PNG: 89 50 4E 47
> - GIF: 47 49 46 38 (ASCII: GIF8)
> - PDF: 25 50 44 46 (ASCII: %PDF)
>
> The bypass works because TWO DIFFERENT SYSTEMS check TWO DIFFERENT THINGS:
> 1. File validation reads magic bytes and says 'this is a JPEG — allowed'
> 2. Web server reads file extension and says 'this is PHP — execute it'
> Neither system talks to the other."

**Q: "The upload renames your file to a random name with forced .jpg extension. Now what?"**

> "If I can't control the extension:
> 1. Path traversal in filename: try uploading with name ../../shell.php
> 2. If there's an LFI vulnerability elsewhere, include the uploaded file regardless of extension
> 3. Upload a .htaccess file that makes the server execute .jpg as PHP
> 4. Upload SVG files (if allowed) — SVG is XML and can contain JavaScript for XSS
> 5. Embed payload in EXIF metadata of a real image, then trigger via LFI
> 6. Race condition: access temp file before server finishes renaming"

---

## Scenario 2: Web Shell Post-Exploitation (Medium-Hard)

**Interviewer:** "You successfully uploaded a web shell. Walk me through what you do next."

### Model Answer:

"**Step 1 — Verify execution and gather info:**
- whoami → which user am I running as?
- id → user ID, group membership
- uname -a → OS and kernel version
- pwd → current directory

**Step 2 — Assess the environment:**
- cat /var/www/.env → database credentials
- env → environment variables (may contain secrets)
- netstat -tlnp → open ports, internal services
- ps aux → running processes

**Step 3 — Establish stable access:**
Set up a proper reverse shell for stability instead of using the web shell for everything.

**Step 4 — Document everything:**
Screenshot every command. RCE is critical severity — report immediately.

**What I would NOT do:**
- Modify or delete any data
- Install persistent backdoors (unless in scope)
- Pivot to other systems without authorization
- Access personal data beyond what proves impact"

---

## Scenario 3: Bypassing Strong Validation (Hard)

**Interviewer:** "Server checks extension, Content-Type, AND magic bytes. How do you still get execution?"

### Model Answer:

"This is strong defense. I would explore several angles:

**1. Polyglot file — valid image AND valid code:**
Create a GIF file that is also valid PHP. GIF89a as the first bytes satisfies magic byte check. PHP code after it gets executed if the extension allows.

**2. Configuration file upload (Apache):**
Upload a .htaccess file with: AddType application/x-httpd-php .gif
Now ANY .gif file in that directory is executed as PHP. Then upload the polyglot GIF.

**3. PHP-FPM configuration:**
Upload a .user.ini file with: auto_prepend_file=shell.gif
Every PHP page in that directory will prepend and execute the GIF content.

**4. Race condition:**
File is saved temporarily before validation completes. Access the temp file during that gap.

**5. Image processing exploits:**
If the server uses ImageMagick to resize, certain crafted images can trigger command execution (ImageTragick CVE-2016-3714 concept).

**Key insight:** The upload itself might be secure, but the file's USAGE might be vulnerable. Think about the full lifecycle: upload, store, process, serve."

---

## Scenario 4: File Upload in APIs (Medium)

**Interviewer:** "The upload is through a REST API, not a form. How does this change your approach?"

### Model Answer:

"API uploads change the mechanics but not the logic. Key fields to manipulate:

1. **filename** in Content-Disposition header — controls the stored filename
2. **Content-Type** inside the multipart part — may be validated
3. **name** field — different upload handlers for different field names

**Base64 encoded uploads:**
Some APIs accept file content as base64 in JSON. The content_type and filename are just strings you control — send whatever you want.

**Additional API-specific tests:**
- Can I specify the storage path in the request body?
- Does the API support different upload methods (PUT, PATCH)?
- Is there a separate 'import' or 'restore' endpoint that accepts ZIP files containing malicious files?
- Does the API resize/process images? Test image processing library vulnerabilities."

---

## Quick Recall Cheat Sheet

```
EXTENSION BYPASS:
  .php5 .phtml .phar .php.jpg
  .pHp (case)  .php. (trailing dot)  .php::$$DATA (Windows)

CONTENT-TYPE BYPASS:
  Change to: image/jpeg, image/png, image/gif

MAGIC BYTES:
  GIF: GIF89a;    JPEG: FF D8 FF E0    PNG: 89 50 4E 47

POLYGLOT:
  GIF89a; followed by PHP code

SERVER CONFIG UPLOADS:
  .htaccess (Apache) — AddType directive
  .user.ini (PHP-FPM) — auto_prepend_file directive
  web.config (IIS) — handler mapping

POST-UPLOAD:
  Find the URL to your uploaded file
  Test if it executes (append ?cmd=whoami)
  If not, look for LFI to include the file

IF EXTENSION IS FORCED:
  LFI to include uploaded file
  EXIF metadata payload in real image
  SVG with JavaScript (for XSS at minimum)
  .htaccess to change handler
  Race condition on temp file
```
