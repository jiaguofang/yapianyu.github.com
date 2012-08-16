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
   `pthread_attr_`命名；
2. **Mutexes**：Mutual exclusion的简称，用于处理线程同步问题。互斥量函数提供了对
   互斥量的create，destroy，lock和unlock操作，同时也包括设置(set)和修改(modify)
   与互斥量相关的属性。以`pthread_mutex_`和`pthread_mutexattr_`命名；
3. **Condition variables**：用来通知共享数据的状态信息。包括对条件变量的create，
   destroy，wait和signal操作，以及设置(set)和查询(query)条件变量属性。以
   `pthread_cond_`和`pthread_condattr_`命名；
4. **Synchronization**：操作读写锁和barrier。以`pthread_rwlock_`和
   `pthread_barrier_`命名。

###Thread management###
####线程的创建(create)和终止(terminate)####
{% highlight cpp %}
/**
 * 创建一个线程
 *
 * @param thread 线程ID
 * @param attr 属性对象，用于设置线程属性，若attr为NULL，将以默认属性创建线程
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
<pthread_exit和return的关系>

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

----------
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
 * 分离线程，当目标线程结束后，其资源将被立刻回收
 *
 * @param thread 目标线程ID
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_detach(pthread_t thread);
{% endhighlight %}

只有被创建为joinable的线程才能被其它线程join。如果目标线程正在运行，调用`pthread_join()`
会阻塞当前线程，直到目标线程终止；如果目标线程已经终止(未被detach，状态为终止态)
，当前线程会立刻返回，不被阻塞。目标线程的返回状态(通过`pthread_exit()`指定)可以
通过`pthread_join()`的第二个参数获取。多个线程同时等待(join)同一个线程的行为是不
可预知的，必须禁止这种做法。

如果使用`PTHREAD_CREATE_DETACH`属性创建线程，或者调用`pthread_detach()`分离线程，
则当线程结束时，其资源将被立刻回收。如果终止线程没有被分离，则它将一直处于终止态
直到被分离(通过`pthread_detach()`)或者被连接(通过`pthread_join`)。

> From \<Programming With POSIX Threads\>:

> 分离一个正在运行的线程不会对线程带来任何影响，仅仅是通知系统当该线程结束时，其
> 所属资源可以被回收。

> 一个没有被分离的线程终止时会保留其虚拟内存，包括它们的堆栈和其他系统资源。分离
> 线程意味着通知系统不再需要此线程，允许系统将分配给它的资源回收。

> 一旦pthread_join获得返回值，终止线程就被pthread_join函数分离，并且可能在
> pthread_join函数返回前被回收。这意味着，返回值一定不要是与终止线程堆栈相关的堆
> 栈地址，因为该地址上的值可能在调用线程能够访问之前就被覆盖了。

-----------
####其它函数####
{% highlight cpp %}
/**
 * 获得调用线程的ID
 *
 * @return 调用线程的ID
 */
pthread_t pthread_self(void);

/**
 * 比较两个线程ID是否相等
 *
 * @param t1 线程ID
 * @param t2 线程ID
 * @return 相等返回非0，否则返回0
 */
int pthread_equal(pthread_t t1, pthread_t t2);
{% endhighlight %}

