---
layout: post
title: 理解TCP/IP
category: network_programming
tags: TCP UDP IP
---

最近闲来无事，端起两本网络编程的神书《TCP/IP Illustrated, Vol. 1: The Protocols》和《Unix Network Programming, Vol. 1: Networking API》开始看，一些以前不清楚的知识点得到了进一步理解。

我认为网络编程学习可以分三个阶段：

1. 熟悉TCP/IP模型及相关协议，比如连接的建立和终止、超时和重传、滑动窗口和拥塞控制……
2. 掌握Socket API及其背后隐藏的细节
3. 对服务器进行性能调优

以我目前的功力，应该暂时处于第1阶段。Anyway, this is just a beginning!

##TCP/IP模型##
**[TCP/IP模型](http://en.wikipedia.org/wiki/TCP/IP)**(TCP/IP model)是一个四层协议模型，自下而上依次为**[链路层](http://en.wikipedia.org/wiki/Link_layer)**→**[网络层](http://en.wikipedia.org/wiki/Internet_layer)**→**[传输层](http://en.wikipedia.org/wiki/Transport_layer)**→**[应用层](http://en.wikipedia.org/wiki/Application_layer)**。但是**TCP/IP模型与TCP和IP协议并无必然联系**，并不一定要使用TCP和IP协议，比如网络层可以是ICMP协议，传输层可以是UDP协议，之所以这样叫是因为TCP和IP协议是最早被定义的，并且是最重要的两个协议。

**[协议族](http://en.wikipedia.org/wiki/Internet_protocol_suite)**(Protocol suite)是一组不同层次上的多个协议的组合，比如TCP/IP。一个**[协议栈](http://en.wikipedia.org/wiki/Protocol_stack)**(Protocol stack)是对协议族的具体实现，比如HTTP + TCP + IP + Ethernet protocol，POP3 + TCP + IP + Ethernet protocol，UDP + IP + Ethernet protocol等等。或者说协议族是对协议的定义，协议栈是它们的软件实现。

![Prototols in relation to TCP/IP model](/images/protocols-in-relation-to-tcp-ip-model.png)

通常，应用程序(用户进程)构建在应用层，只处理应用程序的细节，不关心数据在网络中的传输活动。而下三层一般在操作系统的内核中执行，对应用程序一无所知，但它们要处理所有的通信细节。

当应用程序的数据进入协议栈时，每一层会对收到的数据增加一些头部信息(有时还要增加尾部信息)，最后被当作一串比特流送入网络。

* TCP传给IP的数据单元称作**TCP段**(TCP segment)，UDP传给IP的数据单元称作**UDP数据报**(UDP datagram)
* IP和网络接口层之间传送的数据单元是**分组**(packet)，它既可以是一个**IP数据报**(IP datagram)，也可以是IP数据报的一个**分片**(fragment)
* 在以太网中传输的比特流称作**幀**(frame)

![Encapsulation of data as it goes down the protocol stack](/images/protocol-stack-data-encapsulation.gif)
