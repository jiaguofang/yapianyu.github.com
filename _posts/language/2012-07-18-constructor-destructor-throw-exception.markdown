---
layout: post
title: C++构造函数和析构函数抛出异常
category: language
tags: C++ 构造函数 析构函数 异常 栈回退 智能指针
---

之前似乎从未考虑过能否在构造函数和析构函数中抛出异常？会有什么后果？所以花了点时间研究了一下，找资料时恰好在《More Effective C++》条款10: *Prevent resource leaksinconstructors*和条款11: *Prevent exceptions from leaving destructors*找到类似章节，遂搬来使用。

##构造函数抛出异常##
我们知道构造函数没有返回值，所以当构造函数执行失败时(比如参数错误，运行时错误等等)，为了**通知调用者**，抛出异常是最好的选择。

举个例子：
{% highlight cpp %}
// 版本一：构造函数无异常
#include <iostream>
#include <string.h>
using namespace std;

class Child
{
public:
    Child(const char* name)
    {
        m_name = new char[strlen(name) + 1]; // 申请内存
        strcpy(m_name, name); // 拷贝
    }
    virtual ~Child()
    {
        delete[] m_name; // 释放内存
        cout << "Child destructed!" << endl;
    }
    char* GetName()
    {
        return m_name;
    }
private:
    char* m_name;
};

class Son : public Child
{
public:
    Son(const char* name) : Child(name) {}
    virtual ~Son()
    {
        cout << "Son destructed!" << endl;
    }
};

class Daughter : public Child
{
public:
    Daughter(const char* name) : Child(name) {}
    virtual ~Daughter()
    {
        cout << "Daughter destructed!" << endl;
    }
};

class Parent
{
public:
    Parent(const char* son, const char* daughter) :
    m_son(son), // 初始化儿子
    m_daughter(new Daughter(daughter)) // 初始化女儿
    {
    }

    ~Parent()
    {
        delete m_daughter; // 释放内存
        cout << "Parent destructed!" << endl;
    }

    void PrintChildren()
    {
        cout << "My son is " << m_son.GetName() << endl; // 打印儿子
        cout << "My daughter is " << m_daughter->GetName() << endl; // 打印女儿
    }
private:
    Son m_son;
    Daughter* m_daughter;
};

int main()
{
    Parent* p = new Parent("Li Lei", "Han Meimei");
    p->PrintChildren(); // 打印
    delete p; // 释放内存
}
{% endhighlight %}

打印正常，且无内存泄漏:
{% highlight text %}
yapianyu@yapianyu-pc:~/Desktop$ g++ ctorleak.cpp -g
yapianyu@yapianyu-pc:~/Desktop$ valgrind --leak-check=full ./a.out
==1711== Memcheck, a memory error detector
==1711== Copyright (C) 2002-2010, and GNU GPL'd, by Julian Seward et al.
==1711== Using Valgrind-3.6.1-Debian and LibVEX; rerun with -h for copyright info
==1711== Command: ./a.out
==1711== 
My son is Li Lei
My daughter is Han Meimei
Daughter destructed!
Child destructed!
Parent destructed!
Son destructed!
Child destructed!
==1711== 
==1711== HEAP SUMMARY:
==1711==     in use at exit: 0 bytes in 0 blocks
==1711==   total heap usage: 4 allocs, 4 frees, 58 bytes allocated
==1711== 
==1711== All heap blocks were freed -- no leaks are possible
==1711== 
==1711== For counts of detected and suppressed errors, rerun with: -v
==1711== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 4 from 4)
{% endhighlight %}

现在修改一下代码，在构造函数中抛出异常：
{% highlight cpp %}
// 版本二：构造函数抛出异常
... ...
class Parent
{
public:
    Parent(const char* son, const char* daughter) :
    m_son(son), // 初始化儿子
    m_daughter(new Daughter(daughter)) // 初始化女儿
    {
        throw runtime_error("Exception from Parent!"); // <=== 抛出异常
    }

