---
layout: post
title: Plotted TMS
thumbnail-img: /assets/img/
cover-img: /assets/img/
categories: THM
---

> Happy Hunting!

Tip: Enumeration is key!

## Reconnaissance

First let's start with scanning the machine. 
There are a handful of possiblities to do this - [Nmap](https://nmap.org/), [Rustscan](https://github.com/RustScan/RustScan), [Angry IP Scanner](https://angryip.org/), but I go with the classic - NMAP!

```
nmap -sC -sV <ip-address>
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-19 05:36 EST
Nmap scan report for 10.10.67.203
Host is up (0.21s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a3:6a:9c:b1:12:60:b2:72:13:09:84:cc:38:73:44:4f (RSA)
|   256 b9:3f:84:00:f4:d1:fd:c8:e7:8d:98:03:38:74:a1:4d (ECDSA)
|_  256 d0:86:51:60:69:46:b2:e1:39:43:90:97:a6:af:96:93 (ED25519)
80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
445/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 61.86 seconds
```

## Directory discovery

AKA, directory enumeration - Think about the author's tip!

Let's launch our tool...

```
gobuster dir -u http://<ip-address>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.67.203/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/02/19 05:36:58 Starting gobuster in directory enumeration mode
===============================================================
/axxxx                (Status: 301) [Size: 312] [--> http://<ip-address>/axxxx/]
/sxxxxx               (Status: 200) [Size: 25]                                  
/pxxxxx               (Status: 200) [Size: 25]
```

There is a handful of things that can be found first, but are any of them real?

These three directories are a trap, it seems the author likes base64...

`/axxxx` holds a file named `id_rsa`, but it's too small to hold an actual `id_rsa` key!

These are the contents of it:

```
VHJ1c3QgbWUgaXQgaXMgbm90IHRoaXMgZWFzeS4ubm93IGdldCBiYWNrIHRvIGVudW1lcmF0aW9uIDpE
```

Ha! Well, let's go for the other two.

The second directory found is `/sxxxxx` and holds this `bm90IHRoaXMgZWFzeSA6RA==`. Again, this is a base64 message from the author. 

What will there be in the third directory, `/pxxxxx`?

No surprise there, it holds exactly the same contents as the second...

Let's continue to the next port hosting an apache web server!

Enumerate, enumerate, enumerate!

```
gobuster dir -u http://<ip-address>:<port>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.67.203:445/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/02/19 05:43:15 Starting gobuster in directory enumeration mode
===============================================================
/mxxxxxxxxx           (Status: 301) [Size: 322] [--> http://<ip-address>:<port>/mxxxxxxxxx/]
```

Looks like there is only one directory that might interest us... Let's enumerate some more!

```
gobuster dir -u http://<ip-address>:<port>/mxxxxxxxxx/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.67.203:445/management/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/02/19 05:44:51 Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 330] [--> http://<ip-address>:<port>/mxxxxxxxxx/uploads/]
/pages                (Status: 301) [Size: 328] [--> http://<ip-address>:<port>/mxxxxxxxxx/pages/]  
/admin                (Status: 301) [Size: 328] [--> http://<ip-address>:<port>/mxxxxxxxxx/admin/]  
/assets               (Status: 301) [Size: 329] [--> http://<ip-address>:<port>/mxxxxxxxxx/assets/] 
/plugins              (Status: 301) [Size: 330] [--> http://<ip-address>:<port>/mxxxxxxxxx/plugins/]
/database             (Status: 301) [Size: 331] [--> http://<ip-address>:<port>/mxxxxxxxxx/database/]
```
Huh... Well, there are some interesting directories here...

Let's check the website out!



