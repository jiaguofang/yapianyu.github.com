---
layout: post
title: C/C++内存泄漏检测
category: languages
tags: c c++ memory leak
---

本文利用MS C-Runtime Library的Debug function对内存泄漏进行检测。

### Step 1. 创建LeakWatcher.h

将new和malloc重新定义，malloc最终会调用_malloc_dbg，此时传入文件名和行号就能准确地定位内存泄漏的代码

{% highlight cpp %}
#ifndef LEAKWATCHER
#define LEAKWATCHER

#include <crtdbg.h>

#ifdef _DEBUG

void* operator new(size_t nSize, const char * lpszFileName, int nLine)
{
    return ::operator new(nSize, _NORMAL_BLOCK, lpszFileName, nLine);
}
#define DEBUG_NEW new(THIS_FILE, __LINE__)

#define MALLOC_DBG(x) _malloc_dbg(x, _NORMAL_BLOCK, THIS_FILE, __LINE__);
#define malloc(x) MALLOC_DBG(x)

#endif // _DEBUG

#endif // LEAKWATCHER
{% endhighlight %}

### Step 2. 在每个待检测的CPP中加入如下宏定义，并在程序入口处调用_CrtSetDbgFlag

{% highlight cpp %}
#include "LeakWatcher.h"

// 仿照MFC自动生成的代码
#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
// 如果在CPP中有多处new，则它们用同一个字符串THIS_FILE；
// 否则如果每处都用__FILE__，则会生成多个字符串。
static char THIS_FILE[] = __FILE__;
#endif

void main()
{
	// 在程序退出时自动调用_CrtDumpMemoryLeaks()
	_CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);
	// new的内部实现是通过调用malloc，最终会调用_malloc_dbg
	int *p1 = new int[10];
	// 最终会调用_malloc_dbg(sizeof(int) * 10, _NORMAL_BLOCK, THIS_FILE, __LINE__);
	int *p2 = (int *)malloc(sizeof(int) * 10);
}
{% endhighlight %}

输出：

{% highlight cpp %}
Detected memory leaks!
Dumping objects ->
main.cpp(19) : {94} normal block at 0x00447D08, 40 bytes long.
 Data: <                > CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD 
main.cpp(17) : {93} normal block at 0x00447CA0, 40 bytes long.
 Data: <                > CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD 
Object dump complete.
{% endhighlight %}

References：  
[http://dev.yesky.com/147/2356147_2.shtml](http://dev.yesky.com/147/2356147_2.shtml)  
[http://www.codeproject.com/Articles/2319/Detailed-memory-leak-dumps-from-a-console-app](http://www.codeproject.com/Articles/2319/Detailed-memory-leak-dumps-from-a-console-app)
