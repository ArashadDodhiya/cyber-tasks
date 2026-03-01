# 🔀 Phase 7: Port Redirection & Tunneling (Week 13)

> **Goal**: Master pivoting through compromised hosts to reach internal networks.
> **Your existing notes**: `port_redir_tunneling/` folder (11 docs!)

---

## 📋 Table of Contents

1. [Concepts & Setup](#1-concepts--setup)
2. [SSH Tunneling](#2-ssh-tunneling)
3. [Chisel (HTTP Tunneling)](#3-chisel-http-tunneling)
4. [Socat (Port Forwarding)](#4-socat-port-forwarding)
5. [Ligolo-ng (Modern Tunneling)](#5-ligolo-ng-modern-tunneling)
6. [SSHuttle](#6-sshuttle)
7. [Proxychains](#7-proxychains)
8. [Windows Tunneling (netsh, plink)](#8-windows-tunneling)
9. [Double Pivoting](#9-double-pivoting)
10. [Practice Scenarios](#10-practice-scenarios)
11. [Weekly Schedule](#11-weekly-schedule)
12. [Checklist & Progress Tracker](#12-checklist--progress-tracker)

---

## 1. Concepts & Setup

### What is Pivoting?

```
Scenario: You compromised Machine A, but Machine B is only accessible FROM Machine A.

┌──────┐     ┌──────────┐     ┌──────────┐
│ Kali │────│ Machine A │────│ Machine B │
│ .10  │     │ .20  .30 │     │    .40   │
└──────┘     └──────────┘     └──────────┘
 Network 1    Network 1+2      Network 2

You need to "pivot" through Machine A to reach Machine B.
```

### Types of Tunneling

| Type                     | Direction         | Use Case                                      |
| ------------------------ | ----------------- | --------------------------------------------- |
| **Local Port Forward**   | Remote → Local    | Access remote service through your machine    |
| **Remote Port Forward**  | Local → Remote    | Expose your service through target            |
| **Dynamic Port Forward** | SOCKS Proxy       | Route all traffic through target              |
| **Reverse Tunnel**       | Target → Attacker | When target can't accept incoming connections |

### Lab Setup

```
Option 1: Metasploitable (dual-homed)
- NIC 1: hacklab (192.168.56.x) — reachable from Kali
- NIC 2: internal (10.0.0.x) — internal network only

Option 2: Add a third VM
- VM3 on internal network only (10.0.0.x)
- Machine A bridges both networks
```

---

## 2. SSH Tunneling

### 2.1 — Local Port Forwarding

```bash
# Scenario: Machine B (10.0.0.40) runs a web server on port 80
# You can SSH to Machine A (192.168.56.20)

# Forward Machine B's port 80 to your port 8080
ssh -L 8080:10.0.0.40:80 msfadmin@192.168.56.20

# Now access Machine B's web server:
curl http://localhost:8080
# Or open http://localhost:8080 in your browser

# Multiple ports
ssh -L 8080:10.0.0.40:80 -L 3306:10.0.0.40:3306 msfadmin@192.168.56.20
```

### 2.2 — Remote Port Forwarding

```bash
# Scenario: Expose Kali's port 80 through Machine A
# Useful when target can reach Kali but not directly

ssh -R 8080:localhost:80 msfadmin@192.168.56.20
# Now Machine A's port 8080 forwards to Kali's port 80
# Any machine that can reach Machine A can access your web server
```

### 2.3 — Dynamic Port Forwarding (SOCKS Proxy)

```bash
# Create a SOCKS5 proxy through Machine A
ssh -D 9050 msfadmin@192.168.56.20

# Now configure proxychains to use this proxy
# Edit /etc/proxychains4.conf:
# socks5 127.0.0.1 9050

# Use proxychains to route any tool through the tunnel
proxychains nmap -sT 10.0.0.40
proxychains curl http://10.0.0.40
proxychains firefox
```

### 2.4 — SSH Tips

```bash
# Background (-f) and no terminal (-N)
ssh -f -N -D 9050 msfadmin@192.168.56.20

# Keep alive
ssh -o ServerAliveInterval=60 -D 9050 msfadmin@192.168.56.20

# Compress traffic
ssh -C -D 9050 msfadmin@192.168.56.20

# Using SSH key instead of password
ssh -i id_rsa -D 9050 user@192.168.56.20
```

---

## 3. Chisel (HTTP Tunneling)

> 🔗 Download from: https://github.com/jpillora/chisel/releases

### 3.1 — Forward Proxy (SOCKS)

```bash
# On Kali (server)
./chisel server --reverse -p 8000

# On Target (client)
./chisel client 192.168.56.10:8000 R:9050:socks

# Now configure proxychains: socks5 127.0.0.1 9050
proxychains nmap -sT 10.0.0.40
```

### 3.2 — Port Forwarding

```bash
# On Kali (server)
./chisel server --reverse -p 8000

# On Target — forward specific port
./chisel client 192.168.56.10:8000 R:8080:10.0.0.40:80

# Now access: http://localhost:8080
```

### 3.3 — Why Chisel?

- Works over HTTP/HTTPS (bypasses firewalls)
- Single binary, no dependencies
- Cross-platform (Linux, Windows, macOS)
- No SSH required on target

---

## 4. Socat (Port Forwarding)

```bash
# Simple port forward
# Listen on port 8080, forward to 10.0.0.40:80
socat TCP-LISTEN:8080,fork TCP:10.0.0.40:80

# Port forward in background
socat TCP-LISTEN:8080,fork TCP:10.0.0.40:80 &

# Forward UDP
socat UDP-LISTEN:53,fork UDP:10.0.0.40:53

# Encrypted tunnel with SSL
socat OPENSSL-LISTEN:443,cert=server.pem,verify=0,fork TCP:10.0.0.40:80
```

---

## 5. Ligolo-ng (Modern Tunneling)

> 🔗 Download from: https://github.com/nicocha30/ligolo-ng/releases

```bash
# On Kali (proxy)
sudo ip tuntap add user root mode tun ligolo
sudo ip link set ligolo up
./proxy -selfcert

# On Target (agent)
./agent -connect 192.168.56.10:11601 -ignore-cert

# In proxy console:
session                              # Select session
ifconfig                             # View target's interfaces
start                                # Start tunnel

# Add route on Kali
sudo ip route add 10.0.0.0/24 dev ligolo

# Now directly access internal network — no proxychains needed!
nmap -sT 10.0.0.40
curl http://10.0.0.40
```

### Why Ligolo-ng?

- Creates actual tun interface (no proxychains needed)
- All tools work natively through the tunnel
- Modern, fast, and easy to use
- Cross-platform agent

---

## 6. SSHuttle

```bash
# Route all traffic for a subnet through SSH
sshuttle -r msfadmin@192.168.56.20 10.0.0.0/24

# With SSH key
sshuttle -r user@192.168.56.20 10.0.0.0/24 --ssh-cmd "ssh -i id_rsa"

# Exclude certain IPs
sshuttle -r msfadmin@192.168.56.20 10.0.0.0/24 -x 10.0.0.1

# SSHuttle acts like a VPN — all tools work natively
nmap -sT 10.0.0.40    # Works directly!
curl http://10.0.0.40  # Works directly!
```

---

## 7. Proxychains

```bash
# Configuration file: /etc/proxychains4.conf
# Key settings:

# Proxy types:
# socks4 127.0.0.1 9050
# socks5 127.0.0.1 9050
# http 127.0.0.1 8080

# Chain types:
# strict_chain — all proxies must work
# dynamic_chain — try all, skip dead ones
# random_chain — random proxy order

# Usage with any tool:
proxychains nmap -sT -Pn 10.0.0.40
proxychains curl http://10.0.0.40
proxychains ssh user@10.0.0.40
proxychains crackmapexec smb 10.0.0.0/24
proxychains evil-winrm -i 10.0.0.40 -u user -p pass

# ⚠️ Important: Proxychains only works with TCP!
# No UDP scans through proxychains (use -sT for nmap)
```

---

## 8. Windows Tunneling

### netsh (Built-in Windows)

```cmd
:: Port forwarding with netsh
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=10.0.0.40

:: View active port forwards
netsh interface portproxy show all

:: Remove port forward
netsh interface portproxy delete v4tov4 listenport=8080 listenaddress=0.0.0.0
```

### Plink (PuTTY Command Line)

```cmd
:: Dynamic port forward (SOCKS proxy)
plink.exe -ssh -D 9050 -N msfadmin@192.168.56.20

:: Local port forward
plink.exe -ssh -L 8080:10.0.0.40:80 msfadmin@192.168.56.20

:: Accept host key automatically
echo y | plink.exe -ssh -l msfadmin -pw msfadmin -D 9050 -N 192.168.56.20
```

### Chisel on Windows

```cmd
:: Same syntax as Linux
chisel.exe client 192.168.56.10:8000 R:9050:socks
```

---

## 9. Double Pivoting

```
Scenario: Reach Machine C through Machine A AND Machine B

Kali → Machine A → Machine B → Machine C

Step 1: Create tunnel through Machine A to reach Machine B
ssh -D 9050 user@MachineA

Step 2: Through that tunnel, create another tunnel through Machine B
proxychains ssh -D 9051 user@MachineB

Step 3: Use the second proxy to reach Machine C
# Add second proxy to proxychains config
# socks5 127.0.0.1 9051
proxychains nmap -sT MachineC
```

### With Chisel (Double Pivot)

```bash
# Kali → Machine A
# On Kali:
./chisel server --reverse -p 8000

# On Machine A:
./chisel client KaliIP:8000 R:9050:socks

# Machine A → Machine B (through first tunnel)
# Upload second chisel to Machine A
# On Machine A:
./chisel server --reverse -p 9000

# On Machine B:
./chisel client MachineA_IP:9000 R:9051:socks

# Configure proxychains with both proxies
```

---

## 10. Practice Scenarios

### Scenario 1: Basic SSH Pivot

```
1. SSH into Metasploitable
2. Create dynamic port forward (-D 9050)
3. Configure proxychains
4. Scan internal network through proxychains
```

### Scenario 2: Chisel Through Firewall

```
1. Start Chisel server on Kali
2. Transfer Chisel binary to target
3. Connect as client with reverse SOCKS
4. Scan and access internal services
```

### Scenario 3: Windows Pivot

```
1. Get shell on Windows target
2. Upload Chisel or use plink
3. Create tunnel back to Kali
4. Access internal network
```

### TryHackMe Rooms

| Room      | Focus                             |
| --------- | --------------------------------- |
| Wreath    | Multi-machine pivoting network    |
| Throwback | Full AD network with pivoting     |
| Holo      | Pivoting through web applications |

---

## 11. Weekly Schedule

| Day | Activity                               | Time  |
| --- | -------------------------------------- | ----- |
| Mon | SSH tunneling (local, remote, dynamic) | 2 hrs |
| Tue | Chisel setup and SOCKS proxy           | 2 hrs |
| Wed | Socat + SSHuttle + Ligolo-ng           | 2 hrs |
| Thu | Proxychains with various tools         | 2 hrs |
| Fri | Windows tunneling (netsh, plink)       | 2 hrs |
| Sat | TryHackMe: Wreath (pivoting network)   | 3 hrs |
| Sun | Double pivot practice + review         | 2 hrs |

---

## 12. Checklist & Progress Tracker

### SSH Tunneling
- [ ] Local port forward (`-L`)
- [ ] Remote port forward (`-R`)
- [ ] Dynamic port forward (`-D`)
- [ ] Background SSH tunnel (`-f -N`)

### Chisel
- [ ] Set up server on Kali
- [ ] SOCKS proxy through Chisel
- [ ] Port forwarding through Chisel

### Other Tools
- [ ] Socat port forwarding
- [ ] SSHuttle VPN-like tunnel
- [ ] Ligolo-ng tun interface
- [ ] Proxychains with nmap
- [ ] Proxychains with curl/web
- [ ] Proxychains with CrackMapExec

### Windows
- [ ] netsh port forwarding
- [ ] Plink SSH tunneling
- [ ] Chisel on Windows

### Advanced
- [ ] Double pivoting exercise
- [ ] TryHackMe: Wreath room (or equivalent)

---

> **Previous**: [Phase 6 — Active Directory](../phase_6_active_directory/active_directory.md)
> **Next Phase**: [Phase 8 — Full Attack Chains](../phase_8_attack_chains/attack_chains.md) 🎯
