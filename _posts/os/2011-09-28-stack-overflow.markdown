---
layout: post
title: 堆栈溢出(Stack Overflow)
category: os
tags: [stack overflow]
---

堆栈溢出的本质是**在某次函数调用中，编译器分配的空间超过了堆栈可以允许的范围**。堆栈的大小和许多因素有关：编程语言、机器架构、多线程……操作系统为每个线程分配独立的线程栈，一般情况下为1M。

##较大的堆栈变量##
局部变量一般在堆栈上创建，但是如果局部变量的size太大，比如int a[1000000]，就会发生堆栈溢出。这种情况下，最好从堆上分配这些空间。
{% highlight cpp %}
void foo()
{
    //int x[1000000]; ==>
    int *x = new int[1000000];
}
{% endhighlight %}

##递归(Recursion)层数太深##
无论堆栈大小是1M还是多少，总是有限的。从函数调用中栈帧的变化可以看出，每次递归调用都会将一些必要的信息保存到栈帧，比如寄存器ebp、局部变量、参数、返回地址等等。这些信息再少也会占用一定空间，因此，递归层数过深最终会耗尽堆栈资源，并导致堆栈溢出。
{% highlight cpp %}
#include <stdio.h>

int Sum(int n)
{
    if (n <= 0)
        return 0;
    return n + Sum(n - 1);
}

void main()
{
    Sum(1000000);
}
{% endhighlight %}
{% highlight nasm %}
Sum:
    pushl   %ebp
    movl    %esp, %ebp
    subl    $24, %esp ; 保证栈帧大小是16的倍数
    cmpl    $0, 8(%ebp) ; n <=> 0?
    jg  .L2
    movl    $0, %eax
    jmp .L3
.L2:
    movl    8(%ebp), %eax
    subl    $1, %eax
    movl    %eax, (%esp) ; 把n-1作为参数
    call    Sum
    addl    8(%ebp), %eax ; n + (n - 1)
.L3:
    leave
    ret
{% endhighlight %}
从汇编代码可以看出，每次调用Sum函数，都会占用32Bytes大小的栈帧。因此，Sum(1000000)总共占用大约32M的内存，在Linux下抛出**Segmentation Fault**错误。

对于情形二，除了用非递归的方法来消除堆栈溢出的危险，还可以将递归转化成尾递归，如下：
{% highlight cpp %}
int SumTail(int n, int s)
{
    if (n <= 0)
        return s;
    return SumTail(n - 1, n + s);
}
{% endhighlight %}
{% highlight nasm %}
SumTail:
    pushl   %ebp
    movl    %esp, %ebp
    movl    8(%ebp), %edx ; 参数n
    movl    12(%ebp), %eax ; 参数s
    testl   %edx, %edx
    jle .L2 ; n <= 0?
.L4:
    addl    %edx, %eax
    subl    $1, %edx // n - 1
    jne .L4 ; 注意：这里用了jne，没有用call
.L2:
    popl    %ebp
    ret
{% endhighlight %}
可以看到，SumTail函数多了一个参数s，它的作用是在递归调用时累积之前调用的结果，并将其传入下一次递归调用中。由于SumTail处于方法的最后位置，除两个参数外，不再需要任何其他信息。因此方法之前所积累下的各种状态对于递归调用结果已经没有任何意义，完全可以把本次方法中留在堆栈中的数据清除，把空间让给最后的递归调用。这就是所谓的“尾递归”。

尾递归有几种优化方式，上述用到的是转化为循环处理，还有一种是消除堆栈法，暂时不做介绍。

##References##
[http://en.wikipedia.org/wiki/Stack\_overflow](http://en.wikipedia.org/wiki/Stack_overflow)  
[http://en.wikipedia.org/wiki/Tail\_recursion](http://en.wikipedia.org/wiki/Tail_recursion)
