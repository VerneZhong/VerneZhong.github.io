---
title: JVM 参数配置
date: 2020-08-23 12:47:39
categories: 
- JVM笔记
tags:
- JVM参数
---

![](https://github.com/user-attachments/assets/cf0e9371-c81e-4491-b7a0-2304999da266)

<!-- more -->

## JVM 参数配置

### 虚拟机及垃圾收集器日志

在JDK9以前，HotSpot并没有提供统一的日志处理框架，虚拟机各个功能模块的日志开关分布在不同的参数上，日志级别、循环日志大小、输出格式、重定向等设置在不同功能上都需要单独解决。直到JDK9，这些问题才终于解决，HotSpot将所有功能的日志都收归到了`-Xlog`参数上。

参数格式：`-Xlog[:[selector][:[output][:[decorators][:output-options]]]]`

命令行中最关键的参数是选择器（Selector），它由标签（Tag）和日志级别（Level）共同组成。标签可理解为虚拟机中某个功能模块的名字，它告诉日志框架用户希望得到虚拟机哪些功能的日志输出。垃圾收集器的标签名称为"gc"，由此可见，垃圾收集器日志只是HotSpot众多功能日志的其中一项。

日志级别从低到高，共有Trace、Debug、Info、Warning、Error、Off六种级别，日志级别决定了输出信息的详细程度，默认级别为Info，HotSpot的日志规则与Log4j、Slf4j这类Java日志框架大体上是一致的。

JDK9前后日志参数变化列表。

| JDK9前日志参数                                               | JDK9后配置形式                                               |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| -XX:+PrintGC                                                 | -Xlog:gc                                                     |
| -XX:+PrintGCDetails                                          | -Xlog:gc+heap=debug                                          |
| -XX:+G1PrintHeapRegions                                      | -Xlog:gc+region=trace                                        |
| -XX:+G1PrintRegionLivenessInfo                               | -Xlog:gc+liveness=trace                                      |
| -XX:+G1SummarizeConcMark                                     | -Xlog:gc+marking=trace                                       |
| -XX:+G1SummarizeRSetStats                                    | -Xlog:gc+remset*=trace                                       |
| -XX:+GCLogFileSize, NumberOfGCLogFiles, UseGCLogFileRotation | -Xlog:gc*:file=<file>::filecount=<count>,filesize=<filesize in kb> |
| -XX:+PrintAdaptiveSizePolicy                                 | -Xlog:gc+ergo*=trace                                         |
| -XX:+PrintClassHistogramAfterFullGC                          | -Xlog:classhisto*=trace                                      |
| -XX:+PrintClassHistogramBeforeFullGC                         | -Xlog:classhisto*=trace                                      |
| -XX:+PrintGCApplicationConcurrentTime                        | -Xlog:safepoint                                              |
| -XX:+PrintGCApplicationStoppedTime                           | -Xlog:safepoint                                              |
| -XX:+PrintGCDateStamps                                       | 使用time修饰器                                               |
| -XX:+PrintGCTaskTimeStamps                                   | -Xlog:gc+task=trace                                          |
| -XX:+PrintGcTimeStamps                                       | 使用uptime修饰器                                             |
| -XX:+PrintHeapAtGC                                           | -Xlog:gc+heap=debug                                          |
| -XX:+PrintHeapAtGCExtended                                   | -Xlog:gc+heap=trace                                          |
| -XX:+PrintJNIGCStalls                                        | -Xlog:gc+jni=debug                                           |
| -XX:+PrintOldPLAB                                            | -Xlog:gc+plab=trace                                          |
| -XX:+PrintParallelOldGCPhaseTimes                            | -Xlog:gc+phases=trace                                        |
| -XX:+PrintPLAB                                               | -Xlog:gc+plab=trace                                          |
| -XX:+PrintPromotionFailure                                   | -Xlog:gc+promotion=debug                                     |
| -XX:+PrintReferenceGC                                        | -Xlog:gc+ref=debug                                           |
| -XX:+PrintStringDebuplicationStatisitcs                      | -Xlog:gc+stringdedup                                         |
| -XX:+PrintTaskqueue                                          | -Xlog:gc+task+stats=trace                                    |
| -XX:+PrintTenuringDistribution                               | -Xlog:gc+age=trace                                           |
| -XX:+PrintTerminationStats                                   | -Xlog:gc+task+stats=debug                                    |
| -XX:+PrintTLAB                                               | -Xlog:gc+tlab=trace                                          |
| -XX:+TraceAdaptiveGCBoundary                                 | -Xlog:heap+ergo=debug                                        |
| -XX:+TraceDynamicGCThreads                                   | -Xlog:gc+task=trace                                          |
| -XX:+TraceMetadataHumongousAllocation                        | -Xlog:gc+metaspace+alloc=debug                               |
| -XX:+G1TraceConcRefinement                                   | -Xlog:gc+refine=debug                                        |
| -XX:+G1TraceEagerReclaimHumongousObjectss                    | -Xlog:gc+humongous=debug                                     |
| -XX:+G1TraceStringSymbolTableScrubbing                       | -Xlog:gc+stringtable=trace                                   |

### 垃圾收集器参数

垃圾收集齐全相关的常用参数

|              参数              | 描述 |
| :----------------------------: | ---- |
|          -XX:+/-UseSerialGC          | 虚拟机运行在Client模式下的默认值，打开此开关后，使用Serial + Serial Old的收集器组合进行内存回收 |
|          -XX:+/-UseParNewGC    | 打开此开关后，使用ParNew + Serial Old的收集器组合进行内存回收，在JDK9后不再支持 |
|       -XX:+/-UseConcMarkSweepGC       | 打开此开关后，使用ParNew + CMS + Serial Old的收集器组合进行内存回收。Serial Old收集器将作为CMS收集器出现“Concurrent Mode Failure”失败后的后备收集器使用 |
|         -XX:+/-UseParallelGC   | JDK9之前虚拟机运行在Server模式下的默认值，打开此开关后，使用Parallel Scavenge + Serial Old（PS MarkSweep）的收集器组合进行内存回收 |
|        -XX:+/-UseParallelOldGC        | 打开此开关后，使用Parallel Scavenge + Parallel Old的收集器组合进行内存回收 |
| -Xms256m | 初始堆大小 |
| -Xmx512m | 最大堆大小 |
| -Xmn128m | 新生代大小 |
| -Xss1m | 设置栈大小 |
| -XX:+/-UseTLAB | 开启本地线程分配缓冲（Thread Local Allocation Buffer，TLAB） |
| -XX:FieldsAllocationStyle | 字段存储顺序会受到虚拟机分配策略参数 |
| -XX：HeapDumpOnOutOfMemoryError | 虚拟机内存溢出时导出堆转储快照文件 |
| -XX:MaxPermSize | 最大永久代大小（JDK6） |
| -XX:PermSize | 初始永久代大小（JDK6） |
| -XX:MaxMetaspaceSize | 最大元空间大小（JDK8），默认是-1，即不限制，或者说只受限于本地内存大小 |
| -XX:MetaspaceSize | 指定元空间的初始大小，以字节为单位，达到该单位就会触发垃圾收集进行类型卸载，同时收集器会对该值进行调整：如果释放了大量空间就会降低该值；如果释放很少空间，在不超过最大的元空间大小的前提下，适当提高该值 |
| -XX:MinMetaspaceFreeRatio | 作用是在垃圾收集之后控制最小的元空间剩余容量的百分比，可减少因为元空间不足导致的垃圾收集的频率。 |
| -XX:MaxMetaspaceFreeRatio | 用于控制最大的元空间剩余容量的百分比 |
| -XX:MaxDirectMemorySize | 设置直接内存的最大容量，若不指定，则默认与堆最大值一致 |
|         -XX:SurvivorRatio         | 新生代中Eden区域与Survivor区域的容量比值，默认为8，代表Eden：Survivor=8:1 |
|     -XX:PretenureSizeThreshold     | 直接晋升到老年代的对象大小，设置这个参数后，大于这个参数的对象将直接在老年代分配 |
|      -XX:MaxTenuringThreshold      | 晋升到老年代的对象年龄。每个对象在坚持过一次Minor GC之后，年龄就增加1，当超过这个参数值时就进入老年代 |
|     -XX:UseAdaptiveSizePolicy  | 动态调整Java堆中各个区域的大小以及进入老年代的年龄 |
|     -XX:HandlePromotionFailure     | 是否允许分配担保失败，即老年代的剩余空间不足以应对新生代的整个Eden和Survivor区的所有对象都存活的极端情况 |
|       -XX:ParallelGCThreads    | 设置并行GC时进行内存回收的线程数，也就是用户线程冻结期间并行执行的收集器线程数 |
|          -XX:GCTimeRatio       | GC时间占总时间的比率，默认值为99，即允许1%的GC时间。仅在使用Parallel Scavenge收集器时生效 |
|        -XX:MaxGCPauseMillis        | 设置GC的最大停顿时间。仅在使用Parallel Scavenge收集器时生效 |
| -XX:CMSInitiatingOccupancyFraction | 设置CMS收集器在老年代空间被使用多少后触发垃圾收集。默认值为68%，仅在使用CMS收集器时生效 |
| -XX:UseCMSCompactAtFullCollection | 设置CMS收集器在完成垃圾收集后是否要进行一次内存碎片整理。仅在使用CMS收集器时生效，此参数从JDK9开始废弃 |
| -XX:CMSFullGCsBeforeCompaction | 设置CMS收集器在进行若干次垃圾收集后再启动一次内存碎片整理。仅在使用CMS收集器时生效，此参数从JDK9开始废弃 |
| -XX:UseG1GC | 使用G1收集器，这个是JDK9后的Server模式默认值 |
| -XX:G1HeapRegionSize=n | 设置Region大小，并非最终值 |
| -XX:MaxGCPauseMillis | 设置G1收集过程目标时间，默认值200ms，不是硬性条件 |
| -XX:G1NewSizePercent | 新生代最小值，默认值是5% |
| -XX:G1MaxNewSizePercent | 新生代最大值，默认值是60% |
| -XX:ConcGCThreads=n | 并发标记、并发整理的执行线程数，对不同的收集器，根据其能够并发的阶段，有不同的含义 |
| -XX:InitiatingHeapOccupancyPercent | 设置触发标记周期的Java堆占用率阈值。默认值是45%。这里的Java堆占比值的是non_young_capacity_bytes，包括old+humongous |
| -XX:UseShenandoahGC | 使用Shenandoah收集器。这个选项在OracleJDK中不被支持，只能在OpenJDK 12或者某些支持Shenandoah的Backport发行版本使用。目前仍然要配合-XX:+UnlockExperimentalVMOptions使用 |
| -XX:ShenandoahGCHeuristics | Shenandoah何时启动一次GC过程，其可选值有adaptive、static、compact、passive、aggressive |
| -XX:UseZGC | 使用ZGC收集器，目前仍然要配合-XX:+UnlockExperimentalVMOptions使用 |
| -XX:UseNUMA | 启用NUMA内存分配支持，目前只有Parallel和ZGC支持，以后G1收集器可能也会支持该选项 |

### 总结  

垃圾收集器在许多场景中都是影响系统停顿时间和吞吐能力的重要因素之一，虚拟机之所以提供多种不同的收集器以及大量的调节参数，就是因为只有根据实际应用需求、实现方式选择最优的收集方式才能获取最好的性能。没有固定收集器、参数组合，没有最优的调优方法，虚拟机也就没有什么必然的内存回收行为。如果要实践调优，就必须要了解每个具体收集器的行为、优势劣势、调节参数等。
