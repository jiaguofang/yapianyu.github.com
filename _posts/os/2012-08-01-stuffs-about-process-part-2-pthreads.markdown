---
layout: post
title: 进程那些事(二)：POSIX threads(Pthreads)
category: os
tags: POSIX thread
---

共享进程地址空间就像一把双刃剑，好处是减少线程创建和切换的代价，并且使得线程间通
信更加方便。坏处也很明显，就是共享数据的同步问题，频繁的同步将带来性能上的损失，
同时线程异常终止会导致整个进程的终止。

“Pthread数据类型是不透明的，我们不应该对其实现做任何假设，只能按照标准中描述的方
式使用它们。例如，线程标志符ID可能是整型，或者是浮点型，或者是结构体，任何以不能
兼容所有定义的方式使用线程ID的代码都是错误的。”——为了可移植

函数声明：参数，返回值，发生错误时，特殊情况下
返回值：正常——0+额外的输出参数指向存有“有用结果”的地址，错误——返回<errno.h>头文
件中的错误代码，通过strerror函数获得错误代码的描述

join自己会怎么样，join一个刚刚结束的线程

pthread_create
pthread_equal
pthread_join——为什么第二个参数是指向指针的指针
pthread_detach
pthread_exit

main函数是main thread——return, exit, pthread_exit
线程函数的参数，返回值——return, exit, pthread_exit

"分离一个正在运行的线程不会对线程带来任何影响，仅仅是通知系统当该线程结束时，其
所属资源可以被回收。"——所谓的资源包括哪些？
“一个没有被分离的线程终止时会保留其虚拟内存，包括它们的堆栈和其他系统资源。分离
线程意味着通知系统不再需要此线程，允许系统将分配给它的资源回收。”——堆上的空间呢
，没有被分离的进程在join结束之后又如何
“一旦pthread_join获得返回值，终止线程就被pthread_join函数分离，并且可能在
pthread_join函数返回前被回收。这意味着，返回值一定不要是与终止线程堆栈相关的堆栈
地址，因为该地址上的值可能在调用线程能够访问之前就被覆盖了。”——因此自动回收有两
种方式pthread_detach或pthread_join


多个线程join同一个线程如何

互斥量和条件变量都能引起阻塞
只有互斥量的主人能够解锁它




http://www.ibm.com/developerworks/linux/library/l-posix1/index.html
