---
author: killswitch
comments: false
date: 2016-05-24 14:34:01+00:00
layout: post
link: http://cybersyndicates.com/2016/05/return-empyre-series-3/
slug: return-empyre-series-3
title: The Return of the EmPyre
wordpress_id: 815
tags:
- EmPyre
---

[![empyre_logo_white_background](/wp-content/empyre_logo_white_background.png)](/wp-content/empyre_logo_white_background.png)



This post is number 3 of the EmPyre series and cross post with a fellow friend and ATD co-worker [@xorrior](https://twitter.com/xorrior). Thanks to the entire ATD family and dev team of EmPyre:





[@rvrsh3ll](https://twitter.com/424f424f)  -- [@harmj0y](https://twitter.com/harmj0y) -- [@xorrior](https://twitter.com/xorrior) -- [@CptJesus](https://twitter.com/CptJesus)





EmPyre can be found here: [https://github.com/adaptivethreat/EmPyre](https://github.com/adaptivethreat/EmPyre)





5/12/16 – [**Building an EmPyre with Python**](http://www.harmj0y.net/blog/?p=2637&preview=true)





5/18/16 – [**Operating with EmPyre**](http://www.rvrsh3ll.net/blog/empyre/operating-with-empyre/)



  



## EmPyre Persistence



Mac OS X offers several methods to abuse system functionality and obtain persistence through reboots. One of the most effective methods is [Dylib Hijacking](https://www.virusbulletin.com/virusbulletin/2015/03/dylib-hijacking-os-x/). Dylibs are comparable to DLLs in that they contain code executed by applications at runtime. Dylib Hijacking exists because of how “dyld”, the system dynamic linker, searches and loads these libraries. Let’s briefly examine the Mach-O header to understand why this vulnerability exists. 

**Figure 1**

**[![Picture1](/wp-content/Picture1.png)](/wp-content/Picture1.png)**

**Figure 2**

**[![Picture2](/wp-content/Picture2.png)](/wp-content/Picture2.png)**

Figure 1 depicts the load commands for Xcode. Load commands provide the OS loader with instructions on where to find the application's entry point, offsets for the text and data sections, and all of the libraries needed by the application at runtime. The “Name” field in the LC_LOAD_DYLIB load command identifies the file path to the DVTFoundation library. Notice that the path is prepended with @rpath. This signals to the OS loader to examine the LC_RPATH (Figure 2) load commands in order to expand the @rpath variable. Each path is searched in succession by dyld to locate the required library. Once the library is found it is loaded into the application at runtime. The issue is that any library planted in an LC_RPATH that is found before the legitimate dylib, will be loaded first. A slight variation to this attack, which we won’t cover in this post, involves the LC_LOAD_WEAK_DYLIB load command. This indicates that the specified dylib is not required but will be loaded if found. If the specified library is not present on the system, we can plant the dylib in the specified path and it will be loaded by the application at runtime. For more information on Dylib Hijacks for weak dylibs, please read the [white paper](https://www.virusbulletin.com/virusbulletin/2015/03/dylib-hijacking-os-x/) on Dylib Hijacks by @patrickwardle. The load commands slightly differ but still offer the same way to conduct this attack. We can abuse dyld’s load order to obtain consistent code execution and/or persistence in OS X. 

Before we can weaponize Dylib Hijacks, there is a small problem to address. When an application normally loads a library, the os loader will try to resolve symbols for functions required by the application. If those functions are not found, the application will crash. To remedy this, the hijacking dylib will need to have a LC_REEXPORT_DYLIB load command that provides the path to the legitimate dylib. When the application starts, it will load the attacking dylib first and then load the legitimate dylib.

To carry out this attack in EmPyre, you will need to first run the HijackScanner module in situational_awareness/host/osx/. This is simply an adaption to @patrickwardle’s [python script](https://github.com/synack/DylibHijack/blob/master/scan.py) to scan the system for Mach-O binaries, load each and examine the load commands to determine if the application is vulnerable. 

**Figure 3**

**[![Picture3](/wp-content/Picture3.png)](/wp-content/Picture3.png)**

With the default options for the module, every Mach-O binary on the file system will be examined, which can take quite some time. To speed up the scan, set a path or only scan loaded process executables. Once the scan is finished, your output will provide you with the path to the vulnerable binary and the full path to where an EmPyre dylib should be planted. You will also need to locate the legitimate dylib for the next module in this attack. 

**Figure 4**

**[![Picture4](/wp-content/Picture4.png)](/wp-content/Picture4.png)**

Here we see that the Xcode application is vulnerable to a dylib hijack with the DVTFoundation library. Instructions are provided after the scan output to gather the information necessary for the next module. 

The CreateHijacker module in persistence/osx/ configures an EmPyre dylib to be used in a Dylib Hijack. This is yet another slightly [modified script](https://github.com/synack/DylibHijack/blob/master/createHijacker.py) written by @patrickwardle. This module does all the heavy lifting for configuring the EmPyre dylib and patching in the path to the legitimate dylib.

**Figure 5**

**[![Picture5](/wp-content/Picture5.png)](/wp-content/Picture5.png)**

The Listener, UserAgent, and Arch options are all used to generate the hijacker dylib. Be sure that the architecture of the dylib matches the architecture of the vulnerable application. The “LegitimateDylibPath” option will define the full path to the legitimate dylib loaded into the application. The “vulnerableRPATH” refers to the rpath value returned from the HijackScanner module. Once we have all of our options set, we can execute the module.

**Figure 6**

**[![Picture6](/wp-content/Picture6.png)](/wp-content/Picture6.png)**

In Figure 6, we see that once the module is completed, an EmPyre dylib is configured and copied to the vulnerable rpath we specified. When we start the Xcode application, we receive a new EmPyre agent running in that application’s process! 

**Figure 7**

**[![Picture7](/wp-content/Picture7.png)](/wp-content/Picture7.png)**

**Figure 8**

**[![Picture8](/wp-content/Picture8.png)](/wp-content/Picture8.png)**

Important Note: In some cases, killing the agent will close the application. Closing the application will always kill the agent. 



## LoginHooks



Just like the many supported startup locations and run keys within Windows, OS X offers a few which are deprecated but still functional ([https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/Introduction.html#//apple_ref/doc/uid/10000172i-SW1-SW1](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/Introduction.html#//apple_ref/doc/uid/10000172i-SW1-SW1)). On the newest version of OS X, El Capitan,  “LoginHooks” seem to be an extremely reliable method of persistence and can be removed with ease. Here are a few things to note about “LoginHooks” and how they can be used:
    
  *      Permissions for your script file should include execute privileges for the appropriate users.
  *      The payload will execute for any user that logs in.
  *      Only one copyof each script can be installed at a time and root privileges are needed!
  *      If a user variable is required for login logic the $1 is passed your script.
  *      Other login actions wait until your hook finishes executing

A benefit to using LoginHooks is the ease of installation, and target system info enumeration using the “defaults” tool. This tool also allows you to write settings to multiple sub system settings. Here is manual process to  setup a Hook:

[snippet id="37"]

To remove this hook all you would need to do is use the defaults tool to delete it:


    
    [snippet id="35"]



Lastly to read the hook settings you setup:


    
    [snippet id="36"]



Here is a small demo I setup to show the install process of the LoginHook. First start out by setting up EmPyre, creating a listener, and getting your launcher executed in your test VM. Once the C2 is set up we will need to build out AppleScript that will be used for the persistence execution.

**Figure 9**

**[![Picture9](/wp-content/Picture9.png)](/wp-content/Picture9.png)**

This will output AppleScript with a simple stager to the current working directory. Next, we will have to upload the script to a directory of the operator’s choice. In this case, I just used the /tmp/ directory store my “evilscript”. To set up the hook is very simple, set the user password, this is for the sudo that takes place to install the hook and creating proper permissions on the script. This can be seen being set up in Figure 9.

**Figure 10**

**[![Picture10](/wp-content/Picture10.png)](/wp-content/Picture10.png)**

Running the defaults read command we covered earlier will result in the output of the LoginHook location, this is to ensure our hook is in place:

**Figure 11**

**[![Picture11](/wp-content/Picture11.png)](/wp-content/Picture11.png)**

At this point, the LoginHook is properly setup and installed. Anytime the user logs in, the agent will be executed. It should be noted that this method of persistence is widely known and signatured by anti-virus solutions that look for OS X specific persistence.  
## LaunchDaemon’s

As we talked about earlier, Apple does not approve of the many methods used to stay persistent or they are purely outdated according to the Apple development references ([https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/Introduction.html#//apple_ref/doc/uid/10000172i-SW1-SW1](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/Introduction.html#//apple_ref/doc/uid/10000172i-SW1-SW1)). But when it comes to launching custom daemons, using ‘launchd’ is the preferred method according to Apple's development documentation. One of the major concerns with LoginHooks are the blocking nature during execution. LoginHooks are executed during the start of a logon session, which is an inline execution of the script. If at anytime this hangs or does not launch correctly it will deadlock the user from logging in. Using launchdaemons allows you live outside of user context, giving it an amazing benefit of how execution takes place. Here are few benefits of using a daemon:

  * Supports inetd-style daemons    
  * launchd runs as root ( Keeps it easy to run )
  * Daemons launch on demand, communication requests do not fail if the daemon is not launched ( Just in case you mess up )
  * If taxed they are simply delayed until the daemon can launch and process them. ( important as they are not blocking on login etc)

If these reasons are not enough for an operator, I don’t know what is. It’s an extremely clean and safe method of staying persistent on your target; however few downsides do exist:
    
  * Many defensive tools look here for persistence (as it’s a limited attack surface)    
  * KnockKnock  - [https://objective-see.com/products/knockknock.html](https://objective-see.com/products/knockknock.html)    
  * Uninstallation is a few extra steps.

We should first cover how the OS boot process works at high level to understand where your persistence will be living. When starting to work in the persistence realm, it's extremely important to understand the small nuances such as system daemons and user agent daemons. Here is a chart I put together to quickly understand how daemons are executed upon boot and login. 

**Figure 12**

**[![Picture12](/wp-content/Picture12.png)](/wp-content/Picture12.png)**

Currently EmPyre supports installing a “SYSTEM” level daemon running as root that is not dependent on a user being active. This can be very important in some cases and gives you an advantage compared to other methods. Now in Figure 11 we mentioned that the launchd service will locate the plist (property list) file, this is the core of the service and passes the required options that are associated with the service. When launchd starts up it will parse this file and decide when to start, pass arguments or listen on SOCKETS for IPC (Inter Process Communication) so it important that we have an idea of what we are installing. Here is the plist file used:

**Figure 13**

**[![Picture13](/wp-content/Picture13.png)](/wp-content/Picture13.png)**

A few main things should be noted, the Label key string value will be the name of the daemon, in this case the default Is “com.proxy.initialize”. Whereas the array string will be location plus the executable. Finally the key “RunAtLoad” and “KeepAlive” tells launchd to start at system init and stay running rather than a one-time process.

To install the daemon you perform the following. Start by elevating your context to root.

**Figure 14**

**[![Picture14](/wp-content/Picture14.png)](/wp-content/Picture14.png)**

After you have an elevated agent we can go ahead and setup the persistence, using the “persistence/osx/launchdaemonexecutable”.

**Figure 15**

**[![Picture15](/wp-content/Picture15.png)](/wp-content/Picture15.png)**

That’s it, EmPyre does all the nitty-gritty work of creating the executable, writing it to disk, building the plist and registering it to launchd! When testing is complete we can use the “RemoveDaemon” module to properly clean up. (*Remember to take notes on the paths during install!)

**Figure 16**

**[![Picture16](/wp-content/Picture16.png)](/wp-content/Picture16.png)**

Hope this can get you started and maybe give you some ideas of where persistence can also be installed!

References :

OS X Persistence-

[https://www.blackhat.com/docs/us-15/materials/us-15-Wardle-Writing-Bad-A-Malware-For-OS-X.pdf](https://www.blackhat.com/docs/us-15/materials/us-15-Wardle-Writing-Bad-A-Malware-For-OS-X.pdf)

OS X boot Process-

[https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/Introduction.html](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/Introduction.html)

LoginHooks- [https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CustomLogin.html](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CustomLogin.html)

Launchd-

[https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html)

[https://www.virusbulletin.com/uploads/pdf/conference/vb2014/VB2014-Wardle.pdf](https://www.virusbulletin.com/uploads/pdf/conference/vb2014/VB2014-Wardle.pdf)

Dylib Hijacks-

[https://www.virusbulletin.com/virusbulletin/2015/03/dylib-hijacking-os-x/](https://www.virusbulletin.com/virusbulletin/2015/03/dylib-hijacking-os-x/)
