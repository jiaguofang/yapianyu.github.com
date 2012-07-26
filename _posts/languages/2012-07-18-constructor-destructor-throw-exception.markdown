---
layout: post
title: C++构造函数和析构函数抛出异常
category: languages
tags: C++ 构造函数 析构函数 异常 栈回退 智能指针
---

之前似乎从未考虑过能否在构造函数和析构函数中抛出异常？会有什么后果？所以花了点时
间研究了一下，找资料时恰好在《Effective C++》条款10: Prevent resource leaks in
constructors和条款11: Prevent exceptions from leaving destructors找到类似章节，
遂搬来使用。

###构造函数抛出异常###
我们知道构造函数没有返回值，所以当构造函数执行失败时(比如参数错误，运行时错误
等等)，为了**通知调用者**，抛出异常是最好的选择。先来看段正常的代码：

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
==32130== Memcheck, a memory error detector
==32130== Copyright (C) 2002-2010, and GNU GPL'd, by Julian Seward et al.
==32130== Using Valgrind-3.6.1-Debian and LibVEX; rerun with -h for copyright info
==32130== Command: ./a.out
==32130== 
This is a string object!
This is a pointer to string!
Base destructor!
==32130== 
==32130== HEAP SUMMARY:
==32130==     in use at exit: 0 bytes in 0 blocks
==32130==   total heap usage: 4 allocs, 4 frees, 126 bytes allocated
==32130== 
==32130== All heap blocks were freed -- no leaks are possible
==32130== 
==32130== For counts of detected and suppressed errors, rerun with: -v
==32130== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 4 from 4)
{% endhighlight %}

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
        throw runtime_error("Exception from Base!");
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

