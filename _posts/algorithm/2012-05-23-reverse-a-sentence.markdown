---
layout: post
title: hht算法007. 翻转句子中单词的顺序
category: algorithm
tags: [algorithm, reverse string]
---

##题目##
输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。句子中单词以空格符隔开。为简单起见，标点符号和普通字母一样处理。

例如输入"I am a student."，则输出"student. a am I"。

题目来源：[http://zhedahht.blog.163.com/blog/static/254111742007289205219/](http://zhedahht.blog.163.com/blog/static/254111742007289205219/)

##解题思路##
这答题的解法比较多，比如先分割后合并，或者使用堆栈……但是这些方法都需要额外申请内存，为了避免申请内存，可以先将句子整体反转，再将每个单词单独反转一遍，便可得到所要结果。**整个过程需要遍历句子三次**。

##参考代码##
{% highlight cpp %}
#include <cstring>

/**
 * Reverse a string.
 *
 * @param start Start pointer of the string.
 * @param end End pointer of the string.
 */
void Reverse(char* start, char* end)
{
	if (!start || !end)
		return;
	while (start < end)
	{
		char tmp = *end;
		*end = *start;
		*start = tmp;

		start++;
		end--;
	}
}

/**
 * Reverse a sentence, but keep the char order inside the word.
 *
 * @param s Pointer of the string.
 */
void ReverseSentence(char* s)
{
	if (!s)
		return;
	// reverse the whole sentence
	Reverse(s, s + strlen(s) - 1);
	for (char* start = s; *start != '\0'; start++)
	{
		// skip white space
		if (*start == ' ')
			continue;
		char* end = start;
		// loop until find '\0' or white space
		while (*(end + 1) != '\0' && *(end + 1) != ' ')
			end++;
		// now the word is the same as its original order
		Reverse(start, end);
		start = end;
	}
}
{% endhighlight %}
