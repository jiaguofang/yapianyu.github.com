---
layout: post
title: 经典算法之排列组合算法 
category: algorithm
tags: [algorithm, permutation, combination]
---

###全排列(支持重复元素)###
递归算法：字符集{1，2，3}的全排列为123，132，213，231，321，312。方法是，先确定第一个位子的数，比如1，剩下{2，3}。接着确定{2，3}中的第一个位子，比如2，剩下{3}。把所有的可能都取一遍就是全排列了。

![Permutation](/images/permutation-algorithm.png)

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
        if (!duplicated(s, stub, i)) // support duplicated elements
        {
            swap(s, stub, i); // each element s[i] is going to exchanged with s[stub]
            permute(s, stub + 1); // permute behind s[stub]
            swap(s, stub, i);
        }
}

int main()
{
    char s[] = "1223";
    permute(s, 0);
}
{% endhighlight %}

非递归算法：构造法
http://www.cnblogs.com/autosar/archive/2012/04/08/2437799.html

###组合(n选m)###
递归算法：字符集{1，2，3，4}选两个元素的所有组合为12，13，14，23，24，34。方法是，第一轮先取出1，剩下{2，3，4}。接着从{2，3，4}中取2构成12，取3构成13，取4构成14。第二轮先取2，剩下{3，4}。接着从{3，4}中取3构成23，取4构成24。重复上述过程。

因此，整个过程可以归纳为，第一轮取第一个元素，再从剩下的n-1个元素中选m-1个元素。第二轮取第二个元素，再从剩下的n-2个元素中选m-1个元素。反复做，直到某轮只剩下m个元素，则该轮只有一种取法。

![Combination](/images/combination-algorithm.png)

{% highlight cpp %}
#include <iostream>
#include <cstring>
using namespace std;

void combine(char* s, int si, char* d, int di, int n, int m)
{
    for (int i = si; i <= strlen(s) - m; i++)
    {
        d[di] = s[i];
        if (m - 1 == 0)
            cout << d << endl;
        else
            combine(s, i + 1, d, di + 1, n - 1, m - 1);
    }
}

int main()
{
    char s[] = "123456";
    char d[3] = { 0 };
    combine(s, 0, d, 0, 6, 2);
}
{% endhighlight %}
