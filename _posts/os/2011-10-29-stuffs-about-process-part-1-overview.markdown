---
layout: post
title: 进程那些事(一)：概述
category: os
tags: [process, heap, stack, virtual memory space]
---

进程是对运行中的程序与一些计算机资源(内存，文件，I/O设备等)和状态(程序计数器，寄存器等)的统称。

##虚拟地址空间##
通常，程序以二进制可执行文件形式存储在磁盘上。为了执行，程序被调入并放在进程中。早期的程序访问的内存地址都是实际的**物理地址**，不同的程序可以修改彼此的内存数据(有意或无意)，导致程序运行异常。**虚拟地址空间**(Virtual Address Space)对进程来说就像一个沙盒，从逻辑上将它们隔离。在32位机器上，虚拟地址空间的大小为4GB。操作系统将它分为两部分——**内核空间**(Kernel Space)和**用户空间**(User Space)。内核空间由所有进程共享，但只有运行在内核态的进程才能访问，用户进程可以通过系统调用切换到内核态访问内核空间；用户空间由每个进程独立拥有，运行在用户态和内核态的进程都可以访问用户空间。不同的操作系统中，内核空间和用户空间的比例有所不同，Linux为1:3(1GB和3GB)，Windows为2:2(2GB和2GB，也可以配置成1:3)。

![](/images/kernel-user-memory-split.png)

**进程的地址空间是私有的**，不能被其它进程访问(除非共享)。操作系统为每个进程维持一张**页表**(Page Table)，用来将地址空间内的虚拟地址映射到真实的物理地址。

![](/images/virtual-memory-in-process-switch.png)

蓝色区域表示映射到物理内存的虚拟地址，白色区域则表示尚未映射的虚拟地址。

典型的Linux内存布局如下：

![](/images/linux-flexible-address-space-layout.png)

简单介绍各段含义：  

1. stack  
在heap之上，唯一由高地址向低地址增长的段。存放函数局部变量、临时变量、函数参数、返回地址、寄存器值等(文字常量和静态变量除外)。**每个线程都有各自独立的stack**，[其大小可以在线程创建时设置，Linux/x86-32平台上默认为2MB](http://www.kernel.org/doc/man-pages/online/pages/man3/pthread_create.3.html)。每次调用函数时，会往stack中压入一个**栈帧**(Stack Frame)。由于stack上所有申请和销毁操作都由编译器完成，没有复杂操作，因此效率很高。
2. heap  
向高地址增长，通过malloc/new申请。内部实现时所有空闲内存像链表一样被连接，频繁的申请和销毁操作容易形成碎片。当heap不足时，通过 brk() 系统调用扩张。所有申请和销毁操作由应用程序负责。
3. BSS  
**存放未初始化的全局变量和static变量**，整个区段在程序启动时被初始化为0。
4. data  
**存放已初始化的全局变量和static变量**，生命周期持续到程序结束。
5. text  
**存放可执行代码和文字常量**，只读。

{% highlight cpp %}
static int cntActiveUsers; // 未初始化的static变量
char * gonzo = "God's own prototype"; // gnozo是已初始化的全局变量，"God's own prototype"是文字常量
static int cntWorkerBees = 10; // 已初始化的static变量
{% endhighlight %}
![](/images/mapping-binary-image.png)

##进程调度##
进程在执行时会改变状态。每个进程可能处于下列状态之一：

1. **新的**：进程正在被创建。
2. **运行**：指令正在被执行。
3. **等待**：进程等待一定事件的出现(如I/O完成或收到某个信号)。
4. **就绪**：进程等待被分配给某个处理器。
5. **终止**：进程已完成执行。

![](/images/process-state.gif)

存放进程管理和控制信息的数据结构称为[进程控制块](http://en.wikipedia.org/wiki/Process_control_block)(Process Control Block, PCB)。PCB包含的信息有：进程状态、程序计数器、CPU寄存器、CPU调度信息、内存管理信息、记账信息、I/O状态信息等等。

当CPU切换到另一个进程需要保存原来进程的状态并装入新进程的保存状态。这一任务称为[上下文切换](http://en.wikipedia.org/wiki/Context_switch)(Context Switch)。当发生上下文切换时，内核会将旧进程的关联状态保存在其PCB中，然后装入经调度要执行的新进程的已保存的关联状态。

##References##
[Anatomy of a Program in Memory](http://duartes.org/gustavo/blog/post/anatomy-of-a-program-in-memory)(从这里盗了很多图)  
[Virtual Address Space](http://msdn.microsoft.com/en-us/library/windows/desktop/aa366912(v=vs.85\).aspx)
