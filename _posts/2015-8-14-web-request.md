---
layout: post
title: 一次网页请求的过程
---

我们要在学校上网时一次网页请求的过程是什么呢?

首先将电脑用一条以太网网线连到以太网交换机, 这个交换机连到学校的路由器,该路由器再连到ISP(Internet Service Provider), 就是电信等公司, 电信会提供DNS域名解析服务.

当电脑刚刚连上网络的时候, 需要运行DHCP协议从本地的DHCP服务器来得到一个IP地址。

1. 电脑创建一个DHCP request message, 放到一个UDP段里面, 这个段还包含目标端口(DHCP server)和来源端口(DHCP client). 这个UDP段会放到一个IP数据包里面, 这个包的目标IP地址是广播地址(255.255.255.255), 来源地址是0.0.0.0.

2. IP数据包放在一个Ethernet frame里, 这个以太网帧的目标地址是FF:FF:FF:FF:FF:FF所以该帧会被广播到所有连接在该交换机的设备,该帧的来源MAC地址是电脑的物理地址.

3. 当DHCP服务器收到这个IP数据包后,就会分配一个IP地址. 创建一个包含该IP地址和DNS地址的DHCP ACK message, 发回到我们的电脑.

有了IP地址后就可以上网了。在浏览器上输入URL www.google.com, 浏览器会首先创建一个TCP socket, 然后通过TCP socket发送HTTP请求。 为了创建TCP socket我们首先要知道 www.google.com的IP地址， 该地址由DNS协议得到。

4. 操作系统创建了一个DNS query message, 放进一个目标端口为53（DNS服务器）的UDP段里面， 这个UDP段放进以DNS server IP的为目标的IP数据包里。

5. 包含该IP数据包的Ethernet frame会发往学校的网关路由器。尽管我们的电脑知道学校网关的IP地址，但是它不知道网关路由器的MAC地址， 这个时候就需要用到 ARP 协议。

6. 电脑创建一个包含网关IP地址的ARP query message， 把它放进目标地址为广播地址FF:FF:FF:FF:FF:FF的以太网帧里面。 交换机会把这个帧发送到所有连接的设备，包括网关路由器。

7. 网关路由器发现这个DNS query message的目标IP是自己，则创建一个ARP reply, 把自己的MAC地址发送回去。
操作系统得到网关的MAC地址后， 就可以发送DNS query了。

8. 网关路由器拿到DNS query, 查看它的目标地址，根据forwading table决定传递到哪一个路由器去。IP数据报被放进链接层的帧里面。

9. 拿到该链接层帧的路由器再根据它的forwading table转发，该forwarding table由Internet's intera-domain protocal(如 RIP, OSPF, IS-IS, BGP)维护。

10. 最后DNS server查询DNS query里的网址，得到对应的IP， 发送回我们的电脑。

11. 得到www.google.com的IP地址后，我们就可以创建TCP socket了。 首先TCP要进行三次握手连接到80端口， 然后发送HTTP GET。

12. Google的服务器读取HTTP GET讯息，返回包含网页内容的HTTP response. 我们的电脑就可以展示其中的内容了。

