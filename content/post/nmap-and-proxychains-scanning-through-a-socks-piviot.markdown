---
author: killswitch
comments: false
date: 2015-12-13 19:14:20+00:00
layout: post
link: http://cybersyndicates.com/2015/12/nmap-and-proxychains-scanning-through-a-socks-piviot/
slug: nmap-and-proxychains-scanning-through-a-socks-piviot
title: Scanning Effectively Through a SOCKS Pivot with Nmap and Proxychains
wordpress_id: 683
tags:
- nmap
---

When ever I am on a pen-test and I come across a major problem (Every OP) that requires some creative solution and I love to share it. Not only that I could help someone else one day, but it really helps me retain the knowledge for future use.

This past engagement we where on a very unique network dealing with ICS devices and a ton of Unix backend servers, while still handling our normal pen test methodology on the side (No Time to play). While I feel comfortable operating in most network environments, *NIX based environments generally pose to elevate the difficulty of lateral movement and achieving final end state goals of the OP. In this environment most of the critical assets we where targeting where properly VLAN’ed off and proper network segmentation was in place.

After initial host discovery, we noticed our active host count was low and we figured we needed to gain a foothold on a dual homed box. Gaining a foothold didn’t take long on a lovely Unix jump box. After confirming we could hit multiple VLANs and a known target, it was time to re-scan for our initial host discovery.

In the past I have played around with getting Nmap to scan through Tor, and there are plenty of those tutorials on the web. But this time I had to perform a larger scan of multiple IP’s (/16) for an initial host discovery. I knew I could use the Unix jump box as a SOCKS4/5 proxy to funnel all my scan data through using SSH without dropping anything to disk.

Here is the idea:

[![Screen Shot 2015-12-11 at 5.42.29 PM](/wp-content/Screen-Shot-2015-12-11-at-5.42.29-PM.png)](/wp-content/Screen-Shot-2015-12-11-at-5.42.29-PM.png)

The first step is to setup your SSH SOCKS Proxy:


    
    ssh -DnF 8080 user@jump-box



After that was setup I had to configure ProxyChains:

[![Screen Shot 2015-12-11 at 6.47.24 PM](/wp-content/Screen-Shot-2015-12-11-at-6.47.24-PM.png)](/wp-content/Screen-Shot-2015-12-11-at-6.47.24-PM.png)

Turn on Quite mode for ProxyChains

[![Screen Shot 2015-12-11 at 6.48.36 PM](/wp-content/Screen-Shot-2015-12-11-at-6.48.36-PM.png)](/wp-content/Screen-Shot-2015-12-11-at-6.48.36-PM.png)

As we move forward we will discover the timeouts section will have to be tinkered with. This is key to optimizing your performance with Nmap.

[![Screen Shot 2015-12-11 at 6.55.59 PM](/wp-content/Screen-Shot-2015-12-11-at-6.55.59-PM.png)](/wp-content/Screen-Shot-2015-12-11-at-6.55.59-PM.png)

Lastly set up the strike chain.



### Heart Aches with Nmap



So now we are at the main point of even writing this blog. Before when test scanning through Proxy tunnels I only did one or two address at a time. But in this case I needed to scan a large amount of IP space in a reasonable time table. In the past few months I have greatly optimized my scanning techniques to get the most out of my discovery scans. Working with a team of 3-5 can be tough with coordination and when conducting a internal pen-test I may need to get through generally entire /8 or a few /16’s. In this case I had a potential /16 behind my jump box.

So after following my SOP’s I noticed just firing off Nmap into this SOCKS proxy tunnel wasn’t going to cut it. We experienced large drops in performance and Nmap’s performance and timing parameters weren’t working correctly. Here is my standard Nmap String (Internal scan):

```
Nmap -Pn -n -sS –p 21-23,25,53,111,137,139,445,80,443,3389,5900,8080,8443 --min-hostgroup 255 --min-rtt-timeout 1ms --max-rtt-timeout 70ms --max-retries 0 --max-scan-delay 0 --min-rate 2000 -oA <customer-#> -vvv --open -iL <IPLIST>
```

But after 2 minutes of scanning with this I noticed a few things:

  * Parallel scans weren’t taking place
  * Timing and performance aspects don’t work
  * Nmap cant perform --defeat-rst-ratelimit with full TCP Connect scans –sT    
  * It would wait the full 1500ms for a port to time-out (Rmbr the Proxy config)

