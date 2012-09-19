---
layout: post
title: 理解TCP/IP
category: os
tags: TCP UDP IP
---

最近闲来无事，端起两本网络编程的神书《TCP/IP Illustrated, Vol. 1: The Protocols》和《Unix Network Programming, Vol. 1: Networking API》开始看，不少以前似懂非懂的知识点得到解答。

我认为学习网络编程可以分三个阶段：

1. 熟悉TCP/IP参考模型和相关协议，比如连接的建立和终止、超时和重传、滑动窗口和拥塞控制……
2. 掌握Socket API及其背后隐藏的细节
3. 对服务器进行性能调优

以我目前的功力，应该暂时在第1阶段。Anyway, this is just a beginning!

##TCP/IP参考模型和相关协议##
网络协议是网络上进行数据交换的标准和规范。多个网络协议按照不同功能，不同层次组合在一起就是一个协议族，比如TCP/IP协议族。

TCP/IP模型是一个四层协议模型：链路层……
每一层都有若干符合该层规范的协议，比如传输层上有TCP和UDP
一个协议族，比如TCP/IP，是一组不同层次上的多个协议的组合。TCP/IP并不只包含TCP和IP两个协议(网络层可以是ICMP，传输层可以是UDP)，这样叫是因为TCP和IP是最早被定义的，并且是最重要的两个协议。
协议族举例：
HTTP/TCP/IP/Ethernet protocal
POP3/TCP/IP/Ethernet protocal
UDP/IP/Ethernet protocal
