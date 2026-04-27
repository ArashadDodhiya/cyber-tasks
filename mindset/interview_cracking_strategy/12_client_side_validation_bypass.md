# Bypassing Client-Side Validations — Complete Guide

> Client-side validation is decoration. It exists for user experience, NOT security.
> If security only exists in the browser, it does not exist at all.

---

## Why Client-Side Validation Is Not Security

```
CLIENT-SIDE: Code runs in YOUR browser → YOU control it
SERVER-SIDE: Code runs on THEIR server → THEY control it

Anything in your browser can be:
  - Disabled (turn off JavaScript)
  - Modified (edit the code)
  - Skipped (send requests directly, bypass browser entirely)
```

**Simple analogy:** A sign that says "Employees Only" is client-side validation. A locked door with a keycard reader is server-side validation. Anyone can ignore a sign.

---

## The 5 Universal Methods to Bypass ANY Client-Side Check

### Method 1: Intercept and Modify with Burp Suite

**How:** Let the browser do its validation, submit the form normally, then catch the request in Burp before it reaches the server. Modify whatever you want.

```
Step 1: Fill form normally (pass all JS checks)
Step 2: Burp intercepts the request
Step 3: Change any value in the request
Step 4: Forward the modified request to server
```

**Why it works:** The browser validated → happy. Burp changed the data after validation → server gets the modified version. Server never knew about the client-side check.

**This is the most reliable method. Use it as your default.**

---

### Method 2: Disable JavaScript

**How:** In browser settings, disable JavaScript entirely. All JS-based validations stop working.

```
Chrome: Settings → Privacy → Site Settings → JavaScript → Blocked
Firefox: about:config → javascript.enabled → false
```

**When to use:** Quick test to see if ANY validation is client-side only. If the form submits with JS off, there is no server-side backup.

---

### Method 3: Use Browser Developer Tools

**How:** Right-click → Inspect Element. Modify HTML directly.

```
Removing restrictions:
  - Delete "disabled" attribute from buttons
  - Delete "readonly" attribute from inputs
  - Delete "maxlength" attribute from inputs
  - Change "type=hidden" to "type=text" to see/edit hidden fields
  - Delete entire validation functions from script tags
  - Change form action URL
```

---

### Method 4: Send Requests Directly (No Browser)

**How:** Use curl, Postman, or Python requests. No browser means no JavaScript, no HTML, no client-side anything.

```
curl example:
  curl -X POST https://target.com/api/submit \
    -d "name=test&age=-5&email=not_an_email"

Python example:
  import requests
  requests.post('https://target.com/api/submit', 
    data={'name': 'test', 'age': -5, 'email': 'not_an_email'})
```

**Why it works:** Client-side validation only exists inside a browser. curl/Python are not browsers. They send raw HTTP requests with no validation at all.

---

### Method 5: Replay and Modify in Burp Repeater

**How:** Capture a valid request once, send it to Repeater, then modify and resend as many times as you want.

```
Step 1: Make one legitimate request (captured in Burp History)
Step 2: Right-click → Send to Repeater
Step 3: Modify any parameter
Step 4: Click Send
Step 5: Check response — did the server accept it?
```

---

## Every Type of Client-Side Validation and How to Bypass It

---

### 1. Form Field Length Limits

**What it looks like:**
```html
<input type="text" name="username" maxlength="20">
```

**What it does:** Browser prevents typing more than 20 characters.

**Bypass:**
- Inspect Element → remove `maxlength` attribute → type any length
- Burp → change username value to 500 characters
- Test: Does the server also limit length? Does a very long input cause errors?

**Why it matters:** Buffer overflow, database errors, or denial of service if server does not validate length.

---

### 2. Required Fields

**What it looks like:**
```html
<input type="email" name="email" required>
```

**What it does:** Browser shows "Please fill out this field" if empty.

**Bypass:**
- Inspect Element → remove `required` attribute → submit empty
- Burp → remove the parameter entirely or send empty value
- Test: Does the server handle null/empty values properly?

**Why it matters:** Missing required data might cause server errors, null pointer exceptions, or bypass business logic.

---

### 3. Input Type Restrictions

**What it looks like:**
```html
<input type="email" name="email">       <!-- must look like email -->
<input type="number" name="age">        <!-- only numbers -->
<input type="url" name="website">       <!-- must look like URL -->
<input type="tel" name="phone">         <!-- telephone format -->
<input type="date" name="birthday">     <!-- date picker -->
```

**What it does:** Browser checks format before submitting.

**Bypass:**
- Burp → send any format: age=abc, email=not-an-email, url=anything
- Change input type in DevTools: change `type="number"` to `type="text"`

**What to test:**
```
number field:    Send text, negative numbers, very large numbers, decimals, 0
email field:     Send text without @, special characters, very long strings
url field:       Send javascript:alert(1), file:///etc/passwd
date field:      Send invalid dates, future dates, negative dates
```

---

### 4. Dropdown / Select Menus

