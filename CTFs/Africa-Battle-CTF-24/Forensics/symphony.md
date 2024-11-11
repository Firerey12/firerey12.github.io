---
layout: page
title: Agent 47
categories: CTFs
permalink: /ctfs/africa-battle-ctf-24/forensics/symphony/
---

[Back to Forensics Challenges](../)

## Problem

![description](https://github.com/user-attachments/assets/4d2edd4e-fb62-408e-bca6-ca6f7304f5aa)

Looking at the file we can see a bunch of hex characters, which looks to be a hexdump of a file with the 3rd and 4th characters missing. So it seems what we need to do is recover those characters to get the file back.

![misshex](https://github.com/user-attachments/assets/35ad4659-588b-4042-bed7-d32ce43f8617)

So I decided to look for all magic bytes that begin with 52 49 which led me to the conclusion that it was of a RIFF filetype which is commonly used for audio files.

![possibfile](https://github.com/user-attachments/assets/30825262-bf65-4269-86bb-652492f72d8e)

So I decided to replace the X with 46 and add the 57 41 56 45 after the next 4 bytes to see if that would work, but that didn’t recognize the audio properly. So I decided to just import the raw data using Audacity

Which gave me an audio that seemed to be morse code but sped up. So I slowed the audio down and sure enough it was morse code.

![audacitymorse](https://github.com/user-attachments/assets/7b8d1c25-b097-488d-956f-160fad4b4c83)

Which translated to:

```bash
FLAG: M0RS3 C0D3 !TRANSL4T0R@@WITH 4UD10 FE4TUR3512598648
```

Using the format from the description we get the final flag

```bash
BattleCTF{M0RS3_C0D3_!TRANSL4T0R@@WITH_4UD10_FE4TUR3512598648}
```
