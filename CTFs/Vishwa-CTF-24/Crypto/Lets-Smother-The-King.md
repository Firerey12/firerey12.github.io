---
layout: page
title: Let's Smother The Kings
categories: CTFs
permalink: /ctfs/Vishwa-CTF-24/Cryptography/lets-smother-the-king/
---

# Problem

In my friend circle, Mr. Olmstead and Mr. Ben always communicate with each other through a secret code language that they created, which we never understand. Here is one of the messages Mr. Ben sent to Mr. Olmstead, which I somehow managed to hack and extract it from Ben's PC. However, it's encrypted, and I don't comprehend their programming language. Besides being proficient programmers, they are also professional chess players. It appears that this is a forced mate in a 4-move chess puzzle, but the information needs to be decrypted to solve it. Help me out here to solve the chess puzzle and get the flag.

Flag format: VishwaCTF{move1ofWhite_move1ofBlack_move2ofWhite_move2ofBlack_move3ofWhite_move3ofBlack_move4ofWhite}.

Note: Please use proper chess notations while writing any move.

Author : Naman Chordia

FLAG FORMAT:
VishwaCTF{write the chess moves}

Files Provided:
- [code.txt](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Cryptography/Lets%20smother%20the%20King!/Files/code.txt)

# Solution
This was a very interesting puzzle as it involved chess, a game which I enjoy but am not the best at. Nevertheless, it was an interesting solve. Looking at the description provided we can see some obvious clues, such as the names look like they could give us much information.
The description mentions a programming language and a file which is appropriately named code.txt. This tells us that the code.txt is some sort of weird programming language code or an obfuscated code. After a quick Google search of `olmstead programming language` we come across a wikipedia page for [Malbolge](https://en.wikipedia.org/wiki/Malbolge) a very 
difficult looking programming language, but one that looks a lot like our code.txt

[code.txt](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Cryptography/Lets%20smother%20the%20King!/Files/code.txt)
```

D'`A_9!7};|jE7TBRdQbqM(n&JlGGE3feB@!x=v<)\r8YXtsl2Sonmf,jLKa'edFEa`_X|?UTx;WPUTMqKPONGFjJ,HG@d'=BA@?>7[;4z21U54ts10/.'K+$j(!Efe#z@a}vut:[Zvutsrqj0ngOe+Lbg`edc\"CBXW{[TSRvP8TMq4JONGFj-IBGF?c=a$@9]=6|:3W10543,P*/.'&%$Hi'&%${z@a}vut:xqp6nVrqpoh.fedcb(IHdcba`_X|V[ZYXWPOsMLKJINGkKJI+G@dDCBA#"8=6Z4z21U5ut,P0)(-&J*)"!E}e#"!x>|uzs9qvo5srkpingf,Mchg`_%cbaZBXW{>=YXWPOs65KPImMLEDIBf)(D=a$:?>7<54X210/43,Pqp(',+*#G'gf|B"y?}v^tsr8vuWsrqpi/POkdibgf_%]b[Z_X|?>TYRQu8TMqQPIHGkEJIBA@dD&B;@?8\<5{3W70/.-Q10/on,%Ij('~}|B"!~}_u;y[Zponm3qpoQg-kjihgfeG]#aCBXW{[ZYX:Pt7SRQJONMLEiIHA@?cCB$@9]~<;:921U543,10/.-&J*j('~}${A!~}vuzs9Zponm3qpihmf,jihgfeG]#n

```

After this it was just a matter of finding a compiler for Malbolge. I used [this](https://www.tutorialspoint.com/execute_malbolge_online.php) compiler by tutorialspoint. After running the code we finally get the puzzle mentioned in the description.

[malbolge_result.txt](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Cryptography/Lets%20smother%20the%20King!/Solution/malbolge_result.txt)
```

White- Ke1,Qe5,Rc1,Rh1,Ne6,a2,b3,d2,f2,h2 Black- Ka8,Qh3,Rg8,Rh8,Bg7,a7,b7,e4,g2,g6,h7

```

Thereafter I just decided to use an online chess tool (https://nextchessmove.com/) to set up the board and also solve the moves (since I was lazy to solve the puzzle myself).

![Puzzle Solve](https://github.com/Firerey12/VishwaCTF_2024_Write_Ups/blob/f327b50e93a9f35d43e6578fb8061d0514f04d25/Cryptography/Lets%20smother%20the%20King!/Solution/puzzle_solve.png)

That gave us the moves for the flag:

```

Nc7+;Kb8;Na6;Ka8;Qb8+;Rxb8;Nc7#

```

## Flag

VishwaCTF{Nc7+_Kb8_Na6+_Ka8_Qb8+_Rxb8_Nc7#}
