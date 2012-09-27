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
**[TCP/IP模型](http://en.wikipedia.org/wiki/TCP/IP)**(TCP/IP model)是一个四层协议模型，自下而上依次为**[链路层](http://en.wikipedia.org/wiki/Link_layer)**->**[网络层](http://en.wikipedia.org/wiki/Internet_layer)**->**[传输层](http://en.wikipedia.org/wiki/Transport_layer)**->**[应用层](http://en.wikipedia.org/wiki/Application_layer)**。但是**TCP/IP模型与TCP和IP协议并无必然联系**，并不一定要使用TCP和IP协议，比如网络层可以是ICMP协议，传输层可以是UDP协议，之所以这样叫是因为TCP和IP协议是最早被定义的，并且是最重要的两个协议。

**[协议族](http://en.wikipedia.org/wiki/Internet_protocol_suite)**(Protocol suite)是一组不同层次上的多个协议的组合，比如TCP/IP。一个**[协议栈](http://en.wikipedia.org/wiki/Protocol_stack)**(Protocol stack)是对协议族的具体实现，比如HTTP + TCP + IP + Ethernet protocol，POP3 + TCP + IP + Ethernet protocol，UDP + IP + Ethernet protocol等等。或者说协议族是对协议的定义，协议栈是它们的软件实现。

![Prototols in relation to TCP/IP model](/images/protocols-in-relation-to-tcp-ip-model.png)

通常，应用程序(用户进程)构建在应用层，只处理应用程序的细节，不关心数据在网络中的传输活动。而下三层一般在操作系统的内核中执行，对应用程序一无所知，但它们要处理所有的通信细节。

当应用程序的数据进入协议栈时，每一层会对收到的数据增加一些头部信息(有时还要增加尾部信息)，最后被当作一串比特流送入网络。

* TCP传给IP的数据单元称作**TCP段**(TCP segment)，UDP传给IP的数据单元称作**UDP数据报**(UDP datagram)
* IP和网络接口层之间传送的数据单元是**分组**(packet)，它既可以是一个**IP数据报**(IP datagram)，也可以是IP数据报的一个**分片**(fragment)
* 在以太网中传输的比特流称作**幀**(frame)

![Encapsulation of data as it goes down the protocol stack](/images/protocol-stack-data-encapsulation.gif)

##TCP(Transmission Control Protocol)##
**[TCP](http://en.wikipedia.org/wiki/Transmission_Control_Protocol)**提供一种**[面向连接的](http://en.wikipedia.org/wiki/Connection-oriented_communication)**、**[可靠的](http://en.wikipedia.org/wiki/Reliability_(computer_networking)**字节流服务，它保证发送和接收的字节的内容和顺序是一致的。

![TCP header](/images/tcp-header.png)

###TCP如何提供可靠的服务###
TCP通过如下方式提供可靠性：

* 应用数据被分割成TCP认为最适合发送的数据块。  
  注：因为是流服务，所以可以任意分割任意合并
* 当TCP发出一个段后，它启动一个定时器，等待目的端确认收到这个报文段。如果不能及时收到一个确认，将重发这个报文段。  
  注：超时和重传策略
* 当TCP收到发自TCP连接另一端的数据，它将发送一个确认。  
  注：接收端的确认
* TCP将保持它首部和数据的检验和。这是一个端到端的检验和，目的是检测数据在传输过程中的任何变化。如果收到段的检验和有差错，TCP将丢弃这个文段和不确认收到此报文段(希望发端超时并重发)。
* 既然TCP报文段作为IP数据报来传输，而IP数据报的到达可能会失序，因此TCP报文段的到达也可能会失序。如果必要，TCP将对收到的数据进行重新排序，将收到的数据以正确的顺序交给应用层。
* 既然IP数据报会发生重复，TCP的接收端必须丢弃重复的数据。
* TCP还能提供流量控制。TCP连接的每一方都有固定大小的缓冲空间。TCP的接收端只允许另一端发送接收端缓冲区所能接纳的数据。这将防止较快主机致使较慢主机的缓冲区溢出。

###TCP连接的建立###
TCP通过三次握手(Three-way handshake)建立连接：

![TCP three-way handshake](/images/tcp-three-way-handshake.gif)

1. 客户端发送**SYN**到服务器，同时将`sequence number`设置为`client_ISN`
2. 服务器发送**SYN+ACK**作为应答，同时将`sequence number`设置为`server_ISN`，将`acknowledgment number`设置为`client_ISN+1`
3. 客户端发送**ACK**作为应答，同时将`acknowledgment number`设置为`server_ISN+1`

TCP为应用层提供[全双工](http://en.wikipedia.org/wiki/Full-duplex#Full-duplex)(full-duplex)服务，数据能在两个方向上独立地进行传输。因此，连接的每一端必须保持每个方向上的传输数据序号。当建立一个新的连接时，双方都应该发送**SYN**，并将`sequence number`设为本方的ISN(其值随系统时间而变化)。接收到**SYN**后，双方应发送**ACK**作为应答，并将`acknowledgment number`设为对方`ISN+1`。

为什么SYN、FIN需要占用一个字节，而ACK不需要？
没有数据的数据包，ACK需要加1吗？

###TCP连接的终止###
TCP通过四次握手(Four-way handshake)终止连接：

![TCP connection termination](/images/tcp-connection-termination.gif)

###Wireshark实时会话图###
