---
layout: post
title: 经典算法之排列组合算法 
category: algorithm
tags: [algorithm, permutation, combination]
---

##排列##

###全排列(不含重复元素)###
####递归####
{% highlight cpp %}
#include <iostream>
#include <cstring>
using namespace std;

void swap(char* s, int a, int b)
{
    char c = s[a];
    s[a] = s[b];
    s[b] = c;
}

void permute(char* s, int stub)
{
    if (stub + 1 == strlen(s))
    {
        cout << s << endl;
        return;
    }
    for (int i = stub; i < strlen(s); i++)
    {
        swap(s, stub, i);
        permute(s, stub + 1);
        swap(s, stub, i);
    }
}

int main()
{
    char s[] = "1223";
    permute(s, 0);
}
{% endhighlight %}

####非递归####
构造法

###全排列(含重复元素)###
{% highlight cpp %}
#include <iostream>
#include <cstring>
using namespace std;

void swap(char* s, int a, int b)
{
    char c = s[a];
    s[a] = s[b];
    s[b] = c;
}

bool duplicated(char* s, int stub, int another)
{
    if (stub == another)
        return false;
    if (s[stub] == s[another])
        return true;
    for (int i = stub + 1; i < another; i++)
        if (s[i] == s[another])
            return true;
    return false;
}

void permute(char* s, int stub)
{
    if (stub + 1 == strlen(s))
    {
        cout << s << endl;
        return;
    }
    for (int i = stub; i < strlen(s); i++)
        if (!duplicated(s, stub, i))
        {
            swap(s, stub, i);
            permute(s, stub + 1);
            swap(s, stub, i);
        }
}

int main()
{
    char s[] = "1223";
    permute(s, 0);
}
{% endhighlight %}

http://www.cnblogs.com/autosar/archive/2012/04/08/2437799.html

http://dongxicheng.org/structure/permutation-combination/
