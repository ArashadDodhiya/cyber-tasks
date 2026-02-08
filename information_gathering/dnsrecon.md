# 🔴 What is `dnsrecon`?

**dnsrecon** is a **DNS reconnaissance tool** used to:

* Enumerate DNS records
* Discover subdomains
* Identify misconfigurations
* Attempt zone transfers

👉 It is **ACTIVE information gathering** because it sends DNS queries to name servers.

---

# 1️⃣ Command 1

```bash
dnsrecon -d <domain_name> -t std
```
    
## 🔍 Meaning of each part

### `dnsrecon`

* The tool itself

---

### `-d <domain_name>`

* Target domain
* Example:

```bash
-d example.com
```

---

### `-t std` → **Standard scan**

This tells `dnsrecon` to perform a **standard DNS enumeration**.

---

## 🧠 What `-t std` actually does internally

`std` is **not one scan**, it’s a **bundle of DNS checks**:

It queries:

| Record Type | Purpose            |
| ----------- | ------------------ |
| A           | IPv4 addresses     |
| AAAA        | IPv6 addresses     |
| MX          | Mail servers       |
| NS          | Name servers       |
| SOA         | Zone authority     |
| TXT         | SPF / verification |
| SRV         | Service discovery  |

It also:

* Checks for **zone transfer possibility**
* Tries basic name server enumeration

---

## 🧪 Example output (simplified)

```text
[*] SOA ns1.example.com
[*] NS ns1.example.com
[*] NS ns2.example.com
[*] MX mail.example.com
[*] A example.com 93.184.216.34
```

---

## 🎯 Use case

* First active DNS touch
* Baseline DNS mapping
* Safer than brute force
* Low-to-medium noise

---

# 2️⃣ Command 2

```bash
dnsrecon <domain_name> -D -t brt
```

⚠️ This command has a **syntax issue** as written
Let me correct it first.

---

## ✅ Correct command

```bash
dnsrecon -d <domain_name> -D <wordlist.txt> -t brt
```

---

## 🔍 Meaning of each part

### `-d <domain_name>`

Target domain

---

### `-D <wordlist.txt>`

**Dictionary file** (wordlist)

Example entries:

```text
www
mail
admin
dev
test
api
```

---

### `-t brt` → **Brute force scan**

This tells dnsrecon:

> “Try combining each word in the list with the domain
> and check if that subdomain exists.”

---

## 🧠 What `-t brt` does internally

For each word in the list:

```text
www.example.com
mail.example.com
admin.example.com
```

It sends:

* DNS A queries
* Sometimes AAAA queries

This is:

* 🔥 Much noisier
* 🔥 Much more detectable

---

## 🧪 Example output

```text
[+] Found A www.example.com 93.184.216.34
[+] Found A mail.example.com 93.184.216.35
```

---

## 🔴 Active Recon Warning (important)

`-t brt`:

* Generates **hundreds/thousands** of DNS requests
* Can trigger:

  * DNS rate limiting
  * SOC alerts
  * ISP monitoring

---

# 🆚 `std` vs `brt` (clear comparison)

| Feature             | std     | brt         |
| ------------------- | ------- | ----------- |
| Query count         | Low     | High        |
| Subdomain discovery | Limited | Extensive   |
| Noise               | Low     | High        |
| Wordlist needed     | ❌       | ✅           |
| Detection risk      | Low     | Medium–High |

---

# 🔐 Blue Team Detection View

SOC sees:

```text
Repeated NXDOMAIN queries
Same source IP
Sequential subdomain patterns
```

Classic brute-force signature.

---

# 🧠 When to use what

### Use `std`

* First active recon step
* Low-risk environments
* Quick DNS mapping

### Use `brt`

* Authorized pentest
* Bug bounty scope
* Deep subdomain discovery

---

# ⚠️ Legal note (can’t skip this)

> DNS brute forcing is **ILLEGAL without permission**.

Only use on:

* Labs
* Allowed scopes
* Your own domains

---

# 🔑 Final takeaway (burn this in)

> `-t std` = **Ask politely**
> `-t brt` = **Start guessing names**