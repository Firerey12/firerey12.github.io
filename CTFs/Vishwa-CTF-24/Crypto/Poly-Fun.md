---
layout: page
title: Poly Fun
categories: CTFs
permalink: /ctfs/Vishwa-CTF-24/Cryptography/poly-fun/
---

# Problem
Its a simple symmetric key encryption, I am sure you will be able to solve it (what do you mean the key looks weird)

Author : Revak Pandkar

FLAG FORMAT:
VishwaCTF{}

Files Provided:
- [encoded_flag.txt](Files/encoded_flag.txt)
- [encoded_key.txt](Files/encoded_key.txt)
- [challenge.py](Files/challenge.py)


# Solution
1. Looking at the files provided, we can see that we have an AES encrypted flag and a key which seems to be a substitution based cipher and the program with which the key was encoded.

    [encoded_flag.txt](Files/encoded_flag.txt):
    ```
    
    u5FUKxDUxH9y8yxvfaaU+GSXDwvJS6QxlN/3udOEzpU6fIVUExjDLsB3LKqUTz/x
    
    ```

    [encoded_key.txt](Files/encoded_key.txt):
    ```
    
    ☞➭⥄⫣Ⲋ⸹⿰ㆯ㍶☞⒗☞☞☞➭☞⥄☞⫣☞Ⲋ☞⸹☞⿰☞ㆯ☞㍶➭⒗➭
    
    ```

    [challenge.py](Files/challenge.py):
    ```python
    
    import numpy as np
    import random
    
    polyc = [4,3,7]
    poly = np.poly1d(polyc)
    
    
    def generate_random_number():
        while True:
            num = random.randint(100, 999)
            first_digit = num // 100
            last_digit = num % 10
            if abs(first_digit - last_digit) > 1:
                return num
    
    
    def generate_random_number_again():
        while True:
            num = random.randint(1000, 9999)
            if num % 1111 != 0:
                return num
    
    
    def transform(num):
        number = random.randint(1, 100000)
        org = number
        number *= 2
        number += 15
        number *= 3
        number += 33
        number /= 6
        number -= org
        if number == 13:
            num1 = random.randint(1, 6)
            num2 = random.randint(1, 6)
            number = num1 * 2
            number += 5
            number *= 5
            number += num2
            number -= 25
            if int(number / 10) == num1 and number % 10 == num2:
                number = generate_random_number()
                num1 = int(''.join(sorted(str(number), reverse=True)))
                num2 = int(''.join(sorted(str(number))))
                diff = abs(num1 - num2)
                rev_diff = int(str(diff)[::-1])
                number = diff + rev_diff
                if number == 1088:
                    org = num
                    num *= 2
                    num /= 3
                    num += 5
                    num *= 4
                    num -= 9
                    num -= org
                    return num
                else:
                    number = generate_random_number_again()
                    i = 0
                    while number != 6174:
                        digits = [int(d) for d in str(number)]
                        digits.sort()
                        smallest = int(''.join(map(str, digits)))
                        digits.reverse()
                        largest = int(''.join(map(str, digits)))
                        number = largest - smallest
                        i += 1
    
                    if i <= 7:
                        org = num
                        num *= 2
                        num += 7
                        num += 5
                        num -= 12
                        num -= org
                        num += 4
                        num *= 2
                        num -= 8
                        num -= org
                        return num
                    else:
                        org = num
                        num **= 4
                        num /= 9
                        num += 55
                        num *= 6
                        num += 5
                        num -= 23
                        num -= org
                        return num
            else:
                org = num
                num *= 10
                num += 12
                num **= 3
                num -= 6
                num += 5
                num -= org
                return num
        else:
            org = num
            num += 5
            num -= 10
            num *= 2
            num += 12
            num -= 20
            num -= org
            return num
    
    
    def encrypt(p,key):
        return ''.join(chr(p(transform(i))) for i in key)
    
    
    key = open('key.txt', 'rb').read()
    enc = encrypt(poly,key)
    print(enc)
    
    
    ```
    
    Initially looking at the program, it looked as if it was a complicated encryption using random numbers, but after analyzing the program, I found out that the only thing the encrypt() function was doing was using the equation described at the top of the code (4x<sup>2</sup> + 3x + 7) to get the encoded character code, which finally was returned as a string. The transform() function wasn't doing anything except returning the argument given to it despite the complicated looking code.

2. To decode the key, I came up with a code which solved for x in that equation and returned the original key's character code.

    [decode.py](Solution/decode.py):
    ```python
    
    import numpy as np
    import random
    import sympy
    
    # Opening encoded key
    encrypted = open("../Files/encoded_key.txt", "r", encoding="utf-8").read()
    
    # Setting up an array to store decoded characters
    dec = []
    
    # Setting up symbols for sympy
    x = sympy.symbols('x')
    
    # For each character in encrypted solve the equation for that characters ASCII code and append it to dec
    for char in encrypted:
        dec_code = sympy.solveset(sympy.Eq(4*x**2+3*x+7, ord(char)), x).args[1] # args[1] since the positive answer is stored in index 1
        dec.append(chr(dec_code))
    
    # Finally write the decoded key to file
    decrypted = open("decoded_key.txt", "w").write(''.join(dec))
    
    ```
    
    This finally gave us the decoded key
    ```
    
    12345678910111213141516171819202
    
    ```
    
    Using that key to solve the AES encoded flag, we got the final flag

## Flag
VishwaCTF{s33_1_t0ld_y0u_1t_w45_345y}
