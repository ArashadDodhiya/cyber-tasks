# 🐧 Linux Concepts — Interview Masterguide

> **Goal**: Master Linux so well that you can explain, troubleshoot, and secure any Linux system confidently in an interview.

---

## 1. What is Linux?

Linux is a **free, open-source operating system** based on Unix. It's the backbone of servers, cloud, and cybersecurity.

**Why Linux matters for security**:
- 96% of top web servers run Linux
- Most hacking tools run on Linux (Kali, Parrot)
- All cloud platforms (AWS, Azure, GCP) use Linux
- Android is built on Linux kernel

### Linux vs Windows

| Feature | Linux | Windows |
|---------|-------|---------|
| Cost | Free | Paid license |
| Source Code | Open source | Closed source |
| Security | More secure (permissions) | More targeted by malware |
| CLI Power | Extremely powerful | Limited (PowerShell improving) |
| Server Usage | ~96% of servers | ~4% of servers |
| File System | ext4, xfs | NTFS, FAT32 |
| Case Sensitive | Yes (`File` ≠ `file`) | No |

---

## 2. Linux File System Structure

Everything in Linux is a **file** — even hardware devices!

```
/                   ← Root (top of everything)
├── /bin            ← Essential user commands (ls, cp, cat)
├── /sbin           ← System admin commands (iptables, fdisk)
├── /etc            ← Configuration files (passwords, network config)
├── /home           ← User home directories (/home/arashad)
├── /root           ← Root user's home directory
├── /var            ← Variable data (logs, web files)
│   ├── /var/log    ← System logs
│   └── /var/www    ← Web server files
├── /tmp            ← Temporary files (cleared on reboot)
├── /opt            ← Optional/third-party software
├── /dev            ← Device files (hard drives, USB)
├── /proc           ← Virtual filesystem (running processes info)
├── /usr            ← User programs and libraries
├── /boot           ← Boot loader files (kernel)
└── /mnt            ← Mount point for external drives
```

### Interview Tip — Important Files to Know

| File | Purpose |
|------|---------|
| `/etc/passwd` | User account information |
| `/etc/shadow` | Encrypted passwords (root only) |
| `/etc/hosts` | Local DNS mappings |
| `/etc/resolv.conf` | DNS server configuration |
| `/etc/crontab` | Scheduled tasks |
| `/etc/ssh/sshd_config` | SSH server configuration |
| `/var/log/syslog` | System log |
| `/var/log/auth.log` | Authentication log |
| `/proc/version` | Kernel version |

---

## 3. Essential Linux Commands

### Navigation & File Operations

```bash
# Where am I?
pwd                          # Print working directory

# List files
ls                           # Basic listing
ls -la                       # Detailed + hidden files
ls -lah                      # Human-readable sizes

# Change directory
cd /var/log                  # Go to specific path
cd ..                        # Go up one level
cd ~                         # Go to home directory
cd -                         # Go to previous directory

# Create files and directories
touch newfile.txt            # Create empty file
mkdir myfolder               # Create directory
mkdir -p a/b/c               # Create nested directories

# Copy, Move, Delete
cp file.txt /tmp/            # Copy file
cp -r folder/ /tmp/          # Copy folder recursively
mv file.txt newname.txt      # Rename/move file
rm file.txt                  # Delete file
rm -rf folder/               # Delete folder (DANGEROUS!)

# View file contents
cat file.txt                 # Show entire file
head -20 file.txt            # First 20 lines
tail -20 file.txt            # Last 20 lines
tail -f /var/log/syslog      # Follow log in real-time
less file.txt                # Scrollable viewer
```

### Searching & Filtering

```bash
# Find files
find / -name "passwords.txt"           # Find by name
find / -name "*.conf"                  # Find all .conf files
find / -perm -4000 2>/dev/null         # Find SUID files (security!)
find / -user root -writable 2>/dev/null # Find writable root files
find /home -mtime -7                   # Modified in last 7 days

# Search inside files
grep "error" /var/log/syslog           # Find "error" in file
grep -r "password" /etc/               # Search recursively
grep -i "admin" file.txt               # Case insensitive
grep -n "root" /etc/passwd             # Show line numbers
grep -v "comment" config.txt           # Show lines NOT matching

# Chaining with pipes
cat /etc/passwd | grep "bash"          # Users with bash shell
ps aux | grep "apache"                 # Find apache process
netstat -tuln | grep "80"              # Check port 80
history | grep "ssh"                   # Find past SSH commands
```