线程ID(包括其它pthread数据类型)都是不透明的(opaque)，因此不能用`==``比较两个线程
ID是否相等。

> From \<Programming With POSIX Threads\>:

> Pthread数据类型是不透明的，我们不应该对其实现做任何假设，只能按照标准中描述的
> 方式使用它们。例如，线程标志符ID可能是整型，或者是浮点型，或者是结构体，任何以
> 不能兼容所有定义的方式使用线程ID的代码都是错误的。

-----------
####Sample####
{% highlight cpp %}
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#define NUM_THREADS 12

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
    pthread_exit(arg); // return arg;
}

int main()
{
    pthread_attr_t attr;
    pthread_attr_init(&attr); // init attr

    size_t stack_size;
    pthread_attr_getstacksize(&attr, &stack_size); // get default stack size
    printf("Default stack size is %lu.\n", stack_size);

    pthread_attr_setstacksize(&attr, 3145728); // set new stack size
    pthread_attr_getstacksize(&attr, &stack_size);
    printf("New stack size is %lu.\n", stack_size);

    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE); // set joinable

    int rc;
    pthread_t thread_id[NUM_THREADS];
    for (long i = 0; i < NUM_THREADS; i++)
    {
        rc = pthread_create(&thread_id[i], &attr, fibonacci, (void *) i); // create threads
        if (rc)
        {
            printf("Failed to create thread %ld: %s.\n", i, strerror(rc));
            exit(-1);
        }
    }

    pthread_attr_destroy(&attr);
    void *status;
    for (long i = 0; i < NUM_THREADS; i++)
    {
        rc = pthread_join(thread_id[i], &status); // join every thread
        if (rc)
        {
            printf("Failed to join thread %ld: %s.\n", i, strerror(rc));
            exit(-1);
        }
        printf("Return code from thread is %ld.\n", (long) status); // return code equals to 'arg'
    }

    printf("Main() exits.\n");
}
{% endhighlight %}
{% highlight text %}
yapianyu@yapianyu-pc:~/Desktop$ ./a.out
Default stack size is 8388608.
New stack size is 3145728.
Fibonacci(5) is 5.
Fibonacci(6) is 8.
Fibonacci(7) is 13.
Fibonacci(8) is 21.
Fibonacci(9) is 34.
Fibonacci(10) is 55.
Fibonacci(11) is 89.
Fibonacci(4) is 3.
Fibonacci(3) is 2.
Fibonacci(2) is 1.
Fibonacci(1) is 1.
Fibonacci(0) is 0.
Return code from thread is 0.
Return code from thread is 1.
Return code from thread is 2.
Return code from thread is 3.
Return code from thread is 4.
Return code from thread is 5.
Return code from thread is 6.
Return code from thread is 7.
Return code from thread is 8.
Return code from thread is 9.
Return code from thread is 10.
Return code from thread is 11.
Main() exits.
{% endhighlight %}

###Mutexes###
线程共享进程地址空间是一把双刃剑，优点是减少线程创建和切换的代价，同时使得线程间
通信更加方便。缺点也很明显，就是共享数据的同步问题，频繁的同步将带来性能上的损失
，线程的异常终止还会导致整个进程的终止。

共享数据的同步问题可以通过互斥量解决，互斥量就像一个“君子协议(gentlemen's
agreement)”，各个参与线程遵守这个协议，彼此有序地访问共享数据。

####互斥量的创建与销毁####
{% highlight cpp %}
/**
 * 用参数attr初始化互斥量，若attr为NULL，将以默认值初始化mutex
 *
 * @param mutex 互斥量
 * @param attr 互斥量属性
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);

/**
 * 销毁互斥量，让它的状态变为非初始化的
 *
 * @param mutex 互斥量
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_mutex_destroy(pthread_mutex_t *mutex);
{% endhighlight %}

初始化互斥量的方式有两种：

1. 静态初始化
{% highlight cpp %}
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
{% endhighlight %}
2. 动态初始化
{% highlight cpp %}
pthread_mutex_t mutex;
pthread_mutex_init(&mutex, NULL); // pthread_mutex_init(&mutex, &attr);
pthread_mutex_destroy(&mutex);
{% endhighlight %}

两者的的区别在于，动态初始化可以设置属性`attr`，但是需要调用`pthread_mutex_destroy()`销毁
(两者的内存分配策略依赖于具体的`pthreads`实现，可能动态方法会在堆上申请空间——未
经证实)。

----------
####互斥量的加锁(lock)与解锁(unlock)####
{% highlight cpp %}
/**
 * 加锁互斥量，如果互斥量已经被锁住，阻塞当前线程直到互斥量被解锁
 *
 * @param mutex 互斥量
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_mutex_lock(pthread_mutex_t *mutex);

/**
 * 功能和pthread_mutex_lock()一样，除了当互斥量已经被锁住时，该函数立即返回
 *
 * @param mutex 互斥量
 * @return 加锁成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_mutex_trylock(pthread_mutex_t *mutex);

/**
 * 解锁互斥量，所有阻塞在该互斥量上的线程重新参与获取该互斥量
 *
 * @param mutex 互斥量
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_mutex_unlock(pthread_mutex_t *mutex);
{% endhighlight %}

多个线程同时调用`pthread_mutex_lock()`对同一个互斥量加锁，只有一个线程可以成功，
其它线程将被阻塞，被阻塞的线程处于等待状态，不参与CPU调度，只有当该互斥量被解锁
时，它们才会从等待状态转为就绪状态，重新参与CPU调度，竞争互斥量。

`pthread_mutex_trylock()`的区别在于它不会阻塞当前线程，无论如何都会立即返回。当
互斥量已经被锁住，它会返回指示“busy”的错误代码，因此可以避免发生死锁。

使用`pthread_mutex_unlock()`时应当注意，只有锁住互斥量的线程才能解锁该互斥量，否
则会发生错误。

----------
####避免死锁####
http://www2.chrishardick.com:1099/Notes/Computing/C/pthreads/mutexes.html

----------
####Sample####
下面这个例子中，部分线程调用`pthread_mutex_lock()`，另外一些调用`pthread_mutex_trylock()`，
同时记录`pthread_mutex_trylock()`执行失败的次数。

{% highlight cpp %}
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>

#define NUM_THREADS_LOCK 100 // thread number using pthread_mutex_lock()
#define NUM_THREADS_TRYLOCK 100 // thread number using pthread_mutex_lock()
#define SUM_TO_NUM 1000 // each thread sum from 1 to 1000

long sum = 0;
pthread_mutex_t sum_mutex;

// using pthread_mutex_lock()
void *sum_lock(void *arg)
{
    for (long i = 1; i <= (long) arg; i++)
    {
        // The thread will be blocked if the mutex is locked
        // by other thread.
        pthread_mutex_lock(&sum_mutex);
        sum += i;
        pthread_mutex_unlock(&sum_mutex);
    }
    pthread_exit(NULL);
}

// using pthread_mutex_trylock()
void *sum_trylock(void *arg)
{
    long trylock_failed = 0;
    for (long i = 1; i <= (long) arg; i++)
    {
        // pthread_mutex_trylock() will return a non-zero value
        // if the mutex is locked by other thread.
        while (pthread_mutex_trylock(&sum_mutex))
            trylock_failed++;
        sum += i;
        pthread_mutex_unlock(&sum_mutex);
    }
    pthread_exit((void *) trylock_failed);
}

int main()
{
    pthread_mutex_init(&sum_mutex, NULL); // init mutex

    pthread_attr_t attr;
    pthread_attr_init(&attr); // init attr
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE); // set joinable

    int rc;
    pthread_t thread_id[NUM_THREADS_LOCK + NUM_THREADS_TRYLOCK];
    for (long i = 0; i < NUM_THREADS_LOCK; i++)
    {
        rc = pthread_create(&thread_id[i], &attr, sum_lock, (void *) SUM_TO_NUM); // create threads
        if (rc)
        {
            printf("Failed to create thread %ld: %s.\n", i, strerror(rc));
            exit(-1);
        }
    }
    for (long i = 0; i < NUM_THREADS_TRYLOCK; i++)
    {
        rc = pthread_create(&thread_id[i + NUM_THREADS_LOCK], &attr,
                sum_trylock, (void *) SUM_TO_NUM); // create threads
        if (rc)
        {
            printf("Failed to create thread %ld: %s.\n", i + NUM_THREADS_LOCK, strerror(rc));
            exit(-1);
        }
    }

    void *status;
    long total_trylock_failed = 0;
    for (long i = 0; i < NUM_THREADS_LOCK + NUM_THREADS_TRYLOCK; i++)
    {
        rc = pthread_join(thread_id[i], &status); // join every thread
        if (rc)
        {
            printf("Failed to join thread %ld: %s.\n", i, strerror(rc));
            exit(-1);
        }
        total_trylock_failed += (long) status; // sum up failed pthread_mutex_trylock()
    }
    
    pthread_attr_destroy(&attr);
    pthread_mutex_destroy(&sum_mutex);

    printf("Total times pthread_mutex_trylock() failed = %ld.\n", total_trylock_failed);
    printf("Sum = %ld.\n", sum);
    printf("Actual sum = %ld.\n", (long)
        (NUM_THREADS_LOCK + NUM_THREADS_TRYLOCK) * (1 + SUM_TO_NUM) * SUM_TO_NUM / 2);
}
{% endhighlight %}
{% highlight text %}
yapianyu@yapianyu-pc:~/Desktop$ ./a.out
Total times pthread_mutex_trylock() failed = 2193647.
Sum = 100100000.
Actual sum = 100100000.
{% endhighlight %}
###Condition variables###
条件变量和互斥量一样，也提供对共享数据的同步，唯一区别是前者有条件地进行同步(条
件等待->条件满足->被唤醒)。如果不使用条件变量，要实现相同功能，就需要在代码中不
停地轮询(polling)，直到条件满足，这显然很浪费资源。

####条件变量的创建与销毁####
{% highlight cpp %}
/**
 * 用参数attr初始化条件变量，若attr为NULL，将以默认值初始化cond
 *
 * @param cond 条件变量
 * @param attr 条件变量属性
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_cond_init(pthread_cond_t *cond, const pthread_condattr_t *attr);

/**
 * 销毁条件变量，让它的状态变为非初始化的
 *
 * @param cond 条件变量
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_cond_destroy(pthread_cond_t *cond);
{% endhighlight %}
初始化条件变量的方式有两种：

1. 静态初始化
{% highlight cpp %}
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
{% endhighlight %}
2. 动态初始化
{% highlight cpp %}
pthread_cond_t cond;
pthread_cond_init(&cond, NULL); // pthread_cont_init(&cond, &attr);
pthread_cond_destroy(&cond);
{% endhighlight %}

两者的的区别在于，动态初始化可以设置属性`attr`，但是需要调用`pthread_cond_destroy()`销毁。

----------
####条件变量的等待(wait)和发信号(signal)####
{% highlight cpp %}
/**
 * 在条件变量cond上阻塞线程
 *
 * @param cond 条件变量
 * @param mutex 与条件变量关联的互斥量
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);

int pthread_cond_timedwait(pthread_cond_t *cond, pthread_mutex_t *mutex, const struct timespec *abstime);

/**
 * 唤醒阻塞在条件变量cond上的一个线程(单独发信号)
 *
 * @param cond 条件变量
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_cond_signal(pthread_cond_t *cond);

/**
 * 唤醒阻塞在条件变量cond上的所有线程(广播)
 *
 * @param cond 条件变量
 * @return 成功返回0，否则返回<errno.h>头文件中的错误代码，可以通过strerror()获取错误代码的描述
 */
int pthread_cond_broadcast(pthread_cond_t *cond);
{% endhighlight %}

互斥量与条件变量是**一对多**的关系，即任何条件变量在特定时刻只能与一个互斥量相关
联，而互斥量则可以同时与多个条件变量关联。

`pthread_cond_wait()`和`pthread_cond_timedwait()`执行如下操作：

1. 解锁相关互斥量，阻塞调用线程，并等待被唤醒；
2. 被唤醒并成功返回之后，互斥量被当前线程锁住。

`pthread_cond_signal()`和`pthread_cond_broadcast()`执行如下操作：

1. 唤醒等待在条件变量上的线程(一个或多个)；
2. 在当前线程解锁互斥量之后，被唤醒的线程开始竞争互斥量。

无论有没有线程等待在条件变量上，signal或broadcast后条件变量都将被复位。如果此后
有线程被阻塞在条件变量上，是不会被唤醒的。

`pthread_cond_wait()`和`pthread_cond_signal()`的一般用法如下：
{% highlight cpp %}
// code snippet of pthread_cond_wait()
pthread_mutex_lock(&mutex); // <===== Q1
while(<condition is false>) // <===== Q2
    pthread_cond_wait(&cond, &mutex);
<do some real stuff>;
pthread_mutex_unlock(&mutex);

// code snippet of pthread_cond_signal()
pthread_mutex_lock(&mutex); // <===== Q3
<set condition to true>;
pthread_cond_signal(&cond);
pthread_mutex_unlock(&mutex);
{% endhighlight %}

这里有几个问题：

Q1. 为何要在`pthread_cond_wait()`之前lock mutex？  
Q2. 为何要在`pthread_cond_wait()`之前判断condition是否为`false`，还要放在`while`中？  
Q3. 为何要在`pthread_cond_signal()`之前lock mutex？

Q1和Q3是因为condition与共享数据有关，lock mutex是为了保护共享数据。

书\<Programming With POSIX Threads\>中对Q2也讨论过，文中给出的解释是存在**被拦截
的唤醒(intercepted wakeup)**和**假唤醒(spurious wakeup)**。关于**被拦截的唤醒**，
参见以下多线程代码：

{% highlight text %}
Thread A                            Thread B                           Thread C

pthread_mutex_lock(&mutex);
// thread blocked, mutex released
pthread_cond_wait(&cond, &mutex);
                                    pthread_mutex_lock(&mutex);
                                    <set condition to true>;
                                    pthread_cond_signal(&cond);
                                    pthread_mutex_unlock(&mutex);
                                                                       pthread_mutex_lock(&mutex);
                                                                       <set condition to false>
                                                                       pthread_mutex_unlock(&mutex);
// thread unblocked, mutex aquired
// condition is not true now
<do some real stuff>;
pthread_mutex_unlock(&mutex);
{% endhighlight %}

可以看到Thread A从被Thread B唤醒到lock mutex这段时间内，被Thread C**偷去了**CPU。
当Thread C修改condition并unlock mutex后，Thread A看到的condition已经为`false`了。

正如\<Programming With POSIX Threads\>所说：
> 当线程醒来时，再次测试谓词同样重要。应该总是在循环中等待条件变量，来避免程序错
> 误、多处理器竞争和假唤醒。

----------
####Sample####
下面这个例子是[生产者-消费者问题](http://en.wikipedia.org/wiki/Producer-consumer_problem)的Pthreads解法，
若干个生产者往`buffer`里写内容，若干个消费者从`buffer`中取内容，通过`pthread_cond_broadcast()`
唤醒阻塞的生产者/消费者线程。
{% highlight cpp %}
#include <pthread.h>
#include <stdio.h>
#include <iostream>
#include <deque>

using namespace std;

#define PRODUCER_NUM 10
#define CONSUMER_NUM 10

#define BUFFER_SIZE 100
deque<int> buffer;

int task_id = 0;

#define TOTAL_TASKS 20 // stop producing/consuming if TOTAL_TASKS reaches
int produced = 0; // how many tasks produced
int consumed = 0; // how many tasks consumed

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond_not_full = PTHREAD_COND_INITIALIZER;
pthread_cond_t cond_not_empty = PTHREAD_COND_INITIALIZER;

void *producer(void *arg)
{
    while (true)
    {
        pthread_mutex_lock(&mutex);
        // stop producing if TOTAL_TASKS reaches
        if (produced >= TOTAL_TASKS)
        {
            pthread_mutex_unlock(&mutex);
            break;
        }
        while (buffer.size() == BUFFER_SIZE)
            pthread_cond_wait(&cond_not_full, &mutex); // <===== cond_not_full
        printf("Producer #%ld produces task %d.\n", (long) arg, task_id);
        buffer.push_back(task_id++);
        produced++;
        pthread_cond_broadcast(&cond_not_empty); // <===== cond_not_empty
        pthread_mutex_unlock(&mutex);

        usleep(1000);
    }

    return (NULL);
}

void *consumer(void *arg)
{
    while (true)
    {
        pthread_mutex_lock(&mutex);
        // stop consuming if TOTAL_TASKS reaches
        if (consumed >= TOTAL_TASKS)
        {
            pthread_mutex_unlock(&mutex);
            break;
        }
        while (buffer.size() == 0)
            pthread_cond_wait(&cond_not_empty, &mutex); // cond_not_empty
        printf("\t\t\t\tConsumer #%ld consumes task %d.\n", (long) arg,
                buffer.front());
        buffer.pop_front();
        consumed++;
        pthread_cond_broadcast(&cond_not_full); // cond_not_full
        pthread_mutex_unlock(&mutex);

        usleep(1000);
    }

    pthread_exit(NULL);
}

int main()
{
    pthread_t prod_threads[PRODUCER_NUM];
    pthread_t cons_threads[CONSUMER_NUM];

    for (long i = 0; i < PRODUCER_NUM; i++)
        pthread_create(&prod_threads[i], NULL, producer, (void *) i);

    for (long i = 0; i < CONSUMER_NUM; i++)
        pthread_create(&cons_threads[i], NULL, consumer, (void *) i);

    for (long i = 0; i < PRODUCER_NUM; i++)
        pthread_join(prod_threads[i], NULL);

    for (long i = 0; i < CONSUMER_NUM; i++)
        pthread_join(cons_threads[i], NULL);

    pthread_exit(NULL);
}
{% endhighlight %}
{% highlight text %}
yapianyu@yapianyu-pc:~/Desktop$ ./a.out
Producer #9 produces task 0.
                                Consumer #3 consumes task 0.
Producer #7 produces task 1.
Producer #8 produces task 2.
Producer #5 produces task 3.
                                Consumer #2 consumes task 1.
                                Consumer #1 consumes task 2.
                                Consumer #0 consumes task 3.
Producer #6 produces task 4.
                                Consumer #4 consumes task 4.
Producer #4 produces task 5.
                                Consumer #9 consumes task 5.
Producer #3 produces task 6.
                                Consumer #7 consumes task 6.
Producer #2 produces task 7.
                                Consumer #5 consumes task 7.
Producer #1 produces task 8.
                                Consumer #6 consumes task 8.
Producer #0 produces task 9.
                                Consumer #8 consumes task 9.
Producer #9 produces task 10.
                                Consumer #3 consumes task 10.
Producer #7 produces task 11.
Producer #8 produces task 12.
Producer #5 produces task 13.
                                Consumer #2 consumes task 11.
                                Consumer #1 consumes task 12.
                                Consumer #0 consumes task 13.
Producer #6 produces task 14.
                                Consumer #4 consumes task 14.
Producer #6 produces task 15.
Producer #5 produces task 16.
Producer #8 produces task 17.
                                Consumer #2 consumes task 15.
Producer #7 produces task 18.
                                Consumer #0 consumes task 16.
                                Consumer #1 consumes task 17.
                                Consumer #4 consumes task 18.
Producer #9 produces task 19.
                                Consumer #3 consumes task 19.
{% endhighlight %}
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

[Stack Overflow: pthread_cond_wait and mutex requirement](http://stackoverflow.com/a/6312416/1037167)  
[Stack Overflow: Calling pthread_cond_signal without locking mutex](http://stackoverflow.com/a/4567919/1037167)  
[Stack Overflow: Pthread and wait conditions](http://stackoverflow.com/a/4821672/1037167)