    ~Parent()
    {
        delete m_daughter; // 释放内存
        cout << "Parent destructed!" << endl;
    }

    void PrintChildren()
    {
        cout << "My son is " << m_son.GetName() << endl; // 打印儿子
        cout << "My daughter is " << m_daughter->GetName() << endl; // 打印女儿
    }
private:
    Son m_son;
    Daughter* m_daughter;
};

int main()
{
    try
    {
        Parent* p = new Parent("Li Lei", "Han Meimei");
        p->PrintChildren(); // 打印
        delete p; // 释放内存
    }
    catch (exception& e)
    {
        cout << e.what() << endl;
    }
}
{% endhighlight %}

执行结果如下：
{% highlight text %}
yapianyu@yapianyu-pc:~/Desktop$ g++ ctorleak.cpp -g
yapianyu@yapianyu-pc:~/Desktop$ valgrind --leak-check=full ./a.out
==2027== Memcheck, a memory error detector
==2027== Copyright (C) 2002-2010, and GNU GPL'd, by Julian Seward et al.
==2027== Using Valgrind-3.6.1-Debian and LibVEX; rerun with -h for copyright info
==2027== Command: ./a.out
==2027== 
Son destructed!
Child destructed!
Exception from Parent!
==2027== 
==2027== HEAP SUMMARY:
==2027==     in use at exit: 27 bytes in 2 blocks
==2027==   total heap usage: 6 allocs, 4 frees, 249 bytes allocated
==2027== 
==2027== 27 (16 direct, 11 indirect) bytes in 1 blocks are definitely lost in loss record 2 of 2
==2027==    at 0x4C28B35: operator new(unsigned long) (vg_replace_malloc.c:261)
==2027==    by 0x40138D: Parent::Parent(char const*, char const*) (ctorleak.cpp:53)
==2027==    by 0x400FAF: main (ctorleak.cpp:78)
==2027== 
==2027== LEAK SUMMARY:
==2027==    definitely lost: 16 bytes in 1 blocks
==2027==    indirectly lost: 11 bytes in 1 blocks
==2027==      possibly lost: 0 bytes in 0 blocks
==2027==    still reachable: 0 bytes in 0 blocks
==2027==         suppressed: 0 bytes in 0 blocks
==2027== 
==2027== For counts of detected and suppressed errors, rerun with: -v
==2027== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 4 from 4)
{% endhighlight %}

有几点可以发现：

