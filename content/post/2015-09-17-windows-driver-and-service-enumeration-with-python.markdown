---
author: slacker007
comments: false
date: 2015-09-17 20:13:43+00:00
layout: post
link: http://cybersyndicates.com/2015/09/windows-driver-and-service-enumeration-with-python/
slug: windows-driver-and-service-enumeration-with-python
title: Windows Driver and Service enumeration with Python
wordpress_id: 488
---

With malware becoming more and more sophisticated, it is becoming extremely difficult to identify malware on infected systems.  DLL hijacking, reflected DLL's, and exploiting vulnerable kernel drivers has become extremely popular.  As it turns out Windows makes communication between user mode programs and kernel driver -1 this fairly easy for attackers:

CreateFile() to initialize access to device through its symbolic ling

Communication w/ : DeviceloControl() // IOCTL call

WriteFile()            // pass "stream" data

ReadFile()            // recieve "stream" data

Recent vulnerabilities such as OpenType Font Driver Vulnerability(CVE-2015-2426) are still making use of the ability to exploint DLL's.  This particular vulnerability makes it possible for remote code execution by taking advantage of Window's Adobe Type Manager Library improperly handling specially crafted OpenType fonts.  You can see why it is important to monitor and examine your Drivers, Services, and DLL's.  I wrote a python module for our upcoming OFF-Toolkit (Offensive Forensics Toolkit) that facilitates that analyzing of Drivers and Services.  Some tidbits of information: Service Types are (**normally**) 0x10, 0x20, 0x100: Start is 2, 3, or 4 ONLY.  Driver Type is 0x01 or 0x02; Start is 0 or 1 only..   Services w/o "ObjectName" that is set to LocalSystem, NT AUTHORITY\LocalService, or NT AUTHORITY\NetworkService.... Services starting under the Svchost process must have an entry in SOFTWARE\Microsoft\Windows NT\CurrentVersion\svchost...

These are just a few things to keep in mind when analyzing data retrieved with this module.  For more information and for the source code, check it out at https://github.com/slacker007/Registry_Enumerator/blob/master/DriversAndServices.py

@real_slacker007


