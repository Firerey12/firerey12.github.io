---
layout: page
title: Happy Valentine's Day
categories: CTFs
permalink: /ctfs/Vishwa-CTF-24/Cryptography/happy-valentines-day/
---

# Problem
My girlfriend and I captured our best moments of Valentine's Day in a portable graphics network. But unfortunately I am not able to open it as I accidentally ended up encrypting it. Can you help me get my memories back?

Author : Pushkar Deore

FLAG FORMAT:
VishwaCTF{}

Files provided:
- [enc.txt](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Cryptography/Happy%20Valentines%20Day/Files/enc.txt)
- [source.txt](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Cryptography/Happy%20Valentines%20Day/Files/source.txt)

# Solution
Looking at the files we were provided. We can see that enc.txt is the encrypted file and the source.txt is a Python code that was used to encrypt it.
   
  [source.txt](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Cryptography/Happy%20Valentines%20Day/Files/source.txt):
  ```python
  from PIL import Image
  from itertools import cycle
  
  def xor(a, b):
      return [i^j for i, j in zip(a, cycle(b))]
  
  f = open("original.png", "rb").read()
  key = [f[0], f[1], f[2], f[3], f[4], f[5], f[6], f[7]]
  
  enc = bytearray(xor(f,key))
  
  open('enc.txt', 'wb').write(enc)

  ```

The code itself is pretty simple, all it does is it loads a file, reads it, and then uses its first eight bytes as the key, and then performs an XOR operation using the key on the file (The key is cycled on each run of the XOR operation). It then finally stores the XORed bytes onto a new file. In this case it saves it to a file called 'enc.txt', which is the file we were provided. So all we have to do is reverse this process.

To reverse the process, first we had to gather the key, which was the first eight bytes of the original png. This was easy to get since the first eight bytes of any file is the identifier for it. So I just grabbed the first eight bytes of any random PNG I could find in my PC and modified the original code provided to us.

  [solution.py](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Cryptography/Happy%20Valentines%20Day/Solution/solution.py):
  ```python

  from PIL import Image # Did not really need this
  from itertools import cycle
  
  # Function which performs XOR operation using key on every byte of file given to it
  def xor(a, b):
      return [i^j for i, j in zip(a, cycle(b))]
  
  # Loading Encrypted File
  f = open("../Files/enc.txt", "rb").read()
  
  # key is the first 8 bytes of any intact PNG
  key = [137, 80, 78, 71, 13, 10, 26, 10]
  
  # Saving decrypted bytes to a bytearray
  dec = bytearray(xor(f,key))
  
  # Writing to a new file the decrypted bytes.
  open('recovered.png', 'wb').write(dec)

  ```
Finally after running the code, we finally get the recovered image, which shows us the flag.
   
![recovered](https://github.com/user-attachments/assets/70b64e80-f49b-48ee-a565-486edaf0887b)


## Flag
VishwaCTF{h3ad3r5_f0r_w1nn3r5}
