---
layout: post
title: hht算法009. 链表中倒数第k个结点
category: algorithm
tags: [algorithm, list]
---

##题目##
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

##解题思路##
传统方法——第一次遍历求得链表长度，第二次遍历输出第n-k个结点。快慢指针——快指针先走k步，接着慢指针也开始走，直到快指针走到链表末尾，此时慢指针所指结点就是倒数第k个结点。

**其实，传统方法和快慢指针的时间复杂度相同。区别在于前者需要遍历两次，而后者只需遍历一次。**

##参考代码##
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

##扩展##
1. **输出单向链表的中间结点(如果链表结点数为奇数，输出中间结点；如果链表结点数为偶数，输出中间两个结点前面的一个)。**  
**思路：**快慢指针——快指针每次走两步，慢指针每次走一步，当快指针链表末尾时，慢指针指向的就是中间结点。

2. **判断两个单向链表是否相交，如果相交则输出相交的第一个结点(两个链表都不存在环)。**  
**思路：**先将两个链表遍历一遍，得到链表的长度len_a和len_b(假设len_a>=len_b)。接着从头开始在a上行走(len_a-len_b)步，然后a和b同时走，如果两个链表相交，则指针p_a和p_b指向的结点必定相同。否则不相交。

3. **判断单向链表是否有环，如果有环则输出环的入口结点。**  
**思路：**快慢指针——快指针每次走两步，慢指针每次走一步。如果链表上有环，则快指针必先于慢指针进入环内，且在环上行走足够多的步数之后，快指针与慢指针重合。  
**再求环的入口结点，假设表头距离入口结点m，环长n，先让快指针从表头走n步，此时快指针距离入口结点m-n。接着慢指针指向表头，并和快指针同时行走，当慢指针走过m步时，快慢指针恰好于入口处重合。**
