---
title: DigDug Writeup
published: 2026-03-22
description: Writeup of an easy room by TryHackMe.
tags: [Writeup, Easy, TryHackMe]
category: TryHackMe
draft: false
---

### So today I am going to try another tryhackme room called Dig Dug. This is about DNS servers. I have already completed the DNS room before so this shouldnt be a problem right?
### Let's get started.

### Room Link 
[Dig Dug](https://tryhackme.com/room/digdug)


### Let's get digging. I know a tool called dig which is used to query DNS servers. So I will use that because the room name is dig dug lol.

```
                                                                                                                                                 
┌──(igris㉿kali)-[~/Desktop]
└─$ dig 10.201.125.40 

; <<>> DiG 9.20.11-4+b1-Debian <<>> 10.201.125.40
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 23965
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;10.201.125.40.                 IN      A

;; AUTHORITY SECTION:
.                       86397   IN      SOA     a.root-servers.net. nstld.verisign-grs.com. 2025101301 1800 900 604800 86400

;; Query time: 72 msec
;; SERVER: 8.8.8.8#53(8.8.8.8) (UDP)
;; WHEN: Mon Oct 13 22:52:21 EDT 2025
;; MSG SIZE  rcvd: 117
```
### Well that wasnt so useful. I also did a nmap scan just in case.
```
                                                                                                                                                 
┌──(igris㉿kali)-[~/Desktop]
└─$ nmap -sV 10.201.125.40 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-13 22:39 EDT
Nmap scan report for 10.201.125.40 (10.201.125.40)
Host is up (0.27s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.40 seconds
```

### Wow port 53 didnt show up. I knew that port 53 was for DNS services so why isnt it open????
### I searched online for a while and found out that we can add custom domains in the dig search.
### In the room description there was a hint
```
Oooh, turns out, this 10.201.125.40 machine is also a DNS server!
If we could dig into it, I am sure we could find some interesting records!
But... it seems weird, this only responds to a special type of request for a givemetheflag.com domain?
```
### Here there was a mention of givemetheflag.com so I tried to use that in dig.
```
┌──(igris㉿kali)-[~/Desktop]
└─$ dig -h   
Usage:  dig [@global-server] [domain] [q-type] [q-class] {q-opt}
            {global-d-opt} host [@local-server] {local-d-opt}
            [ host [@local-server] {local-d-opt} [...]]
```
### I got a way to insert the domain in the serach with the command dig givemetheflag.com @{server}. And i used it.
```
┌──(igris㉿kali)-[~/Desktop]
└─$ dig givemetheflag.com @10.201.125.40

; <<>> DiG 9.20.11-4+b1-Debian <<>> givemetheflag.com @10.201.125.40
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61061
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;givemetheflag.com.             IN      A

;; ANSWER SECTION:
givemetheflag.com.      0       IN      TXT     "flag{REDACTED}"

;; Query time: 272 msec
;; SERVER: 10.201.125.40#53(10.201.125.40) (UDP)
;; WHEN: Mon Oct 13 23:00:27 EDT 2025
;; MSG SIZE  rcvd: 86

```

### Just like that we got the flag and the room was over !!!