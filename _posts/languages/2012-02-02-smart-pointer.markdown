---
layout: post
title: 理解智能指针(Smart Pointer)
category: languages
tags: 智能指针 内存泄漏
---

在堆上申请内存就像拿信用卡透支，如果隔三岔五透支(申请内存)却不还款(释放内存)，迟早
有一天会达到上限，到时候再想透支就不行了(Out of Memory)。如果不用智能指针，堆上申
请的内存必须手动释放，否则就是内存泄漏。智能指针就像自动还款一样，在适当的时机向
银行归还透支金额(释放内存)。

###重新造一个轮子###
智能指针的原理很简单，即在指针外面增加一层包装类(wrapper)，一般来说此类的对象都在
栈上生成，当它被自动回收的时候，将指针指向的内存也释放掉。

####引用计数####
引用计数用来记录原始指针指向的内存当前被引用的次数。当引用计数减为0时，通过调用
delete this显式地将自己回收。

{% highlight cpp %}
#ifndef REFERENCECOUNTED_H
#define REFERENCECOUNTED_H

class ReferenceCounted
{
public:
    ReferenceCounted()
    {
        m_counter = 0;
    }
    void AddRef()
    {
        ++m_counter;
    }
    void Release()
    {
        if (m_counter > 0 && --m_counter == 0)
        {
            delete this;
        }
    }
protected:
    // 虚析构函数！
    virtual ~ReferenceCounted(){}
private:
    // 引用计数
    int m_counter;
};

#endif
{% endhighlight %}

所有用户类都必须继承自ReferenceCounted，比如：
{% highlight cpp %}
#include "ReferenceCounted.h"

class MyClass : public ReferenceCounted
{
public:
    MyClass()
    {
        m_num = 0;
    }
    int MyFoo()
    {
        ++m_num;
        return m_num;
    }
    int m_num;
protected:
    ~MyClass(){}
};
{% endhighlight %}

####SmartPtr####
SmartPtr被定义为类模板，可以接收任何继承自ReferenceCounted的用户类。
{% highlight cpp %}
#ifndef SMARTPTR_H
#define SMARTPTR_H

// T必须是一个继承自ReferenceCounted的类
template <class T>
class SmartPtr
{
public:
    // 构造函数一：传入类型为T的指针来构造SmartPtr，必须保证p所指向的内容在堆上
    SmartPtr(T *p);
    // 构造函数二：拷贝构造函数，用一个SmartPtr来构造另一个SmartPtr
    SmartPtr(const SmartPtr<T> &p);
    ~SmartPtr();
    // 重载赋值运算符一：必须保证p所指向的内容在堆上
    SmartPtr& operator =(const T *p);
    // 重载赋值运算符二：智能指针间赋值
    SmartPtr& operator =(const SmartPtr<T> &p);
    // 重载取值运算符
    T& operator *() const;
    // 重载成员选择运算符
    T* operator ->() const;
    // 重置SmartPtr的内部指针
    void Reset(T *p = NULL);
    // 获得内部指针
    T* Get();
private:
    void Dispose();
    T *m_ptr;
};

// 用法
// T *p = new T();
// SmartPtr<T> sp(p);
template <class T>
SmartPtr<T>::SmartPtr(T *p)
{
    m_ptr = p;
    if (m_ptr)
    {
        m_ptr->AddRef();
    }
}

// 用法
// T *p = new T();
// SmartPtr<T> sp1(p);
// SmartPtr<T> sp2(sp1);
template <class T>
SmartPtr<T>::SmartPtr(const SmartPtr<T> &p)
{
    m_ptr = p.m_ptr;
    if (m_ptr)
    {
        m_ptr->AddRef();
    }
}

template <class T>
SmartPtr<T>::~SmartPtr()
{
    Dispose();
}

template <class T>
SmartPtr<T>& SmartPtr<T>::operator =(const T *p)
{
    Reset(p);
    return *this;
}

template <class T>
SmartPtr<T>& SmartPtr<T>::operator =(const SmartPtr<T> &p)
{
    Reset(p.m_ptr);
    return *this;
}

template <class T>
T& SmartPtr<T>::operator *() const
{
    return *m_ptr;
}

template <class T>
T* SmartPtr<T>::operator ->() const
{
    return m_ptr;
}

template <class T>
void SmartPtr<T>::Reset(T *p)
{
    // 避免自赋值self-assignment
    if (m_ptr != p)
    {
        Dispose();
        m_ptr = p;
        if (m_ptr)
        {
            m_ptr->AddRef();
        }
    }
}

template <class T>
T* SmartPtr<T>::Get()
{
    return m_ptr;
}

template <class T>
void SmartPtr<T>::Dispose()
{
    if (m_ptr)
    {
        m_ptr->Release();
    }
}

#endif
{% endhighlight %}

####测试代码####
{% highlight cpp %}
#include "SmartPtr.h"
#include "MyClass.h"
#include <crtdbg.h>
#include <vector>
#include <iostream>
using namespace std;

void main(void)
{
    {
        MyClass *rc1 = new MyClass();
        SmartPtr<MyClass> sp1 = rc1; // 普通构造函数
        SmartPtr<MyClass> sp2 = sp1; // 拷贝构造函数
        SmartPtr<MyClass> sp3(rc1); // 普通构造函数
        SmartPtr<MyClass> sp4(sp1); // 拷贝构造函数

        MyClass *rc2 = new MyClass();
        sp3 = rc2; // 赋值运算符
        sp4 = sp3; // 赋值运算符

        cout << (*sp1).MyFoo() << endl; // 像指针一样操作
        cout << sp1->MyFoo() << endl;

        vector<SmartPtr<MyClass>> v; // 在vector中的表现
        v.push_back(sp1);
        v.push_back(sp2);
        v.push_back(sp3);
        v.push_back(sp4);
    }
    _CrtDumpMemoryLeaks();
}
{% endhighlight %}

###STL智能指针###
http://www.codeguru.com/cpp/cpp/cpp_mfc/stl/article.php/c15361/A-TR1-Tutorial-Smart-Pointers.htm
