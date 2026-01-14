
**Headers = extra information about the HTTP message**

They tell the browser and server:

* How to handle the connection
* How to read the body
* What format the data is in
* How the data was sent

---

## 1️⃣ `Connection` header

### 📌 What it means (simple)

Tells the server:

> “After this request, should we **keep the connection open** or **close it**?”

---

### Example

```http
Connection: keep-alive
```

➡ Reuse the same TCP connection for more requests (faster)

```http
Connection: close
```

➡ Close the connection after this response

---

### Why it matters

* `keep-alive` = faster websites
* `close` = more secure in some cases, but slower

---

## 2️⃣ `Content-Encoding`

### 📌 What it means (simple)

Tells:

> “The response body is **compressed or encoded** in this way.”

---

### Example

```http
Content-Encoding: gzip
```

Meaning:

* Server compressed the content
* Browser must **decompress** it before displaying

---

### Why it matters

* Faster data transfer 🚀
* Common encodings:

  * `gzip`
  * `br` (Brotli)
  * `deflate`

---

## 3️⃣ `Content-Length`

### 📌 What it means (simple)

Tells:

> “The body of this message is **X bytes long**.”

---

### Example

```http
Content-Length: 1024
```

Meaning:

* Browser expects exactly **1024 bytes** in the body

---

### Special case (HEAD request)

For `HEAD`:

* No body is sent
* `Content-Length` tells how big the body **would be** in a `GET`

---

### Security note 🚨

Incorrect `Content-Length` can cause:

* Request smuggling
* Response splitting

(very important in pentesting)

---

## 4️⃣ `Content-Type`

### 📌 What it means (simple)

Tells:

> “What kind of data is in the message body?”

---

### Examples

```http
Content-Type: text/html
```

➡ HTML page

```http
Content-Type: application/json
```

➡ JSON API response

```http
Content-Type: image/png
```

➡ PNG image

---

### Why it matters

* Browser decides **how to render data**
* Wrong type → XSS risk 🚨

Example:

```http
Content-Type: text/html
```

but body contains user input → possible script execution

---

## 5️⃣ `Transfer-Encoding`

### 📌 What it means (simple)

Tells:

> “The body was sent in a **special transfer format**.”

Most common:

```http
Transfer-Encoding: chunked
```

---

### Chunked encoding explained simply

Instead of sending:

* Full body at once
* With a known length

Server sends:

* Data in **small chunks**
* Ends when a `0` chunk is sent

---

### Example (simplified)

```http
Transfer-Encoding: chunked

4
Wiki
5
pedia
0
```

➡ Browser joins chunks → `Wikipedia`

---

### Why it matters (SECURITY 🔥)

If both headers appear:

```http
Content-Length: 100
Transfer-Encoding: chunked
```

➡ **HTTP request smuggling risk**

Attackers exploit disagreement between:

* Front-end server
* Back-end server

---

## 🧠 Quick comparison table

| Header            | Purpose                |
| ----------------- | ---------------------- |
| Connection        | Keep TCP open or close |
| Content-Encoding  | Compression of body    |
| Content-Length    | Size of body           |
| Content-Type      | Type of data           |
| Transfer-Encoding | How body is sent       |

---

## 🧪 Real response example (everything together)

```http
HTTP/1.1 200 OK
Connection: keep-alive
Content-Type: text/html
Content-Encoding: gzip
Content-Length: 2048
```

Meaning:

* Connection stays open
* HTML content
* Compressed with gzip
* Body size = 2048 bytes

---

## 🛡️ Pentester mindset (important for you)

Always check:

* Mismatch between `Content-Length` & actual body
* `Transfer-Encoding + Content-Length` together
* Wrong `Content-Type`
* Unexpected `keep-alive`

These lead to **serious vulnerabilities**.

---

## 🧠 One-line memory trick

> **C**onnection = link
> **E**ncoding = compression
> **L**ength = size
> **T**ype = format
> **T**ransfer = delivery method
