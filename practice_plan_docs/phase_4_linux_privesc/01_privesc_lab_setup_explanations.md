# Linux PrivEsc Lab Setup — Explanations & Troubleshooting

This document contains additional explanations and troubleshooting steps for the `01_privesc_lab_setup.md` guide, based on common issues encountered during the setup process.

---

## General Rule of Thumb for Lab Setup

*   **Setup** is done as `msfadmin` (using `sudo`) on **Metasploitable**.
*   **Attacking/Practicing** is done as `trainee` on **Metasploitable**.
*   **Tool Preparation** is done on **Kali**.

### Understanding the Workflow
Whenever a section is marked with the target icon (🎯 **Practice** or **Practice as trainee**), that is the transition point where setup ends and "hacking" begins:

1.  Log into **Metasploitable** as `msfadmin`.
2.  Run the setup commands using `sudo`.
3.  Switch user by typing `su - trainee` (password: `trainee123`).
4.  **Now you are the "attacker."** Use the `trainee` account to try and exploit the vulnerability to become `root`.
5.  Once you get `root`, type `exit` to drop back to `trainee`, and `exit` again to drop back to `msfadmin` to set up the next step.

---

## Step 6: PATH Hijacking Explained

### What is the `PATH` variable?
In Linux, when you type a command (like `ls` or `service`), the system relies on an environment variable called `$PATH` to go look for the actual program file. It is a list of directories separated by colons (e.g., `/usr/local/bin:/usr/bin:/bin`). The system searches these directories from left to right and runs the first matching executable it finds.

### The Vulnerability
In Step 6, you created a binary with the **SUID bit set**, meaning it executes with the permissions of the file owner (`root`). The critical flaw is that the binary contains the command `service apache2 status` instead of the absolute path `/usr/sbin/service`. It has to rely on the `$PATH` variable of the user running it to find the `service` command.

### The Attack (The Hijack)
As the attacker (`trainee`), you have control over your own `$PATH`.
1.  **Create a fake, malicious program:** You create a fake file named `service` inside `/tmp/` that spawns a shell (`/bin/bash -p`).
2.  **Alter your PATH variable:** You modify your `$PATH` to put `/tmp` at the very front (`export PATH=/tmp:$PATH`).
3.  **Run the vulnerable SUID file:** When you execute the SUID binary, it runs as root, checks the first directory in `$PATH` (`/tmp`), finds your fake `service` file, and executes it as root, giving you a root shell!

*(Note: The SUID setup in Step 6 was updated to use a compiled C program because Linux ignores the SUID bit on bash scripts for security reasons.)*

---

## Step 7: Writable /etc/passwd

**Question:** Do I need to revert Step 6 before proceeding to Step 7?

**Answer:** No! All the practice scenarios in this lab are independent. You can set them all up at once and practice them in any order. The "Reset Everything" section at the end of the guide is only needed when you are completely finished with the entire lab and want to restore the Metasploitable VM.

### Troubleshooting: `su: Authentication failure`
If you attempt to append the new user hash to `/etc/passwd` and cannot `su` into the new account, you likely pasted the hash incorrectly (e.g., duplicating the `$1$hacker$` prefix). Let's say you accidentally entered:
`echo 'hacker:$1$hacker$$1$hacker$maoVUGb6XNp03USLr9Oqq1:0:0::/root:/bin/bash' >> /etc/passwd`

**How to fix:**
1.  Switch to `msfadmin`.
2.  Remove the corrupted lines: `sudo sed -i '/^hacker:/d' /etc/passwd`
3.  Switch back to `trainee`.
4.  Inject the correct line: `echo 'hacker:$1$hacker$maoVUGb6XNp03USLr9Oqq1:0:0::/root:/bin/bash' >> /etc/passwd`

---

## Step 8: Download Enumeration Tools

**Question:** What is the purpose of downloading tools to Kali first?

**Answer:** The purpose is to prepare your attacker machine (Kali) with automated enumeration scripts (like LinPEAS) before transferring them to the target (Metasploitable). 

When you gain initial access to a Linux machine, you don't immediately know what vulnerabilities exist. While you could manually search for them (as done in Steps 2-7), automated scripts are much faster.

In a real-world scenario, target machines often do not have internet access. The standard workflow is:
1.  Download the tools to your Kali machine.
2.  Host them on a temporary web server on Kali (`python3 -m http.server 8080`).
3.  Use the victim machine (Metasploitable) to reach back to your Kali machine and pull the tools (`wget`).

### Troubleshooting: `Connection refused` during `wget`
If Metasploitable says `failed: Connection refused` when running `wget`, it means there is no web server running on the Kali machine at `192.168.56.10:8080`.

**How to fix:**
You need two terminal windows open simultaneously.
1.  **Terminal 1 (Kali):** Start the web server: `python3 -m http.server 8080` and **leave it running**.
2.  **Terminal 2 (Metasploitable):** While the server is running in Terminal 1, execute the `wget` command to download the file.
