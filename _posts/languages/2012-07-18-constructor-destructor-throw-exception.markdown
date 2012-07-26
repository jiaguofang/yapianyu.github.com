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
我们知道构造函数是没有返回值的，所以当构造函数执行失败时(比如参数错误，运行时错
误……)，为了**通知调用者**，抛出异常是最好的选择。先来看段正常的代码：

{% highlight cpp %}
// 版本一：正常代码，无异常
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

