---
layout: post
title: 编程获取获取CPU使用率、内存使用量、uptime、线程数等系统状态参数
category: tech
tags: CPU使用率 内存使用量 uptime 线程数 C++
---

为了监控公司某项目的运行情况，需要在进程内部获取系统和进程的状态参数，谷歌一番之后在这里做个总结。内容有：

* 系统CPU使用率  
* 系统物理内存总大小  
* 系统物理内存使用量  
* 进程CPU使用率  
* 进程物理内存使用量  
* 进程uptime  
* 进程内部的线程数

文中会给出Windows和Linux平台上的解决办法。

###系统CPU使用率###
关于CPU使用率，首先必须澄清并不存在现成的API，如GetSystemCPUUsage()或者
GetProcessCPUUsage()。打开任务管理器，可以看到：

* CPU使用率包括系统的CPU使用率和进程的CPU使用率
* CPU使用率每隔一定时间会刷新一次  
![](/image/windows-task-manager.png)  
也就是说，CPU使用率是采样周期内CPU忙（非空闲）的时间和采样周期的比值，是一个平均
值。因此，系统CPU使用率可以描述为采样周期内系统运行于非idle状态下的时间和采样周
期的比值。

####Windiws Platform####
MSDN对Windows API
[GetSystemTimes](http://msdn.microsoft.com/en-us/library/ms724400(VS.85\).aspx)
是这样描述的：
{% highlight cpp %}
/**
 * Retrieves system timing information. On a multiprocessor system, the values
 * returned are the sum of the designated times across all processors.
 *
 * @param lpIdleTime A pointer to a FILETIME structure that receives the amount
 * of time that the system has been idle.
 * @param lpKernelTime A pointer to a FILETIME structure that receives the
 * amount of time that the system has spent executing in Kernel mode (including
 * all threads in all processes, on all processors). This time value also
 * includes the amount of time the system has been idle.
 * @param lpUserTime A pointer to a FILETIME structure that receives the amount
 * of time that the system has spent executing in User mode (including all
 * threads in all processes, on all processors).
 */
BOOL WINAPI GetSystemTimes(LPFILETIME lpIdleTime, LPFILETIME lpKernelTime, LPFILETIME lpUserTime);
{% endhighlight %}
注意到三点：

* GetSystemTimes的每个返回值表示所有处理器上的时间和
* t_idle已经包括在t_kernel中
* t_kernel + t_user ≈ 采样周期 × 处理器数量

因此，**CPU% = (Δt_kernel + Δt_user - Δt_idle) / (Δt_kernel + Δt_user)**。  
代码请参考[double
SystemInfo::GetSystemCPUUsage()](https://github.com/yapianyu/system-metrics/blob/master/src/SystemInfo.cpp)
