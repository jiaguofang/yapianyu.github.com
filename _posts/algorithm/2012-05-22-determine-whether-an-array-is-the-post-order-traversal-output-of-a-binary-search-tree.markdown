---
layout: post
title: hht算法006. 判断数组是不是二叉查找树的后序遍历结果
category: algorithm
tags: algorithm BST post-order
---

###题目###
输入一个整数数组，判断该数组是不是二叉查找树的后序遍历结果。如果是返回true，否则返回false。

题目来源：[http://zhedahht.blog.163.com/blog/static/25411174200725319627/](http://zhedahht.blog.163.com/blog/static/25411174200725319627/)

###解题思路###
题目的意思相当于根据后序遍历结果重建二叉查找树。我们知道后序遍历的结点访问顺序是左子树->右子树->根结点，因此数组的最后一个元素必定是根结点。再加上二叉查找树的性质：**所有左子树上的节点值小于根节点，所有右子树上的节点值大于根节点**，于是可以从数组起始位置开始遍历，将每个元素与根结点比较，如果小于根结点，则它在左子树上，直到出现大于根结点的元素，这部分对应左子树；从这个元素开始到结尾前面一个元素为止，所有的元素必须大于根结点，这部分对应右子树。根据这样的划分，递归地判断左右两部分是否都是二叉查找树。

###参考代码###
{% highlight cpp %}
/**
 * Determine whether an array is the post order
 * traversal output of a binary search tree.
 *
 * @param a Array of integers.
 * @param left Start index of the array.
 * @param right End index (including) of the array
 * @return True if a is the post order traversal output
 *			of a binary search tree. False otherwise.
 */
bool IsPostOrderTraversal(int a[], int left, int right)
{
	if (left < 0 || right < 0 || left > right)
		return false;

	// the root of tree for post order traversal
	// must be the last element of the array
	int root = a[right];
	int left_end = left - 1;
	// left sub-tree must be less than root
	for (int i = left; i < right; i++)
		if (a[i] < root)
			left_end++;

	int right_start = left_end + 1;
	// right sub-tree must be larger than root
	for (int i = right_start; i < right; i++)
		if (a[i] < root)
			return false;

	// recursively determine whether left sub-tree
	// and right sub-tree are binary search tree
	return (left_end - left <= 0 || IsPostOrderTraversal(a, left, left_end))
			&& (right - 1 - right_start <= 0 || IsPostOrderTraversal(a, right_start, right - 1));
}
{% endhighlight %}
