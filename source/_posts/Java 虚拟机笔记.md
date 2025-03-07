---
title: Java 虚拟机笔记
date: 2020-08-06 16:53:36
categories: 
- JVM笔记
tags:
- JVM内存区域
- JVM对象
---
![](https://github.com/user-attachments/assets/cf0e9371-c81e-4491-b7a0-2304999da266)

<!-- more -->

### 																												JVM 虚拟机笔记

#### JVM运行时数据区域

![](https://github.com/user-attachments/assets/b588cd99-2c55-4e43-b2f7-d83509d15194)

##### 程序计数器（Program Counter Register）

- 内存大小：是一块比较小的内存空间
- 作用：当前线程所执行的字节码的行号指示器，在Java虚拟机的概念模型里，字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令

- 功能
  - 程序控制流的指示器
  - 分支
  - 循环
  - 跳转
  - 异常处理
  - 线程恢复

- 内存属性：各个线程之间计数器互不影响，“线程私有”

##### Java虚拟机栈（Java Virtual Machine Stacks）

- 描述
  - Java方法执行的内存模型，每个方法执行时都会创建栈帧（Stack Frame）

- 栈帧
  - 存储局部变量表
    - 基本数据类型（boolean、byte、char、short、int、float、double、long）
    - 对象引用（reference类型）
    - returnAddress类型（字节码指令地址）
  - 操作数栈
  - 动态连接
  - 方法出口

- 内存属性：线程私有，生命周期与线程相同
- 异常
  - StackOverflowError
    - 线程请求的栈深度大于虚拟机所允许的深度，抛出StackOverflowError异常
  - OutOfMemoryError
    - 如果虚拟机栈容量可扩展，当栈扩展时无法申请到足够内存，抛出OutOfMemoryError异常（Classic虚拟机栈可以动态扩展）
    - 在HotSpot虚拟机的栈容量是不可动态扩展，所以不会因为动态扩展而OOM，只要线程申请栈空间成功了就不会有OOM，但是申请失败仍然会有OOM异常

##### 本地方法栈（Native Method Stack）
- 功能与虚拟机栈非常相似，区别：
  - Java虚拟机栈为虚拟机执行Java方法（字节码）服务
  - 本地方法栈为虚拟机使用Native方法服务
- 异常
  - StackOverflowError
    - 当栈深度溢出则抛出
  - OutOfMemoryError
    - 当扩展失败时抛出

##### Java堆（Java Heap）

- 描述
  - 内存大小：Jvm所管理的内存中最大的一块
  - 被所有线程所共享的区域
  - 作用：存放对象实例，几乎所有的对象实例都在堆里分配内存，但随着JIT编译器的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换优化技术会发生微妙变化
  - 堆是垃圾收集器管理的主要区域，因此被称为GC堆
  - 堆可以处于物理上不连续的内存空间中，只要逻辑上连续即可
  - Java堆既可以被实现成固定大小，也可以是可扩展的（通过参数-Xmx和-Xms设定）

- 分类
  - 内存回收角度
    - 现在收集器基本采用分代收集算法
      - 以G1收集器为分界，都基于经典分代来设计（不适合当下情况）
        - 新生代
          - Eden space
          - To Survivor space
          - From Survivor space
        - 老年代
        - 永久代（PermGen）
          - 在Jdk8已经删除，替换成元空间（Metaspaces）
      - HotSpot目前采用不分代设计的新垃圾收集器
  - 内存分配角度
    - 所有线程共享的Java堆中可以划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer， TLAB），以提升分配对象的效率

- 异常
  - OutOfMemoryError
    - 堆没有内存完成实例分配，并且无法扩展时，抛出OOM异常

##### 方法区（Method Area）

- 描述
  - 和Java堆一样，被所有线程所共享，又称Non-Heap（非堆）
  - 在JDK8以前，又把方法区称为“永久代”，其实两者不等价，只是使用永久代来实现方法区而已，这样HotSpot虚拟机就可以像管理堆一样管理这块内存
  - 对于其他虚拟机是不存在永久代概念，在JDK7以前，是很容易导致OOM PermGen space内存溢出的问题
  - 从JDK6开始HotSpot就有放弃永久代，逐步改为采用本地内存（Native Memory）来实现方法区
  - 从JDK7开始，HotSpot把原来放在永久代的字符串常量池、静态变量等从方法区移出到堆里
  - 到了JDK8已经完全废弃了，HotSpot把永久代剩余的类型信息全部移到元空间中
- 作用：用来存储
  - 类信息
  - 常量
  - 静态变量
  - 即时编译器编译后的代码缓存

- 内存回收：
  - 对常量池对回收
  - 对类型的卸载

- 异常
  - OutOfMemoryError
    - 当方法区无法满足内存分配需求时，抛出OOM异常

###### 运行时常量池（Runtime Constant Pool）
  - 描述
    - 方法区的一部分
    - Class文件中除了有类的版本信息、字段、方法、接口等描述信息外，还有一项是常量池表，用于存放编译期生成等各种字面量与符号引用，这部分内容在类加载后存放到方法区的运行时常量池中
  - 作用
    - 存储编译期生成的各种字面量
    - 符号引用
  - 异常
    - OutOfMemoryError
      - 当常量池无法申请内存时，抛出OOM异常

##### 直接内存（堆外内存）
- 并不是虚拟机运行时数据区的一部分，也不是JVM规范中定义的内存区域，是直接受操作系统管理
- 在NIO中的DirectByteBuffer对象就是进行堆外内存管理和使用的，它会在对象创建的时候就分配堆外内存。DirectByteBuffer类是在Java Heap外分配内存，对堆外内存的申请主要是通过成员变量Unsafe来操作
- 好处：能够在一定程度上减少垃圾回收对应用程序造成的影响
- 若忽略直接内存，会使各个内存区域总和大于物理内存限制，从而动态扩展时出现OOM异常

#### JVM 对象
##### 对象创建
![WX20200813-151534@2x](https://github.com/user-attachments/assets/69badbbf-98fd-4d8e-a264-a9ade5dfa0fe)

- 检查指令参数是否在常量池定位到类的符号引用
- 检查符号引用的类是否已加载、解析、初始化
- 为新生对象分配内存
  - 堆内存规整，则用指针碰撞方式分配
  - 堆内存不规整，则用空闲列表方式分配
  - 堆内存规整与否，取决于采用的垃圾收集器是否带有空间压缩整理的能力
  - 当使用Serial、ParNew等带有压缩整理过程的收集器，系统采用等是指针碰撞方式
  - 使用CMS这种基于清除（Sweep）算法的收集器时，理论上只能采用较为复杂的空闲列表来实现分配
  - 内存分配安全方式
    - 对分配对内存空间动作进行同步处理
      - CAS+失败重试来保证更新操作对原子性
    - 内存分配对动作按照线程划分在不同对空间之中进行
      - 即每个线程在Java堆中预先分配一小块内存，称为本地线程分配缓冲（Thread Local Allocation Buffer, TLAP）

- 将分配到的内存空间都初始化为0（不包括对象头）
- 为对象进行必要设置
- 执行init为对象初始化

##### 对象的内存布局
- 对象头（Header）
  - 存储对象自身的运行时数据，也被称为“Mark Word”
    - 哈希吗（HashCode）
    - GC分代年龄
    - 锁状态标志
    - 线程持有的锁
    - 偏向线程ID
    - 偏向时间戳

  - 类型指针
    - 对象指向它的类元数据的指针
    - 虚拟机通过指针来确定对象是哪个实例
  - 对象若是数组，必须有一块记录数组长度的数据，普通对象JVM可以通过元数据信息获取对象大小，而数组的元数据无法确定大小

- 实例数据（Instance Data）
  - 对象真正存储的有效信息，也是程序定义的各种类型的字段内容
  - HotSpot虚拟机分配策略是相同宽度的字段总是被分配在一起

- 对齐填充（Padding）
  - 不是必要的，起着占位符的作用
  - HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍
  - 当对象实例没有对齐时，就需要通过填充来补全

##### 对象的访问定位
- Java程序通过栈上的reference数据来操作堆上的具体对象，至于通过什么方式定位引用对象，通过以下两种主流方式
- 句柄
  - 堆分配一块内存作为句柄池，reference中存储对象的句柄地址
  - 优势：reference存储句柄地址，对象移动时只会改变句柄中实例数据指针，而reference本身不用修改

- 直接指针
  - reference存储的直接就是对象地址
  - 优势：速度更快，节省了一次指针定位的时间开销，由于对象的访问在Java非常频繁，因为节省的开销是非常可观的
  - HotSpot 就是使用直接指针方式定位对象