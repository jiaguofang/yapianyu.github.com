---
layout: post
title: POSIX Threads(Pthreads)
category: os
tags: POSIX thread
---

[POSIX Threads](http://en.wikipedia.org/wiki/POSIX_Threads)(或称Pthreads)是一套
针对线程的POSIX标准，该标准定义了一系列创建和操作线程的API。在很多符合POSIX标准
的Unix系(\*nix)OS上都有实现，比如FreeBSD，NetBSD，OpenBSD，GNU/Linux，Mac OS X和Solaris。

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

pthread_mutex_init——和PTHREAD_MUTEX_INITIALIZER初始化的区别，后者不需要destroy？
pthread_mutex_destroy
pthread_mutex_lock——死锁
pthread_mutex_trylock——什么情况下会用它
pthread_mutex_unlock

“使用多个互斥量时可能会产生死锁，有两种通用的解决方法：固定加锁层次，试加锁和回
退”

“在一个条件变量上等待会导致以下原子操作：释放相关互斥量，等待其他线程发给该条件
变量的信号（唤醒一个等待者）或广播该条件变量（唤醒所有等待者）。当线程从条件变量
等待中醒来时，它重新继续锁住互斥量。”

“为什么不将互斥量作为条件变量的一部分来创建呢？首先，互斥量不仅与条件变量一起使
用，而且还要单独使用；其次，通常一个互斥量可以与多个条件变量相关。”

无论有没有线程被阻塞，条件变量发出信号后都将被复位。因此如果这时有线程被阻塞，是
不会被唤醒的。

pthread_cond_init
pthread_cond_destroy
pthread_cond_wait
pthread_cond_timewait
pthread_cond_signal
pthread_cond_broadcast

多个线程中使用条件变量的拷贝是没有意义的，必须使用条件变量的指针
互斥量与条件变量是一对多的关系，即任何条件变量在特定时刻只能与一个互斥量相关联，
而互斥量则可以同时与多个条件变量关联



http://www.ibm.com/developerworks/linux/library/l-posix1/index.html
