Write-Up: Shadow SystemRoom InformationRoom Name: Shadow System   Difficulty: Easy / MediumPlatform: TryHackMe   Author: [Your Username]Overview: A Linux boot-to-root CTF challenge focusing on service enumeration, initial access via SSH as tom, and privilege escalation through misconfigured sudo permissions.   Step 1: ReconnaissanceStart by scanning the target machine to identify open ports and running services.Port & Service ScanBashnmap -sC -sV <TARGET_IP>
Check the discovered services (such as SSH on port 22 and HTTP on port 80).   Web EnumerationIf an HTTP service is running, inspect the web application and enumerate hidden directories or files:Bashgobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
Review any exposed pages, comments, configuration files, or backups that point to credentials or an entry point for tom.   Step 2: Initial AccessUsing the credentials or key material discovered during enumeration, connect to the target over SSH as tom:   Bashssh tom@<TARGET_IP>
User FlagOnce logged in as tom, locate and read the user flag:Bashls -la ~
cat ~/user.txt
User Flag: THM{<user_flag_here>}Step 3: Privilege EscalationCheck sudo privileges assigned to the current user to see if a command or binary can be executed as another user or root:Bashsudo -l
Leverage the misconfigured sudo rule (for example, a permitted script or binary that allows spawning a shell or reading restricted files) to escalate privileges to root.Step 4: Root FlagAfter obtaining a root shell, read the root flag from /root:Bashid
ls -la /root
cat /root/root.txt
Root Flag: THM{<root_flag_here>}
