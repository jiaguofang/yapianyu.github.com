---
layout: post
title: 服务器稳定性检查项目小结
category: project
tags: Python thread Event
---

最近做一个服务器稳定性检查的小项目，我的设计如下：

1. 设计基类`BaseChecker`，成员函数`check()`。派生类全部继承自`BaseChecker`，并实现`check()`方法。好处是可扩展，便于做单元测试。

2. 设计线程类`CheckerThread`，继承自`Python Thread`类。构造函数吸收一个`checker`，在`run()`中调用`checker.check()`方法，执行完毕被挂起一段时间。

3. 读配置，在主程序中构造相应的`checker`，并启动相应的线程。接受用户输入，比如`stop`和`CTRL-C`用来停止程序。

原先使用`time.sleep(t)`来实现线程挂起，后来发现不是很好，因为每个`checker`的执行周期可能相差很大——几秒钟到几小时，所以当用户输入`stop`想要停止程序时，必须等待各个线程`sleep`结束，这显然不科学。我需要的功能是能挂起，能中途退出。

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
