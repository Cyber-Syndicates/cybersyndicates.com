---
author: slacker007
comments: false
date: 2016-06-20 01:51:04+00:00
layout: post
link: http://cybersyndicates.com/2016/06/mercenary-linux-1-hunt-dist/
published: false
slug: mercenary-linux-1-hunt-dist
title: Mercenary-Linux Hunt Distro
wordpress_id: 964
categories:
- Open Source Black Team Projects
tags:
- Active Monitoring
- Cyber Defense
- hunt distro
- Intrusion Detection
- linux
- mercenary-linux
- nmap
- open source
- python
---

## **MERCENARY-LINUX**



[![newUI_mhf](http://cybersyndicates.com/wp-content/uploads/2016/06/newUI_mhf-1024x768.png)](http://cybersyndicates.com/wp-content/uploads/2016/06/newUI_mhf.png)



https://twitter.com/cybersyndicates

https://twitter.com/real_slacker007

https://twitter.com/reaperb0t

**[DOWNLOAD NOW!](http://cybersyndicates.com/downloads/)**

**Description:** Mercenary Linux is a lightweight distribution of (mostly) Dockerized tools built for field expedient hunting, forensics, and malware analysis. Mercenary-Linux came about as all good ideas usually do: a _solution_ to a problem.

**Problem: **My particular situation was the unorganized and inefficient methods and platforms that my associates and I were using for hunt/digital forensic tasks.  We needed a way to seamlessly coordinate and correlate our information during hunt team operations.

**Solution: **This problem birthed MHF (Mercenary Hunt Framework).  MHF allows us to perform our hunt operations within a framework that aggregated our data in a centralized repository and share information transparently.  Even with minimal capabilities, it made our life so much easier during hunt operations. This held especially true when the network environment was large and complex.  After a couple uses we decided to add a database backend that we could later use to add additional functionality.  We decided to use the Big-Data analytics beast Neo4j, and it didn't disappoint.  So now we had a pretty useful framework that we could use to perform hunt tasks while aggregating and correlating our findings immediately but most importantly transparently to the user.  Next we created relational algorithms to run on the backend against our data! This provided us with an automated way to more quickly and efficiently put our findings to use.

The longer we worked on the framework, the larger and more useful it became.  That brought about more tools and more complexity.  To alleviate that, we came up with Mercenary-Linux! We decided to aggregate our most useful tools to a single distro so that we could incorporate additional tools into the framework and standardize the output so that our back-end analytics could ingest it.  We also used that as an opportunity address another problem, which was a universal distro that could do all levels of forensics across each platform.  So  whether you're doing network forensics, or analyzing a Windows or *Nix system; Mercenary-Linux can more than handle the task.  This made it easier to offer the framework already installed, on a lightweight Linux distro. This would ensure that all the dependencies are installed, and make the framework much easier to use and maintain.

But, that's not all folks! We utilized docker for many of the tools to avoid the aches and pains of using tools that require different versions of the same dependencies. Lets admit, when you need to use a tool---you need to use a tool. The last thing you want to do is spend an hour (or more) downloading and configuring a tool just to use it once. Or even worse, download a tool and spend hours to configure it, just for it to end up not working. To alleviate those issues with our framework we decided to provide the framework, modules, and correlation separately within one concise distribution. We're proud to say that we've built one of the most universal and lightweight distros for hunting, forensics, and malware analysis!

**Key Features **




    
  * Default Credentials

    
    * Username: mercenary

    
    * Password: mercenary




    
  * Perform Team Based Hunt Ops using MHF (Mercenary Hunt Framework)

    
  * Data relational analytics using a neo4j driven database

    
  * Enumerate Windows Environments using wmic :

    
  * FPC with Netsniff-ng

    
  * Automated Extraction from memory captures with BareMonkey

    
  * And much more!!



**Tools List:**




    
  * [ANALYZEPDF](https://github.com/hiddenillusion/AnalyzePDF)

    
  * [ANGR (Docker)](https://hub.docker.com/r/angr/angr/)

    
  * [BINNAVI](https://github.com/google/binnavi)

    
  * [BRO](https://www.bro.org/)

    
  * [BULK_EXTRACTOR](https://github.com/simsong/bulk_extractor)

    
  * [CHOPSHOP (Docker)](https://hub.docker.com/r/mitrecnd/chopshop/)

    
  * [CRITS (Docker)](https://hub.docker.com/r/remnux/crits/)

    
  * [DNS BEACON DETECTION](https://github.com/slacker007/CS-Beacon-Detector)

    
  * [FAKEDNS](https://github.com/Crypt0s/FakeDns)

    
  * [GRR-GOOGLE RAPID RESPONSE (Docker)](https://hub.docker.com/r/grrdocker/grr/)

    
  * [JSDETOX (Docker)](https://hub.docker.com/r/remnux/jsdetox/)

    
  * [MALTRIEVE (Docker)](https://hub.docker.com/r/remnux/maltrieve/)

    
  * [MASTIFF (Docker)](https://hub.docker.com/r/remnux/mastiff/)

    
  * [MERCENARY HUNT FRAMEWORK](https://github.com/slacker007/MHF)

    
  * [METASPLOIT FRAMEWORK (Docker)](https://hub.docker.com/r/linuxkonsult/kali-metasploit/~/dockerfile/)

    
  * [NEO4J](https://neo4j.com/)

    
  * [NETSNIFF-NG](http://netsniff-ng.org/)

    
  * [NETWORKMINER](http://www.netresec.com/?page=NetworkMiner)

    
  * [NMAP](https://nmap.org/)

    
  * [OFFICEPARSER](https://github.com/unixfreak0037/officeparser)

    
  * [PESCANNER (Docker)](https://hub.docker.com/r/remnux/pescanner/)

    
  * [PYTHON2.7](https://www.python.org/download/releases/2.7/)

    
  * [PYTHON3](https://www.python.org/download/releases/3.0/)

    
  * [PYTHON3.4](https://www.python.org/download/releases/3.4.0/)

    
  * [RADARE2 (Docker)](https://hub.docker.com/r/remnux/radare2/)

    
  * [REKALL (Docker)](https://hub.docker.com/r/remnux/rekall/)

    
  * [SIFT (Docker)](https://github.com/kost/docker-sift)

    
  * [STRACE](http://linux.die.net/man/1/strace)

    
  * [TCPDUMP](http://www.tcpdump.org/)

    
  * [UNXOR](https://github.com/tomchop/unxor)

    
  * [V8 (Docker)](https://hub.docker.com/r/remnux/v8/)

    
  * [VIPER (Docker)](https://hub.docker.com/r/remnux/viper/)

    
  * [VIVISECT](https://github.com/vivisect/vivisect)

    
  * [VOLATILITY (Docker)](https://hub.docker.com/r/remnux/volatility/)

    
  * [WIRESHARK](https://www.wireshark.org/)

    
  * [WMIC-EXE](https://www.aldeid.com/wiki/Wmic-linux)

    
  * [YARA (Docker)](https://hub.docker.com/r/blacktop/yara/)

    
  * [W3AF (Docker)](https://hub.docker.com/r/andresriancho/w3af/)

    
  * [WINEXE](https://www.aldeid.com/wiki/Winexe)




    
  * **How to Use:**

    
    * We are currently developing a Wiki with guides demonstrating the use of Merc-Linux on GitHub.






**[DOWNLOAD NOW!](http://cybersyndicates.com/downloads/)**

[contact-form to='slacker007@cybersyndicates.com' subject='Mercenary-Linux'][contact-field label='Name' type='name' required='1'/][contact-field label='Email' type='email' required='1'/][contact-field label='Website' type='url'/][contact-field label='Comment' type='textarea' required='1'/][/contact-form]
