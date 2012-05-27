---
layout: post
title: hht算法009. 链表中倒数第k个结点
category: algorithm
tags: algorithm list
---

###题目###
输入一个单向链表，输出该链表中倒数第k个结点。链表的倒数第0个结点为链表的尾指针。

链表结点定义如下：  
{% highlight cpp %}
struct ListNode
{
    int m_nKey;
    ListNode* m_pNext;
};
{% endhighlight %}

题目来源：
[http://zhedahht.blog.163.com/blog/static/2541117420072114478828/](http://zhedahht.blog.163.com/blog/static/2541117420072114478828/)

###解题思路###
经典题目，两指针一前一后，前指针先走k步，接着后指针也开始走，直到前指针走到链表末尾，此时后指针就是倒数第k个结点。

###参考代码###
{% highlight cpp %}
struct ListNode
{
    int m_nKey;
    ListNode* m_pNext;
};

ListNode* FindLastKth(ListNode* head, unsigned int k)
{
    if (!head)
        return NULL;
    
    ListNode* p0 = head;
    for (int i = 0; i < k && !p0; i++)
        p0 = p0->m_pNext;
    if (!p0)
        return NULL;

    ListNode* p1 = head;
    while (!p0->m_pNext)
    {
        p0 = p0->m_pNext;
        p1 = p1->m_pNext;
    }

    return p1;
}
{% endhighlight %}
