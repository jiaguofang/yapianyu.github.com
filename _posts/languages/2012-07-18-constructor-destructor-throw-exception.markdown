---
layout: post
title: C++构造函数和析构函数抛出异常
category: languages
tags: C++ 构造函数 析构函数 异常 栈回退 智能指针
---

之前似乎从未考虑过能否在构造函数和析构函数中抛出异常？会有什么后果？所以花了点时
间研究了一下，找资料时恰好在《Effective C++》条款10: *Prevent resource leaks in
constructors*和条款11: *Prevent exceptions from leaving destructors*找到类似章节，
遂搬来使用。

###构造函数抛出异常###
我们知道构造函数没有返回值，所以当构造函数执行失败时(比如参数错误，运行时错误
等等)，为了**通知调用者**，抛出异常是最好的选择。

假设正在执行如下代码，可以看到内存申请/释放和输出结果一切正常：
{% highlight cpp %}
// 版本一：构造函数无异常
#include <iostream>
#include <string>
using namespace std;

class Base
{
public:
    Base() :
    m_sobj("This is a string object!"),
    m_sptr(new string("This is a pointer to string!"))
    {
    }

    ~Base()
    {
        delete m_sptr;
        cout << "Base destructor!" << endl;
    }

    void Print()
    {
        cout << m_sobj << endl;
        cout << *m_sptr << endl;
    }
private:
    string m_sobj;
    string* m_sptr;
};

int main()
{
    Base* b = new Base();
    b->Print();
    delete b;
}
{% endhighlight %}
{% highlight text %}
yapianyu@yapianyu-pc:~/Desktop$ g++ ctorleak.cpp -g
yapianyu@yapianyu-pc:~/Desktop$ valgrind --leak-check=full ./a.out
==5570== Memcheck, a memory error detector
==5570== Copyright (C) 2002-2010, and GNU GPL'd, by Julian Seward et al.
==5570== Using Valgrind-3.6.1-Debian and LibVEX; rerun with -h for copyright info
==5570== Command: ./a.out
==5570== 
This is a string object!
This is a pointer to string!
Base destructor!
==5570== 
==5570== HEAP SUMMARY:
==5570==     in use at exit: 0 bytes in 0 blocks
==5570==   total heap usage: 4 allocs, 4 frees, 90 bytes allocated
==5570== 
==5570== All heap blocks were freed -- no leaks are possible
==5570== 
==5570== For counts of detected and suppressed errors, rerun with: -v
==5570== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 17 from 6)
{% endhighlight %}

我们修改一下代码，在构造函数中抛出一个异常：
{% highlight cpp %}
// 版本二：构造函数抛出异常
#include <iostream>
#include <string>
#include <stdexcept>
#include <memory>
using namespace std;

class Base
{
public:
    Base() :
    m_sobj("This is a string object!"),
    m_sptr(new string("This is a pointer to string!"))
    {
        throw runtime_error("Exception from Base!"); // 抛出异常
    }

    ~Base()
    {
        delete m_sptr;
        cout << "Base destructor!" << endl;
    }

    void Print()
    {
        cout << m_sobj << endl;
        cout << *m_sptr << endl;
    }
private:
    string m_sobj;
    string* m_sptr;
};

int main()
{
    try
    {
        Base* b = new Base();
        b->Print();
        delete b;
    }
    catch (exception& e)
    {
        cout << e.what() << endl;
    }
}
{% endhighlight %}

valgrind的执行结果如下，可以看到：  
1. 输出结果为`Exception from Base!`。与版本一相比，Print()和~Base()中的输出并没
   有打印出来，可见没有被调用到  
2. 第12行发生内存泄漏即`m_sptr(new string("This is a pointer to string!"))`。因
   为其析构函数~Base()没有被调用到

