---
author: killswitch
date: 2016-06-21 20:14:14+00:00
title: EmPyre Collection Operations
categories:
- Github
tags:
- EmPyre
---
{{< img-post path="/image" file="empyre_logo.png" alt="Empyre Logo" type="center" percent="50">}}
This post is number 4 of the EmPyre series. Thanks to the entire ATD family and dev team of EmPyre:

@rvrsh3ll — @harmj0y — @xorrior — @CptJesus

EmPyre can be found here: https://github.com/adaptivethreat/EmPyre

#### Master list of the seriers: [Here](http://www.harmj0y.net/blog/empyre/building-an-empyre-with-python/)
- 5/12/16 – _**[Building an EmPyre with Python](http://www.harmj0y.net/blog/?p=2637&preview=true)**_
- 5/18/16 – _**[Operating with EmPyre](http://Operating with EmPyre)**_
- 5/24/16 – _**[The Return Of the EmPyre](https://www.xorrior.com/the-return-of-the-empyre/)**_
- 5/31/16 – _**[OS X Office Macros with EmPyre](http://www.harmj0y.net/blog/empyre/os-x-office-macros-with-empyre/)**_

### **EmPyre: Introduction to collection operations**
Information gathering may be one of the most vital actions to execute on a target. When it comes to EmPyre, we have implemented a suite of tools and techniques to accomplish a large subset of our needs. The interesting part about collection is it’s often used at nearly every phase of the attack cycle. This week we will be covering the various pre & post collection modules we have built!

#### **Pre-Escalation Collection Methods**
One of our favorite, and easiest, methods is collecting a large subset of user information in OS X. Due the heavy use of SQL to store user information, it is extremely easy to query this data. For instance: with the “browser_dump” module we can start an operation to obtain browser history, which helps gain situational awareness of the target system. While this can be used in many scenarios, it is especially useful because the module does not require an elevated context; as such, it may give you pointers for the target you're currently on. Once you are up and running in EmPyre you can dump our user’s browser history for Safari and Chrome.

After quickly generating some history on my test VM we have data!

{{< img-post path="/image" file="browser_dump_empyre.png" alt="Empyre Browser Dump" type="center" percent="100">}}

Next up on SQL data: the iMessage data store! iMessage data store is used by many targets, and is often synced with iCloud (SMS and IMessage data). The interesting thing about the iMessage app is its ability to integrate multiple chat platforms - not just standard iMessaging from Apple. Examples of platforms with this funcionality include: Yahoo, Jabber, Aol, AIM, Google, etc.  Information collection through data mining is sometimes needed for additional escalation, and iMessage makes a great location to search for data like passwords. Using the “imessage_dump” module we can search for specific terms in the users iMessage data store. This will enumerate the following:
    
  * Account – The corresponding account user
  * Service – The provider of the message
  * Country – Where the message originated
  * Number – If a text message, will be the senders telephone number
  * ROWID – This is the row the message was stored, can be helpful if you pull the DB back
  * Date – Date of the message
  * Message – The text enumerated from the selected message
  * Type – Secondary field for the type of account used within the app

{{< img-post path="/image" file="empyre_message_dump.png" alt="Empyre Message Dump" type="center" percent="100">}}
The next method of collection requires using osascript (Apples scripting language) we are able to force applications to prompt for credentials potentially gathering a password! Here is the “Prompt” module in action:

{{< img-post path="/image" file="empyre_prompt_attack.png" alt="Empyre Message Dump" type="center" percent="100">}}
Once setup we can easily force the App of choice to prompt a dialog box requesting a password. This can be a great method to gather credentials and potentially escalate. This and the next module have been adapted from work from [@FuzzyNop](https://twitter.com/fuzzynop?lang=en), so big shout out the work he puts in.

{{< img-post path="/image" file="empyre_prompt_attack2.png" alt="Empyre Message Dump 2" type="center" percent="100">}}

One of my fellow friends, @enigma0x3 ([https://enigma0x3.net/2015/01/21/phishing-for-credentials-if-you-want-it-just-ask/)](https://enigma0x3.net/2015/01/21/phishing-for-credentials-if-you-want-it-just-ask/)) has an interesting technique to get credentials on engagments using the standard Windows login prompt. The great part is OS X has something similar we are able to implement into our collection strategy. Using the osascript method we can easily weaponize the “ScreenSaverEngine” application to request credentials.  We can force the standard screensaver, request creds, and test them against the current users Apple Key Chain. Using the **security** command we can lock the users key chain and use the creds supplied to attempt to unlock using the follwing command: **security unlock-keychain –p Test. **This can be easily deployed in a loop to only unlock the screensaver when the user successfully enters the correct credentials. While it may not be extremely OPSEC friendly, it may go unnoticed and sucessfuly result in creds.

{{< img-post path="/image" file="empyre_screensaver_prompt.png" alt="Empyre Screen Saver Prompt" type="center" percent="100">}}

A user will be prompted repeatedly until successful creds or the exit count is reached.

{{< img-post path="/image" file="empyre_screensaver_prompt2.png" alt="Empyre Screen Saver Prompt2" type="center" percent="100">}}

One of the key collection modules we use during an engagment is clipboard collection. The main reason clipboard monitoring is key in our tradecraft is due to the ease of collection when keylogging fails to capture creds. This allows you to gather credentials and target specific times when password vaults are used in conjunction with the screenshot. Allowing for targeted collection of potential passwords and sensitive data. Using native API calls NSPasteboard, NSStringPboardType we can prevent using built in commands and potential signatures that could get us caught.

{{< img-post path="/image" file="empyre_clipboard_monitor.png" alt="Empyre Clipboard monitor" type="center" percent="100">}}

### **Post-Exploit Collection Methods**
Often during operations, we move from a recon & escalation stage to a post exploitation information gathering process. This is truly the heart of end state operations and proper tools can help speed up the process as well as help a tester reach the data / show impact. When EmPyre was being developed we often attempted to explain and relate it to processes and procedures used during a windows environment.

The first module I want to start with is the screenshot module. Due to testers reliance on this module it may be one of the most important post exploitation modules in our arsenal. While EmPyre currently supports two different techniques of screenshot, the native_screenshot module may be concern if certain AV products are in place. This is due to using the _screenshot_ command, and potential logging.  While the standard module uses system APIs to capture the screen, parse the image, and drop to disk in a temporary location. During my research I did uncover a few methods to properly parse all the images completely in memory, but currently due to lack of the PIL library standalone 2.7 does not have the ability to parse completely in memory.  So relying on the Quartz API is a current constraint in this method.

{{< img-post path="/image" file="empyre_native_screenshoot.png" alt="Empyre Native Screenshot" type="center" percent="100">}}

One feature within the EmPyre code base is different tasking types. This allows for different code paths of execution. For example, the iMessage module used dynamic code execution while the screenshot uses dynamic code execution with saved output. This is done by dropping to disk, opening the file, base64ing the raw file and the EmPyre server, and then saving it to your downloads folder. This makes for easy file management for modules and keeps things organized.

Once basic situational awareness has been completed on a target, we often transition to reaching end state client goals. This is often referred to as the “Crown Jewels” or high value targets. Generally, it takes a large amount of post exploitation intelligence and the ability to collect on all your endpoints. Keylogging is an amazing technique that has been extremely successful for gathering creds, enumerating users work roles, collecting info, or even understanding the tools they are using to interact with the asset. Bellow you will see this being employed using an adapted ruby keylogger. @ joev  ([https://github.com/gojhonny/metasploit-framework/blob/master/modules/post/osx/capture/keylog_recorder.r)](https://github.com/gojhonny/metasploit-framework/blob/master/modules/post/osx/capture/keylog_recorder.r))

{{< img-post path="/image" file="empyre_keylogger.png" alt="Empyre Keylogger" type="center" percent="100">}}

Once the key logger is up and running you can easily check it using the built in shell commands or once complete kill the PID, use the built in download, and delete the file from disk.

{{< img-post path="/image" file="empyre_keylogger2.png" alt="Empyre Keylogger" type="center" percent="100">}}

Once the file has been retrieved from the target you will have something similar (to the below) representing the application, key stroke, and keyboard commands.

{{< img-post path="/image" file="empyre_keylogger3.png" alt="Empyre Keylogger 3" type="center" percent="100">}}

While operating on OS X you for sure miss the ease of mimikatz for gathering credentials, but while password collection is possible in some cases cracking potential evaluated accounts, admins or service accounts may be extremely useful for lateral movement. Within EmPyre we have built in the ability for hashdump and nicely output these hashes in hashcat ready format!

{{< img-post path="/image" file="empyre_hashdump.png" alt="Empyre Hashdump" type="center" percent="100">}}

It is often thought that due to the out-of-the box setup that lateral movement is just not an option. This is not always the case in the corporate environments. Admins are generally going to administrate, right? They need some method of software, patch, or user management for their end points. This can result in SSH being deployed to most corporate OS X assets, or maybe a custom solution. While this blog does not cover the Kerberos implications, in corporate environments, due to the heavy use of Active Directory it can be an entirely new method of lateral movement. Stay tuned for more on that subject!

The last module we will be covering is the keychaindump. While this module may result in a large subset of user data and crucial passwords it does have limitations. Currently, this will not work against the latest OS X platform. The latest usable version was Yosemite, due to a vulnerability that allowed for researches to pull the master key candidate from memory once in an elevated state. Due to the new System Integrity Protections (SIP) in El Captain the ability to retrieve this master key has been properly stored in memory protected by SIP.  Not all hope is lost though! I have been extremely successful in using other tools to parse the keychain with the user’s credentials. This can be done with either a memory image or the keychain file itself.

{{< img-post path="/image" file="empyre_keychain_dump.png" alt="Empyre Keychain Dump" type="center" percent="100">}}


Once the keychain is local we can use a python tool called chainbreaker ([https://github.com/n0fate/chainbreaker)](https://github.com/n0fate/chainbreaker)). This allows for the use of the password or the master key and will parse and decrypt the entire keychain offline. While this requires the password, a root context is not needed to download the keychain, allowing for offline attempts against the keychain which can be quite useful.

{{< img-post path="/image" file="empyre_keychain_dump2.png" alt="Empyre Keychain Dump2" type="center" percent="100">}}


Hopefully this has helped clear up different use cases when it comes to EmPyre collection operations and the corresponding modules. As always we plan to release blogs near weekly covering the different use cases of EmPyre, next the team will be covering the injection of Kerberos tickets