### Text Processing

```bash
# Cut — extract columns
cut -d: -f1 /etc/passwd               # Get all usernames
# -d: means delimiter is colon, -f1 means first field

# Awk — powerful text processing
awk -F: '{print $1, $3}' /etc/passwd   # Print username and UID
awk '{print $1}' access.log            # Print first column

# Sed — find and replace
sed 's/old/new/g' file.txt             # Replace old with new
sed -i 's/http/https/g' config.txt     # Edit file in-place

# Sort and unique
sort file.txt                          # Sort lines
sort -u file.txt                       # Sort + remove duplicates
uniq -c sorted.txt                     # Count occurrences

# Word count
wc -l file.txt                        # Count lines
wc -w file.txt                        # Count words
```

---

## 4. File Permissions (Interview Favorite!)

### Understanding Permission String

```
-rwxr-xr-- 1 arashad staff 4096 May 6 file.txt
│└┬┘└┬┘└┬┘
│ │  │  └── Others: read only (r--)
│ │  └───── Group: read + execute (r-x)
│ └──────── Owner: read + write + execute (rwx)
└────────── File type (- = file, d = directory, l = link)
```

### Permission Values

| Symbol | Value | Meaning |
|--------|-------|---------|
| r | 4 | Read |
| w | 2 | Write |
| x | 1 | Execute |
| - | 0 | No permission |

### Calculating Permissions (Numeric)

```
rwx = 4+2+1 = 7
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4

So rwxr-xr-- = 754
```

### Common Permission Sets

| Numeric | Symbolic | Meaning | Use Case |
|---------|----------|---------|----------|
| 777 | rwxrwxrwx | Everyone can do everything | NEVER use (insecure!) |
| 755 | rwxr-xr-x | Owner full, others read+execute | Scripts, programs |
| 644 | rw-r--r-- | Owner read+write, others read | Config files |
| 600 | rw------- | Only owner read+write | SSH keys, passwords |
| 400 | r-------- | Only owner read | Sensitive files |

### Changing Permissions

```bash
chmod 755 script.sh          # Set numeric permissions
chmod +x script.sh           # Add execute for everyone
chmod u+w file.txt           # Add write for owner (u=user)
chmod g-w file.txt           # Remove write for group
chmod o-rwx file.txt         # Remove all for others

# Change owner
chown arashad file.txt       # Change owner
chown arashad:staff file.txt # Change owner and group
chown -R arashad:staff dir/  # Recursive
```

### Special Permissions (Security Critical!)

| Permission | Numeric | Effect |
|-----------|---------|--------|
| **SUID** | 4000 | File runs as the file owner, not the user running it |
| **SGID** | 2000 | File runs as the group owner |
| **Sticky Bit** | 1000 | Only file owner can delete in shared directories |

```bash
# Find SUID files (privilege escalation goldmine!)
find / -perm -4000 2>/dev/null

# Example: /usr/bin/passwd has SUID
ls -la /usr/bin/passwd
# -rwsr-xr-x  ← The 's' means SUID is set
# Regular users can change their password because
# passwd runs as root (file owner)
```

---

## 5. User Management

```bash
# View current user
whoami                       # Current username
id                           # UID, GID, groups

# User management (need root)
useradd -m -s /bin/bash john # Create user with home dir and bash
passwd john                  # Set password
usermod -aG sudo john        # Add to sudo group
userdel -r john              # Delete user + home directory

# Understanding /etc/passwd
cat /etc/passwd
# arashad:x:1000:1000:Arashad:/home/arashad:/bin/bash
# username:password:UID:GID:comment:home:shell

# Understanding /etc/shadow (passwords stored here)
sudo cat /etc/shadow
# arashad:$6$salt$hash:18900:0:99999:7:::
# username:hashed_password:last_change:min:max:warn
```

### sudo vs su

```bash
sudo command           # Run ONE command as root
sudo -l                # List what you can run as sudo
su - root              # Switch to root user entirely
sudo su -              # Become root using your password
```

