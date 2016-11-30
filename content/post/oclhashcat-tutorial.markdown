---
author: killswitch
comments: true
date: 2015-06-10 18:04:20+00:00
layout: post
link: http://cybersyndicates.com/2015/06/oclhashcat-tutorial/
slug: oclhashcat-tutorial
title: OclHashcat - Tutorial
wordpress_id: 224
categories:
- Red Teaming / Pen Testing
- WIFI
post_format:
- Video
---

### HashCat Quick Start / Advanced Cracking Methods





* * *







After the  overwhelming amount of views on this subject I wanted to put together something a bit more detailed for those wanting to take it to the next level. When that list doesn't work and you need to crack that has some methods can be used to reduce cycles and greatly increase the chance of success.





#### Understanding the basics of OclHashcat and what you need to know to get started





Types of cracking that matter:






    
  * Brute-Force: In todays environment its not suppressing how often I revert to this

    
  * Brute-Force with custom Masks: This where you can make that $$

    
  * Wordlist: We will cover some of great lists out there that i use offten

    
  * Wordlist with Rules: While many online lists have been published rules build onto a great amount of data.



Hash Types we will cover:


    
  * NTLM - 500 (We see this often and it's a great baseline)

    
  * WPA2 / WPA - 2500



Distributed Cracking:


    
  * HashTopus Works Great!





### Basic Word List Cracking and Rules





WordList's that I use often:




    
  * [CrackStation](https://crackstation.net/buy-crackstation-wordlist-password-cracking-dictionary.htm)



Quick methodology of how I attempt to crack a password:

Run 1-7 all Char Brutes -> Run Simple Word Lists / Rules -> Run Larger hybrid Rule sets -> Custom masking 8+.





* * *





HASH CAT with Aircrack-ng



https://www.youtube.com/watch?v=XIhlAScPnuM

Using Aircrack-ng and ALFA card, I show you how simple it is to Bruteforce and crack a password after you get the WPA handshake through a few different means. Once I crack the key, I than use this key to do a simple decryption of the collected pcap in WireShark.

****LEGAL DISCLAIMER****
DO-not USE on any unauthorized targets
I DID THIS ON MY WIFI
This is informational to demonstrate the ease to test your WIFI
Use all tools at your own risk
JUST DON'T BE STUPID

Any request or comments just ask!

This is the download link
[http://hashcat.net/oclhashcat/](http://hashcat.net/oclhashcat/)

This is the help page!
[https://hashcat.net/wiki/doku.php?id=...](https://hashcat.net/wiki/doku.php?id=cracking_wpawpa2)

Converter for PCAP!
[https://hashcat.net/cap2hccap/](https://hashcat.net/cap2hccap/)

HERE IS THE .BAT SCRIPT --
The Link below is simply patebin so you can copy code!

[At My GitHub](https://github.com/killswitch-GUI/PenTesting-Scripts/blob/master/HASHCAT.bat)

or

https://github.com/killswitch-GUI/PenTesting-Scripts/blob/master/HASHCAT.bat

1) Place in notepad
2) save as .bat
3) place in the Hashcat Directory
4) click and go!
