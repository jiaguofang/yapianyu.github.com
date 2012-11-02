---
layout: post
title: 经典算法——回溯算法
category: algorithm
tags: 回溯
---

八皇后问题
{% highlight cpp %}
#include <iostream>
#include <cmath>
using namespace std;

void output(int x[], int n)
{
    for (int i = 0; i < n; i++)
        cout << x[i] << " ";
    cout << endl;
}

bool validate(int x[], int n)
{
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++)
            if (x[i] == x[j] || (j - i) == abs(x[i] - x[j]))
                return false;
    return true;
}

void backtrack(int n)
{
    int* x = new int[n];
    for (int i = 0; i < n; i++)
        x[i] = 0;

    int idx = 0;
    while (idx >= 0)
    {
        x[idx]++;
        if (x[idx] > n)
        {
            x[idx] = 0; // must reset this
            idx--; // backtrack
        }
        else if (validate(x, idx + 1)) // pruning...same row, column, diagonal?
        {
            idx++;
            if (idx >= n) // find one solution
            {
                output(x, n);
                idx--;
            }
        }
    }
    delete[] x;
}

int main(void)
{
    backtrack(8);
}
{% endhighlight %}

穷举
组合
回溯
