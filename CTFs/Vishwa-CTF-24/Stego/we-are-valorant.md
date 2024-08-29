---
layout: page
title: We Are Valorant
categories: CTFs
permalink: /ctfs/Vishwa-CTF-24/steganography/we-are-valorant/
---

# We are Valorant

# Problem
One day, while playing Valorant, I received a mail from Riot Games that read,

“In a world full of light, sometimes the shadows help you win.” “Your Signature move also helps you a lot ; develop one and ace it now.”

It also had an image and a video/gif attached to it. I am not able to understand what they want to say. Help me find what the message wants to express.

Author : Abhinav Mehta

FLAG FORMAT:
VishwaCTF{}

Files Provided:
- [Astra_!!.mp4](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/7d0d4856e0ab910ae84db95c7a9cb9f9ff6932b7/Steganography/We%20Are%20Valorant/Files/Astra_!!.mp4)
- [we_are_valorant.jpg](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/7d0d4856e0ab910ae84db95c7a9cb9f9ff6932b7/Steganography/We%20Are%20Valorant/Files/we_are_valorant.jpg)

# Solution
Looking at the files we are given, the we_are_valorant.jpg seems to be corrupted, but the Astra_!!.mp4 does play. Initially looking at the mp4 file it doesn't really look like much, but after looking at it a few times, I noticed a slght flicker around the end of the file, so I looked at it Frame by Frame and got a word written saying "Tenz"

![password](https://github.com/user-attachments/assets/93c82055-245e-48d8-932c-71f92230bb95)


Now to figure out where to use this newly found clue. Only thing we had was a corrupted image. I tried to look into the hexdump to see if maybe the Magic Numbers were not intact since that usually is the case with the Easy level Steganography challenges. Turns out that hunch paned out and I corrected the Magic Numbers to that of a jpg file using hexedit.

![hexedit_before](https://github.com/user-attachments/assets/dd1833a5-899e-4f97-8bac-933851866b85) ![hexedit_after](https://github.com/user-attachments/assets/3f96a55b-a668-4379-aaa3-253d429de937)


That finally gives us an image back

![we_are_valorant](https://github.com/user-attachments/assets/25d72eb1-e40b-4a22-a19d-c05e2f8ae4c2)


The image doesn't really have anything much on it, it was just a regular image of some Valorant agents. Then I remembered we have a key ("Tenz"). So I tried to use steghide using "Tenz" as the password to extract any secret if any from the image. Turns out that was it, using that password, steghide gives us a file called not_a_secret.txt which gives us our final Flag.

![extracting_password](https://github.com/user-attachments/assets/e4ace6f0-9f5b-403f-921e-0a61bfc4a435)


## Flag
VishwaCTF{you_are_invited_to_the_biggest_valorant_event}
