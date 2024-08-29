---
layout: post
title: Active Directory Project
categories: SOC-Projects
---

In this project, I'll be setting up an Active Directory (home lab) that includes Splunk, Kali Linux & Atomic Red Team, Windows 10 and Windows Server 2022. I'll explore how a domain environment works, learn how to ingest events to a SIEM and generate telemetry related to attacks seen in the wild to help you detect them in the future.

Special Thanks to Steven over at [MyDFIR]((https://www.youtube.com/@MyDFIR)) for providing an amazing Tutorial on how to get Started with this lab.

## Logical Diagram

The first step was to come up with a logical diagram in order to understand how the data would flow between the nodes. I used draw.io for this since it was the easiest option to come up with a logical diagram

![Logical Diagram](https://github.com/user-attachments/assets/7ea03bf1-1113-43bf-9f52-1756cb94d174)

Once I was done creating the logical diagrams, it was time to move on and set up the Devices. I decided to use VMWare to accomplish this since I was most comfortable with that Virtualization Software.

## Setting up Virtual Machines

### Splunk Server (Ubuntu 22.04.4 Server)

I decided to use Ubuntu server to run the Splunk Indexer to receive logs from the Splunk Forwarder installed in the Domain Computers (Windows 10 and Windows Server 2022). The setup for the VM was pretty simple, I went with the default settings except for a slight change in the network settings where I set up a static IP for the Server as can be seen below.

![Splunk Server Network Settings](https://github.com/user-attachments/assets/fe4c2972-567a-4bf2-9f90-00cff47aa132)

All that is left to do with this is to install the Splunk Indexer.

### Windows Server 2022

I decided to use Windows Server 2022 to act as the Domain Controller for our Virtual Environment which will have installed Splunk Forwarder and Sysmon to send generate and send logs over to our Splunk Indexer.

I installed the Windows Server 2022 Standard (Desktop Experience) so as to get a GUI to interact with so as to make things easier.

![Microsoft Server 2022 Install Settings](https://github.com/user-attachments/assets/158a76f6-62c2-472b-b3ab-be1958a609c8)

The Virtual Hardware was pretty standard since this was just a Home Lab

![Microsoft Server 2022 VM Settings](https://github.com/user-attachments/assets/75b259a7-4b9a-4190-9e66-382a88a8a7d7)

### Windows 10 User

I decided to use a Windows 10 host, since I had that VM already set up with basic configurations from my previous Course related Assignments. All that was needed was to install the Splunk Universal Forwarder.

![Windows 10 User](https://github.com/user-attachments/assets/b46a4b49-867f-4fd4-b217-565df899b868)

### Kali Linux (Attacker)

For the attacker I decided to go with Kali Linux which is the most popular OS for Attacking. I went with the basic setup for this and downloaded the VMWare image from Kali's website.

## Installing & Configuring Software

### Active Directory Domain Services

We need to set up the Windows Server to act as the Active Directory Domain Controller, to do that we need to install the Active Directory Domain Services, we can do that by going to the Dashbord of Server Manager and clicking on Add Roles and Services. and select Active Directory Domain Services

![ADDS](https://github.com/user-attachments/assets/7d29a4a3-7fc2-44e8-a3fc-218ddc215160)

I went with the default settings in setting this up, creating a forest called "soclab.local"

Once that was done, we needed to promote the server to act as the Domain Controller, we do that by clicking the Flag which shows issues and click on Promote Server to DC. The Server would restart afterwards finally promoting our server to a Domain Controller.

Thereafter I created 2 Organizational Units (OUs) in our Domain, namely IT and HR.

![OUs](https://github.com/user-attachments/assets/5a536517-8469-41ab-8cc3-925bfc566f93)

I added 1 user each to the OUs namely John Doe and Jane Doe so as to login to the Windows 10 as.

![New User](https://github.com/user-attachments/assets/f46aefad-3637-49ce-96be-a0d8761f6b6e)

### Adding Windows 10 PC to the Domain

We neeeded to add the Windows 10 PC to the domain we just created (soclab.local), but before doing that we need to ensure that the PC knows where the soclab.local domain is, we can do this by changing the DNS server of the PC to our Windows Server (192.168.10.15). We can change that by heading to the Network Properties and changing the DNS server. Once that is done we can go ahead and add the PC to the domain.

To do that we needed to go to Advanced System Settings and then go the Computer Name tab and then Change Name and then input the details of the domain.

![Add to Domain](https://github.com/user-attachments/assets/bbb9f084-f683-44b2-9538-ff8f14cb2310)

### Sysmon (Windows 10 and Window Server 2022)

We need to install Sysmon on our Windows devices so as to generate logs to send over to our Splunk Server. We can install Sysmon by going to Microsoft's site and grabbing the zip file. Once that is done we need to Install the Sysmon service. We will use the config by https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml to set up our Sysmon. 

![sysmon_install](https://github.com/user-attachments/assets/fa6767f2-a0ad-48dc-9617-1d65ad6c4008)

Once that is done we can move forward with other configurations.

### Splunk Forwarder (Windows 10 and Windows Server 2022)

The Splunk Forwarder was to be installed on the Windows 10 User and the Windows Server 2022. I went with the most default settings

![Splunk Forwarder](https://github.com/user-attachments/assets/f57fcc5a-331d-46b6-a447-9efc355cf4fd)

Setting the indexer to the Splunk Server (192.168.10.10)

![Splunk Indexer Settings](https://github.com/user-attachments/assets/5767a394-42a5-4b5a-9692-7fe87ec7fc5e)

After installing the forwarder what was left was to give privilege to the SplunkForwarder service to actually send the logs to the indexer. This was done by changing the user that was running the service from NT SERVICE\SplunkForwarder to Local System Account

![NT SERVICE](https://github.com/user-attachments/assets/33c7a0c0-40f3-4b36-aceb-e94bf5dd64ae)


![Local System Account](https://github.com/user-attachments/assets/073c3666-f12f-4318-9fdb-e44a34225bc9)


There was one last thing to configure. What logs to be sent to the indexer, this was done by adding a config file to "C:\Program Files\SplunkUniversalForwarder\etc\system\local" and name it inputs.conf

`
[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
`

This config would send Application, Security, System, and Sysmon data to the endpoint index onto the Splunk Indexer

### Splunk Indexer (Ubuntu 22.04 Server)

The Splunk Server was going to be hosting the Splunk Indexer so I installed the Splunk Enterprise 9.3.0 onto the Server.

![Splunk Indexer](https://github.com/user-attachments/assets/33eb99bb-da49-4b82-86e5-9b25f432c62b)

We need to do some initial configuration on our Indexer so that we can receive data from the Forwarders. First, we need to add a receiving port from which to receive the data, we had set this to 9997 on both the Windows Server and Windows 10. We can do this by going to Settings > Forwarding and Receiving > Configure receiving > New Receiving Port > and then type in 9997.

There after we need to creat the index 'endpoint' in which to receive the data as we have set up in our inputs.conf file. We do this by going to Settings > Indexes > New Index > type in index details.

![New Index](https://github.com/user-attachments/assets/63e36308-f443-48d1-8daf-ed95a6f7d6f0)

And now to check if everything is receiving we can go to Search & Reporting, type in index=endpoint to look at telemtry from that endpoint.

![Splunk_Results](https://github.com/user-attachments/assets/b8111664-42d0-4fa7-b815-226db58bd689)

As we see above we can see around a thousand results coming in from our forwarders.

## Generating Malicious Telemetry

### Bruteforce Attack on RDP

Now that we have everything setup all that is left is to generate some malicious telemetry and see if can see that in Splunk. To do that I decided to go with Kali and run a bruteforce attack on the RDP. So before we move on with the attacking let's enable RDP on the Windows 10 machine.

We can do this by going to Advanced System Settings > Remote > Under Remote Desktop > Enable "Allow remote connections to this Computer"

![rdp](https://github.com/user-attachments/assets/2bd2b8c3-7151-49b5-9acd-e926b88cde70)

We still need to add the users who can RDP to the PC, we can do that by clicking Select Users in the Remote tab and adding our users.

![rdp2](https://github.com/user-attachments/assets/b52fbc92-9031-4c07-a3df-d85770fa98d1)

Great, now that we have RDP set up let's try running a bruteforce attack on it.

I decided to use Crowbar to run the attack. I set up a small password list which included the password of the users. I decided to attack Jane Doe's account.

![crowbar](https://github.com/user-attachments/assets/d6bd8d1c-afb4-422f-8dd2-3d3ebeb0b942)

As we can see above we got the password of Jane Doe's account.

Now let's see what kind of Telemetry this generates by searching using the filter "index=endpoint jdoe EventCode=4625" the event code stands 4625 is for Unsucessful Logins.

![logonfailed](https://github.com/user-attachments/assets/a08a8355-1529-4a14-b450-5ac5dd9ca243)

As we can see it generated a bunch of Logon Failed events which Splunk caught, and if we investigate where these requests originated from we can find our kali PC.

![networkinfo](https://github.com/user-attachments/assets/abfc1a47-36df-4b02-839e-23b72f540358)

Now let's see if we can find the successfull logon using the same filter as above but using the event code 4624 which is for Successful Logins

![logonsuccess](https://github.com/user-attachments/assets/472acdd9-f433-412d-8db1-d02870a111d9)

As we can see both the unsucessful and successful logon occur at the same time indicating a bruteforce attack and if we look into the network information of the sucessful logon we can see the Kali PC's IP

![networkinfo](https://github.com/user-attachments/assets/abfc1a47-36df-4b02-839e-23b72f540358)