**What it looks like:**
```html
<select name="role">
  <option value="user">User</option>
  <option value="editor">Editor</option>
</select>
```

**What it does:** User can only pick from predefined options. No "admin" option visible.

**Bypass:**
- Inspect Element → add `<option value="admin">Admin</option>` → select it
- Burp → change `role=user` to `role=admin` in the request
- This is one of the MOST COMMON real-world vulnerabilities

**Why it matters:** Developers assume "the user can only pick from the dropdown" — but the dropdown is in YOUR browser. You decide what to send.

**What to test:**
```
Send values not in the dropdown: role=admin, role=superuser, role=root
Send empty: role=
Send unexpected types: role[]=admin, role=0
```

---

### 5. Hidden Fields

**What it looks like:**
```html
<input type="hidden" name="price" value="99.99">
<input type="hidden" name="user_id" value="1042">
<input type="hidden" name="discount" value="0">
<input type="hidden" name="is_admin" value="false">
```

**What it does:** Stores data in the form that the user "cannot see or change."

**Bypass:**
- Inspect Element → change `type="hidden"` to `type="text"` → edit value
- Burp → modify any hidden field value in the request

**What to test:**
```
price=0.01        → pay almost nothing
price=-10         → get credit?
user_id=1         → become admin
discount=100      → 100% discount
is_admin=true     → elevate privileges
```

**This is a goldmine.** Always check for hidden fields in every form.

---

### 6. Disabled / Readonly Fields

**What it looks like:**
```html
<input type="text" name="email" value="user@test.com" disabled>
<input type="text" name="role" value="user" readonly>
```

**What it does:** `disabled` — field not submitted with form at all. `readonly` — field submitted but cannot be edited in browser.

**Bypass:**
- Inspect Element → remove `disabled` or `readonly` attribute
- Burp → add the parameter manually (for disabled fields that were not sent)
- Burp → change the value (for readonly fields)

**Key difference:**
```
disabled: parameter NOT sent in request → add it yourself in Burp
readonly: parameter IS sent but "locked" → change it in Burp
```

---

### 7. JavaScript Validation Functions

**What it looks like:**
```javascript
function validateForm() {
  if (document.getElementById('age').value < 18) {
    alert('Must be 18 or older');
    return false;   // prevents form submission
  }
  if (!document.getElementById('email').value.includes('@')) {
    alert('Invalid email');
    return false;
  }
  return true;      // allows submission
}
```

**Bypass options:**
- **Console override:** Open DevTools Console and type:
  ```
  document.querySelector('form').onsubmit = null;
  ```
  This removes the validation function entirely.

- **Override the function:**
  ```
  validateForm = function() { return true; }
  ```
  Now the function always returns true (valid).

- **Disable JS entirely** → no validation runs

- **Burp** → let validation pass normally, modify in transit

---

### 8. Regex / Pattern Validation

**What it looks like:**
```html
<input type="text" name="phone" pattern="[0-9]{10}" title="10 digit number">
```

**What it does:** Browser checks input against regex pattern. Won't submit if it does not match.

**Bypass:**
- Inspect Element → remove `pattern` attribute
- Burp → send anything regardless of pattern
- The pattern only exists in HTML, not on the server

**What to test:** Send data that violates the pattern. If server accepts it, there is no server-side validation.

---

### 9. File Upload Client-Side Checks

**What it looks like:**
```html
<input type="file" name="avatar" accept=".jpg,.png">
```
```javascript
function checkFile(input) {
  var file = input.files[0];
  if (file.size > 2000000) { alert('Max 2MB'); return false; }
  if (!file.type.startsWith('image/')) { alert('Images only'); return false; }
}
```

**What it does:** Browser restricts file type via `accept` attribute and JS checks file size and MIME type.

**Bypass:**
- `accept` attribute → only affects the file picker dialog. You can still drag-drop any file or modify in Burp.
- File size check → Burp does not care about JS size checks
- MIME type check → change Content-Type header in Burp to image/jpeg

**What to test:**
```
Upload PHP/JSP/ASPX files with image Content-Type
Upload oversized files
Upload files with double extensions (.php.jpg)
Upload files with manipulated magic bytes
```

---

### 10. Client-Side Price / Calculation Validation

**What it looks like:**
```javascript
var quantity = parseInt(document.getElementById('qty').value);
var price = 29.99;
var total = quantity * price;
document.getElementById('total').value = total;
// Hidden field "total" is sent in form
```

**What it does:** JavaScript calculates the total on the page and sends it.

**Bypass:**
- Burp → change `total=29.99` to `total=0.01` in the request
- If the server trusts the client-calculated total, you pay whatever you want

**Rule:** If ANY value is calculated client-side and sent to server, it can be manipulated.

---

### 11. Character / Input Filtering (Blacklist)

**What it looks like:**
```javascript
function sanitize(input) {
  return input.replace(/[<>"']/g, '');  // removes special chars
}
```

