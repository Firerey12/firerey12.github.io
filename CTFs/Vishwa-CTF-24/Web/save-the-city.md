---
layout: page
title: Save The City
categories: CTFs
permalink: /ctfs/Vishwa-CTF-24/web/save-the-city
---

# Problem
The RAW Has Got An Input That ISIS Has Planted a Bomb Somewhere In The Pune! Fortunetly, RAW Has Infiltratrated The Internet Activity of One Suspect And They Found This Link. You Have To Find The Location ASAP!

Aurthor : Samarth Ghante & Harshali Patil

FLAG FORMAT:
VishwaCTF{}

Provided Information:
> nc 13.234.11.113 32641

# Solution
This was an Easy difficulty Web challenge which involved exploiting a vulnerable libSSH. 

Netcatting to the provided IP and Port provided us with plenty of information to solve this.

![initial_netcat](https://github.com/user-attachments/assets/5d9dbfe3-934f-4040-8f36-073b2a87d288)


This gives us the version of libSSH being used. Googling this gives us a bunch of exploits we can run. I went with mgeeky's [code](https://gist.github.com/mgeeky/a7271536b1d815acfb8060fd8b65bd5d#file-cve-2018-10993-py)
The code was pretty long so I decided not to put it here, but you should check it out at the link provided.

But basically running the code tells us how to use it.

![exploit_arguments](https://github.com/user-attachments/assets/36bf1c74-9b50-4dfd-8787-e8de944bb8f3)


This tells us how to use the program. So all we have to do is give it the IP and port and give it a command to run. I decided to run `ls` to see what files were in the directory.

![exploit_ls](https://github.com/user-attachments/assets/9a4b9867-bd41-481b-9f2f-4e41642d5088)


We can see a file named `location.txt` which is likely the flag.

Running the code again this time with `cat location.txt` as the command we get the Final Flag.

![final_location](https://github.com/user-attachments/assets/144c6e1f-3112-44de-8588-ef5f25c55cda)


## Flag
VishwaCTF{elrow-club-pune}
