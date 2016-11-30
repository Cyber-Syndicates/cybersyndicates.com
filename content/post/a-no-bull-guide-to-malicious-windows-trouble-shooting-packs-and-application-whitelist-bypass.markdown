---
author: killswitch
comments: true
date: 2015-10-16 13:36:50+00:00
layout: post
link: http://cybersyndicates.com/2015/10/a-no-bull-guide-to-malicious-windows-trouble-shooting-packs-and-application-whitelist-bypass/
slug: a-no-bull-guide-to-malicious-windows-trouble-shooting-packs-and-application-whitelist-bypass
title: A No Bull Guide to Malicious Windows Trouble Shooting Packs and Application
  whitelist Bypass
wordpress_id: 506
categories:
- Open Source Red Team Projects
- Red Teaming / Pen Testing
---

### Malicious Windows Trouble Shooting Package 

**_Using Nicholas Berthaume Recent work on WTP:_**

**_[@nberthaume](https://twitter.com/nberthaume)_**

Just some background on this before we deep dive into setup and construction of the package. I was let onto this a few months back from a coworker. I know this was a pretty cool concept and could see the real world benefit of using something like this on a Red Team in a high security environment. Especially on the phishing side of the house, which I have been doing way more of lately!

But at the time I only knew that this could be done for malicious use, it does turn out that their was a talk explaining just about everything we are going to cover except a few things. The key here is all the information I gathered and the step-by-step execution was done by research on my own and just trial and error to see how things worked. This is an import aspect of anything in the Red Team space, it requires the knowledge of what you are throwing into a customers environment as well as understanding what type effects its has on the endpoint to truly own your tradecraft. Upon entry into this space everyone is challenged at some point, questions like what command are you running; why?; do you know what that does?; what process will be spawned vs injecting.

Thats why I took the approach to doing all the research before reaching out to the team for info on where the idea was generated. It seemed to pay off :)


### Ligt Functionality  (Windows "Feature")

So what are WTP (Windows Trouble Shooting Packs)? The question is simple to answer: its a simple package software developers can check issues, that could be causing issues with their software. This WTP is mainly used by Microsoft and most of us have used them and maybe just didn't think about it (Troubleshooting Network Issues). I'm sure we all have seen this before:[![Screen Shot 2015-10-09 at 2.36.47 PM](/wp-content/Screen-Shot-2015-10-09-at-2.36.47-PM.png)](/wp-content/Screen-Shot-2015-10-09-at-2.36.47-PM.png)

Microsofts _Signed_ Binary Involved (Note this):
    
    msdt </id <name> | /path <name> | /cab < name>> <</parameter> [options] … <parameter> [options]>>



Some really cool features of msdt:
    
  * Is a standard windows signed binary and standard with all deployments
  * Allows you to specify paths of the package and even use UNC paths!!!
  * Allows for you to use the flag "/af" to produce a Answers file built in XML to automate the WTP and can also run from a UNC path "Potential Lateral Movement"
  * Answer files can be easily built in Power-shell

### What does the Package consist of?

[![IC306717](/wp-content/IC306717.jpg)](/wp-content/IC306717.jpg)

So I stole this image from MSDN and its a great image to understand the basics of a WTP. A few things we need to understand about the package it self. The WTP structure consist of:
    
  * Id Metadata
  * Detection scripts
  * Resolution scripts
  * Verification scripts
  * Localized scripts

This format is what builds the "Windows Troubleshooting Pack" itself , this is what is given to the Runtime Engine and runs through these and display to the user the results of the package. The bottom line is we really don't need all of these scripts unless you will be weaponizing these for a specific phishing campaign that is unique the environment you will be targeting. The cool part is you can basically use this package to deploy just about any code you can dream of, as long as you can some how deploy it with powershell of course.



### How to setup to build your first WTP

Lets now cover what we will need to build your first WTP. This can be tricky so follow this to the exact step so you won't encounter any crazy errors as this wasn't easy to figure out. I did use VMware and a clean windows 7 pro VM, so I can't speak for any other platform. But the trick here is that you can build this packages manually which will take FOREVER or use windows SDK package manger to build them.

**Step 1:**

Build your VM that is fully patched and latest SP

**Step 2:**

We need to install windows specific visual studio and C++ distribution

YOU MUST USE 2010 Visual Studio

