+++
author = "Alexander Rymdeko-Harvey"
tags = []
title = "OS X Packet Capture & Empire"
description = "Libpcap.dylib & Python"
date = "2017-02-12T21:39:41-05:00"
+++

Recently I have been preparing for some additional support for Empire and the Python agent. With that comes some new modules cooking, finally getting to porting the Linux sniffer over to OS X. While some of you know it’s easy with raw sockets on the Linux side. OS X does not natively support this, and you often see question that relate to how to get this to work on the OS X side. 

## Empire 
Empire will be seeing a major release here in the coming months, moving from a single-track windows agent to a platform for OS X, Linux and Windows. Making it extremely agile. 

TL;DR: The following video is the new OS X sniffer module in action

<iframe src="https://player.vimeo.com/video/203695405?autoplay=1&loop=1&color=ffffff" width="640" height="400" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/203695405">Empire OS X Sniffer Module</a> from <a href="https://vimeo.com/user52367620">Alexander Rymdeko-Harvey</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

## Native Support
Native code capability is crucial due the constraints we have, while we do have Python package imports now from memory there is always limitations.  The Empire dev team often focuses on making all modules as cross capable and native. 

## Extend Python with cTypes and Dylib!
One of the major benefits to python is its low-level support, and with cTypes we have those bindings! We can call the standard C library and much more dynamically. Let’s take a look at some the library’s shipped with OS X:

```
$ ls /usr/lib/ | grep “lib”
...
libpq.5.dylib
libpq.dylib
libprequelite.dylib
libproc.dylib
libpthread.dylib
libpython.dylib
libpython2.6.dylib
libpython2.7.dylib
libquit.dylib
libreadline.dylib
libresolv.9.dylib
libresolv.dylib
librpcsvc.dylib
...
```

Next we can inspect our .dyblib using `nm`  and dumping the export functions in the dynamic library: 

```
$ nm -gU /lib/usr/libpcap.dylib 
...
0000000000018c93 T _pcap_dump
000000000001b28c T _pcap_ng_get_simple_packet_fields
000000000001e15a T _pcap_ng_init_section_info
0000000000014088 T _pcap_ng_offline_read
0000000000013e90 T _pcap_ng_open_offline
0000000000011a4c T _pcap_not_initialized
0000000000012f69 T _pcap_offline_filter
0000000000013f7c T _pcap_offline_read
0000000000011b1a T _pcap_oneshot
0000000000012ec6 T _pcap_open_dead
0000000000012e0c T _pcap_open_dead_with_tstamp_precision
00000000000122e3 T _pcap_open_live
0000000000013c13 T _pcap_open_offline
000000000001249b T _pcap_open_offline_common
...
0000000000011ed3 T _pcap_set_snaplen
0000000000011fc1 T _pcap_set_timeout
00000000000120f7 T _pcap_set_tstamp_precision
0000000000011fff T _pcap_set_tstamp_type
0000000000010c9f T _pcap_set_want_pktap
0000000000012c37 T _pcap_setdirection
0000000000012c29 T _pcap_setfilter
0000000000012b2f T _pcap_setnonblock
0000000000012b78 T _pcap_setnonblock_fd
000000000001ca05 T _pcap_setup_pktap_interface
```

Now comes the ability to extend Python with cTypes and C convention calling. according to Python docs: "cdll loads libraries which export functions using the standard cdecl calling convention" 

``` 
class ctypes.CDLL(name, mode=DEFAULT_MODE, handle=None, use_errno=False, use_last_error=False)

Instances of this class represent loaded shared libraries. Functions in these
libraries use the standard C calling convention, and are assumed to return int.
```
 
In the code I also setup the standard C lib for future use or other functions you may need such as `mmap`

```
OSX_PCAP_DYLIB = '/usr/lib/libpcap.A.dylib'
OSX_LIBC_DYLIB = '/usr/lib/libSystem.B.dylib'
...
libc = ctypes.CDLL(OSX_LIBC_DYLIB, use_errno=True)
...
pcap = ctypes.CDLL(OSX_PCAP_DYLIB, use_errno=True)
```

## libpcap - Standard C Program
using http://www.devdungeon.com/content/using-libpcap-c guide on libcap is a great starting point for most. Here is what is impended in Python:

