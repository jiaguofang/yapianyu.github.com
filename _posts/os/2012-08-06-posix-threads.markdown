---
layout: post
title: POSIX Threads (Pthreads)
category: os
tags: POSIX thread
---

[POSIX Threads](http://en.wikipedia.org/wiki/POSIX_Threads)(Pthreads)是一套针对
线程的POSIX标准，该标准定义了一系列创建和操作线程的API。在很多符合POSIX标准的Unix
系OS上都有实现，如FreeBSD，NetBSD，OpenBSD，GNU/Linux，Mac OS X和Solaris。

Pthreads API可以(非正式地)分为四组：

1. **Thread management**：直接操作线程的API，如create，detach，join等。也包括设
   置(set)和查询(query)线程属性(如joinable，scheduling等)的函数。以`pthread_`和
   `pthread_attr_`开头；
2. **Mutexes**：Mutual exclusion的简称，用于处理线程同步问题。互斥量函数提供了对
   互斥量的create，destroy，lock和unlock操作，同时也包括设置(set)和修改(modify)
   与互斥量相关的属性。以`pthread_mutex_`和`pthread_mutexattr_`开头；
3. **Condition variables**：用来通知共享数据的状态信息。包括对条件变量的create，
   destroy，wait和signal操作，以及设置(set)和查询(query)条件变量属性。以
   `pthread_cond_`和`pthread_condattr_`开头；
4. **Synchronization**：操作读写锁和barrier。以`pthread_rwlock_`和
   `pthread_barrier_`开头。

###Thread management###
####线程的创建(create)和终止(terminate)####
{% highlight cpp %}
/**
 * 创建一个线程
 *
 * @param thread 线程ID
 * @param attr 属性对象，用于设置线程属性，默认值为NULL
 * @param start_routine 指向线程函数的指针，参数和返回值都为void *
 * @param arg 传递给start_routine()的参数
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine)(void*), void *arg);

/**
 * 终止当前线程
 *
 * @param value_ptr 线程的退出状态，可以调用pthread_join获取该状态
 */
void pthread_exit(void *value_ptr);

/**
 * 创建一个线程
 *
 * @param thread 线程ID
 * @param attr 属性对象，用于设置线程属性，默认值为NULL
 * @param start_routine 指向线程函数的指针，参数和返回值都为void *
 * @param arg 传递给start_routine()的参数
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_cancel(pthread_t thread);

/**
 * 用默认值初始化线程属性对象
 *
 * @param attr 线程属性对象
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_attr_init(pthread_attr_t *attr);

/**
 * 销毁线程属性对象
 *
 * @param attr 线程属性对象
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_attr_destroy(pthread_attr_t *attr);
{% endhighlight %}

`main()`函数所在线程被称为“**主线程**”，因此即使没有调用`pthread_create()`，当前
进程也包含一个线程。主线程和进程中其它线程没有任何层次关系，执行顺序完全依赖于系
统的调度算法。如果没做特殊处理，主线程结束时其它线程也将自动终止。

有关线程创建最重要的是，在当前线程从函数`pthread_create()`中返回以及新线程被调度
执行之间不存在同步关系。即，新线程可能在当前线程从`pthread_create()`返回之前就运
行了，甚至可能已经运行完成。

一般来说，终止线程有以下几种方式：

1. 线程函数执行完成，调用`return`结束；
2. 线程调用`pthread_exit()`终止自己；
3. 被另一个线程调用`pthread_cancel()`；
4. 进程调用`exec()`或`exit()`；
5. `main()`函数先于当前线程结束，并且没有调用`pthread_exit()`。

当线程函数以`return`方式结束时，相当于隐式地(implicit)调用了`pthread_exit()`，两
者效果一样。此时线程函数的返回值就是线程的退出状态(exit status)，可以调用
`pthread_join`获取该状态。值得注意的是，**`main()`函数调用`return`相当于调用`exit()`，
整个进程将会被终止**。为了防止这种情况发生，可以在`main()`函数结束时调用
`pthread_exit()`，这样主线程会被阻塞并等待所有其它线程结束。

<pthread_exit和pthread_cancel之间的关系>

线程创建时会被设置以默认的属性(attribute)，其中一部分属性可以通过修改属性对象
(attribute object)来改变，包括：

1. Detached or joinable state
{% highlight cpp %}
int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);
{% endhighlight %}
2. Scheduling inheritance
{% highlight cpp %}
int pthread_attr_setinheritsched(pthread_attr_t *attr, int inheritsched);
int pthread_attr_getinheritsched(const pthread_attr_t *attr, int *inheritsched);
{% endhighlight %}
3. Scheduling policy
{% highlight cpp %}
int pthread_attr_setschedpolicy(pthread_attr_t *attr, int policy);
int pthread_attr_getschedpolicy(const pthread_attr_t *attr, int *policy);
{% endhighlight %}
4. Scheduling parameters
{% highlight cpp %}
int pthread_attr_setschedparam(pthread_attr_t *attr, const struct sched_param *param);
int pthread_attr_getschedparam(const pthread_attr_t *attr, struct sched_param *param);
{% endhighlight %}
5. Scheduling contention scope
{% highlight cpp %}
int pthread_attr_setscope(pthread_attr_t *attr, int contentionscope);
int pthread_attr_getscope(const pthread_attr_t *attr, int *contentionscope);
{% endhighlight %}
6. Stack size(POSIX标准没有规定线程的堆栈大小，随体系架构的不同而不同，
   [Linux/x86-32平台上默认为2MB](http://www.kernel.org/doc/man-pages/online/pages/man3/pthread_create.3.html))
{% highlight cpp %}
int pthread_attr_setstacksize(pthread_attr_t *attr, size_t stacksize);
int pthread_attr_getstacksize(const pthread_attr_t *attr, size_t *stacksize);
{% endhighlight %}
7. Stack address
{% highlight cpp %}
int pthread_attr_setstackaddr(pthread_attr_t *attr, void *stackaddr);
int pthread_attr_getstackaddr(const pthread_attr_t *attr, void **stackaddr);
{% endhighlight %}
8. Stack guard (overflow) size
{% highlight cpp %}
int pthread_attr_getguardsize(const pthread_attr_t *attr, size_t *guardsize);
int pthread_attr_setguardsize(pthread_attr_t *attr, size_t guardsize);
{% endhighlight %}

<pthread_attr_destroy感觉没多大用处>

Sample：
{% highlight cpp %}
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>

void *fibonacci(void *arg)
{
    unsigned long result = 0;
    unsigned long n = (unsigned long) arg;
    unsigned long a = 0, b = 1;
    if (n == 0 || n == 1)
        result = n;
    else
        for (unsigned long i = 2; i <= n; i++)
        {
            result = a + b;
            a = b;
            b = result;
        }
    printf("Fibonacci(%ld) is %lu.\n", (long) arg, result);
    sleep(1); // sleep to wait main() to terminate
    pthread_exit((void *) 0); // the same as "return 0"
}

int main()
{
    pthread_attr_t attr;
    pthread_attr_init(&attr); // init attr

    size_t stack_size;
    pthread_attr_getstacksize(&attr, &stack_size); // get default stack size
    printf("Default stack size is %lu.\n", stack_size);

    pthread_attr_setstacksize(&attr, 3145728); // 3MB
    pthread_attr_getstacksize(&attr, &stack_size);
    printf("New stack size is %lu.\n", stack_size);

    pthread_t thread_id;
    int rc = pthread_create(&thread_id, &attr, fibonacci, (void *) 12);
    if (rc)
    {
        printf("Failed to create thread: %d\n", (rc));
        exit(-1);
    }

    pthread_attr_destroy(&attr); // destroy attr
    printf("Main() exits.\n");
    pthread_exit(NULL); // wait for other threads to complete
}
{% endhighlight %}
{% highlight text %}
Default stack size is 8388608.
New stack size is 3145728.
Main() exits.
Fibonacci(12) is 144.
{% endhighlight %}

####线程的等待(join)和分离(detach)####
{% highlight cpp %}
/**
 * 阻塞调用线程直到目标线程终止
 *
 * @param thread 目标线程ID
 * @param value_ptr 用来获取目标线程的返回状态，通过目标线程的pthread_exit()指定
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_join(pthread_t thread, void **value_ptr);
/**
 * 用默认值初始化线程属性对象
 *
 * @param attr 线程属性对象
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_detach(pthread_t thread);
{% endhighlight %}

如果目标线程正在运行，调用`pthread_join()`会阻塞当前线程，直到目标线程终止；如果
目标线程已经终止(未被detach)，由于其状态为“终止态”，因此当前线程会立刻返回，不被
阻塞。目标线程的返回状态(通过`pthread_exit()`指定)可以通过`pthread_join()`的第二
个参数获取。多个线程同时等待(join)同一个线程的行为是不可预知的，必须禁止这种做法。

线程运行结束后，其资源不会被系统回收，必须调用`pthread_join()`或
`pthread_detach()`通知系统将其回收。《Programming With POSIX Threads》这样写道：

> 分离一个正在运行的线程不会对线程带来任何影响，仅仅是通知系统当该线程结束时，其
> 所属资源可以被回收。

> 一个没有被分离的线程终止时会保留其虚拟内存，包括它们的堆栈和其他系统资源。分离
> 线程意味着通知系统不再需要此线程，允许系统将分配给它的资源回收。

> 一旦pthread_join获得返回值，终止线程就被pthread_join函数分离，并且可能在
> pthread_join函数返回前被回收。这意味着，返回值一定不要是与终止线程堆栈相关的堆
> 栈地址，因为该地址上的值可能在调用线程能够访问之前就被覆盖了。

###Mutexes###
###Condition variables###
###Synchronization###


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



https://computing.llnl.gov/tutorials/pthreads/
http://www.ibm.com/developerworks/linux/library/l-posix1/index.html

