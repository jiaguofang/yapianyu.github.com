---
layout: post
title: 经典算法之回溯算法
category: algorithm
tags: [algorithm, backtrack, eight queen problem, coloring problem]
---

回溯法本质上是一种搜索算法。首先定义解空间树，接着构造可能满足条件的分支，并调用剪枝函数验证当前分支是否具备进一步构造的可能性，从而降低搜索复杂度。

##八皇后问题##

> 八皇后问题，是一个古老而著名的问题，是回溯算法的典型例题。该问题是十九世纪著名的数学家高斯1850年提出：在8X8格的国际象棋上摆放八个皇后，使其不能互相攻击，即任意两个皇后都不能处于同一行、同一列或同一斜线上，问有多少种摆法。

{% highlight cpp %}
#include <iostream>
#include <cmath>
#include <vector>
using namespace std;

void output(vector<int> &v)
{
    for (int i = 0; i < v.size(); i++)
        cout << v[i] << " ";
    cout << endl;
}

bool validate(vector<int> &v)
{
    for (int i = 0; i < v.size(); i++)
        for (int j = i + 1; j < v.size(); j++)
            if (v[i] == v[j] || (j - i) == abs(v[i] - v[j]))
                return false;
    return true;
}

/******************************/
/*        Non-recursive       */
/******************************/
void backtrack(int n)
{
    vector<int> v;
    v.push_back(-1);
    while (true)
    {
        v.back()++; // increase the last element by one
        if (v.back() == n) // backtrack
        {
            v.pop_back();
            if (v.empty())
                break;
        }
        else if (validate(v)) // prune
        {
            if (v.size() == n) // find one solution
                output(v);
            else
                v.push_back(-1);
        }
    }
}

/******************************/
/*         Recursive          */
/******************************/
void DFS(vector<int> v, int n)
{
    for (int i = 0; i < n; i++)
    {
        v.push_back(i);
        if (validate(v)) // prune
        {
            if (v.size() == n)
                output(v);
            else
                DFS(v, n);
        }
        v.pop_back();
    }
}

void backtrack(int n)
{
    vector<int> v;
    DFS(v, n);
}

int main(void)
{
    backtrack(8);
}
{% endhighlight %}

##着色问题##

> m-着色判定问题：给定无向连通图G和m种不同的颜色。用这些颜色为图G的各顶点着色，每个顶点着一种颜色，是否有一种着色法使G中任意相邻的2个顶点着不同颜色?

> m-着色优化问题：若一个图最少需要m种颜色才能使图中任意相邻的2个顶点着不同颜色，则称这个数m为该图的色数。求一个图的最小色数m的问题称为m-着色优化问题。

显然，m-着色判定问题是m-着色优化问题的子问题。算法雷同，不再赘述。
