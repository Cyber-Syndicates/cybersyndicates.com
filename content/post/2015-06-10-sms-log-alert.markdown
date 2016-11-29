---
author: killswitch
comments: true
date: 2015-06-10 18:27:50+00:00
layout: post
link: http://cybersyndicates.com/2015/06/sms-log-alert/
slug: sms-log-alert
title: SMS-LogAlert.py - Alerting TeamServer Infrastructure
wordpress_id: 229
categories:
- Github
- Red Teaming / Pen Testing
- SMS-LogAlert
---

Recently on a red team engagement we where faced with a major issue during our initial access attempts while email phishing. We kept getting our Cobalt Strike (CS) beacon call backs to our team server at weird hours of the night, inherently missing the beacon and we were out of luck. The worst part was this was just about all we had to work with and we needed initial access bad!

With this came a problem, with any problem comes a great idea and some simple code to automate it. After some thought we knew we needed a easy way to alert the team when this payload beaconed home, and alert us that it was time to jump online and get started. Now beacons in CS are memory only, unless some method of persistence is employed. This means upon restart or power cycle access needs to be re-phished if you lose access to the target. But in this case the user was most likely "sleeping" his computer and thats where the random beacons hitting home where coming from.

With issue in hand we knew we needed some way to alert the team using "out of band" communication from the VPS infrastructure we had in place. With this came the idea of sending text messages (SMS) using the SMTP (Email) library in Python. This would easily alert us when he came back up online. We simply set up a IP-Tables rule to handle the work of logging when we had a call back to the server.  After we initiated SMS-LogAlert to tail the file, catch the ip Address, clear the log file, and send out an alert to the team with the IP that hit us. From there we where able to effectively manage our call back and we had a jackpot!

[![Screen Shot 2015-06-10 at 3.36.51 PM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-10-at-3.36.51-PM-1024x582.png)](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-10-at-3.36.51-PM.png)



### Get here on GitHub:




    
    <code><br></br>> git clone https://github.com/killswitch-GUI/SMS-LogAlert
    
    > Python SMS-LogAlert.py -t 10 -s 1oo -e alex@gmail.com -p 1233 -log /var/log/iptables.log
    
    </code>





### [*] Here is the simple Logic:






    
  * Performs simple check on supplied pass and user

    
  * Opens up log and checks for keyword and strips out Source IP

    
  * Sends SMS and clears log file

    
  * Places the IP in a list so multiple SMS arnt sent on previous IP's

    
  * Insures max SMS count has not been reached





### [*] Features:






    
  * Time to prevent log thrashing

    
  * Allows for custom log alerting

    
  * Supports MIME / or even HTML based alerting

    
  * Supports max SMS count

    
  * Sets specfic log file and time between checks





### [*] Limitations:






    
  * T-Mobile Sucks and doesn't allow SMTP to SMS

    
  * Road Map:

    
    * Script is only allowing gmail as its SMTP server currently (will add in support)

    
    * Need to add in IP range Support








### [*] OPTIONS:



-t = Will tell how long between log checks in Secs, defaults to 10 Secs.

-s = Max SMS texts that can recived before it shuts down, default is 100.

-e = Set required email addr user, ex [ale@email.com](mailto:ale@email.com)

-p = Set required email password

-log = Set a log to parse





### [!] Helpful Data:






    
  * AT&T: number@txt.att.net

    
  * T-Mobile: number@tmomail.net

    
  * Verizon: number@vtext.com

    
  * Sprint: number@messaging.sprintpcs.com or number@pm.sprint.com

    
  * Virgin Mobile: number@vmobl.com

    
  * Tracfone: number@mmst5.tracfone.com

    
  * Metro PCS: number@mymetropcs.com

    
  * Boost Mobile: number@myboostmobile.com

    
  * Cricket: number@sms.mycricket.com

    
  * Nextel: number@messaging.nextel.com

    
  * Alltel: number@message.alltel.com

    
  * Ptel: number@ptel.com

    
  * Suncom: number@tms.suncom.com

    
  * Qwest: number@qwestmp.com

    
  * U.S. Cellular: number@email.uscc.net



Source : http://20somethingfinance.com/how-to-send-text-messages-sms-via-email-for-free/

[taq_review]




