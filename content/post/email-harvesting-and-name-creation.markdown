---
author: killswitch
comments: false
date: 2016-02-14 14:23:45+00:00
layout: post
link: http://cybersyndicates.com/2016/02/email-harvesting-and-name-creation/
slug: email-harvesting-and-name-creation
title: Email Verification and Email Name Creation
wordpress_id: 745
tags:
- SimplyEmail
---

Over the past few months we have been rapidly expanding the capability of SimplyEmail. While it is a very simple tool, it has been extremely successful on a few live engagements I have been on. I feel like it is ready for recommendation to say the least. I did however notice a few key features that some of the guys on the team mentioned that would be nice to have integrated. Thanks to [@_wald0](https://twitter.com/_wald0) and his suggestions I have implemented a email verification option. Also I learned a trick or so a few months back from [@harmj0y](https://twitter.com/harmj0y) on sick site called "Connect6", which seems to populate a large name DB of employees for each company. Also a fellow tester and friend ([Joshua Crumbaugh](https://twitter.com/nagasecurity)) let me on to a sick tool for grabbing Linkedin names from bing called [PhishBait](https://github.com/pan0pt1c0n/PhishBait). With all this I set out to build some new capabilities for SimplyEmail and learn some new tricks :)



#### SMTP Email Verification



This process is actually relatively easy to accomplish. Its a simple heuristic of SMTP return codes when attempting to send a email on the target SMTP server. The process takes place by first Identifying the proper MX record to point to. In many cases larger corporations will have more than one SMTP server with multiple MX records. These are for redundancy of course and are ordered by priority, here is a small snip of the code I built fast:

```
    def GetMX(self):
      MXRecord = [] 
      try:
        if self.verbose:
          print helpers.color(' [*] Attempting to resolve MX records!', firewall=True)
        answers = dns.resolver.query(self.domain, 'MX')
        for rdata in answers:
          data = {
            "Host": str(rdata.exchange),
            "Pref": int(rdata.preference),
          }
          MXRecord.append(data)
        # Now find the lowest value in the pref
        Newlist = sorted(MXRecord, key=lambda k: k['Pref']) 
        # Set the MX record
        self.mxhost = Newlist[0]
        if self.verbose:
          val = ' [*] MX Host: ' + str(self.mxhost['Host'])
          print helpers.color(val, firewall=True)
      except Exception as e:
        error = ' [!] Failed to get MX record: ' + str(e)
        print helpers.color(error, warning=True)
```

After obtaining all the MX records we can easily sort, pick the lowest value and feed it to the next function needed. So here is the meat of the verification checks. Before we go ahead and send the SMTP server all of the gathered emails we need to check if this SMTP server supports this. We do this via providing the STMP server a known "invalid" address, and we test for the a return code other than 250 (250 is a valid email code). If we get anything except a 250, we know that the SMTP server isn't just returning a 250 for each address supplied. we can test this pretty simply:

```
 def VerifySMTPServer(self):
      '''
      Checks for code other than 250 for crap email.
      '''
      # Idea from:
      # https://www.scottbrady91.com/Email-Verification/Python-Email-Verification-Script
      hostname = socket.gethostname()
      socket.setdefaulttimeout(10)
      server = smtplib.SMTP(timeout=10)
      server.set_debuglevel(0)
      addressToVerify = "There.Is.Knowwaythiswillwork1234567@" + str(self.domain)
      try:
        server.connect(self.mxhost['Host'])
        server.helo(hostname)
        server.mail('email@gmail.com')
        code, message = server.rcpt(str(addressToVerify))
        server.quit()
        if code == 250:
          return False
        else: 
          return True
      except Exception as e:
        print e
```

So lets say our SMTP server supports this type of check, we can simply build a function to perform the check for our gathered emails. Here is a small small output of it in action:

[![Screen Shot 2016-02-14 at 11.10.11 AM](/wp-content/Screen-Shot-2016-02-14-at-11.10.11-AM.png)](/wp-content/Screen-Shot-2016-02-14-at-11.10.11-AM.png)

Obviously I needed to wrap this in a simple helper class for SimplyEmail, the full code (Class) can be found here:

[https://github.com/killswitch-GUI/SimplyEmail/blob/master/Helpers/VerifyEmails.py](https://github.com/killswitch-GUI/SimplyEmail/blob/master/Helpers/VerifyEmails.py)



#### Connect6 Name Creation

Im always extreamly nervous to add in functionality to SimpleEmail.. hence the name! In some cases name creation can be a pivotal and vital addition to your phishing campaigns. Some times SimplyEmail will only find the standard email addresses or just a few emails. In this case email creation may be your saving grace. On my assessments I with out doubt found Connect6.com is a reliable source to gather names associated with companies.

Too start I throughly attempted to find  way to get the Connect6 URL using their search engine, with no anvil. This may be do to how they want you to pay for their service / API. So I did build a simple Google Dork function to attempt to resolve the correct URL:
```
    def Connect6AutoUrl(self):
      # Using startpage to attempt to get the URL
      # https://www.google.com/search?q=site:connect6.com+domain.com
      try:
        # This returns a JSON object
        urllist = []
        url = "https://www.google.com/search?q=site:connect6.com+%22" + self.domain + '%22'
        r = requests.get(url, headers=self.UserAgent)
      except Exception as e:
        error = "[!] Major issue with Google Search: for Connect6 URL" + str(e)
        print helpers.color(error, warning=True)
      try:
        RawHtml = r.content
        soup = BeautifulSoup(RawHtml)
        for a in soup.findAll('a', href=True):
          try:
            l = urlparse.parse_qs(urlparse.urlparse(a['href']).query)['q']
            if 'site:connect6.com' not in l[0]:
              l = l[0].split(":")
              urllist.append(l[2])
          except:
            pass
        return urllist
      except Exception as e:
        print e
```
Full code can be found here: [https://github.com/killswitch-GUI/SimplyEmail/blob/master/Helpers/Connect6.py](https://github.com/killswitch-GUI/SimplyEmail/blob/master/Helpers/Connect6.py)

Here is a quick sample of the output that you will given if it cant detect the correct URL:

[![Screen Shot 2016-02-14 at 11.20.14 AM](/wp-content/Screen-Shot-2016-02-14-at-11.20.14-AM.png)](/wp-content/Screen-Shot-2016-02-14-at-11.20.14-AM.png)



#### LinkedIn Name Creation



Linkedin is and has been know as a great source for social engineering, and recently I first hand got to see how effective it is to build a email campaign of it. Using the PhishBait tool mention earlier I was able to add in additional functionality and build in LinkedIn name scraping into SimplyEmail. Here is the really simple code I adapted:

```
def LinkedInNames(self):
      '''
      This function simply uses
      Bing to scrape for names and
      returns a list of list names.
      '''
      try:
        br = mechanize.Browser()
        br.set_handle_robots(False)
        self.domain = self.domain.split('.')
        self.domain = self.domain[0]
        r = br.open('http://www.bing.com/search?q=(site%3A%22www.linkedin.com%2Fin%2F%22%20OR%20site%3A%22www.linkedin.com%2Fpub%2F%22)%20%26%26%20(NOT%20site%3A%22www.linkedin.com%2Fpub%2Fdir%2F%22)%20%26%26%20%22'+self.domain+'%22&qs=n&form=QBRE&pq=(site%3A%22www.linkedin.com%2Fin%2F%22%20or%20site%3A%22www.linkedin.com%2Fpub%2F%22)%20%26%26%20(not%20site%3A%22www.linkedin.com%2Fpub%2Fdir%2F%22)%20%26%26%20%22'+self.domain+'%22')
        soup = BeautifulSoup(r)
        if soup:
          link_list = []
          NameList = []
          more_records = True
          Round = False
          while more_records:
            if Round:
              response = br.follow_link(text="Next")
              soup = BeautifulSoup(response)
            # enter this loop to parse all results
            # also follow any seondary links
            for definition in soup.findAll('h2'):
              definition = definition.renderContents()
              if "LinkedIn" in definition:
                name = (((((definition.replace('<strong>','')).replace('</strong>','')).split('>')[1]).split('|')[0]).rstrip()).split(',')[0]
                name = name.split(' ')
                if self.verbose:
```

Finally the full code can found here: [https://github.com/killswitch-GUI/SimplyEmail/blob/master/Helpers/LinkedinNames.py](https://github.com/killswitch-GUI/SimplyEmail/blob/master/Helpers/LinkedinNames.py)
