+++
author = "Alexander Rymdeko-Harvey"
tags = [
]
title = "Egressing Bluecoat with CobaltStike & Let's Encrypt"
description = "Circumvent Bluecoat proxy with Let's Encrypt"
date = "2016-12-04T21:39:41-05:00"

+++
<iframe width="640" height="360" src="https://www.youtube.com/embed/cgwfjCmKQwM?rel=0" frameborder="0" allowfullscreen></iframe>
This post is a bit different than other posts I have done in the past, the material I will talking about directly helped me overcome an extremely challenging engagement. When it comes to egress protections I personally am not often met with an environment that I can't get out of. It's not about the skill level of the operator, just the techniques, and processes that we employee as a team and we stick to ensure successful engagements.

### Contact
-   Author: Daniel West
-   Twitter: @reaperb0t
-   E-mail: admin@homelandofthings.org
-   Website: https://homelandofthings.org

## Background 
During a recent engagement, we followed many of the tips I mentioned in my last post: https://cybersyndicates.com/2016/11/top-red-team-tips/ including using separate C2 transports for each stage of the engagement. Unfortunately, during initial access we were able to obtain DNS callbacks, but never able to transition to our active OP's team server via HTTP or HTTP(S).

### Identifying a Proxy
Once you identified issues with obtaining a callback via HTTP or HTTP(S) you should immediately start thinking:

-   Proxy (web proxy)
-   HTTP(S) interception
-   Domain Categorization 
-   Domain life / Rating

While you could have other issues like HIPS stopping injection, process with no token context. I have also seen people use elevated (SYSTEM) context and attempting to spawn with no way to authenticate to the proxy. 

The first step it to troubleshoot and see if that user can even authenticate to the proxy. I often use PowerShell to easily check to see if I can reach common sites, here is an example: 

```
powershell.exe -w hidden -command $wc = New-Object System.Net.Webclient; $wc.Headers.Add('User-Agent','Mozilla/5.0 (Windows NT 6.1; WOW64;Trident/7.0; AS; rv:11.0) Like Gecko'); $wc.proxy= [System.Net.WebRequest]::DefaultWebProxy; $wc.proxy.credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials; $wc.downloadstring('http://google.com/')
```

Once you have confirmed that the token context or process that your currently in has the ability to authenticate you can move forward. If not try the following:

1.  Target a user process and inject into that to operate within.
2.  Target specific process: exporer.exe, chrome.exe, ie.exe
3.  Move to a different beacon, often proxies will expire connections. I have been in situations where the beacon would go stale every 30 or so mins.

### Egress Constraints 
Once you have confirmed that you are dealing with a proxy, you will have to start troubleshooting the issues. The three things we will cover: Domain Age, Domain Categorization, HTTPS Certs.

#### Domain Age
In most cases domain age does not come into play. While it’s very rare we see an environment with that level of egress protections, its totally reasonable to be doing. Websense can perform this, bluecoat uses WebFilter and domain age of 14 days or less are often blocked by these devices. 

To test for this I often take an older domain that I know is blog personally owned site. These are often only a few years old and they will help you potentially rule out the domain age. You could follow many of the steps and may have to retool to a domain maybe you owned for a good bit. I know I hold about 5 domains myself for testing so domain age *should* not be a major issue. 

#### Domain Categorization 
We often out of habit attempt to get our domain categorized before we start and engagement. This process is scary easy and we often only go to a few different locations. Some safe bets to target:
 
-   Government
-   IT Tech
-   Finance 
-   News

Try to avoid personal tags, streaming service etc, those can often be blocked to prevent bandwidth use or they don't want their employees spending all day on Hulu. You can easily clone a site with:

```
wget –mk www.example.com
```
I suggest taking the time and finding something close with the keywords that you are trying to target for domain categorization. Once you have the website setup submit it to the following:

-   https://sitereview.bluecoat.com/sitereview.jsp
-   https://community.opendns.com/domaintagging/
-   https://www.brightcloud.com/tools/change-request-url-categorization.php
-   https://archive.lightspeedsystems.com
-   https://support.forcepoint.com/KBArticle?id=How-To-Submit-Uncategorized-Sites

#### Domain Certification
The last thing that plays into the domain overall score is if it contains a proper SSL cert. For a long time, up till recently to obtain an SSL cert you had to go through a register that would properly issue you a cert and prove you owned the domain etc. Many levels of authenticity exist for SSL certs. So this added a small level of complexity for most testers due to the monetary investment as well as the time it generally took to be issued the cert.

