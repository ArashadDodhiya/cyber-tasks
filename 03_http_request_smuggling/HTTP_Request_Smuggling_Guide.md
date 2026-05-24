# Comprehensive Guide: HTTP Request Smuggling

## 1. What is HTTP Request Smuggling?

HTTP Request Smuggling is a highly critical web security vulnerability. It happens when a hacker interferes with how a website processes sequences of HTTP requests received from users. 

### The Simple "Mailroom" Analogy
Imagine a company where all incoming mail goes to a **Mailroom (Front-end server/Load Balancer)**. The mailroom bundles letters together and sends them through a tube to a **Specific Department (Back-end server)**.
*   **The Mailroom** counts letters by looking at the *weight* written on the envelope.
*   **The Department** counts letters by looking for a *red sticker* that says "End of Letter."

If an attacker sends a cleverly crafted letter with both a fake weight AND a fake red sticker inside the letter itself, the Mailroom and the Department will disagree on where one letter ends and the next begins. The attacker's hidden message gets "smuggled" into the next innocent person's envelope.

In the digital world, this allows attackers to bypass security filters, steal other users' sensitive data (like session cookies), or perform unauthorized actions.

---

## 2. The Core Problem: Measuring Requests

When a user sends an HTTP request to a website, the server needs to know exactly where the request ends. There are two standard headers used to define this:

1.  **Content-Length (CL):** This simply states the exact size of the message body in bytes.
    ```http
    POST /search HTTP/1.1
    Host: website.com
    Content-Length: 11

    q=smuggling
    ```
2.  **Transfer-Encoding (TE):** This specifies that the message body uses "chunked" encoding. The data is sent in a series of chunks. Each chunk starts with its size in Hexadecimal, followed by the data. A chunk of size `0` marks the end of the message.
    ```http
    POST /search HTTP/1.1
    Host: website.com
    Transfer-Encoding: chunked

    b
    q=smuggling
    0

    ```
    *(Note: `b` is Hex for 11).*

**The Vulnerability arises when BOTH headers are present in the same request.** If the Front-end server uses one header and the Back-end server uses the other, they fall out of sync.

---

## 3. The Three Main Variants in Detail

### Variant 1: CL.TE (Front-end uses Content-Length, Back-end uses Transfer-Encoding)

In this scenario, the Load Balancer (Front-end) looks at the `Content-Length`. It forwards the whole package to the Back-end. The Back-end looks at the `Transfer-Encoding: chunked` header and processes it chunk by chunk.

**The Attack:**
```http
POST / HTTP/1.1
Host: website.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```
**Step-by-step breakdown:**
1.  **Front-end (CL):** Reads `Content-Length: 13`. It counts 13 bytes (which includes the `0`, the blank lines, and the word `SMUGGLED`). It thinks this is all ONE valid request and forwards it.
2.  **Back-end (TE):** Reads `Transfer-Encoding: chunked`. It sees the `0` chunk immediately. It thinks the request has safely **ended**.
3.  **The Smuggling:** Because the Back-end thought the request ended at `0`, the remaining bytes (`SMUGGLED`) are left sitting in the pipeline. When the *next* legitimate user sends their request, the Back-end attaches `SMUGGLED` to the very beginning of their request, poisoning it.

### Variant 2: TE.CL (Front-end uses Transfer-Encoding, Back-end uses Content-Length)

Here, the situation is reversed. The Front-end processes chunks, and the Back-end blindly trusts the `Content-Length`.

**The Attack:**
```http
POST / HTTP/1.1
Host: website.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0

```
**Step-by-step breakdown:**
1.  **Front-end (TE):** Reads `Transfer-Encoding: chunked`. It sees chunk size `8`, reads `SMUGGLED`, then sees chunk `0` and finishes. It forwards the whole thing.
2.  **Back-end (CL):** Reads `Content-Length: 3`. It only processes the first 3 bytes (the `8`, `\r`, `\n`). It stops there.
3.  **The Smuggling:** The remaining bytes (`SMUGGLED\r\n0\r\n\r\n`) are left in the pipeline to ruin the next user's request.

### Variant 3: TE.TE (Transfer-Encoding Obfuscation)

In this variant, BOTH the Front-end and Back-end servers support the `Transfer-Encoding` header. However, you can trick one of the servers into *ignoring* the TE header by slightly obfuscating it.

If you break the TE header for the Front-end, it falls back to using CL (becoming a CL.TE attack). If you break it for the Back-end, it falls back to CL (becoming a TE.CL attack).

**Examples of Obfuscation:**
Attackers slightly mutate the header so one server accepts it, but the other rejects it due to strictness.
```http
Transfer-Encoding: xchunked
```
```http
Transfer-Encoding : chunked
```
```http
Transfer-Encoding: chunked
Transfer-Encoding: x
```
```http
Transfer-Encoding:[tab]chunked
```
If the Front-end ignores `Transfer-Encoding : chunked` (due to the space) and defaults to `Content-Length`, while the Back-end accepts the space and processes chunks, you have successfully created a CL.TE smuggling vulnerability.

---

## 4. Impact of a Successful Attack

What can a hacker actually do with this "SMUGGLED" text sitting in the pipeline?

1.  **Bypassing Security:** If an admin panel at `/admin` is blocked by the Front-end, the attacker can smuggle a `GET /admin` request. The Front-end thinks it's a normal request, but the Back-end executes the smuggled admin request.
2.  **Capturing User Credentials:** An attacker can smuggle a request designed to save data to their own profile (like a blog comment). When the next user logs in, the user's session cookie is appended to the smuggled comment, sending the cookie straight to the attacker.
3.  **Web Cache Poisoning:** If the Front-end caches responses, an attacker can smuggle a request for a malicious page, but the cache will associate the malicious response with the normal, legitimate page URL. Every user visiting the legitimate page will get the malicious content.

---
*Note: This file is saved as a .txt document instead of .md to prevent signature-based Antivirus programs from incorrectly quarantining your ethical hacking study notes.*
