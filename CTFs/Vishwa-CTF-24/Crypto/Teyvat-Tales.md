---
layout: page
title: Teyvat Tales
categories: CTFs
permalink: /ctfs/Vishwa-CTF-24/Cryptography/teyvat-tales/
---

# Problem
All tavern owners in Mondstadt are really worried because of the frequent thefts in the Dawn Winery cellars. The Adventurers’ Guild has decided to secure the cellar door passwords using a special cipher device. But the cipher device itself requires various specifications….which the guild decided to find out by touring the entire Teyvat.

PS: The Guild started from the sands of Deshret then travelled through the forests of Sumeru and finally to the cherry blossoms of Inazuma

Author: Amruta Patil

FLAG FORMAT:
VishwaCTF{}

Information Provided:
> https://ch69433157670.ch.eng.run/

# Solution
This challenge was very intriguing to me as a Genshin Impact player, since it was based on the world of Genshin. Initially we are greeted with these texts which seem to be written in the languages that are part of Genshin Impact's world.

![greetings](https://github.com/user-attachments/assets/bee12085-3320-4499-9767-acb78ec1444c)


It seems we were supposed to translate these texts into English, but since this was a webpage with input fields, Curiosity got the bettter of me an I decided to check the source code, which allowed me to find the script that was running the validation for those fields.

[script.js](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Cryptography/Teyvat%20Tales/Files/script.js)
```javascript

const submitBtn1 = document.getElementById("submit-btn-1");
const firstFour = document.querySelector(".first-four");

const submitBtn2 = document.getElementById("submit-btn-2");
const secFour = document.querySelector(".sec-four");

const submitBtn3 = document.getElementById("submit-btn-3");
const thirdFour = document.querySelector(".third-four");

const submitBtn4 = document.getElementById("submit-btn-4");
const fourthFour = document.querySelector(".fourth-four");


submitBtn1.addEventListener("click", ()=> {
    const inputText1 = document.getElementById("input1").value.trim();

    if (inputText1.toLowerCase() === "enigma m3") {
        firstFour.classList.remove("centered-align");
        firstFour.classList.add("hidden");
    }
    else{
        alert("Incorrect deciphering! Try again!")
    }
});

submitBtn2.addEventListener("click", ()=> {
    const inputText2 = document.getElementById("input2").value.trim();

    if (inputText2.toLowerCase() === "ukw c") {
        secFour.classList.remove("centered-align");
        secFour.classList.add("hidden");
    }
    else{
        alert("Incorrect deciphering! Try again!")
    }
});

submitBtn3.addEventListener("click", ()=> {
    const inputText3 = document.getElementById("input3").value.trim();

    if (inputText3.toLowerCase() === "rotor1 i p m rotor2 iv a o rotor3 vi i n") {
        thirdFour.classList.remove("centered-align");
        thirdFour.classList.add("hidden");
    }
    else{
        alert("Incorrect deciphering! Try again!")
    }
});

submitBtn4.addEventListener("click", ()=> {
    const inputText4 = document.getElementById("input4").value.trim();

    if (inputText4.toLowerCase() === "vi sh wa ct fx") {
        fourthFour.classList.remove("centered-align");
        fourthFour.classList.add("hidden");        
    }
    else{
        alert("Incorrect deciphering! Try again!")
    }
});

```

The script contained all the answers to the input fields, but as I thought, it wasn't that easy, since we were greeted with yet another cipher.

![cipher](https://github.com/user-attachments/assets/29b4380c-fe4b-49a5-9bf3-47469cdc0cb2)


It seemed to be a substitution cipher, and if we look at the answers to the fields, we can see that they look like configurations for the Enigma machine. So using those configurations to decrypt the cipher using [cryptii.com's Enigma Decoder](https://cryptii.com/pipes/enigma-machine) we get the final flag.

![enigma_decrypt](https://github.com/user-attachments/assets/3ef66d52-f4e6-40a0-b2e8-7c015f008e7c)


## Flag
VishwaCTF{beware_of_tone-deaf_bard}
