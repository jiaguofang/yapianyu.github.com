---
layout: post
title: 经典算法之排列组合算法 
category: algorithm
tags: [algorithm, permutation, combination]
---

##排列##

###全排列###
{% highlight cpp %}
#include <iostream>
#include <cstring>
using namespace std;

void swap(char* s, int a, int b)
{
    char c = *(s + a);
    *(s + a) = *(s + b);
    *(s + b) = c;
}

void permute(char* s, int idx)
{
    if (idx + 1 == strlen(s))
    {
        cout << s << endl;
        return;
    }
    for (int i = idx; i < strlen(s); i++)
    {
        swap(s, idx, i);
        permute(s, idx + 1);
        swap(s, idx, i);
    }
}

int main()
{
    char s[] = "123";
    permute(s, 0);
}
{% endhighlight %}

http://www.cnblogs.com/autosar/archive/2012/04/08/2437799.html

http://dongxicheng.org/structure/permutation-combination/
