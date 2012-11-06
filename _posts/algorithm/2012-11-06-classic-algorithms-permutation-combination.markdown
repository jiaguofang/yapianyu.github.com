---
layout: post
title: 经典算法之排列组合算法 
category: algorithm
tags: 排列 组合
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
