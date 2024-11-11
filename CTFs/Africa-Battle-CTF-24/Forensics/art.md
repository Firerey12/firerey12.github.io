---
layout: page
title: Art
categories: CTFs
permalink: /ctfs/africa-battle-ctf-24/forensics/art/
---

[Back to Forensics Challenges](../)

## Problem

![description](https://github.com/user-attachments/assets/ad67419d-3d8b-47ba-a2ea-7e4fe5160aea)

Provided Files:
- Art.txt

Looking at the file initially it looks weird

![weird](https://github.com/user-attachments/assets/0bfa0579-6e0b-4bcd-934d-198b6422a828)


Scrolling down a little we can see something which looks like it could be part of a QR Code

![lookslikeqr](https://github.com/user-attachments/assets/079f7a53-3160-4039-a5aa-030004f99207)


So I decided to Zoom out the file and see if we could get a QR Code. Which sure enough panned out

![qrcode](https://github.com/user-attachments/assets/9099f0fc-16b7-4101-8c27-a3bdbcb29639)


Which after being scanned led me to the below string

```bash
HMUlOqE2RpHGPtEbP3jKUEesRNHwF2P5jTsPPNEXwREP4FWiPcnkRBHQGlEXxDUPqU4aEBl4EAMtaG+FD===
```

It looks like a base64 string, so I tried to base64 decode it but it didn’t give me anything legible.

Looking back at the zoomed out file, there were two places that were darkened compared to the rest of the file at the top and the near bottom. So I tried zooming onto those location to see what it was, and I found these two things. A morse code text and a key Deep.

![morse](https://github.com/user-attachments/assets/3078aee1-d82c-42c5-ba80-4b46dd44b03f)

![key](https://github.com/user-attachments/assets/100ce9be-d82c-484f-bbcf-d59752462770)

I decoded the morse code to see what it said, which gave me this

```bash
LOOKDEEPER,YOUWILLFINDTHEKEY.
```

I assumed it was referring to the key Deep that we found.

After this I tried a bunch of things I tried using Vigenere on the string and then base64 decoding to no luck. So I left this.

Then after a while since no one was able to solve and many people were asking if there was something wrong with the challenge, the organizers gave a hint.

![hint](https://github.com/user-attachments/assets/87f00e33-56c9-48ac-a9eb-a1314ce2d6bf)

So it seems that we have most of it solved, but it says that the data has been compressed using a popular JavaScript library. I searched on Google what the popular JavaScript libraries were for compressing and tried a few until I found one that worked which was LzString. I had to Vigenere Decode the string and then apply LzString decompress. I used CyberChef for this.

Which finally gave us the flag

```bash
BattleCTF{Looking_00DeepInsid3_Fragment&Fr4m3}
```
