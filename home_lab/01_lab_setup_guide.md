# 🏠 Home Lab Setup Guide — Build Your Free Hacking Lab

> **Goal**: Set up a complete, isolated hacking lab on your Windows PC using **free** tools.
> By the end of this guide, you'll have an attacker machine (Kali) and multiple vulnerable targets to practice on.

---

## 📋 Table of Contents

1. [What You Need (Requirements)](#1-what-you-need-requirements)
2. [Step 1: Install VirtualBox](#2-step-1-install-virtualbox)
3. [Step 2: Set Up Kali Linux (Attacker)](#3-step-2-set-up-kali-linux-attacker)
4. [Step 3: Set Up Metasploitable 2 (Vulnerable Target)](#4-step-3-set-up-metasploitable-2-vulnerable-target)
5. [Step 4: Set Up DVWA (Web Hacking Target)](#5-step-4-set-up-dvwa-vulnerable-web-app)
6. [Step 5: Network Configuration](#6-step-5-network-configuration)
7. [Step 6: Verify Everything Works](#7-step-6-verify-everything-works)
8. [Optional: More Targets](#8-optional-add-more-targets)
9. [Lab Safety Rules](#9-lab-safety-rules)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. What You Need (Requirements)

### Hardware Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **RAM** | 8 GB | 16 GB |
| **Disk Space** | 50 GB free | 100 GB free |
| **CPU** | 2 cores | 4+ cores |
| **OS** | Windows 10/11 | Windows 10/11 |

### Software (All Free!)

| Software | What It Does | Download Link |
|----------|-------------|---------------|
| **VirtualBox** | Runs virtual machines | https://www.virtualbox.org/wiki/Downloads |
| **Kali Linux** | Your hacking/attacker machine | https://www.kali.org/get-kali/#kali-virtual-machines |
| **Metasploitable 2** | Intentionally vulnerable Linux | https://sourceforge.net/projects/metasploitable/ |
| **DVWA** | Vulnerable web application | https://github.com/digininja/DVWA |

### ⚠️ Important: Enable Virtualization

Before starting, make sure **virtualization is enabled** in your BIOS:

1. Restart your PC
2. Press `F2`, `F10`, `Del`, or `Esc` (depends on your motherboard) to enter BIOS
3. Look for: **Intel VT-x** or **AMD-V** or **SVM Mode**
4. **Enable** it
5. Save and restart

**How to check if it's already enabled:**
```
Open Task Manager → Performance → CPU → Look for "Virtualization: Enabled"
```

---

## 2. Step 1: Install VirtualBox

### Download & Install

1. Go to https://www.virtualbox.org/wiki/Downloads
2. Click **"Windows hosts"** to download the installer
3. Run the installer → Click **Next** through all steps
4. Click **Install** (it will install network drivers — click **Yes** when prompted)
5. Click **Finish**

### Install VirtualBox Extension Pack (Optional but Recommended)

1. On the same download page, click **"VirtualBox Extension Pack"**
2. Double-click the downloaded file
3. VirtualBox will open → Click **Install** → Accept the license

> **Why?** The Extension Pack adds USB 2.0/3.0 support and other helpful features.

---

## 3. Step 2: Set Up Kali Linux (Attacker)

### Option A: Pre-built VM (Easiest — Recommended!)

1. Go to https://www.kali.org/get-kali/#kali-virtual-machines
2. Download the **VirtualBox (64-bit)** version
3. Extract the `.7z` file (use [7-Zip](https://7-zip.org/) if you don't have it)
4. Open VirtualBox → **File** → **Import Appliance**
5. Select the `.ova` file from the extracted folder
6. Click **Next** → **Import**
7. Wait for the import to complete

### Configure Kali VM Settings

Before starting, tweak the settings:

1. Select the Kali VM → Click **Settings**
2. **System → Motherboard**: Set RAM to **4096 MB** (4 GB) minimum
3. **System → Processor**: Set to **2 CPUs**
4. **Display → Screen**: Set Video Memory to **128 MB**
5. **Network**: We'll configure this later in Step 5
6. Click **OK**

### First Boot

1. Click **Start** to boot Kali
2. Default credentials:
   - **Username**: `kali`
   - **Password**: `kali`

### First Things to Do in Kali

Open a terminal and run:

```bash
# Update the system
sudo apt update && sudo apt upgrade -y

# Check your IP address
ip a

# Test internet connection
ping -c 3 google.com
```

---

## 4. Step 3: Set Up Metasploitable 2 (Vulnerable Target)

> **Metasploitable 2** is an intentionally vulnerable Linux machine. It has tons of
> security holes for you to practice exploiting.

### Download & Import

1. Go to https://sourceforge.net/projects/metasploitable/
2. Click **Download** (it's a `.zip` file, ~800 MB)
3. Extract the `.zip` file to a folder (e.g., `D:\VMs\Metasploitable2`)

### Create VM in VirtualBox

Since Metasploitable comes as a `.vmdk` (disk image) not an `.ova`, you need to create a VM manually:

1. Open VirtualBox → Click **New**
2. Configure:
   - **Name**: `Metasploitable2`
   - **Type**: `Linux`
   - **Version**: `Ubuntu (64-bit)` (or `Other Linux` if Ubuntu isn't listed)
3. Click **Next**
4. **Memory**: Set to **1024 MB** (1 GB is enough)
5. Click **Next**
6. Select **"Use an existing virtual hard disk file"**
7. Click the folder icon → **Add** → Navigate to your extracted folder
8. Select the `.vmdk` file → Click **Choose**
9. Click **Create**

### ⚠️ CRITICAL: NEVER Connect Metasploitable to the Internet!

Metasploitable is **intentionally insecure**. If it's connected to the internet or your
real network, it can be hacked by actual attackers. We'll configure a safe **internal network** in Step 5.

### First Boot

1. Click **Start** to boot Metasploitable
2. Default credentials:
   - **Username**: `msfadmin`
   - **Password**: `msfadmin`

### Verify It's Running

```bash
# Check IP (after network is configured)
ifconfig

# Check running services
netstat -tlnp
```

You should see TONS of open ports — that's the whole point! 🎯

---

## 5. Step 4: Set Up DVWA (Vulnerable Web App)

> **DVWA (Damn Vulnerable Web Application)** is a PHP/MySQL web app that is intentionally
> vulnerable. Perfect for practicing web attacks like SQL injection, XSS, file inclusion, etc.

### Option A: Run DVWA Inside Metasploitable (Easiest)

Good news — **Metasploitable 2 already has DVWA installed!**

After booting Metasploitable, from your Kali machine open a browser and go to:

```
http://<metasploitable-ip>/dvwa/
```

Default login:
- **Username**: `admin`
- **Password**: `password`

### Option B: Run DVWA as a Separate Docker Container (More Control)

If you want a standalone DVWA:

```bash
# On Kali Linux, install Docker
sudo apt install docker.io -y

# Pull and run DVWA
sudo docker run -d -p 80:80 --name dvwa vulnerables/web-dvwa

# Access it at http://localhost/
```

### DVWA Security Levels

DVWA has adjustable difficulty levels — start with **Low** and work your way up:

| Level | Description |
|-------|-------------|
| **Low** | No security — learn the basic attack |
| **Medium** | Some filtering — learn to bypass basic defenses |
| **High** | Strong filtering — learn advanced bypass techniques |
| **Impossible** | Secure code — see how the vulnerability is properly fixed |

---

## 6. Step 5: Network Configuration

> This is the **most important step**. We need to create an isolated network so your
> VMs can talk to each other but are **completely isolated** from your real network and
> the internet.

### Network Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    YOUR WINDOWS PC                      │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │               VirtualBox                          │   │
│  │                                                   │   │
│  │   ┌──────────┐    Internal     ┌───────────────┐ │   │
│  │   │   Kali   │◄──Network──────►│ Metasploitable│ │   │
│  │   │ Attacker │   (Isolated)    │   (Target)    │ │   │
│  │   └──────────┘                 └───────────────┘ │   │
│  │        │                                          │   │
│  │        │ NAT (for                                 │   │
│  │        │ updates                                  │   │
│  │        │ only)                                    │   │
│  │        ▼                                          │   │
│  │    Internet                                       │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Configure Kali (2 Network Adapters)

Kali needs **two** adapters:
- **Adapter 1 (NAT)**: For internet access (updates, downloads)
- **Adapter 2 (Internal Network)**: To talk to Metasploitable

1. Select Kali VM → **Settings** → **Network**
2. **Adapter 1**:
   - ✅ Enable Network Adapter
   - Attached to: **NAT**
3. **Adapter 2** (click the Adapter 2 tab):
   - ✅ Enable Network Adapter
   - Attached to: **Internal Network**
   - Name: `hacklab` (type this — it creates a new internal network)
4. Click **OK**

### Configure Metasploitable (1 Network Adapter)

Metasploitable should **ONLY** be on the internal network — NO internet access!

1. Select Metasploitable VM → **Settings** → **Network**
2. **Adapter 1**:
   - ✅ Enable Network Adapter
   - Attached to: **Internal Network**
   - Name: `hacklab` (same name as Kali's Adapter 2)
3. Click **OK**

### Set Static IPs

After booting both VMs, configure static IPs:

#### On Kali (for the internal network adapter — eth1):

```bash
# Find your internal adapter name (usually eth1)
ip a

# Set a static IP
sudo ip addr add 192.168.56.10/24 dev eth1
sudo ip link set eth1 up
```

**To make it permanent**, edit the network config:

```bash
sudo nano /etc/network/interfaces
```

Add:

```
auto eth1
iface eth1 inet static
    address 192.168.56.10
    netmask 255.255.255.0
```

Save with `Ctrl+O`, Exit with `Ctrl+X`.

#### On Metasploitable:

```bash
# Set static IP
sudo ifconfig eth0 192.168.56.20 netmask 255.255.255.0 up
```

**To make it permanent:**

```bash
sudo nano /etc/network/interfaces
```

Change to:

```
auto eth0
iface eth0 inet static
    address 192.168.56.20
    netmask 255.255.255.0
```

### Alternative: Host-Only Network (Simpler Setup)

If Internal Network is confusing, you can use **Host-Only Adapter** instead:

1. In VirtualBox, go to **File** → **Host Network Manager** (or **Tools** → **Network**)
2. Click **Create** to add a Host-Only Network
3. Note the IP range (usually `192.168.56.1/24`)
4. Assign this Host-Only Network to both VMs

> **Difference**: Host-Only lets your Windows host also access the VMs.
> Internal Network keeps VMs isolated even from the host.

---

## 7. Step 6: Verify Everything Works

### Test 1: Ping Between Machines

From Kali:

```bash
# Ping Metasploitable
ping -c 4 192.168.56.20
```

You should see replies. If not, check your network configuration.

### Test 2: Scan Metasploitable with Nmap

From Kali:

```bash
# Quick scan
nmap 192.168.56.20

# Full scan (takes longer)
nmap -sV -sC -A 192.168.56.20
```

You should see **many open ports** like:

| Port | Service |
|------|---------|
| 21 | FTP (vsftpd 2.3.4) |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 80 | HTTP (Apache) |
| 139/445 | SMB (Samba) |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6667 | IRC |
| 8180 | Apache Tomcat |

### Test 3: Access DVWA

From Kali, open Firefox and go to:

```
http://192.168.56.20/dvwa/
```

Login with `admin` / `password`.

### Test 4: Check Kali Has Internet

```bash
ping -c 3 google.com
```

### Test 5: Confirm Metasploitable Has NO Internet

```bash
# On Metasploitable — this should FAIL (which is correct!)
ping -c 3 google.com
```

**If all 5 tests pass, your lab is ready!** 🎉

---

## 8. Optional: Add More Targets

### VulnHub Machines (Free Downloadable VMs)

Visit https://www.vulnhub.com/ and download beginner-friendly machines:

| Machine | Difficulty | What You Practice |
|---------|-----------|-------------------|
| **Kioptrix Level 1** | Easy | Enumeration, Exploitation |
| **Mr. Robot** | Medium | Web enumeration, Linux priv esc |
| **DC-1** | Easy | Drupal exploitation, Linux priv esc |
| **Basic Pentesting 1** | Easy | Full attack chain |
| **Stapler** | Medium | Multiple attack vectors |

**How to import VulnHub machines:**
1. Download the `.ova` or `.vmdk` file
2. Follow the same steps as Metasploitable (create VM, attach disk)
3. Add to the `hacklab` internal network
4. Assign a unique IP (e.g., `192.168.56.30`, `192.168.56.40`, etc.)

### OWASP Juice Shop (Modern Web App Hacking)

```bash
# On Kali — run with Docker
sudo docker run -d -p 3000:3000 bkimminich/juice-shop

# Access at http://localhost:3000
```

### Windows Target (For Windows Privilege Escalation Practice)

For AD and Windows Priv Esc practice, you'll need a Windows VM:

1. Download **Windows 10 Evaluation** (free, 90-day trial):
   https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise
2. Create a new VM in VirtualBox (4 GB RAM, 40 GB disk)
3. Install Windows
4. Add to the `hacklab` network
5. Intentionally weaken security for practice:
   - Disable Windows Defender
   - Create weak local accounts
   - Install vulnerable services

> **For Active Directory practice**, see your existing guide at `AD_hacking/AD_lab_setup_guide.md`

---

## 9. Lab Safety Rules

> ⚠️ **Follow these rules to stay legal and safe!**

### DO ✅

- ✅ Only attack machines YOU own/control (your VMs)
- ✅ Keep vulnerable VMs isolated (Internal Network only)
- ✅ Take VM snapshots before major changes
- ✅ Document everything you learn
- ✅ Practice in your home lab before trying CTF challenges

### DON'T ❌

- ❌ **NEVER** attack machines you don't own — this is illegal
- ❌ **NEVER** connect Metasploitable/DVWA to the internet
- ❌ **NEVER** bridge vulnerable VMs to your real network
- ❌ **NEVER** use techniques learned here on real-world targets without authorization
- ❌ **NEVER** share exploits or credentials from unauthorized access

### Take Snapshots!

Before starting any attack exercise, take a **snapshot** of your VMs:

```
VirtualBox → Select VM → Snapshots → Take Snapshot → Name it (e.g., "Clean State")
```

This lets you revert to a clean state if you break something.

---

## 10. Troubleshooting

### Problem: VMs Can't Ping Each Other

- ✅ Both VMs are on the **same internal network name** (`hacklab`)
- ✅ IPs are in the **same subnet** (e.g., `192.168.56.x`)
- ✅ The adapters are **enabled** and **up**
- Try restarting networking: `sudo systemctl restart networking`

### Problem: Kali Has No Internet

- ✅ Adapter 1 is set to **NAT**
- ✅ Try: `sudo dhclient eth0`
- ✅ Check DNS: `cat /etc/resolv.conf` (should have `nameserver` entries)

### Problem: Metasploitable Won't Boot

- ✅ Check if virtualization is enabled in BIOS
- ✅ Try setting Version to **"Ubuntu (32-bit)"** instead of 64-bit
- ✅ Make sure the `.vmdk` file wasn't corrupted during download

### Problem: DVWA Shows "Database Not Set Up"

1. Go to `http://<ip>/dvwa/setup.php`
2. Click **"Create / Reset Database"**
3. Login again with `admin` / `password`

### Problem: VirtualBox Crashes / Blue Screen

- ✅ Disable **Hyper-V** in Windows:
  ```powershell
  # Run PowerShell as Administrator
  bcdedit /set hypervisorlaunchtype off
  # Restart your PC
  ```
- ✅ Check that you're not running too many VMs for your RAM

---

## 📊 Summary: Your Lab Architecture

```
YOUR COMPLETE LAB:

    ┌───────────────────────────────────────────────────┐
    │              Internal Network: hacklab             │
    │                                                    │
    │  ┌────────────┐  ┌───────────────┐  ┌──────────┐ │
    │  │    KALI     │  │ METASPLOITABLE│  │  DVWA    │ │
    │  │ 192.168.56  │  │ 192.168.56.20 │  │ (inside  │ │
    │  │   .10       │  │               │  │  Meta)   │ │
    │  │             │  │ Open Ports:   │  │          │ │
    │  │ Tools:      │  │ 21,22,23,25,  │  │ SQL Inj, │ │
    │  │ nmap,msf,   │  │ 80,139,445,   │  │ XSS,     │ │
    │  │ hydra,burp  │  │ 3306,5432...  │  │ CSRF...  │ │
    │  └────────────┘  └───────────────┘  └──────────┘ │
    └───────────────────────────────────────────────────┘
```

---

**Next Step**: Head to [02_practice_plan.md](./02_practice_plan.md) for structured exercises! 🚀
