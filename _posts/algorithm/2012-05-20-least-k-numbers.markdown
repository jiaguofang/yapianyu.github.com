---
layout: post
title: 算法005. 查找最小的k个数
category: algorithm
tags: algorithm
---

###题目###
输入大小为n的数组，输出：  
1\. 最小的k个数  
2\. 第k小的数  
3\. 中位数

题目来源：[http://zhedahht.blog.163.com/blog/static/2541117420072432136859/](http://zhedahht.blog.163.com/blog/static/2541117420072432136859/)

###解题思路###
题目中的三个问题本质上是同一个，本文以第一个为例。下面分析是该问题的若干解法：  
1\. 类似[选择排序](http://en.wikipedia.org/wiki/Selection_sort)的思想，求最小的k个数，只要遍历数组k次，每次取得最小值。**时间复杂度为O(kn)**。  
2\. 先排序再从小到大选择k个数。**时间复杂度为O(nlogn)**。  
3\. 遍历数组，用容量为k的最大堆存放最小的k个数。最大堆未满时，将当前整数插入最大堆。否则比较当前整数和最大堆的堆顶值，如果小于堆顶值就删除堆顶元素并插入当前整数。由于最大堆的插入删除操作均耗时O(logn)(这里为**O(logk)**)，因此该算法的**时间复杂度为O(nlogk)**。该算法的另一优点是，当n特别巨大(不能全部载入内存)而k相对较小时，容量为k的最大堆能全部放在内存中，无须做额外处理。  
4\. [快速选择](http://en.wikipedia.org/wiki/Selection_algorithm)：利用快速排序的思想，选取合适的枢纽元(三数中值、随机)，每次将数组分为小于枢纽元(k1)和大于枢纽元(k2)两部分。如果k小于k1，则递归地在前半部分查找最小的k个数。否则在后半部分查找最小的k-k2个数。该算法的**平均时间复杂度为O(n)**。*值得注意的是快速选择(排序)算法需要把整个数组全部载入内存，当n特别巨大时，该算法不能适用*。

###参考代码###
{% highlight cpp %}
#include <algorithm>
using namespace std;

int median3(int a[], int left, int right)
{
	int center = (left + right) / 2;
	// left is median of 3
	if (a[left] >= a[center] && a[left] <= a[right])
	{
		swap(a[left], a[right]);
	}
	// center is media of 3
	else if (a[center] >= a[left] && a[center] <= a[right])
	{
		swap(a[center], a[right]);
	}
	// right is media of 3, and we do nothing

	return a[right];
}

void quick_select(int a[], int left, int right, int k)
{
	if (left > right || k >= right - left + 1)
		return;

	int pivot = median3(a, left, right);
	// index to be exchanged with current scanning one
	int exchange_index = left;
	for (int i = left; i < right; i++)
		if (a[i] <= pivot)
		{
			swap(a[i], a[exchange_index]);
			exchange_index++;
		}

	// swap the pivot to the correct position
	swap(a[right], a[exchange_index]);

	// unlike quick sort, we only take a single recursive call
	if (k < exchange_index - left)
		quick_select(a, left, exchange_index - 1, k);
	else if (k > exchange_index - left + 1)
		quick_select(a, exchange_index + 1, right, k - (exchange_index - left
				+ 1));
}
{% endhighlight %}
