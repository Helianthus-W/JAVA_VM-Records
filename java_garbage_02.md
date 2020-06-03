---
title: 虚拟机性能监控
---

##### Java中的命令行工具

###### jps - JVM Process Status Tool(虚拟机进程状况)

jps命令可以列出正在运行的虚拟机进程并显示虚拟机执行主类名称以及这些进程的本地虚拟机唯一ID。

###### jstat - JVM Statistics Monitoring Tool(虚拟机统计信息监视工具)

jstat命令用于监视虚拟机各种运行状态信息，它可以显示本地或者远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。

-class: 监视类装载、卸载数量、总空间以及类装载所耗费的时间
-gc: 监视Java对情况，包括Eden区，两个Survivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息
-gcutil: 监视内容与gc基本相同，但输出主要关注已使用的空间占总空间的百分比
-compiler: 输出JIT编译器编译过的方法 、耗时等信息

###### jinfo - Configuration Info for Java(Java配置信息工具)

jinfo命令可以实时查看和调整虚拟机各项参数。

###### jmap - Memory Map for Java(Java内存映像工具)

jmap命令用于生成堆转储快照，也可以查询finalize执行队列、Java堆和永久代的详细信息。

-dump
-histo

###### jhat - JVM Heap Analysis Tool(虚拟机堆转储快照分析工具)

jhat命令一般搭配jmap使用。用来分析jmap生成的堆转储快照文件，分析结束后，可以在浏览器中查看结果。

###### jstack - Stack Trace for Java(Java堆栈跟踪工具)

