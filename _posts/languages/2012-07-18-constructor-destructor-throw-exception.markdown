---
layout: post
title: 构造函数、析构函数抛出异常
category: languages
tags: 构造函数 析构函数 异常
---

http://www.parashift.com/c++-faq-lite/ctors-can-throw.html

http://www.parashift.com/c++-faq-lite/dtors-shouldnt-throw.html

http://www.parashift.com/c++-faq-lite/selfcleaning-members.html

#include <iostream>
using namespace std;

class MyExp
{
public:
    MyExp(){}
};

class A
{
public:
    A(){}
    ~A(){cout << "A destructed!" << endl;}
};

class B
{
public:
    B(){}
    ~B(){cout << "B destructed!" << endl;}
};

//class C
//{
//public:
//  C(){}
//  ~C(){cout << "C destructed!\n";}
//};

class D
{
public:
    A* a;
    D(){a = new A(); B b;throw exception("D throws exception!");}
    ~D(){delete a;cout << "D destructed!" << endl;}
};

int main ()
{
    try
    {
        D d;
    }
    catch(exception& e)
    {
        cout << e.what() << endl;
    }
}