```
#include <stdio.h>
#include <time.h>
#include <pcap.h>
#include <netinet/in.h>
#include <netinet/if_ether.h>

void print_packet_info(const u_char *packet, struct pcap_pkthdr packet_header);

int main(int argc, char *argv[]) {
    char *device;
    char error_buffer[PCAP_ERRBUF_SIZE];
    pcap_t *handle;
    const u_char *packet;
     struct pcap_pkthdr packet_header;
    int packet_count_limit = 1;
    int timeout_limit = 10000; /* In milliseconds */

    device = pcap_lookupdev(error_buffer);
    if (device == NULL) {
        printf("Error finding device: %s\n", error_buffer);
        return 1;
    }

    /* Open device for live capture */
    handle = pcap_open_live(
            device,
            BUFSIZ,
            packet_count_limit,
            timeout_limit,
            error_buffer
        );

     /* Attempt to capture one packet. If there is no network traffic
      and the timeout is reached, it will return NULL */
     packet = pcap_next(handle, &packet_header);
     if (packet == NULL) {
        printf("No packet found.\n");
        return 2;
    }

    /* Our function to output some info */
    print_packet_info(packet, packet_header);

    return 0;
}
```
## libpcap - Python Implementation
Follow along for the explanation and breakdown or skip to the end for code:

### Interface Device Selection
The next step is to obtain a handle to the interface of choice, as you will see libpcap follows a standard calling convention for setup. We will be using:

```
pcap_lookupdev(): finds the default device on which to capture

Returns a pointer  to a string giving the name of a
network device suitable for use with pcap_create() and pcap_activate(),
or with  pcap_open_live(),  and with pcap_lookupnet()
```

To set this up in Python we need to create a pcap_lookupdev() object and set the res type, this is because by default functions are assumed to return the C int type. 

```
# pcap is our pointer to the dylib function
pcap_lookupdev = pcap.pcap_lookupdev
# set the return type to a C char pointer (string)
pcap_lookupdev.restype = ctypes.c_char_p
# now get our device device to capture on
dev = pcap.pcap_lookupdev()
```

### Interface Device Info
We have an option to get some data on the device of choice. Using the pcap_lookupnet function we can get some debug data if necessary.

```
pcap_lookupnet(): find the IPv4 network number and netmask for a device  

int pcap_lookupnet(const char *device, bpf_u_int32 *netp, 
    bpf_u_int32 *maskp, char *errbuf);

Is used to determine the IPv4 network number and mask associated with the
network device. Both netp and maskp are bpf_u_int32 pointers
```

We will need to use the proper cTypes unsigned int for the return pointers. We can do this by using cTypes primitive C compatible data types.

```
net = ctypes.c_uint()
mask = ctypes.c_uint()
pcap.pcap_lookupnet(dev,ctypes.byref(net),ctypes.byref(mask),err_buf)
if DEBUG:
    print "* Device IP to bind: %s" % net
    print "* Device net mask: %s" % mask
```

### Packet Capture Handle
Once we have a interface device we need to obtain a capture handle to use with various libpcap capture types. For this we will use the pcap_open_live function.

```
pcap_open_live(): open a device for capturing  

pcap_t *pcap_open_live(const char *device, int snaplen,
        int promisc, int to_ms, char *errbuf);
Is used to obtain a packet capture handle to look at packets on the network.
```

We will need to provide the following to the function:

    device = string that specifies the network device to open
    snaplen = specifies the snapshot length to be set on the handle
    promisc = specifies if the interface is to be put into promiscuous mode
    to_ms = specifies the packet buffer timeout in milliseconds
    errbuf = if provided 

Once again we will need to setup the return type, the only difference here is that the return type is pcap_t * structure on success. To do this we will use the ctypes.POINTER() function, which we will provide the type as well:

    ctypes.POINTER(type)

    This factory function creates and returns a new ctypes pointer type.
    Pointer types are cached and reused internally, so calling this function
    repeatedly is cheap. type must be a ctypes type.
Now implemented in Python:
```
# static values 
PCAP_ERRBUF_SIZE = 256
PCAP_CAPTURE_COUNT = 1000
promisc = ctypes.c_int(1)
# setup the return pointer 
pcap_open_live = pcap.pcap_open_live
pcap_open_live.restype = ctypes.POINTER(ctypes.c_void_p)
# get pcap_handle pointer
pcap_handle = pcap.pcap_open_live(dev, 1024, promisc, timeout_limit, err_buf)
```

