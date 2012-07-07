---
layout: post
title: 编程获取获取CPU使用率、内存使用量、uptime、线程数等系统状态参数
category: tech
tags: CPU使用率 内存使用量 uptime 线程数 C++
---

为了监控公司某项目的运行情况，需要在进程内部获取系统和进程的状态参数，谷歌一番之后在这里做个总结。内容涉及：

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
GetProcessCPUUsage()。当我们打开任务管理器，可以看到：

* CPU使用率包括系统的CPU使用率和进程的CPU使用率
* CPU使用率每隔一定时间会刷新一次

![](/image/windows-task-manager.png)  
也就是说，CPU使用率是采样周期内CPU忙（非空闲）的时间和采样周期的比值，是一个平均
值。因此，系统CPU使用率可以描述为采样周期内系统运行于非idle状态下的时间和采样周
期的比值。

####Windiws####
函数[GetSystemTimes()](http://msdn.microsoft.com/en-us/library/ms724400\(VS.85\).aspx)
可以用来获取系统idle的时间、运行在kernel和user模式下的时间，MSDN是这样描述的：
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
* t\_idle已经包括在t\_kernel中，计算系统有效运行时间时必须排除掉它
* t\_kernel + t\_user ≈ 采样周期 × 处理器数量

因此，**CPU% = (Δt\_kernel + Δt\_user - Δt\_idle) / (Δt\_kernel + Δt\_user)**。

####Linux####
Linux将各种系统状态信息保存在文件/proc/stat中，比如CPU运行情况、中断统计、启
动时间、上下文切换次数、运行中的进程等等。在我的ubuntu 11.10中运行`cat /proc/stat`
，输出：
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
第一行是所有CPU的统计信息汇总，后面几行'cpuN'分别表示每个CPU的统计信息，因为我们的
目标是求得CPU的整体使用率，所以只需要关注第一行即可。[Linux Programmer's
Manual](http://www.kernel.org/doc/man-pages/online/pages/man5/proc.5.html)这样描
述/proc/stat：

> cpu  3357 0 4313 1362393  
> The amount of time, measured in units of USER_HZ (1/100ths of a second on most
> architectures, use sysconf(_SC_CLK_TCK) to obtain the right value), that the
> system spent in user mode, user mode with low priority (nice), system mode, and
> the idle task, respectively.

因此，**CPU% = (Δt\_user + Δt\_nice + Δt\_system) / (Δt\_user + Δt\_nice + Δ
t\_kernel + Δt\_idle)**。

代码参见 [double SystemInfo::GetSystemCPUUsage()](https://github.com/yapianyu/system-metrics/blob/master/src/SystemInfo.cpp)

###系统物理内存总大小###
Windows和Linux有各自的API：
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
函数[GetProcessTimes()](http://msdn.microsoft.com/en-us/library/windows/desktop/ms683223\(v=vs.85\).aspx)
可以用来获取进程在kernel和user模式下的运行时间。MSDN是这样描述的：
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

####Linux####
Linux下可以通过读取文件/proc/[pid]/stat获得进程在kernel和user模式下的运行时间。在
我的ubuntu 11.10中运行`cat /proc/3761/stat`，输出：
{% highlight cpp %}
> cat /proc/3761/stat
3761 (chrome) S 3654 1925 1925 0 -1 4202560 640712 0 191 0 31802 58992 0 0 20 0
8 0 2980922 1031110656 10654 18446744073709551615 1 1 0 0 0 0 0 67112962 65536
18446744073709551615 0 0 17 1 0 0 305 0 0
{% endhighlight %}
这里最关键的是第14和第15个参数（31802和58992），分别表示进程在user和kernel模式下
的运行时间。

> utime %lu  
> Amount of time that this process has been scheduled in user
> mode, measured in clock ticks (divide by sysconf(_SC_CLK_TCK). This
> includes guest time, guest_time (time spent running a virtual CPU, see
> below), so that applications that are not aware of the guest time field do
> not lose that time from their calculations.

> stime %lu  
> Amount of time that this process has been scheduled in
> kernel mode, measured in clock ticks (divide by sysconf(_SC_CLK_TCK).

因此，**CPU% = (Δt\_proc\_user + Δt\_proc\_kernel) / (Δt\_sys\_user + Δt\_sys\_nice + Δ
t\_sys\_kernel + Δt\_sys\_idle)**。

代码参见 [double ProcessInfo::GetProcessCPUUsage()](https://github.com/yapianyu/system-metrics/blob/master/src/ProcessInfo.cpp)

###进程物理内存使用量###
####Windows####
结构体[PROCESS_MEMORY_COUNTERS](http://msdn.microsoft.com/en-us/library/windows/desktop/ms684877\(v=vs.85\).aspx)
的属性WorkingSetSize表示进程占用的物理内存大小，可以通过函数
[GetProcessMemoryInfo()](http://msdn.microsoft.com/en-us/library/windows/desktop/ms683219\(v=vs.85\).aspx)获取。
其定义如下：
{% highlight cpp %}
typedef struct _PROCESS_MEMORY_COUNTERS {
  DWORD  cb;
  DWORD  PageFaultCount;
  SIZE_T PeakWorkingSetSize;
  SIZE_T WorkingSetSize; // The current working set size, in bytes.
  SIZE_T QuotaPeakPagedPoolUsage;
  SIZE_T QuotaPagedPoolUsage;
  SIZE_T QuotaPeakNonPagedPoolUsage;
  SIZE_T QuotaNonPagedPoolUsage;
  SIZE_T PagefileUsage;
  SIZE_T PeakPagefileUsage;
} PROCESS_MEMORY_COUNTERS, *PPROCESS_MEMORY_COUNTERS;
{% endhighlight %}

####Linux####
文件/proc/[pid]/status中以VmRSS开头的一行表示进程占用的物理内存大小，示例如下：
{% highlight text %}
> cat /proc/1730/status
Name:   chrome
State:  S (sleeping)
Tgid:   1730
Pid:    1730
PPid:   1
TracerPid:  0
Uid:    1000    1000    1000    1000
Gid:    1000    1000    1000    1000
FDSize: 512
Groups: 4 20 24 46 116 118 124 1000 
VmPeak:  1229704 kB
VmSize:  1093232 kB
VmLck:        16 kB
VmHWM:    695656 kB
VmRSS:    158156 kB
VmData:   944400 kB
VmStk:       136 kB
VmExe:     62128 kB
VmLib:     31020 kB
VmPTE:      1152 kB
VmSwap:        0 kB
Threads:    62
SigQ:   0/15999
{% endhighlight %}

代码参见 [double ProcessInfo::GetProcessMemoryUsed()](https://github.com/yapianyu/system-metrics/blob/master/src/ProcessInfo.cpp)

###进程uptime###
####Windows####
函数[GetProcessTimes()](http://msdn.microsoft.com/en-us/library/windows/desktop/ms683223\(v=vs.85\).aspx)
可以获取进程的创建时间，再调用函数
[GetSystemTimeAsFileTime](http://msdn.microsoft.com/en-us/library/windows/desktop/ms724397\(v=vs.85\).aspx)
可以得到系统的当前时间，两者相减就是进程的uptime。

####Linux####
文件/proc/[pid]/stat的第22个参数
[starttime](http://www.kernel.org/doc/man-pages/online/pages/man5/proc.5.html)
表示进程的起始时间（自系统启动后，单位jiffies），再调用[sysinfo()](http://linux.die.net/man/2/sysinfo)
可以得到系统的运行时间（自系统启动后，单位s），两者单位统一后相减就是进程的uptime。

> starttime %llu (was %lu before Linux 2.6)  
> The time in jiffies the process started after system boot.

代码参见 [double ProcessInfo::GetProcessUptime()](https://github.com/yapianyu/system-metrics/blob/master/src/ProcessInfo.cpp)

###进程内部的线程数###
文件/proc/[pid]/stat的第20个参数
[num_threads](http://www.kernel.org/doc/man-pages/online/pages/man5/proc.5.html)
表示进程内部的线程数，直接读取并解析即可。

> num_threads %ld  
> Number of threads in this process (since Linux 2.6).

{% highlight cpp %}
unsigned long ProcessInfo::GetProcessThreadCount()
{
    unsigned long lThreadCnt = -1;

#ifdef _WIN32
    // get a process list snapshot
    HANDLE lSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPALL, 0);
    if (lSnapshot != NULL)
    {
        // initialize the process entry structure
        PROCESSENTRY32 lEntry;
        lEntry.dwSize = sizeof(PROCESSENTRY32);

        // get the first process info
        BOOL lSuccess = Process32First(lSnapshot, &lEntry);
        while (lSuccess)
        {
            if (lEntry.th32ProcessID == mProcessId)
            {
                lThreadCnt = lEntry.cntThreads;
                break;
            }
            lSuccess = Process32Next(lSnapshot, &lEntry);
        }

        CloseHandle(lSnapshot);
    }

#elif defined(__linux__)
    // the 20th value in file /proc/[pid]/stat:
    // num_threads %ld, Number of threads in this process
    char lFileName[256];
    sprintf(lFileName, "/proc/%d/stat", mProcessId);
    FILE* lpFile = fopen(lFileName, "r");
    if (lpFile)
    {
        // skip unnecessary content
        char lTemp[LINEBUFFLEN];
        int lValuesToSkip = 19;
        for (int i = 0; i < lValuesToSkip; i++)
            fscanf(lpFile, "%s", lTemp);
        fscanf(lpFile, "%lu", &lThreadCnt);
        fclose(lpFile);
    }

#endif
    return lThreadCnt;
}
{% endhighlight %}
