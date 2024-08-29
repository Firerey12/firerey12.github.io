---
layout: page
title: Secret Code
categories: CTFs
permalink: /ctfs/Vishwa-CTF-24/steganography/secret-code/
---

# Problem
Akshay has a letter for you and need your help

Author : Ankush Kaudi

FLAG FORMAT:
VishwaCTF{}

Provided Files:
 - [confidential.jpg](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Steganography/Secret%20Code/Files/confidential.jpg)
 - [letter.txt](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Steganography/Secret%20Code/Files/letter.txt)

# Solution
Looking at the files provided to us, we don't really have much of a clue towards the solve, all we have is a letter and a black image.

[confidential.jpg](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Steganography/Secret%20Code/Files/confidential.jpg):

![confidential](https://github.com/user-attachments/assets/ee710193-25a0-40f8-b4db-fb1c739f5f61)

[letter.txt](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Steganography/Secret%20Code/Files/letter.txt):

```

To,
VishwaCTF'24 Participant

I am Akshay, an ex employee at a Tech firm. Over all the years, I have been trading Cypto currencies and made a lot of money doing that. Now I want to withdraw my money, but I'll be charged a huge tax for the transaction in my country.

I got to know that you are a nice person and also your country doesn't charge any tax so I need your help. 

I want you to withdraw the money and hand over to me. But I feel some hackers are spying on my internet activity, so I am sharing this file with you. Get the password and withdraw it before the hackers have the access to my account.

Your friend,
Akshay


```

Considering this is a Steganography challenge, we can assume that something is hidden in the image. Running Binwalk on the image confirms our doubts.

![binwalk](https://github.com/user-attachments/assets/be57a4bd-32b8-4497-afe8-4f487772a737)


We can see that there is a zip file hidden within the image. After extracting it we get two new files to work with. [5ecr3t_c0de.zip](Solution/5ecr3t_c0de.zip) (password-protected) and [helper.txt](Solution/helper.txt)

[helper.txt](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Steganography/Secret%20Code/Solution/helper.txt):
```

Hey buddy, I'm really sorry if this takes long for you to get the password. But it's a matter of $10,000,000 so I can't risk it out.

"I really can't remember the password for zip. All I can remember is it was a 6 digit number. Hope you can figure it out easily"


```

This tells us that the password for the zip file is a 6 digit number. So we just need to bruteforce the password in order to get access to the zip file. We can write a python code to come up with a wordlist which we can then crack using John.

[gennum.py](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/7d0d4856e0ab910ae84db95c7a9cb9f9ff6932b7/Steganography/Secret%20Code/Solution/gennum.py):
```python

with open("nums.txt", "w") as f:
    for i in range(100000, 1000000):
        f.write(f"{i}\n")

```
Now that we have the wordlist, let's crack the password using John. To do that we need to create a hash of the zip using zip2john and then crack that hash.

![john](https://github.com/user-attachments/assets/2073c424-a1ff-4940-b817-728a5c8929c7)


That gives us the password _**945621**_ which we can then use to extract the zip file, which gives us the files [5ecr3t_c0de.txt](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/7d0d4856e0ab910ae84db95c7a9cb9f9ff6932b7/Steganography/Secret%20Code/Solution/5ecr3t_c0de/5ecr3t_c0de.txt)

[5ecr3t_c0de.txt](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/7d0d4856e0ab910ae84db95c7a9cb9f9ff6932b7/Steganography/Secret%20Code/Solution/5ecr3t_c0de/5ecr3t_c0de.txt) (snippet truncated since it was too long):

```

(443, 1096)
(444, 1096)
(445, 1096)
(3220, 1096)
(3221, 1096)
.
.
.
.
(3226, 1165)
(443, 1166)
(444, 1166)
(3221, 1166)
(3222, 1166)

```

[info.txt](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/7d0d4856e0ab910ae84db95c7a9cb9f9ff6932b7/Steganography/Secret%20Code/Solution/5ecr3t_c0de/info.txt):
```

What are these random numbers? Is it related to the given image? Maybe you should find it out by yourself

```

It seems that we have some coordinates and a clue which says the coordinates are related to the image we were given (confidential.jpg), it seems pretty obvious what we are supposed to do.
It seems we just need to flip the pixels at those locations to another color so that they are visible. We can use Python to do this using the Pillow module.

[solution.py](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/7d0d4856e0ab910ae84db95c7a9cb9f9ff6932b7/Steganography/Secret%20Code/Solution/solution.py):
```python

import numpy as np
from PIL import Image

# Loading the image as an Image object using the Pillow's Image.open function.
img = Image.open("../Files/confidential.jpg")

# Loading the image's pixels
pixels = img.load()

# Array to store the coords from the 5ecr3t_c0de.txt
coords = []

# Reading the coords as a tuple from the 5ecr3t_c0de.txt
with open("5ecr3t_c0de/5ecr3t_c0de.txt", "r") as key:
    for line in key.readlines():
        x,y = line.strip().rstrip(")").lstrip("(").split(",")
        coords.append((int(x), int(y)))

# Changing all the pixels at the given coords into white (I decided to use white since the background was black so it would be most visible)
for x,y in coords:
    pixels[x,y] = (255,255,255)

# Saving the image
img.save("decoded.jpg")

```

Finally after all that, we get the Flag
![decoded](https://github.com/user-attachments/assets/632c307a-d6ce-4e7d-b84d-a29b636610d7)


## Flag
VishwaCTF(th15_15_4_5up3r_53cr3t_c0d3_u53_1t_w153ly_4nd_d0nt_5h4re_1t_w1th_4ny0ne}
