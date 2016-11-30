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

```
CreateFile() to initialize access to device through its symbolic ling

Communication w/ : DeviceloControl() // IOCTL call

WriteFile()            // pass "stream" data

ReadFile()            // recieve "stream" data

```
Recent vulnerabilities such as OpenType Font Driver Vulnerability(CVE-2015-2426) are still making use of the ability to exploint DLL's.  This particular vulnerability makes it possible for remote code execution by taking advantage of Window's Adobe Type Manager Library improperly handling specially crafted OpenType fonts.  You can see why it is important to monitor and examine your Drivers, Services, and DLL's.  I wrote a python module for our upcoming OFF-Toolkit (Offensive Forensics Toolkit) that facilitates that analyzing of Drivers and Services.  Some tidbits of information: Service Types are (**normally**) 0x10, 0x20, 0x100: Start is 2, 3, or 4 ONLY.  Driver Type is 0x01 or 0x02; Start is 0 or 1 only..   Services w/o "ObjectName" that is set to LocalSystem, NT AUTHORITY\LocalService, or NT AUTHORITY\NetworkService.... Services starting under the Svchost process must have an entry in SOFTWARE\Microsoft\Windows NT\CurrentVersion\svchost...

These are just a few things to keep in mind when analyzing data retrieved with this module.  For more information and for the source code, check it out at https://github.com/slacker007/Registry_Enumerator/blob/master/DriversAndServices.py

```
##############################################################################################################################
#                               				 OFFENSIVE FORENSIC FRAMEWORK   								   			 #
#											 (WINDOWS DRIVERS AND SERVICES FORENSICS MODULE)											 #
#											Compatible w/ Win 7,8,10 (x86 & 64)												 #
#												(MODIFIED ORIGINAL SEPT 2015)												   			 #
#															 BY 												   			 #
#					                           (slacker007) 					   			 				     #
##############################################################################################################################


######################################### Function & Variable Formatting Guide ###############################################
# Global Variables are all uppercase EX: GLOBAL_VARIABLE = 0
# local Variables are in all lowercase EX: local_variable = 0
# Function Names are written by capitalizing the first letter of each word EX: def FunctionName(): or def Function_Name():


import _winreg
import threading
import Queue
import time
from xml.dom import minidom
from pprint import pprint

#***********************************************************************************
# Global Variables
#***********************************************************************************

########################	Static Registry Keys	################################

SERVICES = r'SYSTEM\\CurrentControlSet\\services'

# List of Static Registry Keys: To add more keys just create a Variable up top with the same syntax and add the variable in this list.. And RUN!!
L_O_K = [SERVICES]

xml_file = open("driversandservices.xml", "w")
xml_file.write("<group name=\"DriversAndServices,\">\n")
q = Queue.Queue()

def get_MF_info(q, r):
	Iterate_Reg_Keys(_winreg.HKEY_LOCAL_MACHINE, r)

def doc_handler(x):
	xml_file.write("</group>")
	xml_file.close()	

def Read_Subkeys (key): #(FUNCTION THAT READS OPENED HIVE KEY DATA INTO A //GENERATOR OBJECT//  TO REDUCE MEMORY FOOTPRINT!)
	counter = 0
	while True:
		try:
			subkey = _winreg.EnumKey(key, counter)
			yield subkey
			counter += 1
		except WindowsError as e:
			break
def Read_Key_Values (key): #(FUNCTION THAT READS THE VALUES OF AN OPENED SUBKEY USING A //GENERATOR OBJECT// TO REDUCE MEMORY FOOTPRINT!)
	counter = 0
	while True:
		try:
			keyvalue = _winreg.EnumValue(key, counter)
			yield keyvalue
			counter += 1
		except WindowsError as e:
			break
def Iterate_Reg_Keys(hkey, key_path, tabs=0): #(FUNCTION THAT CONTROLS THE ITERATION THROUGH SUBKEY & VALUES)
	key = _winreg.OpenKey(hkey, key_path, 0, _winreg.KEY_READ)
	for subkey_name in Read_Subkeys(key): #(LOOP THROUGH THE REGISTRY KEY AND OPEN EACH SUBKEY)
		xml_file.write("\t<subkey name=\"")
		xml_file.write(str(subkey_name))
		xml_file.write("\">\n")
		subkey_path = "%s\\%s" % (key_path, subkey_name)
		Iterate_Reg_Keys(hkey, subkey_path, tabs+1)
		subkey_value_path = _winreg.OpenKey(hkey, subkey_path, 0, _winreg.KEY_READ)
		data_found = False
		for subkey_value in Read_Key_Values(subkey_value_path): #(LOOP THROUGTH THE SUBKEY TO PULL VALUES FROM SUBKEY)
			data_found = True
			if isinstance(subkey_value[1], str):
				try:
					converted_from_ascii = ":".join("{:02x}".format(ord(c)) for c in subkey_value[1])
					value_data2 = str(converted_from_ascii)
				except UnicodeEncodeError:
					pass
			elif not isinstance(subkey_value[1], str):
				try:
					value_data2 = str(subkey_value[1])
				except UnicodeEncodeError:
					pass
			value_data1 = str(subkey_value[0])
			xml_file.write("\t\t<id name=\"")
			xml_file.write(str(value_data1))
			xml_file.write("\">")
			xml_file.write(str(value_data2))
			xml_file.write("</id>\n")
		xml_file.write("	</subkey>\n")
	_winreg.CloseKey(key)

# Execution Control...................................................

for each in L_O_K:
	t = threading.Thread(target=get_MF_info, args = (q, each))
	t.start()
	t.join()
t = threading.Thread(doc_handler(xml_file))
t.start()
t.join
```

@real_slacker007


