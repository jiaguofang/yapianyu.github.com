---
layout: post
title: 服务器稳定性检查项目小结
category: project
tags: Python thread Event
---

最近做一个服务器稳定性检查的小项目，设计如下：

1. 基类`BaseChecker`，成员函数`check()`。派生类全部继承自`BaseChecker`，并实现`check()`方法。好处是可扩展，便于做单元测试。

2. 线程类`CheckerThread`，继承自`Python Thread`类。构造函数接受参数`checker`，在`run()`中调用`checker.check()`方法，执行完毕被挂起一段时间。

3. 读配置，在主程序中构造相应的`checker`，并启动相应的线程。接受用户输入，比如`stop`和`CTRL-C`用来停止程序。

使用`time.sleep(t)`来挂起线程似乎很显然，但是每个`checker`的执行周期相差很大——几秒钟到几小时，所以用户输入`stop`停止程序时，必须等待各个线程`sleep`结束，这显然不科学。我要的功能是能挂起一段时间，挂起的过程中能退出。条件变量`Condition.wait(timeout)` 可以在给定时间内等待状态的变化，要么被信号通知，要么超时退出。`Python`有更直接的解决方案，那就是`Event`。`Event`内部封装一个`bool flag`和`Condition cond`，用`flag`的值表示`event`是否被触发。`is_set()`方法返回`flag`的值，`wait(timeout)`方法阻塞线程并等待条件变量的信号，`set()`方法将`flag`设置为`True`并向所有等待在条件变量上的线程发出信号。

`Event`定义：

{% highlight python %}
def Event(*args, **kwargs):
    return _Event(*args, **kwargs)

class _Event(_Verbose):

    # After Tim Peters' event class (without is_posted())

    def __init__(self, verbose=None):
        _Verbose.__init__(self, verbose)
        self.__cond = Condition(Lock())
        self.__flag = False

    def _reset_internal_locks(self):
        # private!  called by Thread._reset_internal_locks by _after_fork()
        self.__cond.__init__()

    def isSet(self):
        return self.__flag

    is_set = isSet

    def set(self):
        self.__cond.acquire()
        try:
            self.__flag = True
            self.__cond.notify_all()
        finally:
            self.__cond.release()

    def clear(self):
        self.__cond.acquire()
        try:
            self.__flag = False
        finally:
            self.__cond.release()

    def wait(self, timeout=None):
        self.__cond.acquire()
        try:
            if not self.__flag:
                self.__cond.wait(timeout)
            return self.__flag
        finally:
            self.__cond.release()
{% endhighlight %}

项目原型：

{% highlight python %}
from threading import Event, Thread

class MyThread(Thread):
    def __init__(self, interval):
        Thread.__init__(self)
        self.__interval = interval
        self.__stop_event = Event()

    def run(self):
        while True:
            # do the real work
            print 'Thread running interval: %s' % self.__interval
            # sleep some time, if stop, then break
            if self.__stop_event.wait(self.__interval):
                print 'Thread stopping interval: %s' % self.__interval
                break

    def stop(self):
        self.__stop_event.set()

if __name__ == '__main__':
    thread_pool = []
    for i in range(1, 10):
        t = MyThread(i * 10)
        t.start()
        thread_pool.append(t)
        
    while True:
        try:
            user_input = raw_input()
            if user_input.strip().lower() == 'stop':
                for t in thread_pool:
                    t.stop()
                break
            else:
                print 'Invalid command'
        except KeyboardInterrupt:  # KeyboardInterrupt can only be caught in the main thread
            for t in thread_pool:
                t.stop()
            break
{% endhighlight %}
