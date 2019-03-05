---
layout: post
title: "A journey through the network - Hands on"
categories: [theory, network]
tags: [theory, network, iso/osi, tcp/ip, saga]
---

### A journey through the network - Hands on (Part 1)
A long long time ago ([*I can still remember*](https://www.youtube.com/watch?v=uxYpS_bVdhk) as Madonna sings) I started to wrote some posts about the network. For those who missed the previous posts, [the introduction](https://made2591.github.io/posts/network-layers-0), [the physycal layer](https://made2591.github.io/posts/network-layers-1) and [the datalink layer](https://made2591.github.io/posts/network-layers-2). For the previous post I had to go into details about how some parts of the physical layer work but, by going forward with the layers, concepts belonging to separate historical standards - OSI and IP - will intertwine and this entails some troubles from a _logical_ point of view.
As a main source I use [Computer Networks](https://www.amazon.it/gp/product/9332518742/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1) and [TCP/IP Illustrated](https://www.amazon.it/gp/product/9332535957/ref=oh_aui_detailpage_o02_s00?ie=UTF8&psc=1). In this article, I will continue from there by going more practical with some hands-on.

### Network stack: layers recap
As a recapp, this is a summary table with some mapping between what happens and to which levels things happening belongs to, with some example.

| OSI Ref. Layer No. | OSI Layer Equivalent               | TCP/IP Layer     | TCP/IP Protocol Examples                                                         |
| ------------------ | ---------------------------------- | ---------------- | -------------------------------------------------------------------------------- |
| 5,6,7              | Application, session, presentation | Application      | NFS, NIS, DNS, LDAP, telnet, ftp, rlogin, rsh, rcp, RIP, RDISC, SNMP, and others |
| 4                  | Transport                          | Transport        | TCP, UDP, SCTP                                                                   |
| 3                  | Network                            | Internet         | IPv4, IPv6, ARP, ICMP                                                            |
| 2                  | Data link                          | Data link        | PPP, IEEE 802.2                                                                  |
| 1                  | Physical                           | Physical network | Ethernet (IEEE 802.3), Token Ring, RS-232, FDDI, and others                      |

I accidentally started counting from 0, but in the end with respect to this table we are around level three: everyone dealed with the question:

> What happen when you digit the URL `http://www.google.com` in your broswer?

Well, since I started moving bottom up for the network journey, we cannot answer correctly without knowing a little bit more in details network commands you can find in almost every operating systems since the beginning of time. Because - as one of the mentor I had used (and use) to say - *in the beginning, but still today, it's all about network*

### Agenda
In this post we will see at high levels how you can deal with network layer by layer and the tools you can natively use to debug / study / understand networking foundamentals. The commands are almost in order of layers and I will go through some experiments to test their most important features.

| Command Name | Description                                                                                                                              | Reference OSI Layer |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------- |
| `ping`       | tool that uses the `ICMP` protocol's mandatory `ECHO\_REQUEST` datagram to elicit an `ICMP ECHO\_RESPONSE` from a host or gateway.       | 3                   |
| `iptables`   | tool to set up, maintain, and inspect the tables of IP packet filter rules in the os kernel.                                             | 3                   |
| `netcat`     | (also nc) simple Unix utility which reads and writes data across network connections, using `TCP` or `UDP` protocol.                     | 4                   |
| `curl`       | tool to transfer data from or to a server, using one of the supported protocols.                                                         | 4                   |
| `tcpdump`    | prints out a description of the contents of packets on a network interface that match the boolean expression.                            | 4                   |
| `tshark`     | network protocol analyzer.                                                                                                               | 4                   |
| `dig`        | (domain information groper) flexible tool for interrogating `DNS` name servers.                                                          | 5-7                 |
| `openssl`    | cryptography toolkit implementing the *Transport Layer Security* (`TLS v1`) network protocol, as well as related cryptography standards. | 6                   |
| `nmap`       | (network mapper) is an open source tool for network exploration and security auditing.                                                   | 3-7                 |

Actually, there are many other commands to analyze: I just put some of them in the list to cover the most common scenario. I let's start from `ping`.

#### ping
The most probably well known by eveyone `ping` it's actually a trivial command: it simply checks if your system can *reach* a host. To be more precise, it actually also provide some feedback regarding the *latency*. What does it mean reach? Well, a system can reach another if can estabilish a comunication with it throught network connectivity. There's no other way to think about reachability at this moment so - remember that we are at 3rd layer, there's almost nothing under us.
What about the latency? Let's give a definition:

##### About latency and simultaneity communications
The so called `latency` is the delay from input into a system to desired outcome. The term is understood slightly differently in various contexts and latency issues also vary from one system to another. Latency greatly affects how usable and enjoyable electronic and mechanical devices as well as communications are. The latency in communication is demonstrated in live transmissions from various points on the earth as the communication *hops* (a word who dealed with network should know) between a ground transmitter and a satellite and from a satellite to a receiver each take time. People connecting from distances to these live events can be seen to have to wait for responses. This latency is the wait time introduced by the signal travelling the geographical distance as well as over the various pieces of communications equipment. Even fiber optics are limited by more than just the speed of light, as the refractive index of the cable and all repeaters or amplifiers along their length introduce delays.

It could seem trivial but it is not at all: in fact, latency is actually something related to strong concept about how the Universe itself works. Even at infinitesimal distance, two systems cannot speak directly one to each other because - as Einstein stated as second principle in his Theory of Relativity (and it is demonstrated)[^simul]:

> The concept of simultaneity doesn't exist.

##### How to understand ping
To do my experiment, I created a EC2 instance on AWS but you can use any other system you have access to (at admin level, of course). I ensured during creation to assign to my instance a public IP and a security group with no rules. Then open a shell, do

{% highlight sh %}
ping ec2-52-48-17-240.eu-west-1.compute.amazonaws.com
{% endhighlight %}

and... 100% packet lost. Why? Well, the security group doesn't allow traffic. Thus, even sending a `ICMP` protocol's mandatory `ECHO\_REQUEST` datagram will be reject by default. So let's add a rule to that and see what happens. The needed EC2's associated security group inbound rule has to follow the schema:

- **Type**: Custom ICMP rule
- **Protocol**: Echo Request
- **Port**: N/A (could not be changed)
- **Source**: your choice (I would select My IP to be able to ping from the machine you are testing on)

So first of all: port range is not editable. Of course, we are at layer 3. There are no ports. I said once again:

> At layer 3 there are no ports.

TCP or UDP ports are defined in either layer 4 of the OSI model (or layer 3 of the TCP/IP model), both defined as the Transport layer. OSI layer 5 Session layer uses the ports defined in layer 4 to create sockets and sessions between communicating devices/programs/etc.

Since security rules are applied instantly, if you know do

{% highlight sh %}
ping ec2-52-48-17-240.eu-west-1.compute.amazonaws.com
{% endhighlight %}

You should see packets transmitted and respective times required. So let's DESTROY the myth that *if a host doesn't reply to ping, that means it's down* because it's false. Actually, many server doesn't respond to ping.

Since this command is quite basic, I will no go through details about parameters: you can specify custom number of counts with `-c` (that means, stop after sending (and receiving) count ECHO_RESPONSE packets), or `-i` wait seconds between sending each packet, etc. You most probably have to know only about ping.

#### iptables
The most probably second most known by eveyone `iptables` it's instead quite complex to describe: iptables lets you create rules to match network packets and accept, drop, modify, etc... them. It's actually used as a `Firewall` and `NAT`. What are these strangers?

##### Firewall
A firewall is a software that monitor the network traffic. A firewall has a set of rules which are applied to each packet. The rules decide if a packet can pass, or whether it is discarded. Usually a firewall is placed between a network that is trusted, and one that is less trusted. When a large network needs to be protected, the firewall software often runs on a computer that does nothing else.

##### NAT
Network Address Translation (NAT) is a method of remapping one IP address space into another by modifying network address information in the IP header of packets while they are in transit across a traffic routing device. The technique was originally used as a shortcut to avoid the need to readdress every host when a network was moved. It has become a popular and essential tool in conserving global address space in the face of IPv4 address exhaustion. One Internet-routable IP address of a NAT gateway can be used for an entire private network. Actually, we all connect throught internet using a NAT. If you have more than one device inside you house, they can connect to internet even if your ISP provide to you only one Public IP. Now the question is: how is that possible?

In a typical configuration, a local network uses one of the designated private IP address subnets (RFC 1918) - I have mine in 172.16.0.0, many routers sold by default are in the range 192.168.0.0. A router on that network has a private address in that address space (the 192.168.1.1). The router is also connected to the Internet with a public address assigned by an Internet Service Provider. As traffic passes from the local network to the Internet, the source address in each packet is translated on the fly from a private address to the public address. The router tracks basic data about each active connection (particularly the destination address and port). When a reply returns to the router, it uses the connection tracking data it stored during the outbound phase to determine the private address on the internal network to which to forward the reply.

All IP packets have a source IP address and a destination IP address. Typically packets passing from the private network to the public network will have their source address modified, while packets passing from the public network back to the private network will have their destination address modified. To avoid ambiguity in how replies are translated, further modifications to the packets are required. The vast bulk of Internet traffic uses Transmission Control Protocol (TCP) or User Datagram Protocol (UDP). For these protocols the port numbers are changed so that the combination of IP address and port information on the returned packet can be unambiguously mapped to the corresponding private network destination.

#### How to understand iptables
To touch with your hands how iptable works we are gonna simualte a NAT scenario locally, using our old friend Docker. This is the simplest way I found to test iptables and how it operates locally without breaking things

##### The scenario
The scenario is the following: you have an application that exposes a simple web server that outputs the IP addresses of the source and destination. This application is not reachable from outside world - in the scenario, your host - but we have a proxy implemented with a few iptables rules that forward all the traffic to our application and, from it, back to the request. This proxy can dialogue both internally with our web server and externally with outside world - again, your host.

You can get all you need from [this gist](https://gist.github.com/made2591/a53db169ba6f52c24f20d76714e2e37b). Otherwise, just copy and paste the following content in a `docker-compose.yml` file and do a docker compose up

{% highlight yaml %}
version: "3"
services:
  proxy:
    container_name: proxy
    restart: always
    image: debian:jessie
    privileged: true
    ports:
      - "80:80"
    networks:
      - frontend
      - backend
    tty: true

  server:
    container_name: server
    restart: always
    image: yeasy/simple-web:latest
    privileged: true
    networks:
      - backend
    tty: true

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
{% endhighlight %}

This docker will spawn two container: the first is the [simple-web](https://hub.docker.com/r/yeasy/simple-web/) described before, the second is a simple debian jessie image. What you should is that the server is attached only to a network interface called `backend`. The proxy, instead, is attached to both `backend` and `frontend` network interface. This is a scenario that somehow simulate a private and a public subnet.

Actually the debian:jessie doesn't act like a real proxy once is up. It only expose the port 80 to host. The first thing to test once you do `docker-compose up -d` is to test that only the proxy is reachable. How? The fact is even if the proxy doesn't expose anything, since the port 80 is mapped you should receive an empty reply if you curl your localhost over 80. Let's do a curl over the proxy:

{% highlight sh %}
curl localhost 80
{% endhighlight %}

Something like `Empty reply from server` should appear. Not convinced yet? Before going ahead, let's stop the proxy for a moment and repeat our test again. From the same shell inside the host execute:

{% highlight sh %}
docker-compose stop proxy
curl localhost 80
{% endhighlight %}

You should now receive a `Failed to connect to localhost port 80` that confirm that your proxy is not reachable since it's down. Let's start it again with

{% highlight sh %}
docker-compose start proxy
{% endhighlight %}

Now, we have to install iptables inside the proxy docker and setup the rules. Our mission is to let a `curl` from host get the response from the `server` without exposing directly a port - just to test capabilities of iptables. From host, get the container id of the proxy (first column is the id, last is the container name) by doing a `docker ps -a`, then enter inside the

{% highlight sh %}
docker exec -it <YOUR_PROXY_CONTAINER_ID> /bin/bash -c "apt-get update && apt-get install -y iptables"
{% endhighlight %}

This will install iptables inside the container. You can even install things from inside, because actually it is needed for the next step. Before going ahead, let's see the container ip address to write our rules. From host do:

{% highlight sh %}
docker inspect <YOUR_SERVER_CONTAINER_ID>
{% endhighlight %}

At the end you should see the huge JSON describing the container, including the "Networks" interface and respective IP assigned to server. Keep it somewhere, you will need it in a while. From the host shell, enter the proxy container

{% highlight sh %}
docker exec -it <YOUR_PROXY_CONTAINER_ID> /bin/bash
{% endhighlight %}

and then setup the iptables rules:

{% highlight sh %}
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -F
iptables -t nat -F
iptables -X
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination <YOUR_SERVER_CONTAINER_IP>:80
iptables -t nat -A POSTROUTING -j MASQUERADE
{% endhighlight %}

The first line enables IP forwarding in the proxy container by telling that when it receives a packet not destined directly to it, to forward it to the destination (or, better, the next hop) instead of dropping it. This let the proxy works as a gateway to the specific destination.

`-F` flush the selected chain (all the chains in the table if none is given). This is equivalent to deleting all the rules one by one. Of course, `-t nat -F` flush the rule of nat (just to avoid noises in this experiment). `-X` is again a cleaning operation: it deletes the optional user-defined chain specified. If no argument is given, it will attempt to delete every non-builtin chain in the table. Before going ahead, remember that iptables work by - trivial, maybe - tables. You should know that by default there are five tables in the kernel that contain sets of rules. Every tables has different chains that define `when` - and `how` - iptables acts over packets that match the rules.

##### Tables
- **raw** is used only for configuring packets so that they are exempt from connection tracking;
- **filter** is the default table, and is where all the actions typically associated with a firewall take place;
- **nat** is used for network address translation (e.g. port forwarding);
- **mangle** is used for specialized packet alterations;
- **security** is used for Mandatory Access Control networking rules (e.g. SELinux);

The flow is almost like this:

{% highlight sh %}
                               XXXXXXXXXXXXXXXXXX
                             XXX     Network    XXX
                               XXXXXXXXXXXXXXXXXX
                                       +
                                       |
                                       v
 +-------------+              +------------------+
 |table: filter| <---+        | table: nat       |
 |chain: INPUT |     |        | chain: PREROUTING|
 +-----+-------+     |        +--------+---------+
       |             |                 |
       v             |                 v
 [local process]     |           ****************          +--------------+
       |             +---------+ Routing decision +------> |table: filter |
       v                         ****************          |chain: FORWARD|
****************                                           +------+-------+
Routing decision                                                  |
****************                                                  |
       |                                                          |
       v                        ****************                  |
+-------------+       +------>  Routing decision  <---------------+
|table: nat   |       |         ****************
|chain: OUTPUT|       |               +
+-----+-------+       |               |
      |               |               v
      v               |      +-------------------+
+--------------+      |      | table: nat        |
|table: filter | +----+      | chain: POSTROUTING|
|chain: OUTPUT |             +--------+----------+
+--------------+                      |
                                      v
                               XXXXXXXXXXXXXXXXXX
                             XXX    Network     XXX
                               XXXXXXXXXXXXXXXXXX
{% endhighlight %}

Thus, this rule will act over the PREROUTING chain, on packet received over TCP protocol on port 80. The destination NAT is specified using `-j DNAT '` and `--to-destination` option allows you to specify an address or range of IP addresses and optionally one or a range of ports (UDP and TCP protocols only). In our case, our server reachable by the proxy and not from outside.

{% highlight sh %}
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination <YOUR_SERVER_CONTAINER_IP>:80
{% endhighlight %}

This will change the destination address of web traffic in <YOUR_SERVER_CONTAINER_IP>:80. Unfortunately, it is not enought because: the server container will respond to the query but... the proxy would not be able to redirect this response back to the original requester. There is a specialized case of source NAT called masquerading: it should be used only if the IP addresses are assigned dynamically, such as in the case of external connection: in the case of static IP addresses instead use the already cited SNAT.

{% highlight sh %}
iptables -t nat -A POSTROUTING -j MASQUERADE
{% endhighlight %}

If you open a browser to [http://localhost](http://localhost) you should see a webpage showing the requests counted. Ok this is was just to show a simple real case of how iptables works. There are so many things more but let's go ahead with the stack.

#### netcat
Netcat it's really nice because it lets you create `TCP` and `UPD` packets from shell. To start a server you can simply run the command

{% highlight sh %}
nc -l -p 9010 127.0.0.1
{% endhighlight %}

If you run this in your machine everything you will send to 127.0.0.1:9010 will be printed out: let's try! In another, run the command

{% highlight sh %}
echo "hello world" | nc 127.0.0.1 9010
{% endhighlight %}

You will see hello world in the first shell. If two host are on the same network you can use netcat to send a file from a system to another by simply replace the 127.0.0.1 with the receiver local IP and by redirecting the ouput (the receiver) to file with `> myfile`. You can try easily how it works between two containers debian container sharing the same network interface - you can modify easily the docker-compose or try directly with two laptop inside your local network.

#### curl
The well known `curl` lets you make http requests - in a more advanced way with respect to netcat. Let's modify our docker-compose by adding direct expose to the service server (add port expose as for the proxy) and run it alone by doing `docker-compose up server`. Got into a shell and run `curl 127.0.0.1 80` to see the response you first see on browser.

Some common and useful parameters are:

- `-i`: it shows the response headers: let's try it with our server (from docker compose). Got into a shell and run `curl -i 127.0.0.1 80`. In addiction to response, you will also see the response HEADER, something like:

{% highlight sh %}
HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/2.7.14
Date: Tue, 05 Mar 2019 21:12:33 GMT
Content-type: text/html
Content-Length: 351
Last-Modified: Tue, 05 Mar 2019 21:12:33 GMT
{% endhighlight %}

- `-I`: it will show only the response headers shown below by doing a HEAD request. This is very usefull if you want for instance to try availability of object somewhere without *paying the costs* of a real GET request. Let's say you have an object in S3 and you want to actively test if it is reachable or not. HEAD request are free of charge and if you get a status code 200, you know that the object you are looking for is reachable (or at least you know you have to look into it).
- `-X POST/GET/DELETE/PUT/ETC` will send the respective HTTP request. It is often use to test api together with standard GET requests (usually not explicited) with something like `curl --header "Content-Type: application/json" www.google.com`.
- `-L` follow the redirects (`3**` status) - useful when you are working with services authentication (it happened to me I add to test external 3 step auth with OpenID, etc).
- `-v` it's kind of verbose mode; you can of course combine together these parameters together;
- `-k` don't verify ssl certificates - usefull for some local development or if you are below not trusted corporate proxy (believe me or not it happened to me);
- `-k` don't verify ssl certificates - usefull for some local development or if you are below not trusted corporate proxy (believe me or not it happened to me);
- `--data` lets you send data. Unfortunately the simple server of iptables example doesn't work over POST request but ehy! there's the magical netcat that can do it for us. Open a shell and run `nc -l -p 9010 127.0.0.1`, then from another shell try running `curl -X POST http://127.0.0.1:9010 --data '{"hello": "world"}'` and you will se the JSON in output in netcat shell. If you run `curl -X POST http://127.0.0.1:9010 --data @helloworld.json` with `helloworld.json` a JSON a file with the JSON string `{"hello": "world"}` inside, it will work the same way by reading directly from the file. I used already this tecnique in the past for using Mapping APIs from ElasticSearch to index the files in my NAS (have a look [here](https://madeddu.xyz/posts/elasticnas/)).

### Conclusion
Since this blog post it's already long enough, I think I will split it before going ahead so... play around with the gist and some experiments to discover even more parameters and if you have some insights about usefull tips, don't hesitate put them in the comments!

I will continue the experiments from `tcpdump` (`wireshark` for millenials XD) in the next blog post. Stay tuned!

Thank you everybody for reading!

[^simul]: If you want to know more about, follow [this link](https://en.wikipedia.org/wiki/Relativity_of_simultaneity).