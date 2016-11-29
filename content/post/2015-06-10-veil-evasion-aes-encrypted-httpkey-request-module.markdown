---
author: killswitch
comments: true
date: 2015-06-10 02:10:36+00:00
layout: post
link: http://cybersyndicates.com/2015/06/veil-evasion-aes-encrypted-httpkey-request-module/
slug: veil-evasion-aes-encrypted-httpkey-request-module
title: 'Veil-Evasion AES Encrypted HTTPKEY Request: Sand-Box Evasion'
wordpress_id: 132
categories:
- Red Teaming / Pen Testing
tags:
- Veil
- Veil-Evasion
---

* * *



https://www.youtube.com/watch?v=xAWqRoeW8r4

My buddy [@christruncer](https://www.christophertruncer.com/) set me up on a task to build a module that could be integrated into veil, and I was more than happy to help and contribute to the growing Framework that has been developed. It is becoming apparent that even though binaries are a secondary choice with newer advanced techniques to stay off disk, it’s always great to have a backup in case dropping a binary to disk is the last choice. But when you do have to drop an .exe to disk, you better be sure not to trigger a slew of appliances it has to traverse. That’s where [Veil-Evasion](https://github.com/Veil-Framework/Veil-Evasion) makes its money, building payloads that bypass AV.



[![PM](http://cybersyndicates.com/wp-content/uploads/2015/06/PM-1024x576.png)](http://cybersyndicates.com/wp-content/uploads/2015/06/PM.png)





###### _(Above is an image of some of the required options to get started)_



The payload I decided to develop for this month’s V-Day is a bit like the previous modules, but I added in a few extra functions to help get past some of those pesky high security environments. In which your standard binary may just not make it. This payload is AES encrypted using the shell-code inject method. The previous AES encrypted shell-code modules are using a random key and was complied with this random AES key. With this module the core shell-code inject function is the same, but the idea of sending out your payload with its decryption key it not always ideal. This can lead to quick response from the Blue / IR team by either some basic dynamic malware analysis, or simple sandboxing and your burnt! Next thing you know you have FireEye appliances scanning your C2 infrastructure and you’re up a creek.



[![ScreenPM](http://cybersyndicates.com/wp-content/uploads/2015/06/ScreenPM-275x300.png)](http://cybersyndicates.com/wp-content/uploads/2015/06/ScreenPM.png)





###### _(Above is an image of supplied sample wordpress.html page genrated)_







### Understanding SandBox Evasion





* * *



The first thing we need to understand here is we are adding in additional functionality not for heck of it. We are adding in the ability to bypass current and future attempts at analyzing the payload. Providing survivability of our C2 and ability to conduct operations moving forward if our binary is caught and we already have a presence in the the target space.

A great place to start is the different types of sandbox evasion methods being used and categorized by FireEye them self:




    
  * Human interaction - Checking for human input can help identify inhuman like behaviors

    
  * Configuration-specific - **sleep calls**, **time triggers**, process hiding, malicious downloaders, execution name of the analyzed files, volume information, and execution after reboot

    
  * Environment-specific—version, embedded iframes (in flash, swf, jpg files), embedded executable in an image file, and DLL loaders



In this case we want to focus on the sleep calls and how they can be used to circumvent sandbox environments. This all is based off the premises of efficient computing, sandboxes don't have all day to perform Dynamic Analysis. With a enterprise class network comes a range of hundreds to thousands of different files traversing the wire. This is the weak point of sandboxing, it has limited resources and it can only perform analysis on a file so long before it has to move on to the next. This isn't a flaw but merely necessary.

But evading sand boxes aren't just that easy, they fight back by employing a few methods to trick the malware. They can manipulate the time presented in the enviroment by either manipulating the current time to defect time triggers or even trick the malware into thinking more time has elapsed than actually has. If this isn't enough more advanced methods such as Multipath exploration can crawl code and execute or patch sleep functions by jumping them, and running code otherwise not accessible.

[![Screen Shot 2015-06-10 at 8.22.47 PM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-10-at-8.22.47-PM-1024x509.png)](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-10-at-8.22.47-PM.png)

 In this case we are using staged payloads which allows us to defeat all of the above. The only code visible is a MD5, HTTP Request and the sleep function. We have no need to loop a sleep to defeat extra commands taking place or bring along extra code. The payload will be encrypted and if the sandbox dose execlerate the sleep cycle no extra intelligence will be gained about the binary.    

[![Screen Shot 2015-06-10 at 8.23.14 PM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-10-at-8.23.14-PM-1024x99.png)](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-10-at-8.23.14-PM.png) 



### Expanding on Methodology





* * *



Using this module you prevent and can circumvent some of these issues presented with environments that do run a sandbox appliance. Lets walk through some of the basic functionality of the payload:




    
  1. Attacker deploys binary through a couple different methods.

    
  2. Binary is inspected by border devices (SandBox, Content filtering ex..)

    
  3. At runtime in the appliance, the executable starts an HTTP request attempting to query the web server.


<blockquote>_**This Web-server should be outside your C2 infrastructure**_</blockquote>




    
  4. Web-server returns a HTTP 404 code, the sleep timer engages and the appliance will never decrypt the actual payload.


<blockquote>_**You shouldn't be hosting your Key yet**_</blockquote>




    
  5. At runtime on target, the executable starts an HTTP request attempting to query the webserver holding a HTML page with a hidden random string appearing to be wordpress.html.

    
  6.  Once the payload receives a HTTP 200 code (If a failed request takes place it sleeps for default of 60 secs) it takes the source code of the page and runs it through a md5 function to hash the html output and produce the required 16 Byte key. Than injects it into memory, and executes it.





##### **_I built a small diagram to help depict some of this functionality and suggested infrastructure:_**





[![Screen Shot 2015-06-11 at 4.12.27 PM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-11-at-4.12.27-PM.png)](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-11-at-4.12.27-PM.png)-------------------------------





[![Screen Shot 2015-06-15 at 8.40.50 AM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-15-at-8.40.50-AM.png)](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-15-at-8.40.50-AM.png)[
](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-13-at-11.12.51-PM.png)-------------------------------





[![Screen Shot 2015-06-13 at 11.20.05 PM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-13-at-11.20.05-PM-1024x1005.png)](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-13-at-11.20.05-PM.png)





**Some Benefits of this Module**





* * *






    
  * Uses urllib2 to make get request against supplied server and returns the HTML text for md5 hashing.

    
  * Prevents Key being deployed with payload, preventing the payload from running in sandbox if target server is taken offline during initial deployment. (Circumventing flagging)

    
  * Once use of payload is over, attacker can take down web server to prevent future infections and hinder future analysis of the payload.

    
  * Static and future dynamic RE is near impossible without proper data collection.

    
  * Proper web log monitoring will identify when webserver is burnt and crucial to remove key to prevent future ability to RE binary.

    
  * Defenders would need to capture live memory to capture the decryption key.





** **





**Sources:**



[1] https://www.fireeye.com/content/dam/legacy/resources/pdfs/fireeye-hot-knives-through-butter.pdf

[2] http://www.sans.org/reading-room/whitepapers/malicious/sleeping-sandbox-35797

[taq_review]
