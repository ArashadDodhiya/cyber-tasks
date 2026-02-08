## 1️⃣ What is the `whois` command (Linux)?

### Simple definition

`whois` is a **query tool** that asks public domain/IP registration databases:

> “Who owns this domain or IP, and what metadata is publicly available?”

It does **NOT scan**, **NOT exploit**, and **NOT touch the target system directly**.

---

## 2️⃣ Why `whois` exists (conceptually)

Every domain or IP address must be registered with an authority:

* Domains → **Registrars** (GoDaddy, Namecheap, etc.)
* IPs → **RIRs** (ARIN, RIPE, APNIC, LACNIC, AFRINIC)

`whois` simply queries these registries.

---

## 3️⃣ Basic `whois` usage in Linux

### Install (if missing)

```bash
sudo apt install whois
```

### Domain lookup

```bash
whois example.com
```

### IP address lookup

```bash
whois 8.8.8.8
```

---

## 4️⃣ Output explained (VERY IMPORTANT)

A typical `whois` result contains:

### 🔹 Domain Information

* Domain name
* Registrar
* Registration date
* Expiry date
* Name servers

### 🔹 Ownership / Contact Info

* Registrant name (often hidden)
* Organization
* Country
* Abuse contact email

### 🔹 Network Information (for IPs)

* NetRange / CIDR
* ASN (Autonomous System Number)
* ISP / Hosting provider
* Abuse & admin contacts

---

### Example (simplified)

```text
Domain Name: example.com
Registrar: IANA
Creation Date: 1995-08-14
Registry Expiry Date: 2026-08-13
Name Server: A.IANA-SERVERS.NET
Registrant Country: US
```

---

## 5️⃣ What attackers / security analysts look for in `whois`

From an **offensive security** or **SOC** perspective:

| Info          | Why it matters                        |
| ------------- | ------------------------------------- |
| Registrar     | Domain takedown / abuse escalation    |
| Country       | Legal jurisdiction                    |
| ASN           | Map infrastructure                    |
| Name servers  | DNS attack surface                    |
| Email         | Phishing or abuse reporting           |
| Creation date | Newly registered domains = suspicious |

---

## 6️⃣ What is Passive Information Gathering?

### Core idea

**Passive Information Gathering = Collecting information WITHOUT touching the target system**

No packets sent to the victim
No scans
No alerts triggered

🧠 You’re only using **publicly available data**

---

## 7️⃣ Why passive recon is critical

In real attacks:

1. Passive recon comes **first**
2. Active recon comes **later**
3. Good passive recon = less noise = less detection

Blue teams also use passive recon to:

* Track attacker infrastructure
* Investigate phishing domains
* Attribute threats

---

## 8️⃣ Examples of Passive Information Gathering

### 🔹 Domain-based

* `whois`
* DNS records (from public resolvers)
* Certificate Transparency logs

### 🔹 Network-based

* ASN lookups
* IP ownership info
* Hosting provider data

### 🔹 Organization-based

* Company websites
* Job postings (tech stack leaks)
* GitHub repos
* LinkedIn employee roles

### 🔹 Search-engine based

* Google dorking
* Shodan (search engine, still passive)
* Censys

---

## 9️⃣ How `whois` fits into Passive Recon

`whois` is **100% passive** because:

* You query a **registry**, not the target
* The target never knows
* Logs are not generated on victim systems

### Typical recon flow

```text
Domain → whois → registrar → ASN → IP ranges → hosting provider
```

---

## 🔐 10️⃣ Passive vs Active Recon (clear difference)

| Passive     | Active                 |
| ----------- | ---------------------- |
| whois       | nmap                   |
| Google      | port scanning          |
| DNS history | vulnerability scanning |
| Shodan      | brute forcing          |

👉 `whois` = **safe, silent, legal (usually)**
👉 Active scans = **noisy, logged, risky**

---

## 11️⃣ Real-world security use cases

### 🛡 SOC / Blue Team

* Investigate phishing domains
* Identify malicious infrastructure
* Correlate IP ownership

### 🗡 Pentesting / Red Team

* Map target infrastructure silently
* Discover parent organizations
* Identify shared hosting weaknesses

---

## 12️⃣ Limitations of `whois`

* Privacy protection hides real owners
* Data can be outdated
* No technical vulnerability info
* Some registries throttle requests


* command - “whois [example.com](http://example.com/)”
* or [netcraft.com](http://netcraft.com) can also do this process 
* we can use the [shodan.io](http://shodan.io) also → hacker’s google for information about target