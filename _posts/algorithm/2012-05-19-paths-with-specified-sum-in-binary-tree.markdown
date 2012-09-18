---
layout: post
title: hht算法004. 二元树中和为某一值的所有路径
category: algorithm
tags: algorithm tree
---

##题目##
输入一个整数和一棵二元树。从树的根结点开始往下访问一直到**叶结点**所经过的所有结点形成一条路径。打印出和与输入整数相等的所有路径。

例如输入整数22和如下二元树  

![](/images/paths-with-specified-sum-in-binary-tree.png)

则打印出两条路径：10, 12和10, 5, 7。

题目来源：[http://zhedahht.blog.163.com/blog/static/254111742007228357325/](http://zhedahht.blog.163.com/blog/static/254111742007228357325/)

##解题思路##
简单递归，需要注意的是必须遍历到叶结点，不能是中间结点。

##参考代码##
{% highlight cpp %}
#include <iostream>
#include <deque>
using namespace std;

struct Node
{
	int m_value;
	Node* m_left;
	Node* m_right;
};

deque<int> path;

void PrintPath(Node* root, int sum)
{
	if (!root)
		return;
	path.push_back(root->m_value);
	// is leaf
	if (sum == root->m_value && root->m_left && root->m_right)
	{
		for (unsigned int i = 0; i < path.size(); i++)
			cout << path.at(i) << " ";
		cout << endl;
	}
	PrintPath(root->m_left, sum - root->m_value);
	PrintPath(root->m_right, sum - root->m_value);
	path.pop_back();
}
{% endhighlight %}
