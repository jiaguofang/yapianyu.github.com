---
layout: post
title: 算法001. 把二叉查找树转变成排序的双向链表
category: algorithm
tags: algorithm tree list
---

###题目###
输入一棵二叉查找树，将该二叉查找树转换成一个排序的双向链表。要求不能创建任何新的结点，只调整指针的指向。

比如将二元查找树

![](/image/binary-search-tree-to-sorted-double-linked-list.png)

转换成双向链表4=6=8=10=12=14=16。

题目来源：[http://zhedahht.blog.163.com/blog/static/254111742007127104759245/](http://zhedahht.blog.163.com/blog/static/254111742007127104759245/)

###解题思路###
二叉查找树的特点是**所有左子树上的节点值小于根节点，所有右子树上的节点值大于根节点**。因此，如果左、右子树均已转换成排好序的双向链表，只需将根节点附加在左子树末尾，把右子树附加在根节点上。这样得到的链表就是排好序的双向链表。

###参考代码###
{% highlight cpp %}
#include <iostream>
using namespace std;

// define binary search tree node
struct Node
{
	int m_value;
	Node* m_left;
	Node* m_right;
};

// return <min, max> node pair within tree
pair<Node *, Node *> ConvertTreeToDoublelinkedList(Node* root)
{
	if (!root)
		return make_pair<Node *, Node *> (NULL, NULL);

	pair<Node *, Node *> left = make_pair<Node *, Node *> (root, root);
	if (root->m_left)
	{
		// convert left sub-tree
		left = ConvertTreeToDoublelinkedList(root->m_left);
		root->m_left = left.second;
		left.second->m_right = root;
	}

	pair<Node *, Node *> right = make_pair<Node *, Node *> (root, root);
	if (root->m_right)
	{
		// convert right sub-tree
		right = ConvertTreeToDoublelinkedList(root->m_right);
		root->m_right = right.first;
		right.first->m_left = root;
	}

	return make_pair<Node *, Node *> (left.first, right.second);
}
{% endhighlight %}
