+++
date = "2015-08-24T11:06:11+08:00"
menu = "main"
tags = ["android"]
title = "关于 Android 的 CPU 内存监控"

+++

前两周在做关于Android内存和CPU监控的部分，下面给大家分享几种得到应用CPU和内存占用的方法。

内存占用情况
------

Android系统中内存的信息在`/proc/meminfo`中，在`adb shell`的情况下，我们输入`cat /proc/meminfo`可以得到如下信息

    MemTotal:        2015052 kB
    MemFree:          126996 kB
    Buffers:           43024 kB
    Cached:           605476 kB
    PoolFree:          28612 KB
    SwapCached:         4536 kB
    Active:           690640 kB
    Inactive:         847096 kB
    Active(anon):     438632 kB
    Inactive(anon):   489160 kB
    Active(file):     252008 kB
    Inactive(file):   357936 kB
    Unevictable:        2012 kB
    Mlocked:               0 kB
    HighTotal:       1267768 kB
    HighFree:          28248 kB
    LowTotal:         747284 kB
    LowFree:           70136 kB
    SwapTotal:       1048560 kB
    SwapFree:        1040576 kB
    Dirty:                 4 kB
    Writeback:             0 kB
    AnonPages:        891180 kB
    Mapped:           190248 kB
    Shmem:             36544 kB
    Slab:              43896 kB
    SReclaimable:      22728 kB
    SUnreclaim:        21168 kB
    KernelStack:       10016 kB
    PageTables:        17500 kB
    NFS_Unstable:          0 kB
    Bounce:                0 kB
    WritebackTmp:          0 kB
    CommitLimit:     2056084 kB
    Committed_AS:   39649308 kB
    VmallocTotal:     245760 kB
    VmallocUsed:      101140 kB
    VmallocChunk:     103028 kB
    DirectMap4k:      229380 kB
    DirectMap2M:      548864 kB

首行的MemTotal和第二行的MemFree分别代表总共的内存和剩余空闲的内存信息。要想获得应用的内存占用信息，我们需要使用ActivityManager

	/**
     * 获取应用程序占用的内存，单位iKB
     * @return
     */
    public String getPidMemorySize() {
        ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
        int[] myMempid = new int[] { pid };
        Debug.MemoryInfo[] memoryInfo = am.getProcessMemoryInfo(myMempid);
        memoryInfo[0].getTotalSharedDirty();
        int memSize = memoryInfo[0].getTotalPss();
        return memSize + "KB";
    }

我们也可以使用ActivityManager获得空余的内存信息

	/**
     * 获取空余内存，单位KB
     * @return
     */
    public String getFreeMemorySize() {
        ActivityManager.MemoryInfo outInfo = new ActivityManager.MemoryInfo();
        ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
        am.getMemoryInfo(outInfo);
        long avaliMem = outInfo.availMem;
        return (avaliMem / 1024) + "KB";
    }

CPU占用情况
---------

查询Android应用程序的CPU占用情况，以下三种方法用的最多

1. 使用 top -n 1命令获得当前运行所有程序的各项信息
    在`adb shell`下输入`top -n 1`命令，会的到如下的信息
    User 10%, System 20%, IOW 0%, IRQ 0%
    
        User 29 + Nice 4 + Sys 63 + Idle 210 + IOW 0 + IRQ 0 + SIRQ 1 = 307

          PID PR CPU% S  #THR     VSS     RSS PCY UID      Name
          298  0   7% S     6   7880K    944K     shell    /sbin/adbd
        32674  0   5% R     1   1308K    484K     shell    top
         1929  0   3% S    15 877544K  48656K  bg u0_a89   com.baidu.map.location
         
    PID是进程的ID，是唯一的
    
    CPU不用说了，就是CPU占用比率
    
    VSS(Virtual Set Size)虚拟耗用内存(包含共享库占用的内存)
    
    PSS(Proportional Set Size)实际使用的物理内存(比例分配共享库占用的内存)
    
    RSS(Resident Set Size)实际耗用的物理内存(包含共享库占用的内存)
    
    USS(Unique Set Size)进程独自占用的物理内存(不包含共享库占用的内存)
    
    使用此方法的优点就是获取信息简单容易。
    
    使用次方法的缺点也一目了然，就是精确度不是很高。

