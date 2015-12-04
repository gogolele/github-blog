title: Ganglia Running Process的一次异常情况分析
date: 2012-03-05 15:16:26
categories: dev
tags: Ganglia
---

** Ganglia Running Process的计算 **

Ganglia running processes是怎么算出来的？ganglia是通过这个命令获得running processes的。
```
cat /proc/loadavg
0.00 0.28 0.61 1/591 2993
```

其中，1是running process，591是total process。

**问题及排查**

在一个新版hadoop的测试中，意外发现running process数量上涨。为了追踪ganglia图上突然出现的14个running processes，调查了一下。
```
ps Haxh
cat /proc/loadavg
```

这两个命令查询出来的total processes和的total processes是一致的，状态为R的即是running process。
于是又写了个脚本，在启动namenode的期间，每隔0.5秒打印出/proc/loadavg 和 ps Haxh的数据，查看何时出现14个running processes的。
最后发现，不管有没有这个patch，都有可能出现running processes从1上升至6，甚至14的情况。这些processes是namenode的子线程，在某些情况下状态为Rl，R是运行态，l是多线程，在/proc/loadavg中被计算成了running processes。这些子线程运行的时间很短，ganglia是每分钟获得一次数据，很有可能没有采集到。
因此，我之前测试时ganglia图示上的差别，只是巧合，导致我认为加上patch之后有问题。我通过几次实验，看到没有patch时，也会出现ganglia图中running processes上升的情况。
Reference
```
PROCESS STATE CODES
       Here are the different values that the s, stat and state output specifiers (header "STAT" or "S") will display to describe the state of a process.
       D    Uninterruptible sleep (usually IO)
       R    Running or runnable (on run queue)
       S    Interruptible sleep (waiting for an event to complete)
       T    Stopped, either by a job control signal or because it is being traced.
       W    paging (not valid since the 2.6.xx kernel)
       X    dead (should never be seen)
       Z    Defunct ("zombie") process, terminated but not reaped by its parent.

       For BSD formats and when the stat keyword is used, additional characters may be displayed:
       <    high-priority (not nice to other users)
       N    low-priority (nice to other users)
       L    has pages locked into memory (for real-time and custom IO)
       s    is a session leader
       l    is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
       +    is in the foreground process group
```

------END-------

伍涛@成都 重整
2013-07-28
