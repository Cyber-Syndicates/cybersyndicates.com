---
author: killswitch
comments: false
date: 2016-05-26 21:56:58+00:00
layout: post
link: http://cybersyndicates.com/2016/05/email-reconnaissance-phishing-template-generation-made-simple/
slug: email-reconnaissance-phishing-template-generation-made-simple
title: Email Reconnaissance and Phishing Template Generation Made Simple
wordpress_id: 881
categories:
- Github
- Red Teaming / Pen Testing
---

**Author: Alexander Rymdeko-Harvey, @Killswitch_GUI **

First a big thanks to the entire ATD (Adaptive Threat Divsion) team which contributed ideas, support and templates! [https://verisgroup.com/blog/category/adaptive-threat-division/](https://www.verisgroup.com/blog/category/adaptive-threat-division/) 

https://twitter.com/CptJesus

https://twitter.com/sixdub

https://twitter.com/xorrior

https://twitter.com/424f424f

https://twitter.com/enigma0x3

https://twitter.com/mattifestation

https://twitter.com/_wald0

https://twitter.com/bluscreenofjeff



#### Email Reconnaissance and Phishing Template Generation Made Simple



As a red-teamer or pen-tester, the need for tools that speed up the process is absolutely critical. While tools are not everything, they sure do help when it comes to performing an engagement within a short timeframe that a threat actor would have months to execute. With limited time and the need for effective methodologies, phishing can be a tester’s worst nightmare but also the best path to success. I found that proper reconnaissance and preparation are extremely important when it comes to phishing. I set out to speed up the process while still employing effective methodologies for upcoming and future engagements.

**Email Recon Methodology and name creation:**

Over the past few months, I put quite a bit of time into research and current methods that are being used to perform email harvesting. While every tester has a few different methods to get the most out of their recon, most rely upon search indexing and searching for documents. Using this methodology is generally slow and takes up valuable time. One of the major concerns was the ability to cover as much content as possible, with so many documents and locations to search I often missed email data I should have caught.  I knew right away there was room for improvement.

Using tools like theHarvester has been a great resource for myself and other testers. This tool uses Google and Bing to automatically scrape emails by parsing raw HTML. This tool is used often on engagements and was a huge inspiration to building out a few more features. Here are a few of the sources I knew would be gold mines:




    
  * PasteBin

    
  * Past data dumps

    
  * Raw HTML of target site

    
  * EmailHunter – http://emailhunter.co



I discovered that using HTML parsing over more traditional API searches allows a user to search content that may not yet supply an API. This was the case for a ton of the resources that I wanted to target moving forward with my research. While API based searches have their place, I focused on free and easily accessible data for my gathering techniques. Using advanced Google search operators you can retrieve a list of all .xlsx files currently indexed on Google. Parsing the results was all done manually and finding them took some time.  “Google dorks” that I often employed during my recon phase:


    
  * Site:verisgroup.com filetype:doc -- Site: verisgroup.com filetype:docx

    
  * Site: verisgroup.com filetype:xls -- Site: verisgroup.com filetype:doc

    
  * Site: verisgroup.com filetype:pdf

    
  * site:pastebin.com "@target.com"

    
  * “@target.com”



When searching the web for emails, you have to get creative. An issue I found out early on was the reliability of indexed data, which has to do with indexing and other factors. But I found this out time and time again when it came to searching for new and interesting sources, such as Yahoo, Reddit, Ask, Whois Data, or even PGP keys. These queries may only find 1 or 2 more emails but in the long run, but they add up and you can show your client the visibility they have on the web.

While email recon can get you a good start or initial vector of attack, covering a larger set of data or targets requires secondary techniques, in particular, inferring email addresses from names. This has been used and talked about pretty thoroughly. One tool that implements this technique is PhishBait ([https://github.com/pan0pt1c0n/PhishBait)](https://github.com/pan0pt1c0n/PhishBait)), which scrapes LinkedIn names from Bing to build out a potential email using predictable formats. Will Schroeder (@harmj0y) produced a to scrape names from Connect6 - a sourcing database of employees and their companies. Between these two sites, recent phishing campaigns have seen significantly more success. An example “Google dork” for Connect6:




    
  * site:connect6.com target.com



During a standard test it can be useful to attempt to verify the emails gathered. Using the target’s mail server, we can test for SMTP return codes that could potentially support the verify behavior. This simply opens up a connection to the target SMTP server, starts to create a message for an internal recipient and checks for the return codes (250 or 550). By providing the SMTP server a known invalid address, the tester can test if it returns with a 250 code. If this is the case, the server is known as a “catch all”. If anything other than a 250 is returned, we are in luck and can verify the emails gathered and built from name generation!

**The Birth of SimplyEmail:**

After months of performing the above methodology, consuming roughly a half day of a test time with inaccuracy, I knew there was room within the industry to build a tool that was focused on simplifying the process and improving the accuracy of the information gathered. Many tools built in the security industry have many facets and are generally Swiss army knives in their realm. I knew from the start that SimplyEmail had to do only (simply) email, backed by a framework that would allow other members on the team or industry contribute with ease. Thus, the concept of SimplyEmail was born, and has evolved in accuracy and capability in the email reconnaissance realm.

Currently, SimplyEmail has 25 modules ranging in capabilities and fidelity. Major sources searched are:




    
  1. HTML scrape of the targets web site

    
  2. PasteBin

    
  3. Exalead search – PDF –XLSX – DOCX –PDF

    
  4. Google Search – PDF XLS/XLSX –DOC/DOCX –PDF

    
  5. PGP keys

    
  6. Instagram

    
  7. Reddit

    
  8. Ask Search

    
  9. Yahoo Search

    
  10. Whois Search

    
  11. GitHub user – Code – Gist

    
  12. Flickr

    
  13. Cannary Bin – API



The SimplyEmail framework uses a Task controller and a Producer Consumer model that allows testers to easily write modules, while the framework handles parsing, process creation and formatting. To get started with SimplyEmail you simply:


    
  1. 

    
    git clone <a href="http://github.com/killswitch-GUI/SimplyEmail.git">http://github.com/killswitch-GUI/SimplyEmail.git</a>




    
  2. 

    
    ./sh install.sh(Kali 2 or Debian Currently Supported)




    
  3. 

    
    ./SimplyEmail –l or ./SimplyEmail –h






After the installation of SimplyEmail, it’s time to give it a go:


    
  1. ./SimplyEmail –all –e YOURTARGET.com –n –verify

    
    1. –all = Use all modules

    
    2. –e = Your target domain

    
    3. –n = Names generation

    
    4. –Verify = Attempt to verify your discovered emails Address






[![pic1](http://cybersyndicates.com/wp-content/uploads/2016/05/pic1-923x1024.png)](http://cybersyndicates.com/wp-content/uploads/2016/05/pic1.png)

After the initial email scrape is complete name generation is conducted. Using the built-in LinkedIn Bing scraper, SimplyEmail will start building names using LinkedIn & Connect6:

[![pic2](http://cybersyndicates.com/wp-content/uploads/2016/05/pic2.png)](http://cybersyndicates.com/wp-content/uploads/2016/05/pic2.png)

[![pic3](http://cybersyndicates.com/wp-content/uploads/2016/05/pic3.png)](http://cybersyndicates.com/wp-content/uploads/2016/05/pic3.png)

Next, name generation will be completed. SimplyEmail has two methods to determine the email format. First, it attempts to use EmailHunter’s JSON API to detect the format of the emails. If that fails, SimplyEmail has a built-in class designed to detect the following supported formats:




    
  * {first}.{last} = [alex@domain.com](mailto:alex.alex@domain.com)

    
  * {first}{last} = [jamesharvey@domain.com](mailto:jamesharvey@domain.com)

    
  * {f}{last} = [ajames@domain.com](mailto:ajames@domain.com)

    
  * {f}.{last} = [james@domain.com](mailto:a.james@domain.com)

    
  * {first}{l} = [jamesh@domain.com](mailto:jamesh@domain.com)

    
  * {first}.{l} = [amesh@domain.com](mailto:j.amesh@domain.com)

    
  * {first}_{last} = [james_amesh@domain.com](mailto:james_amesh@domain.com)

    
  * {first} = [james@domain.com](mailto:james@domain.com)



[![pic4](http://cybersyndicates.com/wp-content/uploads/2016/05/pic4.png)](http://cybersyndicates.com/wp-content/uploads/2016/05/pic4.png)

Lastly, as testers, we all know the importance of reporting and the ability to digest the data we just captured. SimplyEmail has a few great reporting options that are built into the tool. The standard text report and HTML file that will show all non-unique emails with the corresponding sources where SimplyEmail found those emails.

[![pic5](http://cybersyndicates.com/wp-content/uploads/2016/05/pic5.png)](http://cybersyndicates.com/wp-content/uploads/2016/05/pic5.png)

This generates a clean report on the sources queried and an ordered list of all emails gathered. This also allows for your report to show duplicate email address gathered, which is specifically handy in the case that you need to correlate an email to multiple sources. This helps clients and testers distinguish where trouble areas are and helps make recommendations based on external OSINT presence.  Of course, a standard text file with raw emails is also built.** [![pic6](http://cybersyndicates.com/wp-content/uploads/2016/05/pic6.png)](http://cybersyndicates.com/wp-content/uploads/2016/05/pic6.png)**** **

**Phishing Template Generation is an art: **

Email template generation often takes a lot of time and effort to get right. This is largely due to ROE restrictions and stipulations put in place by different clients. Some clients say send the best you have, others want to pre-approve the template and even the payload in certain cases. Depending on the client they may even want to pick from a subset of templates. To make sure they have options, testers will generally develop multiple templates with varying degrees of sophistication. This can be a major, but necessary, pain for a tester.

Current methodology for template generation is pretty standard and is highly variable per target organization type (government vs. civilian, service vs product based etc.). This complicates template generation for the following reasons:




    
  * Templates must have relevance to the client

    
  * Templates must entice the target to act promptly

    
  * Template payload must be supported by template subject/body

    
  * Templates must be formatted correctly

    
  * Templates should have unique text values build the legitimacy of the message body

    
  * Multiple templates should be generated for failed phishing attempts



While this is quite an extensive list, this is merely the bare minimum to be successful at the initial stage of generating the template, let alone template theme or OSINT that is put into making the proper decisions to support the theme. Here is what my current model looks like from an operational prospective:


    
  1. Conduct quick OSINT overview of the company

    
  2. Pick a template theme based on current news or info retrieved from OSINT

    
  3. Conduct research on the correct person/position to emulate in email if necessary

    
  4. Create a message body that supports either a link or attachment based payload

    
  5. Develop HTML by hand for the message body

    
    1. If necessary, use outlook to build out .EML for rich HTML messages

    
    2. If using an old template, replace/insert necessary data inside of HTML which contains CSS




    
  6. Use Cobalt Strike’s spear phish option to view the email

    
  7. Send test phish and make corrections



This whole process is extremely time consuming to get correct and changes are often necessary to get the template 100%. If one thing is off it can affect the outcome of statistics as well as payload execution, and in many cases you only get one chance at this. I’ve found that when done correctly it was a solid 4-5 hours from start to finish. This is mainly due to reusing advanced templates and making necessary adjustments as needed. I set out to automate this process as much as possible with SimplyTemplate.

**SimplyTemplate**

** **Getting started with SimplyTemplate is extremely easy and can be a great supplement for the above methodology to cut down on template generation time, reduce errors, and generate rich HTML sophisticated emails with ease. This tool aims to automate 80% of the template generation process, with the 20% you perform on your own hopefully resulting in a pull request!  To get started perform the following:




    
  1. 

    
    git clone https://github.com/killswitch-GUI/SimplyTemplate.git




    
  2. 

    
    ./Setup.sh (Kali 2 or Debian currently supported)




    
  3. 

    
    install the required plugin when prompted into Ice Weasel




    
  4. 

    
    ./SimplyTemplate.py –l






To start, it helps to have an understanding of the module types so we can make accurate choices on template selection. All templates will provide you with a small meta tag. This tag will help you quickly identify the capabilities of the module, also what the "content" supports.


    
  * High - Requires proper OSINT / Social Engineering to build and effectively deploy the template. These are generally internal based templates with specific themes.

    
  * Medium - Requires a decent amount of modifications or settings, and are more general of a template external based template.

    
  * Low - Requires little to no modifications of the template and are generally not effective.



Each template will support one or all of the following core options:


    
  * Text - Text based option or output.

    
  * Html - Rich Html Supported for output (generally multipart Email Html/Text).

    
  * Link - Template supports a major link for stats or potential web download of document/Drive-by.

    
  * Attachment - Can support text that tells users to open or use the supplied attachment.



Seeing how these templates are actually rendered is extremely important. This was an issue I’ve had with the current process for templates with anything other than basic HTML tags. Some phishing platforms have issues with rendering certain formats of email templates. I highly advise using a service like Litmus ([https://litmus.com)](https://litmus.com)) or  Mail-Tester ([https://www.mail-tester.com)](https://www.mail-tester.com)) to see how the email renders on multiple applications.




    
  * HTML - This was used by some to view the HTML markup but CSS does not render correctly in some cases. (basic templates)

    
  * eml - Files can be outputted via .EML to open them directly in Icedove or Outlook

    
  * mht - MHTML is the Mail Html Markup used and can directly rendered in Word/IE or Iceweasel via plugin



Let’s get started with building our first template and rendering these templates.

[![pic7](http://cybersyndicates.com/wp-content/uploads/2016/05/pic7.png)](http://cybersyndicates.com/wp-content/uploads/2016/05/pic7.png)

Currently the templates are broken up by the phishing category they support based on the following:




    
  * External – Templates that would most likely come from external sources

    
    * News

    
    * Agency

    
    * Storage

    
    * Social




    
  * Internal – Templates that would come from internal departments or employees

    
    * Marketing

    
    * IT

    
    * HR

    
    * Agency

    
    * Facilities

    
    * Leadership






As you can see this is easily displayed with the (_list)_ command within.  We can also use the (_search_) command to search for modules by sophistication or core options of the templates.

[![pic8](http://cybersyndicates.com/wp-content/uploads/2016/05/pic8.png)](http://cybersyndicates.com/wp-content/uploads/2016/05/pic8.png)

Execute the (_use_) command with the corresponding number and you will be dropped into the relevant template menu. If more information is required, the _(info)_ command can be used to template variables and a more in-depth explanation.

[![pic9](http://cybersyndicates.com/wp-content/uploads/2016/05/pic9.png)](http://cybersyndicates.com/wp-content/uploads/2016/05/pic9.png)

Once in the template menu you have a few core commands. The first is the _(set)_ command, which allows you to set specific variables within the template. There are a set of default values that can be used, or you can provide your own. Each template will require a mix of different required values that must be set. You will also observe that higher the sophistication needed, the more OSINT is required and in turn the more settings that will need to be populated. Once all options are set, SimplyTemplate can generate the template on the fly and display the final outcome within a browser. This allows the tester to quickly make changes to settings to ensure the final result will look good on the targets end. Running the _(info)_ command will show the following changes:

[![pic10](http://cybersyndicates.com/wp-content/uploads/2016/05/pic10.png)](http://cybersyndicates.com/wp-content/uploads/2016/05/pic10.png)

As mentioned before the (_render_) command can be extremely informative to the tester:

[![pic11](http://cybersyndicates.com/wp-content/uploads/2016/05/pic11.png)](http://cybersyndicates.com/wp-content/uploads/2016/05/pic11.png)

Even the more advanced templates render nicely in the. mht format:

[![pic12](http://cybersyndicates.com/wp-content/uploads/2016/05/pic12.png)](http://cybersyndicates.com/wp-content/uploads/2016/05/pic12.png)

Depending on the template and the corresponding sophistication level the (_edit)_ function may be available. This will eventually be used in a majority of the modules to give testers greater freedom in generating templates. When using this function, you can edit the raw paragraphs of the template before rending or template generation takes place. This is a more advanced feature but allows you to quickly add small changes with out digging through hundred lines of template code to make a spelling or grammar change. This will spawn a custom Text Editor which you can make and save the changes.

[![pic13](http://cybersyndicates.com/wp-content/uploads/2016/05/pic13.png)](http://cybersyndicates.com/wp-content/uploads/2016/05/pic13.png)

Finally use the (_gen_) command to quickly export the final product!

As always I’m looking for improvements or suggestions on both SimplyEmail or SimplyTemplate, every bit helps produce a more useful platform!



Here are the links:

[https://github.com/killswitch-GUI/SimplyEmail](https://github.com/killswitch-GUI/SimplyEmail)

[https://github.com/killswitch-GUI/SimplyTemplate](https://github.com/killswitch-GUI/SimplyTemplate)