Three types of certs we will see; Wildcard, Multi-Domain, and EV (Extended Validation)/SAN certs. While different proxies likely filter and block on the type of cert, I would imagine most environments will allow properly signed certs through. In my case, I was using the self-signed certificate from CobaltStrike and being blocked. Using a self-signed cert not only looks highly suspicious but also could be easily blocked. We will be using Let's Encrypt next to help build that domain ranking, score and hopefully find a path out.

Its general worth just using the PowerShell one liner to request your C2 HTTPS site. Letting you see a returned a status code or HTML that fingerprint the Proxy you are being blocked by. In my case I was able to see the Bluecoat ProxySG page displayed for “blocked content”.

The last thing we will test is the ability to reach a “sample” site that would match your parameters. Using PowerShell once again we will attempt to find a site that has the correct domain categorization and contains a Let's Encrypt Cert. In my case I used Cybersyndicates.com since I host my site with a LetsEncypt cert and I have IT / Computer Security Rating with Bluecoat. 
```
powershell.exe -w hidden -command $wc = New-Object System.Net.Webclient; $wc.Headers.Add('User-Agent','Mozilla/5.0 (Windows NT 6.1; WOW64;Trident/7.0; AS; rv:11.0) Like Gecko'); $wc.proxy= [System.Net.WebRequest]::DefaultWebProxy; $wc.proxy.credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials; $wc.downloadstring('https://cybersyndicates.com/')
```

This worked since Bluecoat trusted the root chain of the cert which is signed by DST. We now know a bit of intel of what is stopping us and how we can circumvent this. 

## CobaltStrike Integration with Let's Encypt Process
Once you have found a combination that works its time move forward with setting up your C2 Server. The process is a bit long manually, so follow along and at the end, I will provide an easy to use setup script for this entire process.

1.  Install the following:
```
Sudo apt-get install keytool openssl git apache2 -y
```

2.  Start Apache2 
```
service apache2 start
```

3.  Check to ensure Apache2 is listening on 80, and the return value should 1. *note: this is important as Let's Encrypt will use this to verify you own the domain. 
```
lsof -nPi | grep -i apache | grep -c ":80 (LISTEN)"
```

4.  In my case with Ubuntu images allow port 80,443 through the firewall:
```
ufw allow 80/tcp
ufw allow 443/tcp
```

5.  Install Let's Encrypt:
```
git clone https://github.com/certbot/certbot /opt/Let's Encrypt
cd /opt/Let's Encrypt
```

6.  Run Let's Encrypt ($domain is you domain name): *note: I found the --register-unsafely-without-email flag hidden in the documentation, which will allow you easily run this script without any prompts from Let's Encrypt 
```
./Let's Encrypt-auto --apache -d $domain -n --register-unsafely-without-email --agree-tos
cat /etc/Let's Encrypt/live/$domain/fullchain.pem
```

7.  Following some documentation from CobaltStrike (https://www.cobaltstrike.com/help-malleable-c2) we find we will need a Java KeyStore, to do this will first need make our PKCS12 certs.
```
cd /etc/Let's Encrypt/live/$domain
openssl pkcs12 -export -in fullchain.pem -inkey privkey.pem -out $domainPkcs -name $domain -passout pass:$password
```

8.  Building java keystore via keytool:
```
keytool -importkeystore -deststorepass $password -destkeypass $password -destkeystore $domainStore -srckeystore $domainPkcs -srcstoretype PKCS12 -srcstorepass $password -alias $domain
```

9.  Make a directory for https in CS dir, and copy the new store:
```
mkdir ~/CobaltStrike/https
cp $domainStore ~/CobaltStrike/https/
```

10. Setup CobaltStrike Malleable C2 Profile to use SSL cert:
```
wget https://raw.githubusercontent.com/rsmudge/Malleable-C2-Profiles/master/normal/amazon.profile --no-check-certificate -O amazon.profile
```

11. Add the following lines to the new amazon profile with the settings you built the keystore with:
```
https-certificate {
    set keystore "domain.store";
    set password "mypassword";
}
```

## CobaltStrike Integration with Let's Encypt Automated
Of course, I had to do this on MANY domains so I automated the process so you don’t make mistakes and this whole process takes less that 2 mins top! 

<script src="https://gist.github.com/killswitch-GUI/bdf00b3bc7035abf7d42c516933d0cac.js"></script>