2. 使用`dumpsys cpuinfo`命令获得CPU占用信息
	我们在`adb shell`命令下输入`dumpsys cpuinfo`，会得到如下的信息
    
    	Load: 0.84 / 0.72 / 1.0
        CPU usage from 16074ms to 5139ms ago:
          10% 298/adbd: 0.6% user + 10% kernel / faults: 1331 minor
          5% 15252/com.sina.weibo: 3.2% user + 1.7% kernel
          2% 185/surfaceflinger: 0.8% user + 1.2% kernel
          2% 12783/com.mi.vtalk:remote: 0.5% user + 1.4% kernel / faults: 4 minor
          1.5% 97/cfinteractive: 0% user + 1.5% kernel
          1.4% 131/dhd_dpc: 0% user + 1.4% kernel
        
    这个能够统计到所有应用的CPU占用信息，要比`top -n 1`获得的信息更加详细一些。
    
    缺点就是这种方法会有权限问题，ROOT了之后才能使用。
    
3. 通过stat句柄
	在`adb shell`命令下输入`cat /proc/stat`能够获得系统的CPU的使用情况
    
        shell@pisces:/ $ cat /proc/stat
        cpu  127277 11744 303884 1487545 2534 618 3184 0 0 0
        cpu0 96804 9072 280861 1426520 1384 618 3128 0 0 0
        cpu1 12179 970 9100 13830 522 0 45 0 0 0
        cpu2 10146 991 7402 12210 375 0 8 0 0 0
        cpu3 8147 710 6521 34984 252 0 2 0 0 0
		
    **参数**  **解析**(单位：jiffies)
    (jiffies是内核中的一个全局变量，用来记录自系统启动以来产生的节拍数，在Linux中，一个节拍大致可以理解为操作系统进程调度的最小时间片，不同的Linux内核可能值有所不同，通常在1ms到10ms之间)
    
    主要有用的是首行，记录了各项指标在CPU中的运行时间。因为stat中记录的信息是从CPU ON开始的，所以为了更准确的记录CPU的使用情况，需要多次采样。
    
    具体的例子如下
    
        measure 1: cpu 8000 2000 1000 9000
        measure 2: cpu 9500 2500 1500 9500
        
	在measure1和measure2上的cpu占有率如下
    
    9500 - 8000 = 1500 in user
    
    2500 - 2000 = 500 in system
    
    1500 - 1000 = 500 in nice
    
    9500 - 9000 = 500 in idle

    1500 + 500 + 500 + 500 = 3000 in total
    
    因此，CPU占有率为
    
    1500/3000 = 0.5 in user
    
    500/3000 = 0.16 in system
    
    500/3000 = 0.16 in nice
    
    500/3000 = 0.16 in idle
    
    然而，以上的部分只是所有应用的CPU占用率，要想获得应用的CPU占用率，需要从`/proc/pid/stat`中获取，获取一个进程的CPU信息是这样的
    	
        1929 (du.map.location) S 186 186 0 0 -1 4194624 20666 0 2 0 1740 755 0 0 20 0 15 0 3265 898633728 12216 4294967295 1 1 0 0 0 0 4612 0 38136 4294967295 0 0 17 0 0 0 0 0 0 0 0 0
        
    通常我们取13,14位的和进行多次采样。
    

上述内容的具体代码部分在[Android内存和CPU自测](https://github.com/MaybeMercy/CPU-Memory-Monitor)


参考
---

[Android内存之VSS/RSS/PSS/USS](http://hubingforever.blog.163.com/blog/static/17104057920114411313717/)

[Android上的CPU信息监测](http://my.oschina.net/u/229093/blog/261191)

[Emmagee - a practical, handy performance test tool for specified Android App](https://github.com/NetEase/Emmagee)

[Linux平台Cpu使用率的计算](http://www.blogjava.net/fjzag/articles/317773.html)

