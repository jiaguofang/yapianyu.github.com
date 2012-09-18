---
layout: post
title: hht算法003. 求最大子数组和
category: algorithm
tags: algorithm dp
---

##题目##
输入一个整形数组，数组里有正数也有负数。数组中连续的一个或多个整数组成一个子数组，每个子数组都有一个和。求所有子数组的和的最大值。要求时间复杂度为O(n)。

题目来源：[http://zhedahht.blog.163.com/blog/static/254111742007219147591/](http://zhedahht.blog.163.com/blog/static/254111742007219147591/)

##解题思路##
假设f(i)表示以a[i]结尾的某个子数组的最大和，那么f(i)可以表示为：

1. 当i=0，f(i) = a[0]；
2. 当i>0，f(i) = max(f(i-1) + a[i], a[i])；

所有子数组的最大和就是max(f(i))。

##参考代码##
{% highlight cpp %}
#include <algorithm>
using namespace std;

int MaxSum(int a[], int n)
{
	int max_sum = 0;
	int sum_i = 0; // max sum of the sub-array ended with a[i]
	for (int i = 0; i < n; i++)
	{
		if (i == 0)
		{
			sum_i = a[0];
			max_sum = a[0];
		}
		else
		{
			sum_i = max(a[i], sum_i + a[i]);
			max_sum = max(max_sum, sum_i);
		}
	}

	return max_sum;
}
{% endhighlight %}