1. 打印结果来自`m_son`的析构和`main()`函数中异常的捕捉。  
`m_son`被析构依赖于异常处理的栈回退([Stack Unwind](http://en.wikibooks.org/wiki/C%2B%2B_Programming/Exception_Handling#Stack_unwinding))机制，主要用来确保异常被捕捉之前所有本地变量(local or stack variables)的析构函数都会被调用到。

2. 第53行`m_daughter(new Daughter(daughter))`发生内存泄漏，说明`~Parent()`没有被调用，从而导致`m_daughter`没有被析构。  
按照C++标准，**尚未完全构造好的对象，其析构函数永远不会被调用**。因此由于`Parent`的构造函数抛出异常，其析构函数永远不会被调用。进一步发现对于`Parent* p = new Parent("Li Lei", "Han Meimei")`，一旦异常抛出，p的值依然是null，即使在catch里调用`delete p`也不会调用`Parent`的析构函数。

因此，为了将`m_daughter`析构掉，唯一可以做的就是在抛出异常时自己捕捉，自己清理，之后将异常重新抛出。
{% highlight cpp %}
// 版本三：构造函数抛出异常后自己清理heap上申请的内存
... ...
class Parent
{
public:
    Parent(const char* son, const char* daughter) :
    m_son(son), // 初始化儿子
    m_daughter(new Daughter(daughter)) // 初始化女儿
    {
        try
        {
            throw runtime_error("Exception from Parent!"); // <=== 抛出异常
        }
        catch (...)
        {
            delete m_daughter; // <=== 自己释放内存
            throw; // <=== 重新抛出
        }
    }

    ~Parent()
    {
        delete m_daughter; // 释放内存
        cout << "Parent destructed!" << endl;
    }

    void PrintChildren()
    {
        cout << "My son is " << m_son.GetName() << endl; // 打印儿子
        cout << "My daughter is " << m_daughter->GetName() << endl; // 打印女儿
    }
private:
    Son m_son;
    Daughter* m_daughter;
};

int main()
{
    try
    {
        Parent* p = new Parent("Li Lei", "Han Meimei");
        p->PrintChildren(); // 打印
        delete p; // 释放内存
    }
    catch (exception& e)
    {
        cout << e.what() << endl;
    }
}
{% endhighlight %}

执行结果如下，可以看到`m_daughter`已被析构，且无内存泄漏：
{% highlight text %}
yapianyu@yapianyu-pc:~/Desktop$ g++ ctorleak.cpp -g
yapianyu@yapianyu-pc:~/Desktop$ valgrind --leak-check=full ./a.out
==2432== Memcheck, a memory error detector
==2432== Copyright (C) 2002-2010, and GNU GPL'd, by Julian Seward et al.
==2432== Using Valgrind-3.6.1-Debian and LibVEX; rerun with -h for copyright info
==2432== Command: ./a.out
==2432== 
Daughter destructed!
Child destructed!
Son destructed!
Child destructed!
Exception from Parent!
==2432== 
==2432== HEAP SUMMARY:
==2432==     in use at exit: 0 bytes in 0 blocks
==2432==   total heap usage: 6 allocs, 6 frees, 249 bytes allocated
==2432== 
==2432== All heap blocks were freed -- no leaks are possible
==2432== 
==2432== For counts of detected and suppressed errors, rerun with: -v
==2432== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 4 from 4)
{% endhighlight %}

最后，我们再来做一点改进，将自动销毁的任务交给智能指针`auto_ptr`来管理：
{% highlight cpp %}
// 版本四：利用智能指针auto_ptr自动清理heap上申请的内存
#include <iostream>
#include <string.h>
#include <stdexcept>
#include <memory>
using namespace std;

class Child
{
public:
    Child(const char* name)
    {
        m_name = new char[strlen(name) + 1]; // 申请内存
        strcpy(m_name, name); // 拷贝
    }
    virtual ~Child()
    {
        delete[] m_name; // 释放内存
        cout << "Child destructed!" << endl;
    }
    char* GetName()
    {
        return m_name;
    }
private:
    char* m_name;
};

class Son : public Child
{
public:
    Son(const char* name) : Child(name) {}
    virtual ~Son()
    {
        cout << "Son destructed!" << endl;
    }
};

class Daughter : public Child
{
public:
    Daughter(const char* name) : Child(name) {}
    virtual ~Daughter()
    {
        cout << "Daughter destructed!" << endl;
    }
};

class Parent
{
public:
    Parent(const char* son, const char* daughter) :
    m_son(son), // 初始化儿子
    m_daughter(new Daughter(daughter)) // 初始化女儿
    {
        throw runtime_error("Exception from Parent!"); // <=== 抛出异常
    }

    ~Parent()
    {
        cout << "Parent destructed!" << endl;
    }

    void PrintChildren()
    {
        cout << "My son is " << m_son.GetName() << endl; // 打印儿子
        cout << "My daughter is " << m_daughter->GetName() << endl; // 打印女儿
    }
private:
    Son m_son;
    auto_ptr<Daughter> m_daughter; // <=== 交给智能指针管理
};

int main()
{
    try
    {
        Parent* p = new Parent("Li Lei", "Han Meimei");
        p->PrintChildren(); // 打印
        delete p; // 释放内存
    }
    catch (exception& e)
    {
        cout << e.what() << endl;
    }
}
{% endhighlight%}

##析构函数抛出异常##
《More Effective C++》条款11指出，在两种情况下析构函数会被调用：

1. 在正常情况下被销毁，也就是当它离开了它的生存空间(Scope)或是被明确地删除(delete)；
2. 被exception处理机制——也就是被exception传播过程中的栈回退(Stack Unwind)机制——销毁。

在第2种情况下，如果栈回退过程中某个析构函数也抛出异常，C++会调用`terminate()`函数，直接将程序终止掉(太狠了)。

举个例子：
{% highlight cpp %}
nclude <iostream>
#include <string.h>
#include <stdexcept>
using namespace std;

class Baby
{
public:
    Baby(const char* name)
    {
        m_name = new char[strlen(name) + 1]; // 申请内存
        strcpy(m_name, name); // 拷贝
    }

    ~Baby()
    {
        throw runtime_error("Exception from ~Baby()!"); // <=== 析构函数抛出异常
        delete[] m_name; // 由于抛出异常，这行将不会被执行
        cout << "Baby destructed!" << endl;
    }

    void Drink()
    {
        cout << "I'm drinking!" << endl;
    }

    void Eat()
    {
        cout << "I'm eating!" << endl;
    }

    void Sleep()
    {
        throw runtime_error("Exception from Sleep()!"); // <=== 抛出异常
        cout << "I'm sleeping!" << endl;
    }
private:
    char* m_name;
};

int main()
{
    try
    {
        Baby b("Little Tomy");
        b.Drink();
        b.Eat();
        b.Sleep();
    }
    catch (exception& e)
    {
        cout << e.what() << endl;
    }
}
{% endhighlight %}

执行结果如下：
{% highlight text %}
yapianyu@yapianyu-pc:~/Desktop$ g++ ctorleak.cpp -g
yapianyu@yapianyu-pc:~/Desktop$ valgrind --leak-check=full ./a.out
... ...
I'm drinking!
I'm eating!
terminate called after throwing an instance of 'std::runtime_error'
  what():  Exception from ~Baby()!
... ...
==5256== HEAP SUMMARY:
==5256==     in use at exit: 285 bytes in 5 blocks
==5256==   total heap usage: 6 allocs, 1 frees, 317 bytes allocated
... ...
==5256== LEAK SUMMARY:
==5256==    definitely lost: 0 bytes in 0 blocks
==5256==    indirectly lost: 0 bytes in 0 blocks
==5256==      possibly lost: 280 bytes in 4 blocks
==5256==    still reachable: 5 bytes in 1 blocks
==5256==         suppressed: 0 bytes in 0 blocks
... ...
Aborted
{% endhighlight %}

可以看到程序不仅被终止，而且还有内存泄漏(析构函数`~Baby()`没有执行完整)。**因此强烈建议不要在析构函数中抛出异常，否则后果自负！**。

##结论##
当构造函数执行失败时，勇敢地抛出异常。此时：

1. 抛出异常的那个类的析构函数永远不会被调用；
2. 所有已构造完成的成员对象的析构函数会被自动调用；
3. 当前被构造的对象(如`Parent`)在抛出异常后，所占用的内存会被自动释放掉；
4. 良好的做法是，当构造函数中抛出异常，立即捕捉掉，同时释放内存，最后重新抛出。当然释放内存的操作可以交给智能指针管理。

对于析构函数，我们应该全力阻止其抛出异常，因为：

1. 这样可以避免`terminate()`函数在exception传播过程的栈回退机制中被调用；
2. 可以确保析构函数完成其应该完成的所有任务，不会中途退出。

##References##
[C++ FAQ 17.8 How can I handle a constructor that fails?](http://www.parashift.com/c++-faq-lite/ctors-can-throw.html)  
[C++ FAQ 17.10 How should I handle resources if my constructors may throw exceptions?](http://www.parashift.com/c++-faq-lite/selfcleaning-members.html)  
[Will the below code cause memory leak in c++](http://stackoverflow.com/questions/147572/will-the-below-code-cause-memory-leak-in-c)  
[C++ FAQ 17.9 How can I handle a destructor that fails?](http://www.parashift.com/c++-faq-lite/dtors-shouldnt-throw.html)
