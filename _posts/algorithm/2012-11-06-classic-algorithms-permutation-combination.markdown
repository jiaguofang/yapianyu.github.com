---
layout: post
title: 经典算法之排列组合算法 
category: algorithm
tags: [algorithm, permutation, combination]
---

###全排列###
####递归算法(支持重复元素)####
字符集{1，2，3}的全排列为123，132，213，231，321，312。先确定第一个位子的数，比如1，剩下{2，3}。接着确定{2，3}中的第一个位子，比如2，剩下{3}，那就是123。如果先取3，那就是132。把所有的可能都取一遍就是全排列了。

用递归的思想描述为，枚举第一个数，对后面的数递归地进行全排列。

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

####非递归算法(支持重复元素)####
非递归的方法采用构造法，从最小的排列开始，根据前一个排列找到比它略大的排列，假定要找8342666411的下一个排列，可以这样：

1. 从右向左检查相邻的两个数，找到第一对数，使得左边的数小于右边的数，将左边的数记为A——8342666411中的两个数是2和6  
2. 从右向左查找第一个比A大的数，记为B——8342666411中2右边第一个比2大的是4  
3. 交换A和B——变为8344666211  
4. 反转A位置之后的序列顺序——变为8344112666

重复上述步骤，直到找不到A为止。

{% highlight cpp %}
#include <iostream>
#include <cstring>
#include <cstdlib>
using namespace std;

// transform the range of elements [first, last) into the
// lexicographically next greater permutation of the elements
template<class T>
bool next_permutation(T * first, T * last)
{
    if (first >= last)
        return false;

    // given two adjacent elements [i-1] and [i], scan from right side to left,
    // find the first pair where [i-1] < [i]
    T* pA = last - 1;
    while (pA - 1 >= first && *(pA - 1) >= *pA)
        pA--;
    if (pA == first)
        return false;
    pA--;

    // find the first element larger than pA from right side
    T* pB = last - 1;
    while (*pA >= *pB)
        pB--;

    // swap
    T t = *pA;
    *pA = *pB;
    *pB = t;

    // reverse
    for (T * pFront = pA + 1, *pEnd = last - 1; pFront < pEnd; pFront++, pEnd--)
    {
        T t = *pFront;
        *pFront = *pEnd;
        *pEnd = t;
    }

    return true;
}

int compar(const void * a, const void * b)
{
    return (*(char*) a - *(char*) b);
}

void permute(char* s)
{
    qsort(s, strlen(s), sizeof(char), &compar);
    do
    {
        cout << s << endl;
    } while (next_permutation(s, s + strlen(s)));
}

int main()
{
    char s[] = "12233";
    permute(s);
}
{% endhighlight %}

###组合(n选m)###
####递归算法####
字符集{1，2，3，4}选两个元素的所有组合为12，13，14，23，24，34。第一种取法先取第1个位子的数，接着在第2、3、4个位子取1个数(就是12，13，14)。第二种取法先取第2个位子的数，接着在第3、4个位子取1个数(23，24)。第三种取法先取第3个位子的数，接着在第4个位子取1个数(就是34)。

用递归的思想描述为，从头往尾取一个数，在这个数后面的数中递归地选出m-1个数的组合。

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
