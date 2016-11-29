---
author: killswitch
comments: false
date: 2016-02-14 14:23:45+00:00
layout: post
link: http://cybersyndicates.com/2016/02/email-harvesting-and-name-creation/
slug: email-harvesting-and-name-creation
title: Email Verification and Email Name Creation
wordpress_id: 745
categories:
- Github
- Open Source Red Team Projects
- Red Teaming / Pen Testing
tags:
- the
---

Over the past few months we have been rapidly expanding the capability of SimplyEmail. While it is a very simple tool, it has been extremely successful on a few live engagements I have been on. I feel like it is ready for recommendation to say the least. I did however notice a few key features that some of the guys on the team mentioned that would be nice to have integrated. Thanks to [@_wald0](https://twitter.com/_wald0) and his suggestions I have implemented a email verification option. Also I learned a trick or so a few months back from [@harmj0y](https://twitter.com/harmj0y) on sick site called "Connect6", which seems to populate a large name DB of employees for each company. Also a fellow tester and friend ([Joshua Crumbaugh](https://twitter.com/nagasecurity)) let me on to a sick tool for grabbing Linkedin names from bing called [PhishBait](https://github.com/pan0pt1c0n/PhishBait). With all this I set out to build some new capabilities for SimplyEmail and learn some new tricks :)



#### _SMTP Email Verification_



This process is actually relatively easy to accomplish. Its a simple heuristic of SMTP return codes when attempting to send a email on the target SMTP server. The process takes place by first Identifying the proper MX record to point to. In many cases larger corporations will have more than one SMTP server with multiple MX records. These are for redundancy of course and are ordered by priority, here is a small snip of the code I built fast:

[snippet id="29"]

After obtaining all the MX records we can easily sort, pick the lowest value and feed it to the next function needed. So here is the meat of the verification checks. Before we go ahead and send the SMTP server all of the gathered emails we need to check if this SMTP server supports this. We do this via providing the STMP server a known "invalid" address, and we test for the a return code other than 250 (250 is a valid email code). If we get anything except a 250, we know that the SMTP server isn't just returning a 250 for each address supplied. we can test this pretty simply:

[snippet id="30"]

So lets say our SMTP server supports this type of check, we can simply build a function to perform the check for our gathered emails. Here is a small small output of it in action:

[![Screen Shot 2016-02-14 at 11.10.11 AM](http://cybersyndicates.com/wp-content/uploads/2016/02/Screen-Shot-2016-02-14-at-11.10.11-AM-1024x590.png)](http://cybersyndicates.com/wp-content/uploads/2016/02/Screen-Shot-2016-02-14-at-11.10.11-AM.png)

Obviously I needed to wrap this in a simple helper class for SimplyEmail, the full code (Class) can be found here:

[https://github.com/killswitch-GUI/SimplyEmail/blob/master/Helpers/VerifyEmails.py](https://github.com/killswitch-GUI/SimplyEmail/blob/master/Helpers/VerifyEmails.py)



#### _Connect6 Name Creation_



Im always extreamly nervous to add in functionality to SimpleEmail.. hence the name! In some cases name creation can be a pivotal and vital addition to your phishing campaigns. Some times SimplyEmail will only find the standard email addresses or just a few emails. In this case email creation may be your saving grace. On my assessments I with out doubt found Connect6.com is a reliable source to gather names associated with companies.

Too start I throughly attempted to find  way to get the Connect6 URL using their search engine, with no anvil. This may be do to how they want you to pay for their service / API. So I did build a simple Google Dork function to attempt to resolve the correct URL:

[snippet id="32"]

Full code can be found here: [https://github.com/killswitch-GUI/SimplyEmail/blob/master/Helpers/Connect6.py](https://github.com/killswitch-GUI/SimplyEmail/blob/master/Helpers/Connect6.py)

Here is a quick sample of the output that you will given if it cant detect the correct URL:

[![Screen Shot 2016-02-14 at 11.20.14 AM](http://cybersyndicates.com/wp-content/uploads/2016/02/Screen-Shot-2016-02-14-at-11.20.14-AM-1024x605.png)](http://cybersyndicates.com/wp-content/uploads/2016/02/Screen-Shot-2016-02-14-at-11.20.14-AM.png)



#### _LinkedIn__ Name Creation_



Linkedin is and has been know as a great source for social engineering, and recently I first hand got to see how effective it is to build a email campaign of it. Using the PhishBait tool mention earlier I was able to add in additional functionality and build in LinkedIn name scraping into SimplyEmail. Here is the really simple code I adapted:

[snippet id="33"]

Finally the full code can found here: [https://github.com/killswitch-GUI/SimplyEmail/blob/master/Helpers/LinkedinNames.py](https://github.com/killswitch-GUI/SimplyEmail/blob/master/Helpers/LinkedinNames.py)
