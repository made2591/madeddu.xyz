---
title: "A journey through the network - Hands on"
tags: [theory, network, iso/osi, tcp/ip, saga]
---

### A journey through the network - Hands on
A long long time ago ([*I can still remember*](https://www.youtube.com/watch?v=uxYpS_bVdhk) as Madonna sings) I started to wrote some posts about the network. For those who missed the previous posts, [the introduction](https://madeddu.xyz/posts/network-layers-0), [the physycal layer](https://madeddu.xyz/posts/network-layers-1) and [the datalink layer](https://madeddu.xyz/posts/network-layers-2). For the previous post I had to go into details about how some parts of the physical layer work but, by going forward with the layers, concepts belonging to separate historical standards - OSI and IP - will intertwine and this entails some troubles from a _logical_ point of view.
As a main source I use [Computer Networks](https://www.amazon.it/gp/product/9332518742/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1) and [TCP/IP Illustrated](https://www.amazon.it/gp/product/9332535957/ref=oh_aui_detailpage_o02_s00?ie=UTF8&psc=1). In this article, I will continue from there by going into detail about first comunication example

### First recap
As a recapp, this is a summary table with some mapping between what happens and to which levels things happening belongs to, with some example.

| OSI Ref. Layer No. | OSI Layer Equivalent               | TCP/IP Layer     | TCP/IP Protocol Examples                                                         |
|--------------------|------------------------------------|------------------|----------------------------------------------------------------------------------|
| 5,6,7              | Application, session, presentation | Application      | NFS, NIS, DNS, LDAP, telnet, ftp, rlogin, rsh, rcp, RIP, RDISC, SNMP, and others |
| 4                  | Transport                          | Transport        | TCP, UDP, SCTP                                                                   |
| 3                  | Network                            | Internet         | IPv4, IPv6, ARP, ICMP                                                            |
| 2                  | Data link                          | Data link        | PPP, IEEE 802.2                                                                  |
| 1                  | Physical                           | Physical network | Ethernet (IEEE 802.3), Token Ring, RS-232, FDDI, and others                      |

I accidentally started counting from 0, but in the end with respect to this table we are around level three: everyone dealed with the question

> What happen when you digit the URL `http://www.google.com` in your broswer?

Well, since I started moving bottom up for the network journey, we cannot answer correctly to this questions if we are not confindent first in answering its small sister, that is:

> What happen when you digit `curl -X GET http://`

### AR

The application, in this case a Web browser, calls a special function to parse
the URL to see if it contains a host name. Here it does not, so the application
uses the 32-bit IPv4 address 10.0.0.1.
1. The application asks the TCP protocol to establish a connection with 10.0.0.1.
2. TCP attempts to send a connection request segment to the remote host by
sending an IPv4 datagram to 10.0.0.1. (We shall see the details of how this is
done in Chapter 15.)
4. Because we are assuming that the address 10.0.0.1 is using the same network prefix as our sending host, the datagram can be sent directly to that
address without going through a router.
5. Assuming that Ethernet-compatible addressing is being used on the IPv4
subnet, the sending host must convert the 32-bit IPv4 destination address
into a 48-bit Ethernet-style address. Using the terminology from [RFC0826],
a translation is required from the logical Internet address to its corresponding physical hardware address. This is the function of ARP. ARP works in
its normal form only for broadcast networks, where the link layer is able to
deliver a single message to all attached network devices. This is an important requirement imposed by the operation of ARP. On non-broadcast networks (sometimes called NBMA for non-broadcast multiple access), other,
more complex mapping protocols may be required [RFC2332].
6. ARP sends an Ethernet frame called an ARP request to every host on the
shared link-layer segment. This is called a link-layer broadcast. We show the
broadcast domain in Figure 4-1 with a crosshatched box. The ARP request
contains the IPv4 address of the destination host (10.0.0.1) and seeks an
answer to the following question: “If you are configured with IPv4 address
10.0.0.1 as one of your own, please respond to me with your MAC address.”

##### CURLOPT_ERRORBUFFER, CURLOPT_STDERR, CURLOPT_HEADERFUNCTION, CURLOPT_HEADERDATA
Ok you got the idea about how that code is implemented.

### Let's implement this function



You typically type an web address (Uniform resource locator) in a Web browser. Web browser uses something called  Hypertext Transfer Protocol which is an Application layer protocol.

