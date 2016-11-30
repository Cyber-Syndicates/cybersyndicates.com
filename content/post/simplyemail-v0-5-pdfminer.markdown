---
author: killswitch
comments: false
date: 2015-11-21 18:32:48+00:00
layout: post
link: http://cybersyndicates.com/2015/11/simplyemail-v0-5-pdfminer/
slug: simplyemail-v0-5-pdfminer
title: 'SimplyEmail v0.5: PDFMiner!'
wordpress_id: 658
tags:
- simplyemail
---

## Major addition to SimplyEmail Core



When I first started this project I published a pretty in-depth “path” of progression. I told my self I would be following this to help build a tool that would actually enhance and build off what is already open source. I knew I had to have unique methods and content. One thing that I use quite often is google dorking for multiple Intel gathering techniques for a OP, so I knew this had to be added!

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">SimplyEmail v0.5: PDFMiner just added! Email recon with PDF scraping. <a href="https://t.co/awX3eoRGNK">https://t.co/awX3eoRGNK</a> (or on GitHub) <a href="https://t.co/JVsc7yrflE">https://t.co/JVsc7yrflE</a></p>&mdash; Alex Rymdeko-harvey (@Killswitch_GUI) <a href="https://twitter.com/Killswitch_GUI/status/668137062055452673">November 21, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

### Intel Gathering Stage



During my tests I weigh heavy on open source information  to help me paint a picture of the culture and potential target rich opportunities once internal. During this stage we are often tasked to Phish external for statistics or payloads to simulate and external threat.

This generally encompasses the ability to generate a custom phishing template that is somewhat unique to the target. I can sometimes locate sensitive information using the **(site:target.com file:pdf) **method to gather documents they may have published. We generally gather the following information:

  * CEO and main leadership    
  * Current issues or press releases that are related to the target
  * What the company sells



I take all the above into account when developing potential targeted emails. Funny enough these documents are a great source for emails as well. As they generally have contact information outside the standard [info@company.com](mailto:info@company.com). This source however can be hard to gather from the HTML scrapping modules already in place. One the documents can take a good amount of time to gather, and for the sake of speed I have always found using googles indexed files always seemed to speed things up.

[![Screen Shot 2015-11-21 at 1.13.55 PM](/wp-content/Screen-Shot-2015-11-21-at-1.13.55-PM.png)](http://cybersyndicates.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-21-at-1.13.55-PM.png)



### PDFMiner



With some basic requirements in hand I knew I could use the previous built module to handle most of the google search requirements. I just need a few small tweaks and I could start parsing the raw HTML of google for PDF files. Using PDFMiner I was able to convert a bulk of them to Text for parsing.

[![Screen Shot 2015-11-21 at 1.14.06 PM](http://cybersyndicates.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-21-at-1.14.06-PM.png)](http://cybersyndicates.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-21-at-1.14.06-PM.png)

Suppersingly enough this wasn’t as hard as I thought it was going to be! Mainly since the method I use to handle all the background work has been handled for me!

Expect to see upcoming this month:

  * Excel Parsing
  * Doc / Docx Parsing


As always you can check out the code here: https://github.com/killswitch-GUI/SimplyEmail/blob/master/Modules/GooglePDFSearch.py

Sample Output and test using the new Verbose options:

``` 
    root@vapt-kali:~/Desktop/SimplyEmail# ./SimplyEmail.py -t GooglePDF -e mandiant.com -v
```

[![Screen Shot 2015-11-21 at 1.22.37 PM](http://cybersyndicates.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-21-at-1.22.37-PM.png)](http://cybersyndicates.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-21-at-1.22.37-PM.png)