**What it does:** JavaScript removes "dangerous" characters before submission.

**Bypass:**
- Burp → characters are removed in browser, but you add them back in Burp
- The server receives the unfiltered input
- Even if server ALSO filters, the client-side filter tells you WHAT they consider dangerous

**What to test:**
```
Characters removed client-side → test them server-side
If < and > are removed → they fear XSS → test XSS payloads via Burp
If ' is removed → they fear SQLi → test injection via Burp
```

---

### 12. Button / UI Disabling

**What it looks like:**
```html
<button id="deleteBtn" disabled>Delete Account</button>
<button id="adminBtn" style="display:none">Admin Panel</button>
<div class="premium-only" style="visibility:hidden">Premium Content</div>
```

**What it does:** Buttons are disabled, hidden, or invisible in the UI.

**Bypass:**
- Inspect Element → remove `disabled`, change `display:none` to `display:block`
- Or skip the UI entirely: find what API endpoint the button calls and call it directly via Burp/curl

**What to test:**
```
Hidden buttons often reveal admin or premium features
The API endpoint behind the button usually has NO server-side check
JavaScript often contains the endpoint URL — search for it
```

---

### 13. Anti-Automation / Timer Checks

**What it looks like:**
```javascript
var startTime = Date.now();
document.querySelector('form').onsubmit = function() {
  if (Date.now() - startTime < 5000) {
    alert('Please wait before submitting');
    return false;
  }
};
```

**What it does:** Prevents form submission within 5 seconds (anti-bot).

**Bypass:**
- Wait 5 seconds (if lazy)
- Console: remove the onsubmit handler
- Burp → submit directly (no timer in HTTP requests)

---

### 14. Referrer / Origin Checks (Client-Side)

**What it looks like:**
```javascript
if (document.referrer !== 'https://trusted-site.com') {
  window.location = '/error';
}
```

**Bypass:**
- Burp → add or modify Referer header: `Referer: https://trusted-site.com`
- This check is client-side anyway — it runs in your browser, you control it

---

### 15. Token / Nonce in Hidden Fields

**What it looks like:**
```html
<input type="hidden" name="csrf_token" value="a1b2c3d4e5">
<input type="hidden" name="nonce" value="12345">
```

**What to test:**
```
1. Remove the token entirely → does server still accept?
2. Send empty token → does server accept?
3. Send previous/old token → does server accept?
4. Send another user's token → does server accept?
5. Send random garbage as token → does server accept?
```

If any of these work, the token validation is broken.

---

## Step-by-Step Testing Methodology

```
For EVERY form/feature you test:

Step 1: USE IT NORMALLY
  → Fill everything correctly, submit, observe the request in Burp

Step 2: READ THE HTML SOURCE
  → Look for hidden fields, disabled buttons, JavaScript validation
  → Search for: hidden, disabled, readonly, maxlength, pattern, required
  → Search JS for: validate, check, verify, submit, onclick

Step 3: TEST EACH FIELD
  → Empty value
  → Very long value (1000+ chars)
  → Special characters (' " < > & ; | ` $)
  → Negative numbers (if numeric)
  → Zero
  → Wrong data type (text in number field, number in text field)
  → Array instead of string (param[]=value)
  → Null / undefined

Step 4: REMOVE OR MODIFY HIDDEN FIELDS
  → Change prices, user IDs, roles, flags
  → Add extra fields (is_admin=true, role=admin)

Step 5: BYPASS DROPDOWNS AND RADIO BUTTONS
  → Send values that are not in the options

Step 6: CHECK SERVER RESPONSE
  → Did the server accept the modified data?
  → Did it cause an error? (error = no server validation = potential exploit)
  → Did it change something it should not have? (success = vulnerability)
```

---

## Real Interview Scenario

**Interviewer:** "You see a form with client-side validation. Walk me through your approach."

**Your answer:**

> "Client-side validation runs in the user's browser, which means I control it. It is never a security boundary.
>
> My approach is:
>
> **First,** I use the form normally and capture the request in Burp Suite. This gives me a valid baseline request.
>
> **Second,** I read the HTML source looking for hidden fields, disabled fields, dropdown values, and JavaScript validation functions. Hidden fields with prices, user IDs, or role values are immediate targets.
>
> **Third,** I test each field by sending modified values through Burp — empty values, extremely long strings, wrong data types, special characters, and values outside the expected range like negative numbers.
>
> **Fourth,** I specifically test dropdown and radio button fields by sending values that are not in the visible options. For example, if a role dropdown only shows 'user' and 'editor', I send 'admin'.
>
> **Fifth,** I check if the server has its own validation. If the server accepts my modified data without complaint, the client-side validation was the ONLY protection, and the application is vulnerable.
>
> The key principle is: anything the browser enforces can be bypassed because I control the browser. Real security must happen on the server."

---

> **Remember:** When an interviewer asks about client-side validation, the answer is always: "It is not security. I bypass it with Burp or by sending requests directly. Then I test if the server has its own checks."