---

## 6. Process Management

```bash
# View processes
ps aux                       # All processes with details
ps aux | grep apache         # Find specific process
top                          # Real-time process monitor
htop                         # Better version of top (install first)

# Process info
# USER  PID %CPU %MEM    VSZ   RSS TTY STAT  TIME COMMAND
# root    1  0.0  0.1  16984  4852 ?   Ss    0:03 /sbin/init

# Kill processes
kill PID                     # Graceful kill (SIGTERM)
kill -9 PID                  # Force kill (SIGKILL)
killall nginx                # Kill all nginx processes
pkill -f "python script"     # Kill by pattern

# Background processes
command &                    # Run in background
nohup command &              # Keep running after logout
jobs                         # List background jobs
fg %1                        # Bring job 1 to foreground
bg %1                        # Send job 1 to background
```

---

## 7. Networking Commands in Linux

```bash
# Check IP address
ip addr show                 # Modern way
ifconfig                     # Old way (still works)

# Check open ports
ss -tuln                     # Modern way
netstat -tuln                # Old way
# -t=TCP, -u=UDP, -l=listening, -n=numeric

# Check connections
ss -tun                      # Active connections
netstat -an                  # All connections

# DNS lookup
nslookup google.com
dig google.com
host google.com

# Test connectivity
ping -c 4 google.com         # Send 4 pings
traceroute google.com        # Trace route

# Download files
wget https://example.com/file.zip
curl https://api.example.com/data
curl -o output.html https://example.com

# Network scanning
nmap -sV 192.168.1.0/24      # Scan entire subnet
nmap -p- 192.168.1.1          # All ports

# Firewall (iptables)
sudo iptables -L              # List rules
sudo iptables -L -n           # List with numeric ports

# Firewall (ufw - simpler)
sudo ufw status
sudo ufw allow 22/tcp
sudo ufw deny 23/tcp
sudo ufw enable
```

---

## 8. Service Management (systemd)

```bash
# Start/Stop services
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx    # Reload config without restart

# Enable/Disable at boot
sudo systemctl enable nginx    # Start on boot
sudo systemctl disable nginx   # Don't start on boot

# Check status
sudo systemctl status nginx
sudo systemctl is-active nginx
sudo systemctl is-enabled nginx

# List all services
systemctl list-units --type=service
systemctl list-units --type=service --state=running
```

---

## 9. Cron Jobs (Scheduled Tasks)

```bash
# Edit crontab
crontab -e                   # Edit your cron jobs
crontab -l                   # List your cron jobs
sudo crontab -u root -l      # List root's cron jobs

# Cron format
# ┌───────── minute (0-59)
# │ ┌─────── hour (0-23)
# │ │ ┌───── day of month (1-31)
# │ │ │ ┌─── month (1-12)
# │ │ │ │ ┌─ day of week (0-7, 0 and 7 = Sunday)
# * * * * * command

# Examples
*/5 * * * * /script.sh       # Every 5 minutes
0 2 * * * /backup.sh         # Daily at 2 AM
0 0 * * 0 /weekly.sh         # Every Sunday midnight
30 8 1 * * /monthly.sh       # 1st of month at 8:30 AM
```

### Security Note
Always check cron for persistence mechanisms during incident response:
```bash
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
for user in $(cut -d: -f1 /etc/passwd); do crontab -u $user -l 2>/dev/null; done
```

---

## 10. SSH (Secure Shell)

```bash
# Connect to remote server
ssh user@192.168.1.100
ssh -p 2222 user@server.com  # Custom port

# SSH key authentication (more secure than passwords)
ssh-keygen -t rsa -b 4096   # Generate key pair
ssh-copy-id user@server     # Copy public key to server

# SCP — Secure file copy
scp file.txt user@server:/tmp/           # Upload file
scp user@server:/var/log/syslog ./       # Download file
scp -r folder/ user@server:/tmp/         # Upload folder

# SSH Tunneling (Port Forwarding)
ssh -L 8080:localhost:80 user@server     # Local forwarding
# Now localhost:8080 → server's port 80

ssh -R 9090:localhost:3000 user@server   # Remote forwarding
ssh -D 1080 user@server                  # SOCKS proxy
```

