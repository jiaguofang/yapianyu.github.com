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

####Windiws####
MSDN对
[GetSystemTimes()](http://msdn.microsoft.com/en-us/library/ms724400\(VS.85\).aspx)
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
BOOL WINAPI GetSystemTimes(
    __out_opt  LPFILETIME lpIdleTime,
    __out_opt  LPFILETIME lpKernelTime,
    __out_opt  LPFILETIME lpUserTime
);
{% endhighlight %}
注意到三点：

* GetSystemTimes的每个返回值都是所有处理器上的时间和
* t\_idle已经包括在t\_kernel中
* t\_kernel + t\_user ≈ 采样周期 × 处理器数量

因此，**CPU% = (Δt\_kernel + Δt\_user - Δt\_idle) / (Δt\_kernel + Δt\_user)**。

代码参见[double
SystemInfo::GetSystemCPUUsage()](https://github.com/yapianyu/system-metrics/blob/master/src/SystemInfo.cpp)

####Linux####
Linux将各种系统状态统计信息保存在/proc/stat文件中，比如CPU运行情况、中断统计、启
动时间、上下文切换次数、运行中的进程等等。在我的ubuntu 11.10中运行`cat /proc/stat`
，输出如下：
{% highlight text %}
> cat /proc/stat
cpu  204090 1068 325035 33868070 91116 2 5869 0 0 0
cpu0 102357 590 156005 16939090 41971 2 2736 0 0 0
cpu1 101732 478 169029 16928980 49145 0 3132 0 0 0
intr 41083974 622 43673 0 1 17 0 3 0 1 0 0 0 619393 0 0 521381 55283 312957 115
510993
ctxt 168458112
btime 1341372511
processes 9402
procs_running 1
procs_blocked 0
softirq 15551046 0 3479000 34799 511332 489536 0 21879 2048010 12831 8953659
{% endhighlight %}
第一行是所有CPU的统计信息汇总，后面的'cpuN'分别表示每个CPU的统计信息，因为我们的
目标是求得CPU的整体使用率，所以只需要关注第一行即可。[Linux Programmer's
Manual](http://www.kernel.org/doc/man-pages/online/pages/man5/proc.5.html)这样描
述/proc/stat：

> cpu  3357 0 4313 1362393  
> The amount of time, measured in units of USER_HZ (1/100ths of a second on most
> architectures, use sysconf(_SC_CLK_TCK) to obtain the right value), that the
> system spent in user mode, user mode with low priority (nice), system mode, and
> the idle task, respectively.

因此，**CPU% = (Δt\_user + Δt\_nice + Δt\_system) / (Δt\_user + Δt\_nice + Δ
t\_system + Δt\_idle)**。

代码参见[double
SystemInfo::GetSystemCPUUsage()](https://github.com/yapianyu/system-metrics/blob/master/src/SystemInfo.cpp)

###系统物理内存总大小###
Windows和Linux都有现成的API：
[GlobalMemoryStatusEx()](http://msdn.microsoft.com/en-us/library/windows/desktop/aa366589\(v=vs.85\).aspx)
和[sysinfo()](http://linux.die.net/man/2/sysinfo)。

{% highlight cpp %}
double SystemInfo::GetSystemMemoryTotal()
{
    double lMemTotal = -1;

#ifdef _WIN32
    MEMORYSTATUSEX lMemInfo;
    lMemInfo.dwLength = sizeof(MEMORYSTATUSEX);
    BOOL lSuccess = GlobalMemoryStatusEx(&lMemInfo);

    // size in MB
    if (lSuccess)
        lMemTotal = lMemInfo.ullTotalPhys / (1024.0 * 1024.0);

#elif defined(__linux__)
    struct sysinfo lSysinfo;
    int lReturn = sysinfo(&lSysinfo);

    if (lReturn == 0)
        lMemTotal = (lSysinfo.totalram * lSysinfo.mem_unit) / (1024.0 * 1024.0);

#endif
    return lMemTotal;
}
{% endhighlight %}

###系统物理内存使用量###
同上！

{% highlight cpp %}
double SystemInfo::GetSystemMemoryUsed()
{
    double lMemUsed = -1;

#ifdef _WIN32
    MEMORYSTATUSEX lMemInfo;
    lMemInfo.dwLength = sizeof(MEMORYSTATUSEX);
    BOOL lSuccess = GlobalMemoryStatusEx(&lMemInfo);

    // size in MB
    if (lSuccess)
        lMemUsed = (lMemInfo.ullTotalPhys - lMemInfo.ullAvailPhys) / (1024.0 * 1024.0);

#elif defined(__linux__)
    struct sysinfo lSysinfo;
    int lReturn = sysinfo(&lSysinfo);

    if (lReturn == 0)
        lMemUsed = (lSysinfo.totalram - lSysinfo.freeram) * lSysinfo.mem_unit / (1024.0 * 1024.0);

#endif
    return lMemUsed;
}
{% endhighlight %}

###进程CPU使用率###
进程的CPU使用率与系统的计算方法类似，只不过改为采样周期内进程运行于处理器上的时间和
采样周期的比值。

####Windiws####
我们可以通过调用GetProcessTimes()来获取进程在kernel和user mode下的运行时间。MSDN对
[GetProcessTimes()](http://msdn.microsoft.com/en-us/library/windows/desktop/ms683223\(v=vs.85\).aspx)
是这样描述的：
{% highlight cpp %}
/**
 * Retrieves timing information for the specified process.
 *
 * @param lpKernelTime A pointer to a FILETIME structure that receives the
 * amount of time that the process has executed in kernel mode. The time that
 * each of the threads of the process has executed in kernel mode is determined,
 * and then all of those times are summed together to obtain this value.
 * @param lpUserTime A pointer to a FILETIME structure that receives the amount
 * of time that the process has executed in user mode. The time that each of the
 * threads of the process has executed in user mode is determined, and then all
 * of those times are summed together to obtain this value.
 */
BOOL WINAPI GetProcessTimes(
  __in   HANDLE hProcess,
  __out  LPFILETIME lpCreationTime,
  __out  LPFILETIME lpExitTime,
  __out  LPFILETIME lpKernelTime,
  __out  LPFILETIME lpUserTime
);
{% endhighlight %}

因此，**CPU% = (Δt\_proc\_kernel + Δt\_proc\_user) / (Δt\_sys\_kernel + Δt\_sys\_user)**。

代码参见[double
ProcessInfo::GetProcessCPUUsage()](https://github.com/yapianyu/system-metrics/blob/master/src/ProcessInfo.cpp)

####Linux####
Linux下可以通过读取/proc/[pid]/stat文件获得进程在kernel和user mode下的运行时间。在
我的ubuntu 11.10中运行`cat /proc/3761/stat`，输出如下：
{% highlight cpp %}
> cat /proc/3761/stat
3761 (chrome) S 3654 1925 1925 0 -1 4202560 640712 0 191 0 31802 58992 0 0 20 0
8 0 2980922 1031110656 10654 18446744073709551615 1 1 0 0 0 0 0 67112962 65536
18446744073709551615 0 0 17 1 0 0 305 0 0
{% endhighlight %}
