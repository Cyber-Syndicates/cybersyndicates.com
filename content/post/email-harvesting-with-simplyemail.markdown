---
author: killswitch
comments: false
date: 2015-11-21 14:58:47+00:00
layout: post
link: http://cybersyndicates.com/2015/11/email-harvesting-with-simplyemail/
slug: email-harvesting-with-simplyemail
title: Email Harvesting with SimplyEmail
wordpress_id: 633
categories:
- Github
- Open Source Red Team Projects
- Red Teaming / Pen Testing
---

## Email Recon Made Easy


More often than not email enumeration is hit or miss, depending on the sources used throttle limits are in place or they just plain don’t index everything. As you may know from my past blog write-up I wrote a HTML scrapper to integrate into theHarvester. After that experience I knew there was for some improvement on 3 major key areas:

  * Speed - It needed to be able to handle way more locations to search    
  * Framework – People needed to be able to easily contribute to the project
  * Reporting – I wanted it to be elegant and easy to discover the sources

With these requirements in hand I started my build on SimplyEmail!

TL;DR:
<div class="github-card" data-github="killswitch-gui/SimplyEmail" data-width="400" data-height="153" data-theme="default"></div>
<script src="//cdn.jsdelivr.net/github-cards/latest/widget.js"></script>

### Benefits of SimplyEmail

In the underlying code I built the program to dynamically load the all modules that where placed in the modules folder. The question is why? Well after using other tools and methods, I found that searching just about anywhere you can get your hands on can be beneficial to the results you obtain (Even if duplicates). In certain cases, Google Captcha block would engage and Yahoo Search would come to save the day! The key here was the ability to run multiple modules at the same time, this proved to be quite an improvement for web intense modules. Here is a small snippet of the code that makes this happen:

  1. All modules are dynamicaly loaded into a list
  2. Modules are placed into a Multiprocessing queue
  3. Execution takes place in a standardized format so that all processing takes place on child process rather than parent
  4. Results are Parsed with a custom parser that handles about all the raw html and text based parsing you could need.
  5. These results are placed into a Dictionary with a Key of the source and email address
  6. Html results are generated on the fly, and raw results are appended to a running list with time stamps.

TL;DR:
<div class="github-card" data-github="killswitch-gui/SimplyEmail" data-width="400" data-height="153" data-theme="default"></div>
<script src="//cdn.jsdelivr.net/github-cards/latest/widget.js"></script>

**Multiprocessing Controller:**

[![Screen Shot 2015-11-17 at 9.36.26 AM](/wp-content/Screen-Shot-2015-11-17-at-9.36.26-AM.png)](/wp-content/Screen-Shot-2015-11-17-at-9.36.26-AM.png)



**How the modules are instantiated:**



[![Screen Shot 2015-11-17 at 9.38.55 AM](/wp-content/Screen-Shot-2015-11-17-at-9.38.55-AM.png)](/wp-content/Screen-Shot-2015-11-17-at-9.38.55-AM.png)



### Simple CLI Interface:


To get started always run the Setup script first:


    
    root@kali:~/Desktop/SimplyEmail# sh Setup.sh
    or
    root@kali:~/Desktop/SimplyEmail# ./Setup.sh



Next take a look at the help menu:


    
    root@kali:~/Desktop/SimplyEmail# ./SimplyEmail -h



[![Screen Shot 2015-11-17 at 9.50.39 AM](/wp-content/Screen-Shot-2015-11-17-at-9.50.39-AM.png)](/wp-content/Screen-Shot-2015-11-17-at-9.50.39-AM.png)

Running the tool can be accomplished by either running all modules or one specific module for testing:


    
    root@kali:~/Desktop/SimplyEmail# ./SimplyEmail -all -e enron.com
    
    root@kali:~/Desktop/SimplyEmail# ./SimplyEmail -t Google -e enron.com



[![Screen Shot 2015-11-17 at 10.03.13 AM](/wp-content/Screen-Shot-2015-11-17-at-10.03.13-AM.png)](/wp-content/Screen-Shot-2015-11-17-at-10.03.13-AM.png)



## Understanding Reporting Options:

One of the most frustrating aspects of Pen-testing is the tools' ability to report the findings and make those easily readable. This may be for the data provided to a customer or just the ability to report on source of the data.

So I'm making it my goal for my tools to take that work off your back and make it as simple as possible! Let's cover the two different reports generated.

### Text Output:

With this option results are generated and appended to a running text file called Email_List.txt. this makes it easy to find past searches or export to tool of choice. Example:

[![Screen Shot 2015-11-21 at 10.01.52 AM](/wp-content/Screen-Shot-2015-11-21-at-10.01.52-AM.png)](/wp-content/Screen-Shot-2015-11-21-at-10.01.52-AM.png)

### HTML Output:

As I mentioned before a powerful function that I wanted to integrate was the ability to produce a visually appealing and rich report for the user and potentially something that could be part of data provided to a client. Please let me know with suggestions!

#### Email Source:
[![Screen Shot 2015-11-11 at 5.27.15 PM](/wp-content/Screen-Shot-2015-11-11-at-5.27.15-PM.png)](/wp-content/Screen-Shot-2015-11-11-at-5.27.15-PM.png)





#### Email Section:
    
  * Html report now shows Alerts for Canary Search Results![![Screen Shot 2015-11-11 at 5.27.31 PM](/wp-content/Screen-Shot-2015-11-11-at-5.27.31-PM-1024x528.png)](/wp-content/Screen-Shot-2015-11-11-at-5.27.31-PM.png)