### Hardening SSH (`/etc/ssh/sshd_config`)

```bash
PermitRootLogin no              # Disable root login
PasswordAuthentication no       # Only key-based auth
Port 2222                       # Change default port
MaxAuthTries 3                  # Limit login attempts
AllowUsers arashad john         # Whitelist users
```

---

## 11. Log Analysis (Incident Response)

```bash
# Important log files
/var/log/syslog        # General system log
/var/log/auth.log      # Authentication attempts
/var/log/kern.log      # Kernel messages
/var/log/apache2/      # Apache web server logs
/var/log/nginx/        # Nginx logs
/var/log/faillog       # Failed login attempts
/var/log/lastlog       # Last login info

# Useful log commands
tail -f /var/log/auth.log              # Watch auth in real-time
grep "Failed password" /var/log/auth.log  # Failed SSH logins
grep "Accepted" /var/log/auth.log      # Successful logins
last                                    # Login history
lastb                                   # Failed login history
who                                     # Currently logged in users
w                                       # Who + what they're doing
```

---

## 12. Package Management

### Debian/Ubuntu (apt)

```bash
sudo apt update                # Update package list
sudo apt upgrade               # Upgrade all packages
sudo apt install nginx         # Install package
sudo apt remove nginx          # Remove package
sudo apt purge nginx           # Remove + delete config
sudo apt search keyword        # Search packages
dpkg -l                        # List installed packages
```

### RedHat/CentOS (yum/dnf)

```bash
sudo yum update                # Update packages
sudo yum install nginx         # Install
sudo yum remove nginx          # Remove
sudo dnf install nginx         # Modern replacement for yum
rpm -qa                        # List installed packages
```

---

## 13. Disk & Storage Management

```bash
# Disk usage
df -h                          # Disk space (human readable)
du -sh /var/log/               # Size of a directory
du -sh * | sort -rh | head -10 # Top 10 largest items

# Mount drives
lsblk                         # List block devices
mount /dev/sdb1 /mnt/usb      # Mount a drive
umount /mnt/usb               # Unmount
cat /etc/fstab                 # Permanent mount config
```

---

## 14. Bash Scripting Basics

```bash
#!/bin/bash
# Shebang line — tells system to use bash

# Variables
name="Arashad"
echo "Hello, $name"

# User input
read -p "Enter IP: " target_ip
echo "Scanning $target_ip..."

# If-else
if [ -f "/etc/passwd" ]; then
    echo "File exists"
else
    echo "File not found"
fi

# Loops
for ip in $(seq 1 254); do
    ping -c 1 192.168.1.$ip > /dev/null 2>&1
    if [ $? -eq 0 ]; then
        echo "192.168.1.$ip is alive"
    fi
done

# Functions
check_root() {
    if [ "$(id -u)" -ne 0 ]; then
        echo "Run as root!"
        exit 1
    fi
}
check_root
```

### Useful One-Liners for Security

```bash
# Find all SUID binaries
find / -perm -4000 2>/dev/null

# Check for world-writable files
find / -perm -o+w -type f 2>/dev/null

# List all listening services
ss -tuln | awk 'NR>1 {print $5}' | sort -u

# Check for unauthorized SSH keys
find / -name "authorized_keys" 2>/dev/null

# Monitor network connections in real-time
watch -n 1 'ss -tun'

# Extract all IPs from a log file
grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' access.log | sort -u
```

---

## 15. Linux Privilege Escalation (Security Interview Gold!)

### Checklist for Privesc

```bash
# 1. Check current user and permissions
whoami
id
sudo -l                        # What can I sudo?

# 2. Check SUID/SGID binaries
find / -perm -4000 2>/dev/null
# Look these up on GTFOBins!

# 3. Check cron jobs
cat /etc/crontab
ls -la /etc/cron*

# 4. Check writable files/directories
find / -writable -type d 2>/dev/null

# 5. Check kernel version (for exploits)
uname -a
cat /proc/version

# 6. Check running processes
ps aux

# 7. Check environment variables
env
cat /etc/profile

# 8. Check for stored credentials
find / -name "*.conf" -exec grep -l "password" {} \; 2>/dev/null
cat ~/.bash_history
```

### GTFOBins — SUID Exploitation Examples

