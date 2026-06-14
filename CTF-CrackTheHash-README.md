# TryHackMe — Crack The Hash

**Platform:** TryHackMe  
**Room:** [Crack The Hash](https://tryhackme.com/room/crackthehash)  
**Category:** Cryptography / Password Cracking  
**Difficulty:** Easy  

---

## Overview

This room focuses on hash identification and password cracking using both online tools and command-line tools. It covers common hash types encountered in real-world CTF and penetration testing scenarios.

**Tools Used:**
- Hashes.com — online hash cracker
- John the Ripper
- Hashcat
- hashid — hash identification tool
- Kali Linux (own machine via OpenVPN)

**Reference Used:**  
[Hashcat Example Hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) — for identifying hash types

---

## Task 1 — Level 1

Most hashes in this level can be cracked using [Hashes.com](https://hashes.com) (recommended) or [CyberChef](https://gchq.github.io/CyberChef/).

For tool-based cracking, John the Ripper or Hashcat can be used.

---

### Hash 1 — MD5
```
48bb6e862e54f2a795ffc4e541caed4d
```
**Method:** Hashes.com — paste hash, solve captcha, submit.

Tool commands if preferred:
```bash
# John the Ripper
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Hashcat
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
```

---

### Hash 2 — SHA1
```
CBFDAC6008F9CAB4083784CBD1874F76618D2A97
```
**Method:** Hashes.com

Tool commands:
```bash
# John the Ripper
john --format=raw-sha1 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Hashcat
hashcat -m 100 hash.txt /usr/share/wordlists/rockyou.txt
```

---

### Hash 3 — SHA256
```
1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
```
**Method:** Hashes.com

Tool commands:
```bash
# John the Ripper
john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Hashcat
hashcat -m 1400 hash.txt /usr/share/wordlists/rockyou.txt
```

---

### Hash 4 — Bcrypt
```
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
```
**Method:** Hashes.com

Tool commands:
```bash
# John the Ripper
john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Hashcat
hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt
```

---

### Hash 5 — MD4
```
279412f945939ba78ce0758d3fd83daa
```
**Method:** Hashes.com

Tool commands:
```bash
# John the Ripper
john --format=raw-md4 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Hashcat
hashcat -m 900 hash.txt /usr/share/wordlists/rockyou.txt
```

---

## Task 2 — Level 2

This level requires tools — online crackers won't work here. John the Ripper is used throughout.

**Setup:** Save each hash into a file first:
```bash
echo "HASH_HERE" > hash.txt
```

Use `hashid` to identify unknown hash types:
```bash
hashid hash.txt
```

---

### Hash 1 — SHA256
```
F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
```
```bash
# John the Ripper
john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Hashcat
hashcat -m 1400 hash.txt /usr/share/wordlists/rockyou.txt
```

---

### Hash 2 — NTLM
```
1DFECA0C002AE40B8619ECF94819CC1B
```
```bash
# John the Ripper
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Hashcat
hashcat -m 1000 hash.txt /usr/share/wordlists/rockyou.txt
```

---

### Hash 3 — SHA512crypt
```
$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02
```
```bash
# John the Ripper
john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Hashcat
hashcat -m 1800 hash.txt /usr/share/wordlists/rockyou.txt
```

---

### Hash 4 — HMAC-SHA1
```
e5d8870e5bdd26602cab8dbe07a942c8669e56d6
```
```bash
# John the Ripper
john --format=hmac-sha1 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Hashcat
hashcat -m 110 hash.txt /usr/share/wordlists/rockyou.txt
```

---

## Key Takeaways

- Always identify the hash type before attempting to crack it — use `hashid` or the Hashcat example hashes reference
- Hashes.com works well for common hash types in Level 1
- Level 2 hashes require tools like John the Ripper or Hashcat with a wordlist (rockyou.txt)
- Bcrypt and SHA512crypt are significantly slower to crack due to their design — computationally expensive by intention
- NTLM hashes are commonly found in Windows environments during real penetration tests

---

## What I Learned

- How to identify MD5, SHA1, SHA256, Bcrypt, MD4, NTLM, and SHA512crypt hash types
- How to use John the Ripper with correct format flags per hash type
- How to use Hashcat with correct mode numbers (-m flag) per hash type
- The difference between fast hashes (MD5, SHA1) and slow hashes (Bcrypt, SHA512crypt)
- Why password hashing algorithms matter from a defensive security perspective