### Check Our Handle
As all good programing practices say we should check our handle, this will insure we have a handle/interface that can go into monitor mode. To do this we will use pcap_can_set_rfmon.

    pcap_can_set_rfmon(): checks whether monitor mode could be set on a capture
    handle when the handle is activated

    pcap_set_rfmon() returns 0 if monitor mode could not be set, 
    1 if monitor mode could be se

Impmentation:
```
pcap_can_set_rfmon = pcap.pcap_can_set_rfmon
pcap_can_set_rfmon.argtypes = [ctypes.c_void_p]
if (pcap_can_set_rfmon(pcap_handle) == 1):
    if DEBUG:
        print "* Can set interface in monitor mode"
```

### Setup Dump File
If you take a look at the raw socket sniffer in Empire https://github.com/killswitch-GUI/NIX-Sniffer-Examples/blob/master/linux-socket-sniffer.py#L40 it creates the pcap header and file header manually. Using the libpcap lib we can use pcap_dump_open() to handle this for us. Using the pcap_dump() function we can call this pointer for the packet handling.
```
pcap_dump_open(): open a file to which to write packets  

pcap_dumper_t *pcap_dump_open(pcap_t *p, const char *fname);
    is called to open a ``savefile'' for writing. fname specifies the name of
    the file to open. The file will have the same format as those used by
    tcpdump(1) and tcpslice(1). The name "-" is a synonym for stdout.
```
implementation: 
```
PCAP_FILENAME = 'test.pcap'
# setup pointer
pcap_dump_open = pcap.pcap_dump_open
pcap_dump_open.restype = ctypes.POINTER(ctypes.c_void_p)
# get pointer 
pcap_dumper_t = pcap.pcap_dump_open(pcap_handle,PCAP_FILENAME)
```

### Pcap Capture
All the prior functionality is setup; we will need to setup a loop for packet capture. We can do this a few ways within libpcap, the first is using pcap_next_ex() or pcap_loop(). Using pcap_loop requires a callback to be setup, which I was unable to do properly and have must of it commented out. So, if you know how to do that, some help would be great. For the mean time, we will use pcap_next_ex in a loop since its blocking.
```
pcap_next_ex():read the next packet from a pcap_t  

int pcap_next_ex(pcap_t *p, struct pcap_pkthdr **pkt_header,
    const u_char **pkt_data);

reads the next packet and returns a success/failure indication. If the packet
was read without problems, the pointer pointed to by the pkt_header argument
is set to point to the pcap_pkthdr struct for the packet, and the pointer
pointed to by the pkt_data argument is set to point to the data in the packet.
```
In this implementation we will also use custom cTypes structures, just like you would see in a C struc. Using this method we will setup a pcap_pkthdr() structure to populate, we than get a cTypes pointer to feed to pcap_next_ex().
```
class pcap_pkthdr(ctypes.Structure):
    _fields_ = [("tv_sec", ctypes.c_long), ("tv_usec", ctypes.c_long),
    ("caplen", ctypes.c_uint), ("len", ctypes.c_uint)]

pcap_pkthdr_p = ctypes.POINTER(pcap_pkthdr)()
packetdata = ctypes.POINTER(ctypes.c_ubyte*65536)()

c = 0
while True:
    if (pcap.pcap_next_ex(pcap_handle, ctypes.byref(pcap_pkthdr_p), ctypes.byref(packetdata)) == 1):
        # our output function pcap_dump()
        pcap.pcap_dump(pcap_dumper_t,pcap_pkthdr_p,packetdata)
        #pkthandler(pcap_pkthdr_p,packetdata)
        c += 1
    if c > PCAP_CAPTURE_COUNT:
        if DEBUG:
            print "* Max packet count reached!"
        break
```

****

> The full repo can be found here:

<div class="github-card" data-github="killswitch-gui/NIX-Sniffer-Examples" data-width="400" data-height="153" data-theme="default"></div>
<script src="//cdn.jsdelivr.net/github-cards/latest/widget.js"></script>

> The full script:

<script src="https://gist.github.com/killswitch-GUI/ce347f79f5cdb90bd8056100c90e9be2.js"></script>





