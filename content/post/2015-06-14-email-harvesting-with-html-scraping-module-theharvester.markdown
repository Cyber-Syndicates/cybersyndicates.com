---
author: killswitch
comments: true
date: 2015-06-14 01:26:31+00:00
layout: post
link: http://cybersyndicates.com/2015/06/email-harvesting-with-html-scraping-module-theharvester/
slug: email-harvesting-with-html-scraping-module-theharvester
title: 'Email Harvesting with HTML Scraping Module: theHarvester'
wordpress_id: 304
categories:
- Github
- Red Teaming / Pen Testing
---

* * *



After a recent Red Team training , I was faced with a issue where I had to conduct a Email phishing campaign to gain initial access using Email Harvesting. This isn't anything necessarily new or a subject that has been touched on a million times. But my goal wasn't to recreate the wheel or create a entirely new tool. I knew from past engagements that there are some great tools like [theHarvester](http://www.edge-security.com/theharvester.php) project out there:

[![Screen Shot 2015-06-11 at 9.32.34 PM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-11-at-9.32.34-PM-1024x377.png)](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-11-at-9.32.34-PM.png)

But I came across a ridiculous issue; I had no access to external net. Only internal web servers for the cyber range we where conducting the training in. This scenario isn't as unrealistic as you may think, you could be on an internal network assessment and may have to gain secondary access through different means if phishing is in scope. In this case with the limited network ability the target surface was limited and quite small, but yet we didn't have access to the normal means of recon. Using the extended set of options of the harvester can yield some great results, but all of its current modules are online dependent. With search options using Google and Bing you can gather tons of data, but these functions wouldn't help me a bit.

[![Screen Shot 2015-06-11 at 10.51.51 PM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-11-at-10.51.51-PM-1024x114.png)](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-11-at-10.51.51-PM.png)



##### _(Current Search Options for TheHarvester)_



So I knew I wasn't going to search a page head-to-toe when time counts. The obvious email address's on the home page is a start, but I needed something fast and that would yield a 100 percent insurance that I had all the emails required to conduct the lab. Of course it was time for some simple command line kung-fu!



### Step-By-Step Execution





* * *



_**Here are the preliminary steps I took to begin searching a domain for Email addresses:**_



1. First I started off knowing that I needed to clone the website for the ability to search and browse the site for other info I may need down the road. I knew theHarvester had no offline ability so I moved to Wget. This command bellow will allow you to reclusively mirror a site:




    
    wget --directory-prefix=/root/Desktop/ --domains target.com --recursive --no-clobber --page-requisites --html-extension --convert-links --restrict-file-names=windows www.target.com





--directory-prefix=/root/Desktop/   ==  Allows you to specify the output location of the cloned site (wget outputs a dir)





--recursive == Follows links and directory structure, by default has a depth of 5





--level=1500 == The depth (directory structure) you want the recursive search to go. **You may want to add this**





--wait=5 == Sets the time to wait between request, helps to not get blocked by pesky admins **You may want to add this**





--limit-rate=10K == sets max bandwidth for wget to consume, helps to not get blocked by pesky admins **May want to add this**





--read-timeout=15 == Sets max time spent attempting to download file before it moves on **You may want to add this**





--no-clobber == Incase you have to restart it won't overwrite what has been saved





--page-requisites == Download all the files that are necessary to properly display a given HTML page





--html-extension == download CSS files as well





--convert-links  == Convert the links in the document to make them suitable for local viewing





--domains target.com == Sets you scope so you don't follow links outside the target domain



The next step is to parse this data and grep for any email addresses within the HTML source code. This is relativity easy to accomplish when recessively searching for the data.


    
    grep -r "@" | grep -i -o '[A-Z0-9._%+-]\+@[A-Z0-9.-]\+\.[A-Z]\{2,4\}'





-r "@" == Using the -r will recursively search the directory
-i == Perform case insensitive matching sources
-o == Show only the part of a matching line that matches PATTERN.



![Screen Shot 2015-06-12 at 2.05.03 PM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-12-at-2.05.03-PM.png)

At this point I was able to simply output line by line the email address's I gathered and still had a mirrored site for further recon if needed.



### Automation





* * *



I knew at this point that I wanted a easy way to do this and at the same time I knew that making some simple script wouldn't help anyone. So I set off to add this function into the harvester so other could use this simple way of gathering email addresses.

[![Screen Shot 2015-06-13 at 9.10.24 PM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-13-at-9.10.24-PM-1024x725.png)](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-13-at-9.10.24-PM.png)

After looking at the code for roughly a few hours I knew that adapting this particular function wasn't going to be hard, but I knew I had to match the coding style of Christian Martorella otherwise it would never get added. So I set off and built a simple way of automating this. I did run into a few problems when it came to recursive mirroring in pythons current modules. With documented issues with proxies and VPN support in Urlib, this was really a turn off for me as I may be using a VPN to conduct work. Well nothing seemed to do it near as good as Wget implemented it. So using pythons _sub.process_ module was going to be the most effective way of implementation and was supported on almost any nix system.

[![Screen Shot 2015-06-13 at 8.43.00 PM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-13-at-8.43.00-PM-1024x555.png)](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-13-at-8.43.00-PM.png)

Some other major issues were using default values in the current command line parser. Currently theHarvester doesn't use _argparse_ so I had in a few nasty if statements (If anyone knows a better way please let me know bellow!).

[![Screen Shot 2015-06-13 at 8.47.00 PM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-13-at-8.47.00-PM.png)](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-13-at-8.47.00-PM.png)

lastly a simple demo:

![Screen Shot 2015-06-13 at 8.50.04 PM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-13-at-8.50.04-PM-1024x43.png)




    
    python theHarvester.py -d www.puttargethere.com -b html-source
    or more advance options:
    python theHarvester.py -d www.puttargethere.com -b html-source -j 10 -w 3 -r 200k -i 3 -o /foo/bar/



As you can see the output format hasn't changed and would simple be another option to add:
![Screen Shot 2015-06-13 at 8.50.30 PM](http://cybersyndicates.com/wp-content/uploads/2015/06/Screen-Shot-2015-06-13-at-8.50.30-PM.png)



### NOTE:



**As of today I will be forking, and creating a branch for a pull request. Hopefully he would add in the module I built! If not it won't hurt my feelings.. You can always come download [my forked version on Git](https://github.com/killswitch-GUI/theHarvester/tree/Html_source).**

Sources:

[*] http://www.linuxjournal.com/content/downloading-entire-web-site-wget

[*] https://github.com/laramies/theHarvester


