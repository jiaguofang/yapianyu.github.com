---
layout: post
title: C++适合开发web application吗？
category: language
tags: [c++, web application]
---

C++可以开发Financial systems, Embeded systems, Games, OS……那么它适合开发web application吗?

开发web application需要处理字符串, 进行网络通信, 访问数据库, 这些C++都可以做. 目前也有不少C++ web开发框架, 比如[Wt](http://www.webtoolkit.eu/wt)和[cppcms](http://cppcms.com/wikipp/en/page/main). 这样就能使C++成为一门出色的web开发语言了吗? 不是, 现代的web application一个很大的目标是快速进入市场, 而C++语法复杂, 缺乏强有力的库支持(除了STL, Boost), 开发周期长, 维护难度大, 这些都不足以让它成为主流的web开发语言. 尽管C++代码执行效率高, 但是大部分web application的时间主要消耗在网络连接和数据库访问上, 因此不同语言的代码执行效率对整体的性能影响不大. 当前的主流应该是Java(J2EE), Ruby(Ruby on Rails), Python(Django), PHP.

> C++ can do anything, but is not suited for everything!

**References**  
[How popular is C++ for making websites/web applications?](http://stackoverflow.com/questions/417816/how-popular-is-c-for-making-websites-web-applications)  
[Can C++ be used as a server-side web development language?](http://programmers.stackexchange.com/questions/53624/can-c-be-used-as-a-server-side-web-development-language)
