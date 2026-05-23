# Comprehensive Guide: Encoding, Hashing, and Encryption

This guide covers everything from the basics to advanced concepts required for the **"2. Every decoding is another encoding"** task. It breaks down the differences between encoding, hashing, and encryption, and provides practical examples, code snippets, and terminal commands to complete each objective.

---

## Part 1: Core Concepts (The Basics)

Before diving into the specific tasks, it is crucial to understand the fundamental differences between the three main pillars of data transformation:

1. **Encoding:** 
   - **What it is:** The process of converting data into a new format using a publicly available scheme so that it can be easily reversed (decoded). 
   - **Purpose:** Data usability and transfer. It ensures data can be safely transmitted over protocols that might not support raw binary data.
   - **Key:** **No key is required.** Anyone can decode encoded data.
   - **Examples:** Base64, Hexadecimal, ASCII, URL Encoding.

2. **Hashing:**
   - **What it is:** A mathematical algorithm that maps data of arbitrary size to a fixed-size string of characters. 
   - **Purpose:** Data integrity verification. It is a **one-way** function; you cannot computationally "unhash" data to retrieve the original input. Even a tiny change in the input produces a drastically different hash (the "Avalanche Effect").
   - **Key:** **No key is required** (except in specific cases like HMACs).
   - **Examples:** MD5, SHA-1, SHA-256.

3. **Encryption:**
   - **What it is:** The process of scrambling data into a secure, unreadable format that can only be read by someone possessing the correct key.
   - **Purpose:** Data confidentiality and security. It is a **two-way** process (encryption and decryption).
   - **Key:** **A key is strictly required.**
   - **Examples:** AES (Symmetric), RSA (Asymmetric).

---

## Part 2: Task Breakdown & Advanced Implementation

### Task 1. Decoding Messages

*   **The Goal:** The attachment "1. Decode messages.xlsx" contains strings that have been transformed using various encoding methods. You must identify the scheme and reverse it.
*   **Common Encodings to recognize:**
    *   **Base64:** Uses `A-Z`, `a-z`, `0-9`, `+`, `/` and often ends with `=` or `==` as padding. (Example: `SGVsbG8=` -> "Hello")
    *   **Hexadecimal (Hex):** Uses numbers `0-9` and letters `A-F`. (Example: `48656c6c6f` -> "Hello")
    *   **Binary:** Uses only `0`s and `1`s.
    *   **Caesar Cipher / ROT13:** A simple substitution cipher where letters are shifted in the alphabet.
