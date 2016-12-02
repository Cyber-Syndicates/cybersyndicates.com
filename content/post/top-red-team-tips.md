+++
author = "Alexander Rymdeko-Harvey"
tags = [ 'red-team'
]
date = "2016-11-30T16:08:28-05:00"
title = "6 Red Team Infrastructure Tips"
description = "Top tips you need to implement on your next engagement"
+++
## TL;DR
1. CLI Logging of All Operator Terminal
2. Restricted Payload Delivery 
3. Separate Servers / C2 Phases for all Stages of an Engagement
4. Always Use Redirectors in Front of Core C2
5. Full Session / Metadata Network Capture on Redirectors and C2
6. Never Share C2 IOC’s

****
As some of you may know I made a huge transition recently from my former team ATD (Adaptive Threat Division) at Veris Group to Sony’s internal Red Team. This was, of course, a major move, but the opportunity to help develop and stand up that capability was hard to pass up. This was a dream for me and to do it with some extremely dedicated and talented coworkers is always a bonus.

This brings me to our subject, recently I had put a ton of time into thinking out red team architecture and wanted to share. This will cover security and tips for a successful engagement. While some of this may seem “excessive”, you have to take this type of work very serious as you will be holding the keys to the kingdom of your company or target! With data dumps like shadow brokers apparently owning the equation group (Not here to say that’s true or valid data) nothing is safe and we must reduce our risk and only expose limited attack surface just as we help our clients.  
****
**1. CLI Logging of All Operator Terminals:**
You may be thinking why in world would I need this level of fidelity and data? Well, a really good buddy of mine [@sixdub](https://www.sixdub.net/) let me on to this a year or so ago for a potential integrity and data preservation method for potential discrepancies or de-conflictions. I’m taking this opportunity to push this forward on some of our first OP’s. Bellow is a tool built for just the right solution:  

*Find the setup here:*
<div class="github-card" data-github="killswitch-gui/lterm" data-width="400" data-height="153" data-theme="default"></div>
<script src="//cdn.jsdelivr.net/github-cards/latest/widget.js"></script>

*Here is a small example of global terminal logging:*
<iframe width="560" height="315" src="https://www.youtube.com/embed/3rbCTW_IBrk" frameborder="0" allowfullscreen></iframe>
****
**2. Restricted Payload Delivery :**
This is something I have not done in a bit and generally only required during specific circumstances. I’m here to tell you that we may be doing it all wrong, after an incident where a payload was spread via a secondary source and made its way out of scope you will want to take this advice. Ensure that not only are there clearly defined scoping and external IP space but take the time to setup your C2 to only allow communication to your payload stagger to trusted sources. A couple ways you can go about this which is some really cool work from @bluescreenofjeff is using Apache mod_rewrite rules. In a combination of the mod rewrite edits you can raise the sophistication level of uncovering your C2.   [bluescreenofjeff mod_rewrite series]( https://bluescreenofjeff.com/tags#mod_rewrite) has some great ways we can get supper paranoid during setup. Using rules for OS, mobile hits (how you want to handle these), browser based (using an HTA like this for IE:  [HTA-Example]( https://github.com/killswitch-GUI/PenTesting-Scripts/blob/master/ProxyAware-ps-Stager.hta)) and even handling user redirection not matching our C2 Profile we use. One of the main concerns is we want to ensure that our payload just does not stage to external IP space that me out of scope or even an IR toolset, this will increase the duration and sophistication to a long living C2 architecture.
{{< img-post path="/image" file="apache_mod_overview.png" alt="VPS with apache mod_rewrite" type="center" percent="100">}}

> Image Credit: https://bluescreenofjeff.com/2016-03-22-strengthen-your-phishing-with-apache-mod_rewrite-and-mobile-user-redirection/

****
**3. Separate Servers / C2 Phases for all Stages of an Engagement:**
This tip is crucial during the construction of your engagements infrastructure. Splitting up your C2 will provide you flexibility, stealth and ability to roll C2 on the unfortunate day you get burnt. So, what do I mean by “splitting” up the Infrastructure? Well in some cases you may be conducting a OP with one team server, but during a red team, all phases of the OP should be separated. 
<iframe width="560" height="315" src="https://www.youtube.com/embed/4w7krkqxRck" frameborder="0" allowfullscreen></iframe>
This is a bit of safeguard against burning all of your C2 at one time as well as providing redundancy, and a separate team server for coworkers. I have found splitting this up into three core components as seems to be quite effective! Bellow is a basic layout and beginning of what our team arch should look like:  
{{< img-post path="/image" file="seprate_vps.jpg" alt="seprate vps red team" type="center" percent="100">}}
****
**4. Always Use Redirectors in Front of Core C2:**
This is not always enforced depending on engagement and type of operation you are conducting. Heck, I have broken this rule in all types of scenarios, for phishing or another quick testing. The reality of the matter your core C2 servers or “Team Servers” if you are a CS user are HVA (High-Value Assets) and need to be secured like one. This mean full-scale firewall to only allow the ports used for an engagement (80,443,8080) and IPTable rules for allowed IP space for ssh and inbound connections from our redirectors. While our redirectors will come and go we need to trust our core C2 servers, and if they are exposed at any given time all it takes is a patient attacker to gain access. Lastly, never re-use a redirector. NEVER CROSS STREAMS! 
{{< img-post path="/image" file="vps_redirectors.jpg" alt="red team redirctors" type="center" percent="100">}}

If you have the time I highly suggest looking at CobaltStikes video on setting up redirectors as this will most likely apply to multiple situations or tools. In most cases, i have found during my setup that Socat was reliable and solid as a brick. I even started asking around and it seems it's the go too. Here is the Socat example from CobaltStrike, the video bellow is a great walk through of this process:
 
```
socat TCP4-LISTEN:80,fork TCP4:54.197.3.16:80
```
<iframe width="560" height="315" src="https://www.youtube.com/embed/PyJu_LYosts" frameborder="0" allowfullscreen></iframe>

****
**5. Full Session / Metadata Network Capture on Redirectors and C2:**
I know you going to question this! Well, its necessary after recent events with cobalt strike and Metasploit our tools are being targeted! I know asking for full PCAP on our redirectors will not be possible for most, while ill be shooting for a certain retention time on core architecture we need at least PCAP metadata (IP SRC/DST, PORT SRC/DST). This can be a simple text file, but this will give you something to go back to god forbid you have multiple redirectors and need to find an originating point of a callback. Something as simple as this can give you a ton of insight if an incident does happen. But in today's age maybe we should be shooting for full pcap? What do you think? It could be as simple as this:  
```
$ sudo tcpdump -n > test.txt
```
****
**6. Never Share C2 IOC’s!** Let’s be honest we have gone far out of our way to stand up and build solid and secure architecture, let’s not get its all burnt with one mistake. When we hear IOC we think malware etc, in this case, we mean IOC’s that would link or unravel our C2 easily without a full blown investigation. 

  1. Do not share domain names by using sub-domains across any part of the separate core C2 servers. 

  2. Register all domain with different POCs or only use private registration. With new tools like Whoisology.com doing reverse register name lookups takes seconds. Within minutes the blue team will be able to use hard indicators to destroy your foothold.
  * We setup three servers for Phishing, Active OP’s, and Long Term and we put them all in the same geographical region and /24 IP-space. Well, that was an easy win for the blue team! Please use separate geographical regions and IP-space for redirectors and core C2 servers. 
  * I have seen this done often and it's generally not an issue, but if you use the same C2 transport for all three separate servers you may have just burnt it all. Let’s say I use CS-HTTP stagers and the blue team catches my phishing server. That team is going to write custom signatures to ID that sort of traffic. Take the time use DNS/HTTP/HTTPS at all stages to give them a treat 
  * Following the last recommendation CS supports malleable C2 profiles, use them they work trust me! With that, you need to be careful if your target is Government related think twice with using streaming services. Use this in good taste or write custom ones that your target will most likely use, and think about what they may block. 
  * Going to host a payload? Take the time and stand up a custom droplet and web server to host payloads that will be used for phishing or lateral movement. This way you're not opening your core C2 to more potential attack surface. Go ahead and look at my baseline apache server, you’re not going to get far. 

  {{< img-post path="/image" file="vps_full_setup.jpg" alt="red team redirctors" type="center" percent="100">}} 
****
Any questions or concerns do not hastate to ask, as always hope I was able to share some basic tips for your next engagement. 



