---
layout: page
title: Agent 47
categories: CTFs
permalink: /ctfs/africa-battle-ctf-24/forensics/agent-47/
---

[Back to Forensics Challenges](../)

## Problem

![description](https://github.com/user-attachments/assets/9c358a36-ea01-4ecf-8bd9-ea9b6b3b8e99)


Files Provided:
- Agent 47
  
Opening this file in a text editor we get a scrambled file.

![scrambled](https://github.com/user-attachments/assets/d945604c-2569-421c-aa09-addc96f262cb)


Looking closer we can see that it’s a PNG file with every two bytes being swapped with each other for example %PNG has become P%GN and IHDR has become HIRD.

By using a simple Python script we can reverse this process and recover the original PNG file.

```python
def swap_bytes_in_file(input_filename, output_filename):
    with open(input_filename, "rb") as infile, open(output_filename, "wb") as outfile:
        while True:
            # Read two bytes at a time
            bytes_pair = infile.read(2)
            
            # If less than two bytes are read, stop the loop
            if len(bytes_pair) < 2:
                # If only one byte remains, write it without swapping
                if len(bytes_pair) == 1:
                    outfile.write(bytes_pair)
                break
            
            # Swap the two bytes and write them to the output file
            outfile.write(bytes_pair[1:2] + bytes_pair[0:1])

# Usage
input_filename = "Agent47"
output_filename = "output.png"
swap_bytes_in_file(input_filename, output_filename)

```

This gives us the below PNG:

![output](https://github.com/user-attachments/assets/a7e28560-6f76-4f0c-ae74-8d1ae93f700f)


Looking into it we don’t really find anything interesting. Even after using tools such as Binwalk, Exiftool and Stegsolve, we don’t find anything interesting.

Then I decided to look at the challenge description. What stood out to me was Agent 47 and it saying to try 46, which hinted to there being a String rotated 46 characters. So I ran strings on the file and came with this weird looking string, which could be the flag rotated 46 characters.

```bash
r3FF>7s&vMzAa@1F:71d6Hc@FGD71;@1d8D;5pQehifcO
```

After rotating it 46 characters using CyberChef we get the final flag

```bash
BattleCTF{Jo1n_the_4dv3nture_in_4fric@!58963}
```
