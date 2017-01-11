---
author: slacker007
comments: true
date: 2015-06-20 01:57:53+00:00
layout: post
link: http://cybersyndicates.com/2015/06/advance-active-monitoring-with-ossec/
slug: advance-active-monitoring-with-ossec
title: Advanced Active Monitoring with OSSEC
wordpress_id: 402
categories:
- Open Source Black Team Projects
tags:
- Active Monitoring
- Cyber Defense
- Intrusion Detection
- OSSEC
---

Tools Used For this tutorial.

##############################################################

OSSEC 2.8.2 ([http://www.ossec.net](http://www.ossec.net))

Ubuntu 14.04 ([http://www.ubuntu.com/download/desktop](http://www.ubuntu.com/download/desktop))

Gmail Account ([https://www.gmail.com](https://www.gmail.com))

Postfix ([www.postfix.org](http://www.postfix.org))

mailutils (installed from bash )

##############################################################

Step (1) Download Ubuntu 14.04 (Update: **apt-get update; apt-get upgrade**)

Step (2) Download OSSEC 2.8.2 Software

Step(2b) OPTIONAL Download Web UI 0.8

Step(3) Install OSSEC Software




    
  * Two primary methods for installing OSSEC includes using the default settings (install.sh) or by manually installing it. Manual installation is simply just creating the required directories and copying the files into the OSSEC directory. Compiling each of the files and moving them to the appropriate directories. And lastly creating users and providing them with permissions to the files. Editing the install.sh script and/or the binaries can also be done to customize the install process. This is especially useful when configuring OSSEC for an unattended install, in which you will want to automate various aspects of the install to meet your environment needs.






    
  * The method that will be used in this demonstration will be the default install, with additional configurations performed after installing.






    
  * On your Ubuntu system, cd into the directory where you have downloaded the files and unpack them.






    
  * **tar –xzf ossec-hids-2.8.2.tar.gz**




    
  * Run the install.sh script and answer each of the questions.

    
    * ./install.sh

    
    * You’ll notice that during the question/answer video I selected “Server” as my choice. Although this will be the only system deployed for this post, it will still perform the same way. It just now gives you the option to add clients at a later time.






[embed]http://youtu.be/dbKrLRJLpRA[/embed]





[embed]http://youtu.be/D_b5QI5ABsI[/embed]




    
  * Once OSSEC has been installed ensure that you allow the necessary communication for syslog traffic on your firewall. The default port is UDP 1514





**Primary Files For OSSEC**

** **

**(path/on/your/system/ossec.conf) – **This file is the primary configuration file for the OSSEC framework. The first section by default is a <global**> **tag that contains the parameters that specify the email notification address and SMTP server that OSSEC will use. Ensure that you have the 5 entries that are included in the screenshot below:

<email_notifications> should be “yes”; This tells OSSEC that you want to receive alerts.

<Email_to>should be [yourgmailaddress@gmail.com](mailto:yourgmailaddress@gmail.com)

<smtp_server>should be “localhost”;




    
  * This setting is critical and it is COMPLETELY based off of your specific environment. For this test environment I am on a private network that is NAT’d, behind a firewall that is allowing outbound SMTP. It is important to fully understand the rules within your environment prior to implanting OSSEC if you wish to receive alerts outside of your network. Also keep in mind that many ISP’s now block SMTP and in that case it is best to use the one they provide for you. A few Google searches should answer that question for you, and if that is the case then wherever you see us use gmail, supplement the SMTP-RELAY-SERVER for your ISP.   For this test we will be using gmail as our (SMTP-RELAY-SERVER) and (localhost(which will be POSTFIX)) as our internal SMTP-SERVER. So basically OSSEC will generate an alert and send it to POSTFIX, which will then send it to whichever SMTP-RELAY-SERVER(gmail for this example), and from there it will be delivered to the email address in the <email_to> tag field. If you’re new to OSSEC, Linux, &/or PostFix you’re probably wondering why not just input gmail’s SMTP-RELAY-ADDRESS in the <smtp_server> tag field and cut out the middleman (PostFix). The reason why Postfix is used is that it is much better suited for handling emails. It handles encryption, transmission, & authentication much better than OSSEC does. Not to say that OSSEC couldn’t provide this functionality because after all it is Open Source, therefore whatever it lacks can always be written into it. Another reason is the separation of duties/functions. Its best practice to separate roles whenever possible and to limit the amount of services that are nested.



<email_from> can be whatever email you’d like to appear in the “From” line (**NOW YOU SEE WHY MANY ISP’S BLOCK SMTP)**

<email_maxperhour> you will have to add this line yourself, but it is important if you are using a free SMTP-RELAY-SERVICE. Many of them have a threshold for the amount of emails that they’ll relay per day. This number is up to you and can be anywhere from **1 – 9999 **messages per hour. Keep in mind that this is only the cap on the amount o f emails that are sent per hour and does not stop OSSEC from generating alerts that remain on the Server itself.



 [![OssecEmailSettings](/wp-content/OssecEmailSettings-300x203.png)](/wp-content/OssecEmailSettings.png)



The next section of this file is the <include> tag field which is used to include separate files that contain rules for OSSEC

Further down in this file there is a <syscheck> segment that dictates how often the file systems will be checked. The default setting is 79200 seconds (22 hours) You can adjust this time based off your unique environmental needs.

In addition to the frequency of the syscheck, you can also dictate which files to check and which to ignore.

The <rootcheck> tag field is also included in this file. This area should not be altered at this time. There will be another post in the near future in which we will discuss custom configuration for rootkit identification based on current indicators of compromise.

The <alerts> tag field contains two important fields: log_alert_levels (what is the min level of alert you wish to be logged) & email_alert_levels(min level of alerts that you wish to be sent immediately to your monitored email account) These fields are configurable based on your risk versus noise tolerance from **1-10. **

** **

The last tag field that we will discuss in this file will be the <active-response> tag field. The two fields within this section that will be important for us will be the <level> tag and the <timeout> tag. The           < level> tag will be set based off your unique environment’s risk tolerance and IR response time. The <timeout> tag is critical and should be set carefully. The number by default is 600 (seconds) and will automatically respond to network based alerts by blocking the IP for the length of time you specify. This can be tricky because if your IR response time is slow then you would ideally want to block for a length of time that supports their responsiveness so that they can be effective. What makes this the tricky part is that too short of a lockout gives the attacker more time to compromise a system or exfiltrate data before your IR team has time to react but too long and a legitimate system may be locked out which could degrade production.



**decoder.xml**

Decoder.xml is probably one of the most important files to understand in order to customize the functionality of this software to your environment. Briefly explained, it is the file which dictates which files are to be monitored by OSSEC. The basic method for adding logs to the file is:

<filename to be logged: EX”**<mail>**”>

<log_format>syslog</log_format> **(syslog is the default format but “snort-full, snort-fast, squid, iis, eventlog, and eventchannel are also allowed)**

<location>absolute/path/to/log</location>

**</mail**>



The second part of this file includes custom decoders. The default path to this file is /var/ossec/etc. This file is explained thoroughly by OSSEC on their documentation link. [Http://ossec-docs.readthedocs.org](Http://ossec-docs.readthedocs.org) According to OSSEC, they are broken into phases, with the first phase performing the pre-decoding. The structure is:

**<decoder name="ossec-exampled">**

** <program_name>ossec-exampled</program_name>**

**</decoder>**

Phase 2 now correctly identifies this log message as coming from ossec-exampled. There is still some very important information in the log message that should be decoded, namely the source IP and test-protocol1. To decode these a child decoder will be added. It will set the ossec-exampled decoder as a parent, and use prematch to limit its use to the correct log message.

**<decoder name="ossec-exampled-test-connection">**

** <parent>ossec-exampled</parent>**

** <prematch offset="after_parent">^test connection </prematch> <!-- offset="after_parent" makes OSSEC ignore anything matched by the parent decoder and before -->**

** <regex offset="after_prematch">^from (\S+) via (\S+)$</regex> <!-- offset="after_prematch" makes OSSEC ignore anything matched by the prematch and earlier-->**

** <order>srcip, protocol</order>**

**</decoder>**

Breaking this down piece by piece:




    
  * <decoder name="ossec-exampled-test-connection"> - Declaring this to be a decoder and giving it a name.

    
  * <parent>ossec-exampled</parent> - This decoder will only be checked if ossec-exampled also matched.

    
  * <prematch offset="after_parent">^test connection </prematch> - If a log message does not contain the data in the prematch, it will not use that decoder. Setting the offset tells OSSEC to only look at data after the parent (ossec-exampled[9123]: in this case), in an effort to speed up matches.

    
  * <regex offset="after_prematch">^from (\S+) via (\S+)$</regex> - The regex line can be used to pull data out of the log message for use in rules. In this instance the first \S+ matches the IP address, and the second matches the protocol. Anything between the parenthesis will be able to be used in rules.

    
  * <order>srcip, protocol</order> - Defines what the entries in the regex line are labeled as. The IP address will be labeled as srcip, and the protocol by proto.



The photo below is an actual screenshot of the structure of the decoder.xml file.



** [![xmlfile](/wp-content/xmlfile.png)](/wp-content/xmlfile.png)
**



For this post, the decoder.xml file will be the primary focus. Try adding custom entries to capture alerts for log-files for thirdparty software that you’ve added yourself. Remember to restart ossec after you’ve made changes and use the OSSEC documentation link provided above for additional instructions. We will address future customization techniques in future posts.

The final part for this setup will be installing and configuring postfix. Start by entering “**sudo apt-get install postfix and sudo apt-get install mailutils**" into your shell.




# See /usr/share/postfix/main.cf.dist for a commented, more complete version





# Debian specific: Specifying a file name will cause the first





# line of that file to be used as the name. The Debian default





# is /etc/mailname.





# myorigin = /etc/mailname


smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)

biff = no

appending .domain is the MUA's job.

append_dot_mydomain = no



# Uncomment the next line to generate "delayed mail" warnings





# delay_warning_time = 4h



readme_directory = no



# TLS parameters



smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem

smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key

smtpd_use_tls=yes

smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache

smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache



# See /usr/share/doc/postfix/TLS_README.gz in the postfix-doc package for





# information on enabling SSL in the smtp client.



smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem

smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key

smtpd_use_tls=yes

smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache

smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache



# See /usr/share/doc/postfix/TLS_README.gz in the postfix-doc package for





# Information on enabling SSL in the smtp client.



smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_un$

myhostname = localhost

alias_maps = hash:/etc/aliases

alias_database = hash:/etc/aliases

myorigin = /etc/mailname

mydestination = CyberSyndicates.com, CyberSyndicates-OSSEC, localhost.localdoma$

relayhost =

mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128

mailbox_size_limit = 0

recipient_delimiter = +

inet_interfaces = localhost

inet_protocols = all
Each of the lines above is in the main.conf file by default. There are a few changes that should be made to fit your particular email address and system information. View the entries that are in bold. These highlighted configs are important, especially the TLS section which contains some of the important information that will facilitate the encrypted connection to gmail.

The lines below are not added by default and should be added manually or copied and changed to fit your custom information. They are to be placed at the bottom of the main.conf file.

##################   Added by Me #######################

smtp_tls_loglevel =1

smtp_tls_security_level=encrypt

smtp_sasl_auth_enable=yes

smtp_sasl_password_maps=hash:/etc/postfix/sasl/passwd

smtp_sasl_security_options = noanonymous

######################## map slacker007@localhost to slacker007.dev@gmail.com #$

smtp_generic_maps=hash:/etc/postfix/generic

relayhost=[smtp.gmail.com]:587

In addition to adding the lines above and making all of the configurations you will also create two files (If they don’t already exist on your system and add the following lines)

Type vi   /etc/postfix/sasl/passwd

Add the following:

[smtp.gmail.com]:587 [USERNAME@gmail.com:YOUR_PASSWORD](mailto:USERNAME@gmail.com:YOUR_PASSWORD)

Save and exit vi

Run the following two commands:

sudo chmod 400 /etc/postfix/sasl/passwd

sudo postmap /etc/postfix/sasl/passwd

cat  /etc/ssl/certs/Thawt_Premium_Server_CA.pem | sudo tee –a /etc/postfix/cacert.pem

sudo /etc/init.d/postfix reload

finally test your mail by entering the command below:

echo “Test email | mail –s “Test Mail” [youremail@gmail.com](mailto:youremail@gmail.com)

If you have not seen the email show up in your email’s inbox, check the junk mail folder. If it is not there, check the /var/log/mail.log file on your system go to the bottom of the file and copy and paste the messages that refer to the email attempt to this blog and ask any questions you may have.

This configuration may seem like a small step but the amount of power gained with a properly configured active response setup is enormous. Below are some images of this system and the test email and some alerts that were generated from it. In addition to those images, are some additional images that contain the results of one of my research VPS’s and the types of alerts that it gets HOURLY..!!!!! It is attacked several times per hour by IP addresses from all over the world. The system built during this write-up was constructed in a lab behind a firewall so therefore there were no “attack or scan” alerts generated.



 [![hashnotification](/wp-content/hashnotification-300x175.png)](/wp-content/hashnotification.png)





[![failedSSHD](/wp-content/failedSSHD.png)](/wp-content/failedSSHD.png)





[![ReverseMapping](/wp-content/ReverseMapping1.png)](/wp-content/ReverseMapping1.png)



The alerts from the VPS !!!

**CHECK OUT WHERE THE IP ADDRESSES ABOVE RESOLVED TO…..!!!!!!!!!!!!!!!!!!!!!**



 [![chinaIP](/wp-content/chinaIP-300x153.png)](/wp-content/chinaIP.png)





[![honkong](/wp-content/honkong.png)](/wp-content/honkong.png)



Get additional information from:

**OSSEC. **[**http://ossec-docs.readthedocs.org/en/latest/manual/rules-decoders/create-custom.html**](http://ossec-docs.readthedocs.org/en/latest/manual/rules-decoders/create-custom.html)
