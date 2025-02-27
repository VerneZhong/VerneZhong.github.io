---
title: JVM 性能监控与故障处理工具
date: 2020-08-20 16:59:59
categories: 
- JVM笔记
tags:
- JVM性能监控工具
- JVM故障处理工具
- JDK工具
---

![](https://github.com/user-attachments/assets/cf0e9371-c81e-4491-b7a0-2304999da266)

<!-- more -->

## 									JVM 性能监控与故障处理工具

### JDK 命令行工具

Java开发人员肯定知道JDK的bin目录中有`java.exe`、`javac.exe`这两个命令行工具，但还有很多其他工具提供稳定而强大的功能，能在处理应用程序性能问题、定位故障时发挥很大的作用。下面来介绍一下Java提供的那些强大并且很有帮助的工具吧。

jdk命令行的主要工具：

|  名称  | 作用                                                         |
| :----: | :----------------------------------------------------------- |
|  jps   | JVM Process Status Tool，显示指定系统内所有的HotSpot虚拟机进程 |
| jstat  | JVM Statistics Monitoring Tool，用于收集HotSpot虚拟机各方面的运行数据 |
| jinfo  | Configuration Info for Java，显示虚拟机配置信息              |
|  jmap  | Memory Map for Java，生成虚拟机的内存转储快照（heapdump文件） |
|  jhat  | JVM Heap Dump Browser，用于分析heapdump文件，它会建立一个HTTP/HTML服务器，让用户可以在浏览器上查看分析结果 |
| jstack | Stack Trace for Java，显示虚拟机的线程快照                   |

### jps： 虚拟机进程状况工具

JDK的很多小工具的名字都参考了Unix命令的命名方式，jps（JVM Process Status Tool）是其中的典型。功能是可以列出正在运行的虚拟机进程，并显示虚拟机执行主类（Main Class，main()函数所在的类）名称以及这些进程的本地虚拟机唯一ID（Local Virtual Machine Identifier，LVMID）。

命令格式： `jps [options] [hostid]`

```powershell
C:\Program Files\Java\jdk1.8.0_171\bin>jps -l
9968 org.jetbrains.jps.cmdline.Launcher
14248 sun.tools.jps.Jps
8812 org.jetbrains.idea.maven.server.RemoteMavenServer36
```

jps工具的主要选项：

| 选项 | 作用                                                |
| :--: | --------------------------------------------------- |
|  -q  | 只输出`LVMID`，省略主类的名称                       |
|  -m  | 输出虚拟机进程启动时传递给主类`main()`函数的参数    |
|  -l  | 输出主类的全名， 如果执行进程的是jar包，输出jar路径 |
|  -v  | 输出虚拟机进程启动时JVM参数                         |

### jstat： 虚拟机统计信息监视工具

jstat（JVM Statistics Monitoring Tool）是用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据，在没有GUI图形界面，只提供了纯文本控制台环境的服务器上， 它将是运行期定位虚拟机性能问题的首选工具。

命令格式： `jstat [option vmid [interval[s|ms] [count]] ]`

参数interval和count代表查询间隔和次数，如果省略这两个参数，说明只查询一次。

假设需要没250ms查询一次进程2764垃圾收集状况，一共查询20次，那么命令应该是：

jstat -gc 2764 250 20

```shell
C:\Program Files\Java\jdk1.8.0_171\bin>jstat -gcutil 8812 1000 20
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00 100.00  76.74   4.42  92.98  82.74      3    0.027     0    0.000    0.027
  0.00 100.00  76.74   4.42  92.98  82.74      3    0.027     0    0.000    0.027
  0.00 100.00  76.74   4.42  92.98  82.74      3    0.027     0    0.000    0.027
  0.00 100.00  76.74   4.42  92.98  82.74      3    0.027     0    0.000    0.027
```

选项option代表着用户希望查询的虚拟机信息，主要分为3类：类加载、垃圾收集、运行期编译状况。

jstat工具的主要选项：

|             选项              | 作用                                                         |
| :---------------------------: | ------------------------------------------------------------ |
|            -class             | 监视类加载、卸载数量、总空间以及类加载所消耗的时间           |
|              -gc              | 监视Java堆状况，包括Eden区、两个Survivor区、老年代、元空间等的容量、已用空间、GC时间合计等信息 |
|          -gccapacity          | 监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间 |
|            -gcutil            | 监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比 |
|           -gccause            | 与-gcutil功能一样，但是会额外输出导致上一次GC产生的原因      |
|            -gcnew             | 监视新生代GC状况                                             |
|        -gcnewcapacity         | 监视内容与-gcnew基本相同，输出主要关注使用到的最大、最小空间 |
|            -gcold             | 监视老年代GC状况                                             |
|        -gcoldcapacity         | 监视内容与-gcold基本相同，输出主要关注使用到的最大、最小空间 |
| -gcpermcapacity（jdk8已移除） | 输出永久代使用到的最大、最小空间                             |
|  -gcmetacapacity（jdk8新增）  | 输出元空间使用到的最大、最小空间                             |
|           -compiler           | 输出JIT编译器编译过的方法、耗时等信息                        |
|       -printcompilation       | 输出已经被JIT编译的方法                                      |

```shell
C:\Program Files\Java\jdk1.8.0_171\bin>jstat -gcutil 8812 250 20
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00 100.00  76.74   4.42  92.98  82.74      3    0.027     0    0.000    0.027
  0.00 100.00  76.74   4.42  92.98  82.74      3    0.027     0    0.000    0.027
  0.00 100.00  76.74   4.42  92.98  82.74      3    0.027     0    0.000    0.027
  0.00 100.00  76.74   4.42  92.98  82.74      3    0.027     0    0.000    0.027
  0.00 100.00  76.74   4.42  92.98  82.74      3    0.027     0    0.000    0.027
  0.00 100.00  76.74   4.42  92.98  82.74      3    0.027     0    0.000    0.027
  0.00 100.00  76.74   4.42  92.98  82.74      3    0.027     0    0.000    0.027
  0.00 100.00  76.74   4.42  92.98  82.74      3    0.027     0    0.000    0.027
  0.00 100.00  76.74   4.42  92.98  82.74      3    0.027     0    0.000    0.027
```

查询结果：

`E代表Eden区，使用了76.74%的空间；`

`两个Survivor区（S0、S1）里面一个是空的，另一个使用了100%；`

`O代表老年代，使用了4.42%`

`M代表元空间，使用了92.98%`

`CCS是压缩的类空间利用率（以百分比表示）是82.74%`

`YGC是程序运行一来来共发生Minor GC（YGC，表示Young GC）3次，总耗时YGCT是0.027s`

`FGC发生Full GC是0次，Full GC的FGCT的耗时也是0`

`GCT（表示GC Time）是所有的GC的总耗时是0.027s`

### jinfo： Java配置信息工具

jinfo（Configuration Info for Java）的作用是实时地查看和调整虚拟机各项参数。使用jps命令的-v参数可以查看虚拟机启动时显式指定的参数列表，但如果想指定未被显式指定的参数的系统默认值，就可以使用`jinfo`的`-flag`选项进行查询，`jinfo`还可以使用`-sysprops`选项把虚拟机进程的`System.getProperties()`的内容打印出来。

`jinfo的命令格式：`

`jinfo [ option ] pid `

### jmap： Java内存映像工具

jmap（Memory Map for Java）命令用于生成堆转储快照（一般称为dump或是heapdump文件）。如果不使用jmap命令，要想获取Java堆转储快照，可以使用

`-XX:+HeapDumpOnOutOfMemoryError`参数，可以让虚拟机在OOM异常出现之后自动生成dump文件，通过`-XX:+HeapDumpOnCtrlBreak`参数则可以使用`[Ctrl] + [Break]`键让虚拟机生成dump文件，又或者在Linux系统下通过`Kill -3`命令发送进程退出信号，让虚拟机生成dump文件。

jmap的作用并不仅仅是为了获取dump文件，它还可以查询`finalize`执行队列、Java堆、元空间（Jdk8以后）或永久代（Jdk8以前）的详细信息，如空间使用率、当前用的是哪种收集器等。

`jmap的命令格式：`

`jmap [ option ] pid `

jmap工具的主要选项：

|      选项      | 作用                                                         |
| :------------: | ------------------------------------------------------------ |
|     -dump      | 生成Java堆转储快照。格式为：`-dump:[live, ]format=b, file=<filename>`，其中live子参数说明只dump存活的对象 |
| -finalizerinfo | 显示在F-Queue中等待Finalizer线程执行finalize方法的对象。只在Linux/Solaris平台有效 |
|     -heap      | 显示Java堆详细，如使用哪种垃圾收集器、参数配置、分代状况等。只在Linux/Solaris平台有效 |
|     -histo     | 显示堆中对象统计信息，包括类、实例数量、合计容量             |
|   -permstat    | 以ClassLoader为统计口径显示永久代或元空间内存状态。只在Linux/Solaris平台有效 |
|       -F       | 当虚拟机进程对-dump选项没有响应时，可使用这个选项强制生成dump快照。只在Linux/Solaris平台有效 |

以下是jmap生成dump快照的示例

```shell
C:\Program Files\Java\jdk1.8.0_171\bin>jmap -dump:format=b,file=D:\dump.hprof 8812
Dumping heap to D:\dump.hprof ...
Heap dump file created
```

### jhat： 虚拟机堆转储快照分析工具

Sun JDK提供jhat（JVM Heap Analysis Tool）命令与jmap搭配使用，来分析jmap生成的堆转储快照。jhat内置了一个微型的HTTP/HTML服务器，生成的dump文件的分析结果后，可以在浏览器中查看。一般不建议使用。可以使用其他工具分析，比如Eclipse Memory Analyzer、IBM HeapAnalyzer或JProfiler等工具，都可以实现更强大更专业的分析功能。

```shell
C:\Program Files\Java\jdk1.8.0_171\bin>jhat d:\dump.hprof
Reading from d:\dump.hprof...
Dump file created Thu Aug 20 21:22:50 CST 2020
Snapshot read, resolving...
Resolving 617466 objects...
Chasing references, expect 123 dots...........................................................................................................................
Eliminating duplicate references...........................................................................................................................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

在浏览器输入`http://localhost:7000`即可查看分析结果。

### jstack ：Java堆栈跟踪工具

jstack（Stack Trace for Java）命令用于生成虚拟机当前时刻的线程快照（一般称为threaddump或者javacore文件）。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等等都是导致线程长时间停顿的原因。线程出现停顿的时候通过jstack来查看各线程的调用堆栈，就可以知道没有响应的现场到底在后台做些什么事情，或者等待什么资源。

`jtack的命令格式：`

`jstack [ option ] vmid `

jstack工具主要选项

| 选项 | 作用                                         |
| :--: | -------------------------------------------- |
|  -F  | 当正常输出的请求不被响应时，强制输出线程堆栈 |
|  -l  | 除堆栈外，显示关于锁的附加信息               |
|  -m  | 如果调用到本地方法的话，可以显示C/C++的堆栈  |

```shell
C:\Program Files\Java\jdk1.8.0_171\bin>jstack 3984
2020-08-23 12:26:14
Full thread dump OpenJDK 64-Bit Server VM (11.0.6+8-b765.40 mixed mode, sharing):

Threads class SMR info:
_java_thread_list=0x0000022dbefdb6d0, length=17, elements={
0x0000022da3b28000, 0x0000022dbcc65800, 0x0000022dbcc67000, 0x0000022dbcc88800,
0x0000022dbcc8b800, 0x0000022dbcc8d800, 0x0000022dbccf9800, 0x0000022dbd501000,
0x0000022dbd5e4800, 0x0000022dbd5ea800, 0x0000022dbe0b6800, 0x0000022dbe0ab000,
0x0000022dbe140800, 0x0000022dbe0e6800, 0x0000022dbe1bd800, 0x0000022dbe1ae800,
0x0000022dbf12f000
}

"main" #1 prio=5 os_prio=0 cpu=484.38ms elapsed=1227.43s tid=0x0000022da3b28000 nid=0x1ec0 in Object.wait()  [0x000000e64f7ff000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
        at java.lang.Object.wait(java.base@11.0.6/Native Method)
        - waiting on <0x00000000d00e2010> (a java.lang.Object)
        at com.intellij.execution.rmi.RemoteServer.start(RemoteServer.java:91)
        - waiting to re-lock in wait() <0x00000000d00e2010> (a java.lang.Object)
        at org.jetbrains.idea.maven.server.RemoteMavenServer36.main(RemoteMavenServer36.java:23)

"Reference Handler" #2 daemon prio=10 os_prio=2 cpu=0.00ms elapsed=1227.39s tid=0x0000022dbcc65800 nid=0x29e8 waiting on condition  [0x000000e64feff000]
   java.lang.Thread.State: RUNNABLE
        at java.lang.ref.Reference.waitForReferencePendingList(java.base@11.0.6/Native Method)
        at java.lang.ref.Reference.processPendingReferences(java.base@11.0.6/Reference.java:241)
        at java.lang.ref.Reference$ReferenceHandler.run(java.base@11.0.6/Reference.java:213)

"Finalizer" #3 daemon prio=8 os_prio=1 cpu=0.00ms elapsed=1227.39s tid=0x0000022dbcc67000 nid=0x3158 in Object.wait()  [0x000000e64fffe000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(java.base@11.0.6/Native Method)
        - waiting on <0x00000000d00e23c0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(java.base@11.0.6/ReferenceQueue.java:155)
        - waiting to re-lock in wait() <0x00000000d00e23c0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(java.base@11.0.6/ReferenceQueue.java:176)
        at java.lang.ref.Finalizer$FinalizerThread.run(java.base@11.0.6/Finalizer.java:170)

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 cpu=0.00ms elapsed=1227.38s tid=0x0000022dbcc88800 nid=0x2364 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Attach Listener" #5 daemon prio=5 os_prio=2 cpu=0.00ms elapsed=1227.38s tid=0x0000022dbcc8b800 nid=0x21d8 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #6 daemon prio=9 os_prio=2 cpu=1109.38ms elapsed=1227.38s tid=0x0000022dbcc8d800 nid=0x57c waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C1 CompilerThread0" #8 daemon prio=9 os_prio=2 cpu=671.88ms elapsed=1227.37s tid=0x0000022dbccf9800 nid=0x18bc waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"Sweeper thread" #9 daemon prio=9 os_prio=2 cpu=0.00ms elapsed=1227.36s tid=0x0000022dbd501000 nid=0x12d4 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Service Thread" #10 daemon prio=9 os_prio=0 cpu=0.00ms elapsed=1227.33s tid=0x0000022dbd5e4800 nid=0xe38 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Common-Cleaner" #11 daemon prio=8 os_prio=1 cpu=0.00ms elapsed=1227.33s tid=0x0000022dbd5ea800 nid=0x488 in Object.wait()  [0x000000e6507fe000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
        at java.lang.Object.wait(java.base@11.0.6/Native Method)
        - waiting on <0x00000000d01a1f98> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(java.base@11.0.6/ReferenceQueue.java:155)
        - waiting to re-lock in wait() <0x00000000d01a1f98> (a java.lang.ref.ReferenceQueue$Lock)
        at jdk.internal.ref.CleanerImpl.run(java.base@11.0.6/CleanerImpl.java:148)
        at java.lang.Thread.run(java.base@11.0.6/Thread.java:834)
        at jdk.internal.misc.InnocuousThread.run(java.base@11.0.6/InnocuousThread.java:134)
        
 // .................
 
"RMI TCP Connection(idle)" #27 daemon prio=5 os_prio=0 cpu=0.00ms elapsed=24.77s tid=0x0000022dbf12f000 nid=0x1724 waiting on condition  [0x000000e64f5fe000]
   java.lang.Thread.State: TIMED_WAITING (parking)
        at jdk.internal.misc.Unsafe.park(java.base@11.0.6/Native Method)
        - parking to wait for  <0x00000000d00b9568> (a java.util.concurrent.SynchronousQueue$TransferStack)
        at java.util.concurrent.locks.LockSupport.parkNanos(java.base@11.0.6/LockSupport.java:234)
        at java.util.concurrent.SynchronousQueue$TransferStack.awaitFulfill(java.base@11.0.6/SynchronousQueue.java:462)
        at java.util.concurrent.SynchronousQueue$TransferStack.transfer(java.base@11.0.6/SynchronousQueue.java:361)
        at java.util.concurrent.SynchronousQueue.poll(java.base@11.0.6/SynchronousQueue.java:937)
        at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@11.0.6/ThreadPoolExecutor.java:1053)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@11.0.6/ThreadPoolExecutor.java:1114)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@11.0.6/ThreadPoolExecutor.java:628)
        at java.lang.Thread.run(java.base@11.0.6/Thread.java:834)

"VM Thread" os_prio=2 cpu=31.25ms elapsed=1227.40s tid=0x0000022dbcc60000 nid=0x1d6c runnable

"GC Thread#0" os_prio=2 cpu=31.25ms elapsed=1227.42s tid=0x0000022da3b44000 nid=0x2d7c runnable

"GC Thread#1" os_prio=2 cpu=31.25ms elapsed=1226.21s tid=0x0000022dbe605800 nid=0x2220 runnable

"GC Thread#2" os_prio=2 cpu=15.63ms elapsed=1226.21s tid=0x0000022dbe58c800 nid=0x2200 runnable

"GC Thread#3" os_prio=2 cpu=15.63ms elapsed=1226.21s tid=0x0000022dbe3bf800 nid=0x22e0 runnable

"G1 Main Marker" os_prio=2 cpu=0.00ms elapsed=1227.42s tid=0x0000022da3b58800 nid=0x191c runnable

"G1 Conc#0" os_prio=2 cpu=0.00ms elapsed=1227.42s tid=0x0000022da3b5a000 nid=0x8a0 runnable

"G1 Refine#0" os_prio=2 cpu=15.63ms elapsed=1227.41s tid=0x0000022da3be4000 nid=0x148c runnable

"G1 Refine#1" os_prio=2 cpu=0.00ms elapsed=1226.20s tid=0x0000022dbd68e800 nid=0x1460 runnable

"G1 Young RemSet Sampling" os_prio=2 cpu=0.00ms elapsed=1227.41s tid=0x0000022da3be7000 nid=0x1e44 runnable
"VM Periodic Task Thread" os_prio=2 cpu=0.00ms elapsed=1227.33s tid=0x0000022dbd5e5800 nid=0x4a8 waiting on condition

JNI global refs: 17, weak refs: 0
```



### 基础工具总结

#### 基础工具

用于支持基本的程序创建和运行

|     名称     | 主要作用                                                 |
| :----------: | -------------------------------------------------------- |
| appletviewer | 在不使用Web浏览器的情况下运行和调试Applet、JDK11中被移除 |
|   extcheck   | 检测Jar冲突的工具，从JDK9移除                            |
|     jar      | 创建和管理Jar文件                                        |
|     java     | Java运行工具，用于运行Class文件或Jar文件                 |
|    javac     | 用于Java的编程语言的编译器                               |
|   javadoc    | Java的API文档生成器                                      |
|    javah     | C语言头文件和Stub函数生成器，用于编写JNI方法             |
|    javap     | Java字节码分析工具                                       |
|    jlink     | 将Module和它的依赖打包成一个运行时镜像文件               |
|     jdb      | 基于JPDA协议的调试器，以类似于GDB的方式进行调试Java代码  |
|    jdeps     | Java类依赖性分析器                                       |
|  jdeprscan   | 用于搜索Jar包中使用了“Deprecated”的类，从JDK9开始提供    |

#### 安全

用于程序签名、设置安全测试等

|    名称    | 主要作用                                                     |
| :--------: | ------------------------------------------------------------ |
|  keytool   | 管理密钥库和证书。主要用于获取或缓存Kerberos协议的票据授权票据。允许用户查看本地凭据缓存和秘钥表中的条目 |
| jarsigner  | 生成并验证Jar签名                                            |
| policytool | 管理策略文件的GUI工具，用于管理用户策略文件（.java.policy），在JDK10中被移除 |

#### 国际化

用于创建本地语言文件

|     名称     | 主要作用                                                     |
| :----------: | ------------------------------------------------------------ |
| native2ascii | 本地编码到ASCII编码的转换器（Native-to-ASCII Converter），用于“任意受支持的字符编码”和与之对应的“ASCII编码和Unicode转义”之间的相互转换 |

#### 远程方法调用

用于跨Web或网络的服务交互

|    名称     | 主要作用                                                     |
| :---------: | ------------------------------------------------------------ |
|    rmic     | Java RMI编译器，为使用JRMP或IIOP协议的远程对象生成Stub、Skeleton和Tie类，也用于生成OMG IDL |
| rmiregistry | 远程对象注册表服务，用于在当前主机的指定端口上创建并启动一个远程对象注册表 |
|    rmid     | 启动激活系统守护进程，允许在虚拟机中注册或激活对象           |
|  serialver  | 生成并返回指定类的序列化版本ID                               |

#### Java IDL与RMI-IIOP

在JDK 11中结束了十余年的CORBA支持，这些工具不在提供

|    名称    | 主要作用                                                     |
| :--------: | ------------------------------------------------------------ |
| tnameserv  | 提供对命名服务的访问                                         |
|    idlj    | IDL转Java编译器（IDL-to-Java Compiler），生成映射OMG IDL接口的Java源文件，并启用以Java编程语言编写的使用CORBA功能的应用程序的Java源文件。IDL意即接口定义语言（Interface Definition Language） |
|    orbd    | 对象请求代理守护进程（Object Request Broker Daemon），提供从客户端查找和调用CORBA环境服务端上的持久化对象的功能。使用CORBA代理瞬态命名服务tnameserv。ORBD包括瞬态命名服务和持久化命名服务。ORBD工具集成了服务器管理器、互操作命名服务和引导名称服务器的功能。当客户端想进行服务器时定位、注册和激活功能时，可以与servertool一起使用 |
| servertool | 为应用程序注册、注销、启动和关闭服务器提供易用的接口         |

#### 部署工具

用于程序打包、发布和部署。

|     名称     | 主要作用                                                     |
| :----------: | ------------------------------------------------------------ |
| javapackager | 打包、签名Java和JavaFX应用程序，在JDK11中被移除              |
|   pack200    | 使用Java GZIP压缩器将JAR文件转换为压缩的Pack200文件。压缩的压缩文件是高度压缩的JAR，可以直接部署，节省带宽并减少下载时间 |
|  unpack200   | 将Pack200生成的打包文件解压提取为JAR文件                     |

#### Java Web Start

|  名称  | 主要作用                                                |
| :----: | ------------------------------------------------------- |
| javaws | 启动Java Web Start并设置各种选项的工具。在JDK11中被移除 |

#### 性能监控和故障处理

用于监控分析Java虚拟机运行信息，排查问题

|   名称    | 主要作用                                                     |
| :-------: | ------------------------------------------------------------ |
|    jps    | JVM Process Status Tool，显示指定系统内所有的HotSpot虚拟机进程 |
|   jstat   | JVM Statistics Monitoring Tool，用于收集HotSpot虚拟机各方面的运行数据 |
|  jstatd   | JVM Statistics Monitoring Tool Daemon，jstat的守护程序，启动一个RMI服务器应用程序，用于监测测试的HotSpot虚拟机的创建和终止，并提供一个界面，运行远程监控工具附加到在本地系统上运行的虚拟机。在JDK9中集成到了JHSDB中 |
|   jinfo   | Configuration Info for Java，显示虚拟机配置信息。在JDK9中集成到了JHSDB中 |
|   jmap    | Memory Map for Java，生成虚拟机的内存转储快照（heapdump文件）。在JDK9中集成到了JHSDB中 |
|   jhat    | JVM Heap Analysis Tool，用于分析堆转储快照，它会建立一个HTTP/Web服务器， 让用户可以在浏览器上查看分析的结果。在JDK9中被JHSDB代替 |
|  jstack   | Stack Trace for Java，显示虚拟机的线程快照。在JDK9中集成到了JHSDB中 |
|   jhsdb   | Java HotSpot Debugger，一个基于Serviceability Agent的HotSpot进程调试器，从JDK9开始提供 |
| jsadebugd | Java Serviceability Agent Debug Daemon，适用于Java的可维护性代理调试守护程序，主要用于附加到指定的Java进程、核心文件，或充当一个调试服务器 |
|   jcmd    | JVM Command，虚拟机诊断命令工具，将诊断命令请求发送到正在运行的Java虚拟机。从JDK7开始提供 |
| jconsole  | Java Console，用于监控Java虚拟机的使用JMX规范的图形工具。它可以监控本地和远程Java虚拟机，还可以监控和管理应用程序 |
|    jmc    | Java Mission Control，包含用于监控和管理Java应用程序的工具，而不会引入与这些工具相关联的性能开销。开发者可以使用jmc命令来创建JMC工具，从JDK7 Update 40开始集成到OracleJDK中 |
| jvisualvm | Java VisualVM，一种图形工具，可在Java虚拟机中运行时提供有关基于Java技术的应用程序（Java应用程序）的详细信息。Java VisualVM提供内存和CPU分析、堆转储分析、内存泄漏检测、MBean访问和垃圾收集。从JDK6 Update 7开始提供；从JDK9开始不再打包入JDK中，但仍然保持更新发展，可以独立下载使用 |

#### WebService工具

与CORBA一起在JDK11被移除

|   名称    | 主要作用                                                     |
| :-------: | ------------------------------------------------------------ |
| schemagen | 用于XML绑定的Schema生成器，用于生成XML Schema文件            |
|   wsgen   | XML Web Service 2.0的Java API，生成用于JAX-WS Web Service的JAX-WS便携式产物 |
| wsimport  | XML Web Service 2.0的Java API，主要用于根据服务端发布的WSDL文件生成客户端 |
|    xjc    | 主要用于根据XML Schema文件生成对应的Java类                   |

#### REPL和脚本工具

|    名称    |                           主要作用                           |
| :--------: | :----------------------------------------------------------: |
|   jshell   |     基于Java的Shell REPL（Read-Eval-Print Loop）交互工具     |
|    jjs     | 对Nashorn引擎的调用入口。Nashorn是基于Java实现的一个轻量级高性能JavaScript运行环境 |
| jrunscript | Java命令行脚本外壳工具（Command Line Script Shell），主要用于解释执行JavaScript、Groovy、Ruby等脚本语言 |

### 总结

本文介绍了JDK提供的自带的常用故障处理工具，在日常线上环境使用的尤为频繁，JDK还提供了可视化的工具就不介绍了，有兴趣的可以自己去尝试使用，如：JConsole和VisualVM等可视化工具。
