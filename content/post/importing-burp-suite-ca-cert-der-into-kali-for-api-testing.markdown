---
author: killswitch
comments: false
date: 2015-10-01 15:15:53+00:00
layout: post
link: http://cybersyndicates.com/2015/10/importing-burp-suite-ca-cert-der-into-kali-for-api-testing/
slug: importing-burp-suite-ca-cert-der-into-kali-for-api-testing
title: Importing Burp Suite CA cert .der into Kali for API testing
wordpress_id: 491
categories:
- Red Teaming / Pen Testing
---

## Importing Burp Suite CA



Just recently I had to test some API calls with a custom tool written to work with RestClient, unfortunately RestClient uses OpenSSL to check the SSL against the known root CA. This throws a nice error, like invalid certificate error when trying to point your bash shell at the HTTP proxy like Burp. With tons of headache I really couldn't find any good info on importing the SSL certs to Kali from burps CA export function.

After a bit of research I did however find a few posts for each step, just no in one easy location. Sometimes I write these posts just for my own repository of headache fixes!


Hope this helps some out, it will help me in the future:


### Step-By-Step Execution

In Burp go-to Proxy -> Options -> Export CA Certificate:

[![Screen Shot 2015-10-01 at 10.52.29 AM](/wp-content/Screen-Shot-2015-10-01-at-10.52.29-AM.png)](http://cybersyndicates.com/wp-content/uploads/2015/10/Screen-Shot-2015-10-01-at-10.52.29-AM.png)

Save that cert to your Desktop or anywhere you want really. Now its time convert your cert from a .DER to a .PEM
```
openssl x509 -inform DER -outform PEM -in burpca.der -out myca.crt.pem
```

Move cert:
```
cp  myca.crt.pem /etc/ssl/certs/myca.crt.pem
```

Go and run just for fun:




    
    update-ca-certificates


 Than add your HTTP proxy environmental variable for the RestClient to pick up, most CLI tools do check this for proxy information. This could be helpful for other tools as well.

    export http_proxy=http://127.0.0.1:8080/





 At this point you should be up and running for the use of burp and your API you fuzz.
