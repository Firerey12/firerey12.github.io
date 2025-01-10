---
layout: post
title: VulnHub DrippingBlues-1 Writeup
categories: VulnHub Machines
permalink: /vulnhub/dripping-blues-1
---

Machine: [https://www.vulnhub.com/entry/dripping-blues-1,744/](https://www.vulnhub.com/entry/dripping-blues-1,744/)
Machine Author: [tasiyanci](https://www.vulnhub.com/author/tasiyanci,765/)

## Nmap

```bash
# Nmap 7.94SVN scan initiated Fri Jan 10 05:29:37 2025 as: /usr/lib/nmap/nmap -Pn -p- -A --min-rate 5000 -oN scan.txt 192.168.56.138
Nmap scan report for 192.168.56.138
Host is up (0.00097s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxrwxrwx    1 0        0             471 Sep 19  2021 respectmydrip.zip [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.56.103
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 9e:bb:af:6f:7d:a7:9d:65:a1:b1:a1:be:91:cd:04:28 (RSA)
|   256 a3:d3:c0:b4:c5:f9:c0:6c:e5:47:64:fe:91:c5:cd:c0 (ECDSA)
|_  256 4c:84:da:5a:ff:04:b9:b5:5c:5a:be:21:b6:0e:45:73 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 2 disallowed entries 
|_/dripisreal.txt /etc/dripispowerful.html
MAC Address: 08:00:27:2A:B3:DB (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.98 ms 192.168.56.138

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jan 10 05:29:48 2025 -- 1 IP address (1 host up) scanned in 11.24 seconds

```

We seem to have a few ports open ftp, ssh and http. We should be focused on enumerating ftp and http. It seems nmap already provided us with a lot of information.

We can see that we are allowed to ftp anonymously and there seems to be a file respectmydrip.zip on the ftp server.

We can also see that there are two disallowed entries on the http server /dripisreal.txt and /etc/dripispowerful.html

Let’s start with the FTP.

## FTP

I downloaded the file respectmydrip.zip to my local machine and tried to unzip it but it seems to be password protected.

```bash
unzip respectmydrip.zip   
Archive:  respectmydrip.zip
[respectmydrip.zip] respectmydrip.txt password: 
   skipping: respectmydrip.txt       incorrect password
  inflating: secret.zip
```

Let’s go ahead and try to crack the password using John.

```bash
john hash.hash --show                                    
respectmydrip.zip/respectmydrip.txt:072528035:respectmydrip.txt:respectmydrip.zip::respectmydrip.zip

1 password hash cracked, 0 left

```

Unzipping them and looking at the files we have a note and another password protected zip. Let’s see if we can crack it using john again.

```bash
cat respectmydrip.txt 
just focus on "drip"                                                                                                                                                                                                                                            

unzip secret.zip       
Archive:  secret.zip
[secret.zip] secret.txt password: 
   skipping: secret.txt              incorrect password
```

```bash
john hash2.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:00 DONE (2025-01-10 05:41) 0g/s 16299Kp/s 16299Kc/s 16299KC/s "2parrow"..*7¡Vamos!
Session completed. 
```

It seems not. Let’s try enumerating the http to see if maybe there is something there.

Looking at the /dripisreal.txt we get this note

```bash
hello dear hacker wannabe,

go for this lyrics:

https://www.azlyrics.com/lyrics/youngthug/constantlyhating.html

count the n words and put them side by side then md5sum it

ie, hellohellohellohello >> md5sum hellohellohellohello

it's the password of ssh
```

Let’s go ahead and do as the note says and we get the below password

```bash
6f301f34ad3c7be06644d1e2f470669f
```

But we still don’t know what user to ssh as.

Let’s look at the other file that was also disallowed /etc/dripispowerful.html

![1](https://github.com/user-attachments/assets/0fcb9c30-4393-41d1-926f-7dd6348a5557)


It seems there isn’t a file.

After running feroxbuster, I was only able to find an index.php page.

![2](https://github.com/user-attachments/assets/fbb0350b-4927-4eb9-8b06-20cf26b3cd46)


I wasn’t able to find anything for a while, then while I was looking through what we had found I saw that note said focus on the ‘drip’ which made me think of trying to use drip as a parameter to index.php, which panned out.

![3](https://github.com/user-attachments/assets/df38ccfc-2344-4ef7-b05d-736986dee402)


We now have LFI on the system. Let’s try looking at the /etc/dripispowerful.html file now

![4](https://github.com/user-attachments/assets/fc81b475-6afe-4706-821c-6602b254de44)


There is just an image on the page. Looking deeper into the source code we find something interesting.

```bash
password is:
imdrippinbiatch
```

We seem to have found a password for something. Let’s try unzipping secret.zip using this password.

That doesn’t seem to work. Looking back at the /etc/passwd, there seems to be a user thugger. We can try using the password to ssh as thugger. That seems to work and we have ssh as thugger successfully.

```bash
thugger@drippingblues:~$ id
uid=1001(thugger) gid=1001(thugger) groups=1001(thugger)
thugger@drippingblues:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
thugger@drippingblues:~$ cat user.txt 
5C50FC503A2ABE93B4C5EE3425496521
```

Now time to escalate our Privileges

## Priv Esc

After looking around the system as thugger, I was not able to find much of anything to escalate privileges. So I decided to run linpeas to see what it could find, and it says that the system is vulnerable to the PwnKit. Running that gives me root.

```bash
root@drippingblues:~# ls
root.txt
root@drippingblues:~# cat root.txt 
78CE377EF7F10FF0EDCA63DD60EE63B8
```