按照C++标准，**尚未完全构造好的对象，其析构函数永远不会被调用**。而此时对于
`Base* b = new Base()`，一旦异常抛出，b的值依然是null，所以即使在其catch里调用
`delete b`也不会调用Base的析构函数。现在唯一可以做的是在抛出异常时自己捕捉，自己
清理，清除后重新将异常抛出。
与版本一相比，Base的Print()和~Base()并没有被调用
{% highlight text %}
yapianyu@yapianyu-pc:~/Desktop$ g++ ctorleak.cpp -g
yapianyu@yapianyu-pc:~/Desktop$ valgrind --leak-check=full ./a.out
==5392== Memcheck, a memory error detector
==5392== Copyright (C) 2002-2010, and GNU GPL'd, by Julian Seward et al.
==5392== Using Valgrind-3.6.1-Debian and LibVEX; rerun with -h for copyright info
==5392== Command: ./a.out
==5392== 
Exception from Base!
==5392== 
==5392== HEAP SUMMARY:
==5392==     in use at exit: 45 bytes in 2 blocks
==5392==   total heap usage: 6 allocs, 4 frees, 227 bytes allocated
==5392== 
==5392== 45 (4 direct, 41 indirect) bytes in 1 blocks are definitely lost in loss record 2 of 2
==5392==    at 0x402842F: operator new(unsigned int) (vg_replace_malloc.c:255)
==5392==    by 0x8048C63: Base::Base() (ctorleak.cpp:12)
==5392==    by 0x8048B05: main (ctorleak.cpp:37)
==5392== 
==5392== LEAK SUMMARY:
==5392==    definitely lost: 4 bytes in 1 blocks
==5392==    indirectly lost: 41 bytes in 1 blocks
==5392==      possibly lost: 0 bytes in 0 blocks
==5392==    still reachable: 0 bytes in 0 blocks
==5392==         suppressed: 0 bytes in 0 blocks
==5392== 
==5392== For counts of detected and suppressed errors, rerun with: -v
==5392== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 17 from 6)
{% endhighlight %}
{% highlight cpp %}
// 版本三：构造函数抛出异常后立即释放heap上申请的内存
#include <iostream>
#include <string>
#include <stdexcept>
#include <memory>
using namespace std;

class Base
{
public:
    Base() :
    m_sobj("This is a string object!"),
    m_sptr(new string("This is a pointer to string!"))
    {
        try
        {
            throw runtime_error("Exception from Base!");
        }
        catch (...)
        {
            delete m_sptr;
            throw;
        }
    }

    ~Base()
    {
        delete m_sptr;
        cout << "Base destructor!" << endl;
    }

    void Print()
    {
        cout << m_sobj << endl;
        cout << *m_sptr << endl;
    }
private:
    string m_sobj;
    string* m_sptr;
};

int main()
{
    try
    {
        Base* b = new Base();
        b->Print();
        delete b;
    }
    catch (exception& e)
    {
        cout << e.what() << endl;
    }
}
{% endhighlight %}

{% highlight cpp %}
// 版本四：利用智能指针auto_ptr自动清理heap上申请的内存
#include <iostream>
#include <string>
#include <stdexcept>
#include <memory>
using namespace std;

class Base
{
public:
    Base() :
    m_sobj("This is a string object!"),
    m_sptr(new string("This is a pointer to string!"))
    {
        throw runtime_error("Exception from Base!");
    }

    ~Base()
    {
        cout << "Base destructor!" << endl;
    }

    void Print()
    {
        cout << m_sobj << endl;
        cout << *m_sptr << endl;
    }
private:
    string m_sobj;
    auto_ptr<string> m_sptr;
};

int main()
{
    try
    {
        Base* b = new Base();
        b->Print();
        delete b;
    }
    catch (exception& e)
    {
        cout << e.what() << endl;
    }
}
{% endhighlight%}

http://www.parashift.com/c++-faq-lite/ctors-can-throw.html

http://www.parashift.com/c++-faq-lite/dtors-shouldnt-throw.html

http://www.parashift.com/c++-faq-lite/selfcleaning-members.html

#include <iostream>
#include <string.h>
using namespace std;

class Son
{
public:
    Son(const char* name)
    {
        m_name = new char[strlen(name) + 1];
        strcpy(m_name, name);
    }
    ~Son()
    {
        delete m_name;
        cout << "Son destructed!" << endl;
    }
    char* GetName()
    {
        return m_name;
    }
private:
    char* m_name;
};

class Daughter
{
public:
    Daughter(const char* name)
    {
        m_name = new char[strlen(name) + 1];
        strcpy(m_name, name);
    }
    ~Daughter()
    {
        delete m_name;
        cout << "Daughter destructed!" << endl;
    }
    char* GetName()
    {
        return m_name;
    }
private:
    char* m_name;
};

class Parent
{
public:
    Parent(const char* son, const char* daughter) :
    m_son(son),
    m_daughter(new Daughter(daughter))
    {
    }

    ~Parent()
    {
        delete m_daughter;
        cout << "Parent destructed!" << endl;
    }

    void PrintChildren()
    {
        cout << "My son is " << m_son.GetName() << endl;
        cout << "My daughter is " << m_daughter->GetName() << endl;
    }
private:
    Son m_son;
    Daughter* m_daughter;
};

int main()
{
    Parent* p = new Parent("Li Lei", "Han Meimei");
    p->PrintChildren();
    delete p;
}

