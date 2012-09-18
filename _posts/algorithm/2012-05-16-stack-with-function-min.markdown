---
layout: post
title: hht算法002. 设计包含min函数的栈
category: algorithm
tags: algorithm stack
---

##题目##
定义栈的数据结构，要求添加一个min函数，能够得到栈的最小元素。要求函数min、push以及pop的时间复杂度都是O(1)。

题目来源：[http://zhedahht.blog.163.com/blog/static/25411174200712895228171/](http://zhedahht.blog.163.com/blog/static/25411174200712895228171/)

##解题思路##
创建辅助栈，此栈的栈顶元素定义为**当前数据栈最小元素**。即假设辅助栈当前有n个元素，则辅助栈栈顶元素表示数据栈n个元素的最小值。当执行push操作时，总是与辅助栈的栈顶元素比较，如果小于辅助栈栈顶元素，则将该值push到辅助栈，否则将辅助栈栈顶元素再次push到辅助栈。当执行pop操作时，将数据栈和辅助栈同时pop。当执行min操作时，辅助栈栈顶元素就是所求。

##参考代码##
{% highlight cpp %}
#include <stack>
using namespace std;

template<typename T> class StackWithMin
{
public:
	void push(const T&);
	void pop();
	T& top();
	T& min();
private:
	stack<T> m_data; // store actual data
	stack<T> m_min; // store minimum data
};

template<typename T> void StackWithMin<T>::push(const T& value)
{
	m_data.push(value);

	if (m_min.empty() || value < m_min.top())
		m_min.push(value);
	else
		m_min.push(m_min.top());
}

template<typename T> void StackWithMin<T>::pop()
{
	m_data.pop();
	m_min.pop();
}

template<typename T> T& StackWithMin<T>::min()
{
	return m_min.top();
}

template<typename T> T& StackWithMin<T>::top()
{
	return m_data.top();
}
{% endhighlight %}