Download: [Link](http://microsoft-visual-cpp-express.soft32.com/free-download/)

**Step 3:**

Use the following selections

[![Screen Shot 2015-10-14 at 9.56.18 PM](/wp-content/Screen-Shot-2015-10-14-at-9.56.18-PM.png)](/wp-content/Screen-Shot-2015-10-14-at-9.56.18-PM.png)

(Important packages you will be installing so we can install the SDK)

[![Screen Shot 2015-10-14 at 9.56.57 PM](/wp-content/Screen-Shot-2015-10-14-at-9.56.57-PM.png)](/wp-content/Screen-Shot-2015-10-14-at-9.56.57-PM.png)

**Step 4:**
So no matter how hard I tired I couldn't get the latest SDK to work so I jumped back to 7.0

Download: [link](http://www.microsoft.com/en-us/download/details.aspx?id=3138)

**Step 5:**

Check the following boxes so you ensure you install the complete package even though it installs .NET 3.5!


[![Screen Shot 2015-10-14 at 10.06.00 PM](/wp-content/Screen-Shot-2015-10-14-at-10.06.00-PM.png)](/wp-content/Screen-Shot-2015-10-14-at-10.06.00-PM.png)

**Building your first WTP**

At this point you should be set up to use the TSPDesigner package that comes with windows SDK.  While this isn't currently easily automated, Nicholas Berthaume talked about releasing a toolkit for linux to build these packages. We will cover some of the tradecraft concerns and case use's in this post in a bit.

Lets get started by starting up the designer and build out with the wizzard for a quick start.

<blockquote>Goto All Programs -> Microsoft windows SDK v7.0 -> Tools -> Windows Troubleshooting Pack Designer</blockquote>

Click on create pack  -> name it "Really Bad Here" and click on create:

[![Screen Shot 2015-10-15 at 6.34.40 PM](/wp-content/Screen-Shot-2015-10-15-at-6.34.40-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-6.34.40-PM.png)

Go right ahead and fill out the package fields (*set URL it won't have to match your code signing cert):

[![Screen Shot 2015-10-15 at 6.36.59 PM](/wp-content/Screen-Shot-2015-10-15-at-6.36.59-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-6.36.59-PM.png)

Before we move forward lets take a quick look at an actually trouble shooting pack, so we have it fresh in our minds of how execution takes place. This structure can give us a great amount of information about where you may want inject your payload of choice. Head over to your kali and download this Microsoft signed Troubleshooting package, we will be unarchiving it in Kali 2.0:

Download: [Link](http://download.microsoft.com/download/F/2/2/F22D5FDB-59CD-4275-8C95-1BE17BF70B21/wushowhide.diagcab) or http://download.microsoft.com/download/F/2/2/F22D5FDB-59CD-4275-8C95-1BE17BF70B21/wushowhide.diagcab if you don't trust me ;)

Lets take a look at this package of what a potential victim would see if they unpacked it and compared, as you move forward with any tool or technique its important that you understand what right looks like and what your victim could possibly see if they had the right tools.

<blockquote>Right click on the .diagcab and select Open With Archive Manager, you should see:</blockquote>

[![Screen Shot 2015-10-15 at 7.09.07 PM](/wp-content/Screen-Shot-2015-10-15-at-7.09.07-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-7.09.07-PM.png)

You can simply open any package you build with a achieve manager, and see all the unique parts of the package that will be built with the WTP designer. The important concept here is to take a look around at the structure and get an idea of how you would like you package to be built. So as you can see we can easily pull out the unique PS scripts, we may want to obfuscated using some simple power-shell techniques (ex. Encoded base64).

Lets move onto finishing the setup, and fill in the following. This data will be displayed when the user checks the advanced results  within the package, so make it interesting:

[![Screen Shot 2015-10-15 at 8.26.15 PM](/wp-content/Screen-Shot-2015-10-15-at-8.26.15-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-8.26.15-PM.png)

Next we will setup what will happen if we discover the root cause. A interesting discovery is the ability to run the package at a elevated level. This could be interesting, so lets select to run this at a elevated context! (This can Bypass UAC)

[![Screen Shot 2015-10-15 at 8.29.49 PM](/wp-content/Screen-Shot-2015-10-15-at-8.29.49-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-8.29.49-PM.png)

Set up the Resolver which won't really do anything in our case so don't set elevated privileges:

[![Screen Shot 2015-10-15 at 8.31.48 PM](/wp-content/Screen-Shot-2015-10-15-at-8.31.48-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-8.31.48-PM.png)

Using a verifier we can use some simple logic to insure the root cause is fixed.. (Endless possibilities to deploy your initial attack vector.).To keep it simple we won't need to add a Verifier but you will see the logic as we move on:

[![Screen Shot 2015-10-15 at 8.39.37 PM](/wp-content/Screen-Shot-2015-10-15-at-8.39.37-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-8.39.37-PM.png)

Now lets start by editing your Root Cause PS script to start up notepad.exe as an example. Select "Edit TroubleShooter Script":

[![Screen Shot 2015-10-15 at 8.40.12 PM](/wp-content/Screen-Shot-2015-10-15-at-8.40.12-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-8.40.12-PM.png)

At this point your built in Power-shell IDE will be launched for simple editing of your logic:

[![Screen Shot 2015-10-15 at 8.53.55 PM](/wp-content/Screen-Shot-2015-10-15-at-8.53.55-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-8.53.55-PM.png)

If you want to test out a payload its very simple to use CobaltStrikes Payload Generator to build a Staged PS script and just place that code in the logic section to gain code execution (Please note that this section does get executed twice for some reason). If you want to use CobaltStrikes DNS Hybrid listener you can build a fully Staged DNS beacon and export it as a PS script and place that code in the logic area as well.

Go ahead and hit the save button, and your package is ready to be tested out. We will be starting out using a self signed package for testing and will sign the package in a bit. Remember that a self signed package won't run on any computer other than the one you built it on. SO don't try sending a self signed one out to a target!

<blockquote>Build -> Build Pack -> Ok -> Yes -> Launch Trouble Shooting Pack</blockquote>

You should imeidently have Notepad pop and you have code execution!

[![Screen Shot 2015-10-15 at 9.04.14 PM](/wp-content/Screen-Shot-2015-10-15-at-9.04.14-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-9.04.14-PM.png)Now lets explore the detailed info that our target may see when they View Detailed information:

[![Screen Shot 2015-10-15 at 9.05.30 PM](/wp-content/Screen-Shot-2015-10-15-at-9.05.30-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-9.05.30-PM.png)



### Setup for Code Signing

Get a code signing cert: [Here](https://www.digicert.com/code-signing/)  ($179 but you can find way cheaper)

In my case I had a DigiCert Code Signing Cert, that was in a .P7B format which cannot be used with the WTP (Must have the secret key re-Attached to the cert to build a .PFX Cert. This can be a tricky step so I will cover how I generated my .PFX cert before we go into signing the package.

Using Kali 2.0 built in OpenSSL package we can convert it to the correct format:

    openssl pkcs7 -print_certs -in MyCodeSigningCert.p7b -out certificate.cer



Using this new built .Cer we will now generate our .PFX using our privatekey.key.

    openssl pkcs12 -export -out certificate.pfx -inkey privateKey.key -in certificate.crt -certfile CACert.crt

<blockquote>Return back to windows package designer -> Project -> Options -> Specify a Certificate</blockquote>

[![Screen Shot 2015-10-15 at 9.21.46 PM](/wp-content/Screen-Shot-2015-10-15-at-9.21.46-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-9.21.46-PM.png)

You should now be able to Build your package and now have a fully signed package for deployment. In the next image the publisher has been obfuscated so I don't share the world with my cert. At this point its  time to move into testing of our new WTP!

[![Screen Shot 2015-10-15 at 9.24.35 PM](/wp-content/Screen-Shot-2015-10-15-at-9.24.35-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-9.24.35-PM.png)



### Application Whitelisting and Script Bypass Testing

Lets now setup a hardened environment on our fresh Windows 7 _Pro_ VM. Using windows Applocker we can easily setup a simulated environment that we can test against.

Using windows Group Policy editor we can make the correct changes needed ( run Gpcedit.msc ):

Navigate to: Computer Configuration \ Windows Settings \ Security Settings \ Application Control Policies \ AppLocker

[![Screen Shot 2015-10-15 at 9.37.47 PM](/wp-content/Screen-Shot-2015-10-15-at-9.37.47-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-9.37.47-PM.png)

Now we need click on "Configure rule enforcement" and enable Execution Rules and Script Rules:

[![Screen Shot 2015-10-15 at 9.39.32 PM](/wp-content/Screen-Shot-2015-10-15-at-9.39.32-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-9.39.32-PM.png)

Next right click Executable Rules -> Create New Rule:

[![Screen Shot 2015-10-15 at 9.42.25 PM](/wp-content/Screen-Shot-2015-10-15-at-9.42.25-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-9.42.25-PM.png)

[![Screen Shot 2015-10-15 at 9.43.02 PM](/wp-content/Screen-Shot-2015-10-15-at-9.43.02-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-9.43.02-PM.png)

[![Screen Shot 2015-10-15 at 9.43.38 PM](/wp-content/Screen-Shot-2015-10-15-at-9.43.38-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-9.43.38-PM.png)

<blockquote>Click Create -> Yes to Default rules</blockquote>

At this point go ahead delete the Admin rule that allows and admin to execute any executable, your rules should match this:

[![Screen Shot 2015-10-15 at 9.47.01 PM](/wp-content/Screen-Shot-2015-10-15-at-9.47.01-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-9.47.01-PM.png)

Go ahead and setup the Script Rules and block out the temp dir:

[![Screen Shot 2015-10-15 at 9.49.18 PM](/wp-content/Screen-Shot-2015-10-15-at-9.49.18-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-9.49.18-PM.png)

Lets start up the required services needed for Applocker to enforce rules properly:
    sc config appIDSvc start= auto
    sc start appIDSvc



[![Screen Shot 2015-10-15 at 9.55.18 PM](/wp-content/Screen-Shot-2015-10-15-at-9.55.18-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-9.55.18-PM.png)

Lastly we need to update our Group Policies. Reboot your box and run this (you may have to reboot first, if its domain auth you must have the DC on in your VM environment):
    
    Gpupdatep /force



So lets test out our new setup in Applocker, first I will attempt to run beacon.exe in my restricted environment:

[![Screen Shot 2015-10-15 at 10.29.07 PM](/wp-content/Screen-Shot-2015-10-15-at-10.29.07-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-10.29.07-PM.png)

Great this is exactly what we needed to see, now PS in this location:

[![Screen Shot 2015-10-15 at 10.36.39 PM](/wp-content/Screen-Shot-2015-10-15-at-10.36.39-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-10.36.39-PM.png)

Now lets test out our new package:

[![Screen Shot 2015-10-15 at 10.38.10 PM](/wp-content/Screen-Shot-2015-10-15-at-10.38.10-PM.png)](/wp-content/Screen-Shot-2015-10-15-at-10.38.10-PM.png)

Boom! Now we have our payload and the ability to bypass AppLocker using some of the most restrictive settings. Now Nicholas Berthaume didn't talk about this Applocker bypass (maybe I just missed this but this is a pretty sweet technique and all props go out to him)!

By why does this work? Well we are actually using MSDT.exe to execute our .diagcab package. This is a Microsoft Signed binary which is inherently trusted by Windows! So we can now use this binary to gain code execution just like other App Bypasses using instal-util etc..

If you don't know much about application whitelist bypasses here is a great list of the current ones that SubTee put out:

https://github.com/subTee/ApplicationWhitelistBypassTechniques/blob/master/TheList.txt

    1. IEExec -This technique may work in certain envirionments. Its relies on the fact that many organizations trust executables signed by Microsoft. We can misuse this trust by launching a specially crafted .NET application. Example Here: http://www.room362.com/blog/2014/01/16/application-whitelist-bypass-using-ieexec-dot-exe/
    2. Rundll32.exe
    3. ClickOnce Applications dfsvc.exe dfshim.dll
    4. XBAP - XML Browser Applications WPF PresentationHost.exe    
    5. MD5 Hash Collision
     http://www.mathstat.dal.ca/~selinger/md5collision/
    6. PowerShell
     Specifically Reflective Execution
     http://clymb3r.wordpress.com/2013/04/06/reflective-dll-injection-with-powershell/
     https://www.defcon.org/images/defcon-21/dc-21-presentations/Bialek/DEFCON-21-Bialek-PowerPwning-Post-Exploiting-by-Overpowering-Powershell.pdf
    7. .HTA Application Invoke PowerShell Scripts
     Launched by mshta.exe, bypasses IE security settings as well.
    8. bat, vbs, ps1
     1. cmd.exe /k < script.txt
     2. cscript.exe //E:vbscript script.txt
     3. Get-Content script.txt | iex



**Tradecraft Concerns:**

  1. Phishing should work great, the .diagcab probably won't be inspected by boarder protections? (let me know ppl)
  2. Code signing cert will need to look legit to the target (so keep that in mind)
  3. Powershell script can easily be pulled from the package, so use something encrypted for stager


I really hope you got something out of this! Let me know on twitter @Killswitch_gui what you think or ideas that you have, I love to learn from my peers so let me know what you think or are using this for. All credit for this goes out to:

https://www.youtube.com/watch?v=-E-JTYjk8_Q



Resources:

https://technet.microsoft.com/en-us/library/ee424379.aspx

Code Signing Help: https://support.globalsign.com/customer/portal/articles/1353601-converting-certificates---openssl

https://msdn.microsoft.com/en-us/library/dd776530.aspxb

Simple AppLocker Setup: http://www.howtogeek.com/howto/6317/block-users-from-using-certain-applications-with-applocker/