*   **Advanced Pro-Tip:** Use [CyberChef](https://gchq.github.io/CyberChef/) (known as the "Cyber Swiss Army Knife"). You can paste your strings there and use the "Magic" module, which utilizes brute force and heuristics to automatically detect and decode the message.

---

### Task 2 & 3. Hash Collisions (MD5 and SHA-1)

*   **The Goal:** Find two different strings or files that result in the exact same hash output.
*   **The Concept:** A robust cryptographic hash function should be "collision-resistant." A **collision** happens when `Hash(Input A) == Hash(Input B)` even though `Input A != Input B`. 
*   **Advanced Reality:**
    *   **MD5 Collisions:** MD5 is cryptographically broken. It is computationally trivial to find two different files or strings with the same MD5 hash today. Researchers like Xiaoyun Wang pioneered these attacks. There are many known MD5 collision strings available online, and tools like `fastcoll` can generate two completely different executable files that share the same MD5 hash.
    *   **SHA-1 Collisions:** SHA-1 is also broken. In 2017, researchers from Google and CWI Amsterdam executed the **SHAttered** attack, demonstrating the first practical technique for generating two different PDF files with the identical SHA-1 hash.
*   **How to complete this:** You do not need a supercomputer to calculate these yourself. You can search online for "known MD5 collision strings" and download the "SHAttered PDFs" to satisfy these requirements.

---

### Task 4. Automated SHA1 Hash Generation (Brute-forcing)

*   **The Goal:** Generate SHA-1 hashes for every 4-letter alphabetical combination from `AAAA` to `ZZZZ` and output it to a CSV file.
*   **The Concept:** This is essentially generating a "Rainbow Table." Rainbow tables are precomputed dictionaries used by hackers to rapidly reverse hashed passwords.
*   **Advanced Implementation (Python):** Calculating $26^4$ (456,976) hashes manually is impossible, so we use automation. Here is the Python script to do it efficiently:

```python
import hashlib
import itertools
import csv

# Define the character set (A-Z) and string length (4)
alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
length = 4

print(f"Generating combinations and calculating SHA-1 hashes...")

# Open a CSV file for writing
with open('sha1_hashes.csv', 'w', newline='') as file:
    writer = csv.writer(file)
    
    # Generate the cartesian product (all combinations)
    for combo in itertools.product(alphabet, repeat=length):
        # Join the tuple into a single string (e.g., ('A', 'A', 'B', 'C') -> "AABC")
        string_val = "".join(combo)
        
        # Calculate SHA1 hash
        # .encode() converts string to bytes, .hexdigest() returns the readable hex string
        sha1_hash = hashlib.sha1(string_val.encode()).hexdigest()
        
        # Write the string and its hash to the CSV
        writer.writerow([string_val, sha1_hash])
        
print("Success! File saved as sha1_hashes.csv")
```

---

### Task 5. Symmetric Encryption (AES-256) with OpenSSL

*   **The Goal:** Encrypt a string and a file using AES-256 with a specific Key, IV, and Salt.
*   **The Concepts:**
    *   **Symmetric Encryption:** Uses the **exact same key** for both locking (encrypting) and unlocking (decrypting) the data.
    *   **AES-256:** Advanced Encryption Standard utilizing a massive 256-bit key. It is the military-grade standard.
    *   **Salt:** Random data added to a password before deriving the actual encryption key. It defends against precomputed dictionary attacks.
    *   **IV (Initialization Vector):** A random block of data used to start the encryption process. It ensures that if you encrypt the exact same file twice with the same key, the resulting ciphertext will look completely different, preventing pattern analysis.
*   **OpenSSL Execution:**
    If you want OpenSSL to handle the Salt and derive the Key/IV automatically from a password:
    
    **Encrypting:**
    ```bash
    openssl enc -aes-256-cbc -salt -in my_file.txt -out encrypted_file.enc -pass pass:MySecretPassword
    ```
    
    **Decrypting:**
    ```bash
    openssl enc -d -aes-256-cbc -in encrypted_file.enc -out decrypted_file.txt -pass pass:MySecretPassword
    ```

    *Advanced (Strict manual Key & IV mode):* If you want to explicitly define the Hex keys:
    ```bash
    # Generate 256-bit (32 byte) Hex Key and 128-bit (16 byte) Hex IV
    openssl rand -hex 32  
    openssl rand -hex 16  
    
    # Encrypt (replace <key> and <iv> with generated hex strings)
    openssl enc -aes-256-cbc -in my_file.txt -out encrypted.enc -K <key> -iv <iv>
    
    # Decrypt
    openssl enc -d -aes-256-cbc -in encrypted.enc -out decrypted.txt -K <key> -iv <iv>
    ```

---

### Task 6. Asymmetric Encryption (RSA 2048) with OpenSSL

*   **The Goal:** Generate RSA 2048-bit keys, encrypt a file with the public key, and decrypt it with the private key.
*   **The Concepts:**
    *   **Asymmetric Encryption:** Uses a mathematically linked **Key Pair**. 
    *   **Public Key:** Shared openly with everyone. Used *only* to encrypt data intended for you.
    *   **Private Key:** Kept strictly secret. Used *only* to decrypt data that was encrypted with your public key.
    *   Because the math is complex, RSA is slow and is generally used to encrypt small things (like AES keys or digital signatures), not large files directly.
*   **OpenSSL Execution:**

    **1. Generate the Private Key (2048-bit):**
    ```bash
    openssl genrsa -out my_private_key.pem 2048
    ```
    
    **2. Extract the Public Key from the Private Key:**
    ```bash
    openssl rsa -in my_private_key.pem -pubout -out my_public_key.pem
    ```
    
    **3. Encrypt a text file using the Public Key:**
    ```bash
    openssl pkeyutl -encrypt -pubin -inkey my_public_key.pem -in secret.txt -out secret.enc
    ```
    
    **4. Decrypt the file using the Private Key:**
    ```bash
    openssl pkeyutl -decrypt -inkey my_private_key.pem -in secret.enc -out decrypted_secret.txt
    ```

---

### Task 7. PGP and Mailvelope Workflow

*   **The Goal:** Set up PGP using Mailvelope, find a specific public key, and send an encrypted email + attachment.
*   **The Concepts:**
    *   **PGP (Pretty Good Privacy):** A standard utilizing both symmetric and asymmetric cryptography to provide end-to-end encryption for emails and files.
    *   **Mailvelope:** An open-source browser extension that brings PGP encryption directly into webmail clients (Gmail, Outlook, etc.) seamlessly.
*   **The Advanced Workflow:**
    1.  **Installation & Keygen:** Install the Mailvelope extension in Chrome/Firefox. Open its dashboard and generate your own PGP Key Pair.
    2.  **The Web of Trust (Key Servers):** PGP works by looking up people's public keys on distributed servers. To email `ravi@net-square.com`, you need his lock (Public Key).
    3.  **Finding the Target Key:** Go to Mailvelope's "Import" tab and search for `ravi@net-square.com`. Mailvelope queries global public key servers (like the Ubuntu key server or `keys.openpgp.org`).
    4.  **Import:** Once found, import his public key into your Mailvelope Keyring.
    5.  **Sending the Email:** Go to your webmail (e.g., Gmail). Click compose. You will see a Mailvelope icon in the compose window. Click it to open the secure editor.
    6.  **Encryption:** Because you have Ravi's public key, Mailvelope will automatically encrypt the message body and any attachments using his public key before it even leaves your browser.
    7.  **Result:** The email sent over the internet will look like garbled PGP ciphertext. Only Ravi, who holds the corresponding Private Key on his machine, will be able to decrypt and read it.

---

# Cryptography Concepts Reference

## 🟢 Beginner Level

### Encoding vs. Hashing vs. Encryption
It is crucial to understand the difference between these three concepts:

*   **Encoding:** This is simply **translation**. It changes data into a new format for safe transport across different systems. 
    *   *Analogy:* Translating an English book into Morse Code.
    *   *Key:* **No secret key.** Anyone who knows the encoding method can decode it.
    *   *Example:* **Base64**. If you encode `Hello` into Base64, it becomes `SGVsbG8=`. Anyone can reverse this back to `Hello`.
*   **Hashing:** This creates a **digital fingerprint**. It is a one-way mathematical process that turns any amount of data into a fixed-size string of characters.
    *   *Analogy:* Taking a person's fingerprint. You can identify the person from the fingerprint, but you cannot rebuild the person just from the fingerprint.
    *   *Key:* **No secret key.** It's a one-way street.
    *   *Example:* If you hash `Hello` using MD5, it becomes `8b1a9953c4611296a827abf8c47804d7`. You cannot easily reverse this to get `Hello`.
*   **Encryption:** This is a **secure lockbox**. It scrambles data so that it can only be read by someone who has the correct key.
    *   *Analogy:* Putting a letter in a safe and locking it with a key.
    *   *Key:* **Requires a secret key.** It is a two-way street (encrypt and decrypt).
    *   *Example:* **AES-256**. Encrypting `Hello` with a secret password turns it into gibberish. Only the person with the password can decrypt it back to `Hello`.

### MD5 & SHA Basics
These are specific mathematical algorithms used for hashing.
*   **MD5 (Message Digest 5):** Produces a 128-bit hash (32 hex characters). It is fast but **cryptographically broken**. It should not be used for security anymore because it is too easy to find collisions.
*   **SHA (Secure Hash Algorithm):** A family of hash functions.
    *   **SHA-1:** Produces a 160-bit hash. Like MD5, it is now considered broken and unsafe.
    *   **SHA-256 (Part of SHA-2):** Produces a 256-bit hash. It is the current industry standard and is considered highly secure. It's used in Bitcoin and standard SSL certificates.

### Checksums & Integrity
*   **Checksum:** A checksum is simply a hash value used specifically to check for errors in files. When you download a large file, the website might provide an MD5 or SHA-256 checksum. After downloading, you hash the file on your computer. If your hash matches the website's hash, the download was successful.
*   **Integrity:** This means data has not been altered or tampered with. Hashes provide integrity due to the **Avalanche Effect**: changing even a single comma in a 1-gigabyte file will completely and drastically change the final hash. 

---

## 🟡 Intermediate Level

### Salting
If two users have the same password (e.g., "password123"), their database hashes will be identical. Hackers can use this to their advantage.
*   **What it is:** A "Salt" is random data added to a password *before* it is hashed.
*   **Example:** Instead of hashing `password123`, the system generates a random salt `Zq9#x` and hashes `password123Zq9#x`. Even if two users have the password "password123", their salts are different, so their final hashes are completely different. This protects against pre-computed tables.

### Rainbow Tables
*   **What it is:** A massive, pre-calculated database mapping plain-text passwords to their corresponding hashes.
*   **How it works:** Instead of trying to guess a password one by one (which takes time), a hacker steals a hashed password, looks it up in the Rainbow Table, and instantly finds the original text. **Salting completely defeats Rainbow Tables** because the tables do not account for random, unique salts for every user.

### Password Cracking
When hackers steal a database of hashed passwords, they try to reverse them using two main methods:
*   **Brute-Force Attack:** The computer guesses every possible combination of letters, numbers, and symbols (`a`, `b`, `c`... `aa`, `ab`...). It is guaranteed to work eventually but can take millions of years for long passwords.
*   **Dictionary Attack:** The computer uses a massive text file containing known passwords, common words, and leaked passwords (like the famous "rockyou.txt") and hashes each one to see if it matches the stolen hash. This is much faster and highly effective against weak passwords.

### Collision Attacks
A collision happens when a hash function takes **two different inputs** but spits out the **exact same hash output**. 
*   **Example:** If `Hash(File_A) == Hash(File_B)`, that is a collision.
*   **Why it's bad:** If MD5 is used to verify software updates, an attacker could write a virus, manipulate the virus's file bytes until its MD5 hash matches a legitimate software update, and trick your computer into installing the virus. MD5 and SHA-1 are vulnerable to practical collision attacks today.

### Digital Signatures
A digital signature proves that a message was created by a known sender and wasn't altered in transit. It relies on **Asymmetric Encryption** (Public/Private Key pairs).
*   **How it works:**
    1.  Alice writes a document. She hashes the document.
    2.  Alice **encrypts the hash** using her **Private Key**. This encrypted hash is the "Digital Signature."
    3.  Alice sends the document and the signature to Bob.
    4.  Bob hashes the document himself. He then uses Alice's **Public Key** to decrypt the signature.
    5.  If Bob's hash matches the decrypted hash, he knows the document is exactly as Alice wrote it (Integrity) and that only Alice could have signed it (Authenticity).

### Essential Tools
*   **`md5sum` / `sha1sum` / `sha256sum`:** Built-in Linux/macOS command-line tools used to quickly generate the checksum of a file.
*   **Hashcat:** The world's fastest and most advanced password recovery utility. It uses the massive parallel processing power of Graphics Cards (GPUs) to brute-force hashes at blistering speeds.
*   **John the Ripper (John):** A famous, versatile, and highly customizable password cracker. It excels at dictionary attacks and rule-based cracking (e.g., trying "Password123" if the dictionary has "password"). Often used heavily with CPUs.

---

## 🔴 Advanced Level

### Merkle–Damgård Construction
This is the internal "engine" inside many older hash functions, including MD5, SHA-1, and SHA-2.
*   **How it works:** Hash functions cannot process a 10GB file all at once. The Merkle-Damgård construction breaks the input data into fixed-size blocks (e.g., 512 bits). It takes the first block, mixes it with an initialization vector using a "compression function," takes that output, mixes it with the second block, and so on, chaining them together until the final block produces the resulting hash.

### Birthday Attacks
This is a mathematical attack against hash functions based on the "Birthday Paradox" (in a room of just 23 people, there is a 50% chance two people share a birthday).
*   **Concept:** In cryptography, it means it is significantly faster to find *any two random files* that share the same hash (a generic collision) than it is to take *one specific file* and try to find another file that matches its hash (a preimage attack). 

### Chosen-Prefix Collisions
This is a highly advanced and dangerous type of collision attack.
*   **Concept:** In a standard collision, attackers just generate two random garbage files that collide. In a **chosen-prefix collision**, the attacker can choose the beginning (the prefix) of two different files (e.g., the header of a legitimate PDF and the header of a malicious PDF), and then calculate a specific "collision block" of data to append to both files so they end up with the exact same hash.
*   **Real-world impact:** The infamous **Flame Malware (2012)** used a chosen-prefix collision attack against MD5 to forge a Microsoft digital certificate, tricking Windows into installing malware as a legitimate Windows Update.

### Length Extension Attacks
A specific vulnerability targeting hash algorithms built on the Merkle-Damgård construction (like SHA-256).
*   **The Flaw:** If a server uses a secret key and data to create a hash `Hash(Secret + Data)` and gives that hash to a user, the user can append their own malicious data to the end and calculate the new valid hash `Hash(Secret + Data + MaliciousData)` **without ever knowing the Secret key**. 
*   **Impact:** This breaks custom authentication systems that naively use simple concatenation `Hash(Secret + Message)` to verify API requests or cookies.

### HMAC (Hash-based Message Authentication Code)
HMAC is the specific cryptographic solution to Length Extension Attacks.
*   **What it is:** Instead of simply concatenating `Secret + Data`, HMAC uses a specific algorithm that mixes the secret key with the message data, hashes it, then mixes the key in *again*, and hashes it a second time. 
*   **Use Case:** Modern web APIs (like AWS, Stripe, or JWT tokens) use HMAC-SHA256 to securely verify that a web request hasn't been tampered with and genuinely comes from someone holding the correct API secret key.

### PKI (Public Key Infrastructure)
PKI is the massive, global framework of systems, policies, and mathematics that manages digital keys and certificates. It is what makes the modern secure internet possible.
*   **How it works:** PKI relies on **Certificate Authorities (CAs)**—highly trusted organizations (like Let's Encrypt, DigiCert). If you want to prove your website is really `google.com`, you generate a Public/Private key pair. You send your Public Key to a CA. The CA verifies you own the domain, then uses *their* highly guarded Private Key to digitally sign your Public Key. This signed package is your Certificate.

### TLS Certificates (Transport Layer Security)
TLS (the modern successor to SSL) uses certificates generated by PKI to secure web traffic (HTTPS).
*   **The Handshake:** When you visit `https://bank.com`:
    1.  The bank's server sends your browser its **TLS Certificate**.
    2.  Your browser checks the digital signature on the certificate using the Public Keys of trusted CAs already installed on your computer.
    3.  Once verified, your browser and the server use the Bank's Public Key (found in the certificate) to securely agree on a temporary, super-fast **Symmetric Key** (like AES-256).
    4.  All your banking traffic is then encrypted using that symmetric key. This combines the trust and authenticity of Asymmetric encryption (PKI) with the sheer speed of Symmetric encryption.