```bash
# If you find /usr/bin/find has SUID:
find . -exec /bin/sh -p \;

# If /usr/bin/vim has SUID:
vim -c ':!/bin/sh'

# If /usr/bin/python3 has SUID:
python3 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

---

## 16. Linux Hardening Checklist

| Action | Command/Config |
|--------|---------------|
| Update system | `apt update && apt upgrade` |
| Disable root SSH | `PermitRootLogin no` in sshd_config |
| Use SSH keys | `ssh-keygen` + disable password auth |
| Set firewall | `ufw enable` + allow only needed ports |
| Remove SUID from unneeded binaries | `chmod u-s /path/to/binary` |
| Check open ports | `ss -tuln` — close unnecessary ones |
| Set password policy | Edit `/etc/login.defs` |
| Enable audit logging | Install and configure `auditd` |
| Disable unused services | `systemctl disable service_name` |
| Set file permissions | Sensitive files should be 600 or 400 |

---

## 17. I/O Redirection & Piping

```bash
# Output redirection
echo "hello" > file.txt       # Write (overwrite)
echo "world" >> file.txt      # Append

# Error redirection
command 2> errors.txt         # Redirect errors only
command > output.txt 2>&1     # Both output and errors
command > /dev/null 2>&1      # Discard everything

# Piping (send output of one command to another)
cat access.log | grep "404" | wc -l    # Count 404 errors
ps aux | grep nginx | awk '{print $2}' # Get nginx PIDs
```

---

## 18. Top Interview Questions & Answers

### Q1: "What is the difference between hard link and soft link?"
> **Hard link**: Another name for the same file (same inode). Deleting original doesn't affect it.  
> **Soft link (symlink)**: A shortcut pointing to the original file. Breaks if original is deleted.
> ```bash
> ln file.txt hardlink.txt      # Hard link
> ln -s file.txt softlink.txt   # Soft link
> ```

### Q2: "How do you find which process is using a specific port?"
> ```bash
> ss -tuln | grep :80
> lsof -i :80
> fuser 80/tcp
> ```

### Q3: "Explain Linux file permissions with an example."
> Every file has permissions for Owner, Group, and Others. Each can have Read (4), Write (2), Execute (1). For example, `chmod 755 script.sh` means owner can do everything (7), group and others can read and execute (5).

### Q4: "What is the /etc/shadow file?"
> It stores encrypted user passwords. Only root can read it. Each line has the username, hashed password (using SHA-512 etc.), and password aging info. This is why password cracking targets this file.

### Q5: "How do you check system resource usage?"
> ```bash
> top / htop        # CPU and memory per process
> free -h           # Memory usage
> df -h             # Disk space
> uptime            # Load average
> iostat            # Disk I/O
> vmstat            # Virtual memory stats
> ```

### Q6: "What is the sticky bit?"
> The sticky bit on a directory means only the file's owner can delete their files, even if others have write permission. `/tmp` uses this so users can't delete each other's temp files.
> ```bash
> ls -ld /tmp
> drwxrwxrwt   ← The 't' at the end = sticky bit
> ```

### Q7: "How would you investigate a compromised Linux server?"
> 1. Check auth logs: `grep "Failed" /var/log/auth.log`
> 2. Check running processes: `ps aux` — look for suspicious ones
> 3. Check network connections: `ss -tun` — look for unknown IPs
> 4. Check cron jobs: `crontab -l` + `/etc/cron*`
> 5. Check recently modified files: `find / -mtime -1`
> 6. Check user accounts: `cat /etc/passwd` — look for new users
> 7. Check bash history: `cat ~/.bash_history`
> 8. Check SUID files: `find / -perm -4000`

### Q8: "Difference between /etc/passwd and /etc/shadow?"
> `/etc/passwd` is readable by everyone and contains user info (username, UID, home dir, shell). `/etc/shadow` is readable only by root and contains the actual password hashes. They were separated for security — so normal users can read passwd for user info without seeing password hashes.

---

> 💡 **Pro Tip**: During interviews, if asked about a command you don't remember exactly, say: "I would check the man page with `man command_name`" — it shows you know how to find information, which is more valuable than memorizing everything.

---

*Last Updated: May 2026 | Created for Cybersecurity Interview Preparation*
