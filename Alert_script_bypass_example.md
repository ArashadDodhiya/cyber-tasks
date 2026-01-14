
---

![Image](https://portswigger.net/support/images/methodology_xss_filters_5.png)

![Image](https://media.geeksforgeeks.org/wp-content/uploads/20250923113454058543/3.webp)

## Given Malicious Input

```html
"><script>alert("foo")</script>
```

The goal is to **smuggle this through** the filter so that it **becomes dangerous only after decoding**.

---

## Key Weakness (Very Important)

👉 **URL decoding happens AFTER filtering**

That means:

* The filter looks at **encoded data**
* But the browser executes **decoded data**

This is a **canonicalization flaw**.

---

## Step-by-Step Bypass

### Step 1: URL-encode the dangerous parts

We encode `<script>` so the filter **doesn’t see it**.

Encoded payload:

```text
%22%3E%3Cscript%3Ealert(%22foo%22)%3C/script%3E
```

This is just a **safe-looking string** to the filter.

---

## Now Let’s Walk Through the Validation Steps

### 🔹 Step 1: Strip `<script>` expressions

❌ Nothing removed
Why? Because `<script>` is **URL-encoded**, not visible yet.

---

### 🔹 Step 2: Truncate to 50 characters

✔ Payload fits under 50 characters
Nothing breaks.

---

### 🔹 Step 3: Remove quotation marks (`"`)

❌ No quotes found
They are `%22`, not actual quotes.

---

### 🔹 Step 4: URL-decode the input

🔥 **Now the magic happens**

Decoded output becomes:

```html
"><script>alert("foo")</script>
```

---

### 🔹 Step 5: If items were deleted, repeat

❌ No deletion happened earlier
So **validation stops**

---

## Final Result

✔ The payload passes validation
✔ Browser receives real `<script>`
🚨 **XSS successfully smuggled**

---

## Why This Worked (In Simple Words)

* The filter cleaned the input **before understanding its real meaning**
* Dangerous content was **hidden using encoding**
* After decoding, it became active **but was never re-validated**

---

## Core Security Lesson (Exam Gold)

> **Always decode input before validating it.**

Correct order:

```
Decode → Normalize → Validate → Encode on output
```

Wrong order (used here):

```
Validate → Decode → Done ❌
```

---

## One-Line Answer (Exam Style)

> Yes, the validation can be bypassed by URL-encoding the `<script>` tags. Because decoding occurs after filtering, the malicious script is reconstructed after validation and executes in the browser.

---

## Bonus: Why Blacklist Filters Fail

This filter tried to:

* Remove `<script>`
* Remove quotes

Attackers just:

* Encode
* Split
* Reassemble later

That’s why **output encoding**, not input stripping, is the real defense.
