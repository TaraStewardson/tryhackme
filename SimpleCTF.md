# [SimpleCTF](https://tryhackme.com/room/easyctf) 

Difficulty: Easy  
Completed: 30/11/23

### 1. How many services are running under port 1000?  
I used nmap to identify ports in use: nmap -sC -sV IP  
It returned 3 ports in use: 
- 21/tcp running vsftpd 3.0.3
- 80/tcp Apache httpd 2.4.18
- 2222/tcp running OpenSSH 7.2p2  

### 2. What's running on the highest port?
As we know, OpenSSH is running on 2222/tcp, so the answer is ssh.

### 3. What's the CVE you're using against the application?
Since Apache Httpd is running, I access the website to see if there is anything interesting. 
I use GoBuster to find any directories of interest: gobuster dir -u http://IP -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

*Note: This wordlist (and location) is default in the TryHackMe HackBox*

GoBuster locates a directory, /simple, which is running CMS Made Simple. A search for CMS Made Simple CVE Exploit returns information on CVE-2019-9053, a sql injection exploit (aka sqli). This answers questions 3 & 4.  

### 4. To what kind of vulnerability is the application vulnerable?
As we know, its vulnerable to an SQL Injection, or sqli. 

### 5. What's the password?
Googling the CVE led me to [this exploit](https://www.exploit-db.com/exploits/46635) for it. 
*I learned later that on the hackbox, you can search the searchsploit database fot the cve: searchsploit cms made simple 2.2.8*  
I made some edits to the file (as the print statements were not in brackets and it was failing), and then ran it: python3 cmsexploit.py -u http://IP/simple  

It found the:

  - Password Salt: 1dac0d92e9fa6bb2
  - Username: mitch
  - Email: admin@admin.com
  - Password: 0c01f4468bd75d7a84c7eb73846e8d96

This password is not correct, so I tried again using rockyou.txt (in usr/share/wordlists on the AttackBox)

python3 cmsexploit.py -u http://IP/simple -w /usr/share/wordlists/rockyou.txt

Still it only returned the encrypted password, so I ran it again with --crack.

I found the password was [this](secret).  

### 6. Where can you login with the details obtained?
You can ssh into the machine with the username and password: ssh mitch@IP 
and I'm in!

### 7. What's the user flag?
Once logged in, using ls we can see one file in the directory home/mitch: user.txt. Using cat user.txt, we can see it has the flag!

### 8. Is there any other user in ther home directory?
Using cd .., we can see there is another user named sunbath.

### 9. What can you leverage to spawn a privileged shell?
If we run sudo -l, we can see that /usr/bin/vim can be run sudo without a password. So we can leverage vim.

### 10. Whats the flag?
Searching GTFOBins for a privelege escalation exploit using vim I found a list. First I tried: sudo vim -c ':!/bin/sh'. Then ran whoami to find that I was root!
I then found the flag in /root/root.txt

## My takeaways
When looking for files, start by listing the contents of the current directory.
Use GTFOBins for privelege escalation.
share/wordlists on the AttackBox has a lot of handy wordlists (rockyou.txt for passwords, dirbuster for directory location). Use the AttackBox searchsploit keyword (using app name and version) to find CVE exploits for that app. Typically, start with any webservers that are running.