A ton of these problems stem from the fact that we are using a SOCKS proxy, and the nature of the connect scan. When you scan from your Kali system you perform the following Nmap -> Proxy-chains -> SOCKS (Jump Box) -> Target System. I discovered that even though you have all the timing settings setup Nmap is going to interpret that the Half Open connection with the SOCKS proxy as open and will let it hang till ProxyChains literally kills the connection due to a (TCP-Read-Timeout). At first I thought I could solve this issue using the "--host-timeout 80ms" flag. But I discovered once that host-timeout was reached it will completely kill the connection (NMAP ADD IN A PORT TIMEOUT!).

Instantly I knew I had to build out a quick script to help reduce the time it would take get through this scan. So I quickly hacked together a multi processed Python script to launch separate Proxychain / Nmap instances. This script sped up the time dramatically, and incase one process was hung the others could continue on rather than coming to a dead stop. I’m sure this could be done way better, but I thought I could atleast post what I did on the fly to speed up my scans. Also my Jump box couldn't handle (Popen and shell=True) for the sub process call, It was going a bit too fast. Right now I used call_check and just escaped that returned data for the moment. In the coming weeks I do plan to clean up the code a bit. All in all it took some time to trouble shoot the problem and thought I would share.

The script can be found here: [https://github.com/killswitch-GUI/PenTesting-Scripts/blob/master/Proxychains-Nmap.py](https://github.com/killswitch-GUI/PenTesting-Scripts/blob/master/Proxychains-Nmap.py)
```
    import multiprocessing
    import argparse
    import Queue
    import threading
    import os
    import sys
    import subprocess
    from random import randint




    
    def cli_parser():
     parser = argparse.ArgumentParser(add_help=False, description='''This script Simply routes your nmap scan in a "sort-of" fast way 
     through a ProcyChain that has been setup.
     \n\t(1) You will find out that when routing nmap through a Proxychain connection that Timing performace is out the window.
     \n\t(2) This is do to the nature of a SOCKS proxy and SYN->SYN/ACK connection is already established in NMAPS Eyes.
     \n\t(3) It out puts random (#) of .gnmap file for each IP for parsing. (MAKE A FOLDER) :)
     ''')
     parser.add_argument("-i", metavar="iplist.txt", help="Set Ip List of IPs Delimited by line")
     parser.add_argument('-h', '-?', '--h', '-help', '--help', action="store_true", help=argparse.SUPPRESS)
     args = parser.parse_args()
     if args.h: 
     parser.print_help()
     sys.exit()
     if not args.i:
     print "[!] I need a list IP's!"
     sys.exit()
     return args.i




    
    def Execution(Task_queue):
     while True:
     Ip = Task_queue.get()
     # If the queue is emepty exit this proc
     # Setup a simple output in the folder, For gnmap Parser
     IpName = str(Ip).replace('.',"-") + str(".gnmap")
     if Ip is None:
     break
     try:
     print "[*] On Ip: " + Ip
     test = subprocess.check_output(["proxychains", "nmap", "-Pn", "-n", "-sT", "--max-scan-delay", "0", "-p111,445,139,21-23,80,443", "-oG", IpName, "--open", Ip])
     test = ""
     except:
     pass




    
    def TaskSelector(Task_queue, verbose=False):
     total_proc = int(8)
     for i in xrange(total_proc):
     Task_queue.put(None)
     procs = []
     for thread in range(total_proc):
     procs.append(multiprocessing.Process(target=Execution, args=(Task_queue,)))
     for p in procs:
     p.daemon = True
     p.start()
     for p in procs:
     p.join()
     Task_queue.close()




    
    def Ip_List(Task_queue, cli_IpList):
     items = []
     cli_IpList = str(cli_IpList)
     try:
     with open(cli_IpList, "r") as myfile:
     lines = myfile.readlines()
     for line in lines:
     line = line.rstrip('\n')
     items.append(line)
     for item in items:
     Task_queue.put(item)
     return Task_queue
     except Exception as e:
     print "[!] Please check your Ip List: " + str(e)
     sys.exit(0)




    
    def main():
     cli_IpList = cli_parser()
     Task_queue = multiprocessing.Queue()
     Task_queue = Ip_List(Task_queue, cli_IpList)
     TaskSelector(Task_queue)




    
    if __name__ == "__main__":
     try: 
     main()
     except KeyboardInterrupt:
     print 'Interrupted'
     try:
     sys.exit(0)
     except SystemExit:
     os._exit(0)
```