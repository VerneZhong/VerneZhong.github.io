---
title: Java Lock之ReentrantLock
date: 2020-09-12 10:53:30
categories:
- Java技术
tags:
- Lock
- ReentrantLock
---

![](https://github.com/user-attachments/assets/79f83b3c-bc5c-4bd5-80c9-e63ea97380a7)

<!-- more -->

## Java Lock之ReentrantLock

### Java 锁

通常，在多线程环境时，为了确保线程安全，我们会使用`synchronized`关键字来保证线程安全。在大多数情况下，`synchronized`关键字是解决之道，但它由一些缺点，导致我们放弃使用它。Java 1.5 Concurrency API附带了带有接口和一些实现类的`java.util.concurrent.locks`软件包，`Lock`以改进对象锁定机制。

### Java  Lock 结构

```java
public interface Lock {

    /**
     * 获取锁
     */
    void lock();

    /**
     * 获取锁直到当前线程被中断
     */
    void lockInterruptibly() throws InterruptedException;

    /**
     * 仅在调用时释放锁时才获取锁。
     */
    boolean tryLock();

    /**
     * 如果锁在给定的等待时间内是空闲的，并且当前线程尚未被中断，则获取该锁。
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    /**
     * 释放锁
     */
    void unlock();

    /**
     * 返回绑定到此Lock实例的新Condition实例。
     */
    Condition newCondition();
}
```

Java Lock API 中的一些重要接口和类：

1. **Lock**：这是Lock API的基本接口。它提供了`synchronized`关键字的所有功能，并提供了其他方式来创建不同的锁定条件，从而为线程等待锁定提供了超时。一些重要的方法是：lock() 获取锁，unlock() 释放锁，tryLock() 等待锁一定时间，newCondition() 创建Condition 阻塞队列接口等。

2. **Condition**：Condition对象类似于`Object wait-notify`模型，具有附加功能以创建不同的wait集。Condition对象始终由Lock对象创建。一些重要的方法是类似于wait() 和signal() 的await() ，类似于notify() 和notifyAll() 方法的signalAll()。

3. **ReadWriteLock**：它包含一对关联的锁，一个用于只读操作，另一个用于写入。只要没有写程序线程，读锁就可以同时由多个读程序线程持有。写锁是排他的。

4. **ReentrantLock**：这是Lock接口使用最广泛的实现类。此类以与synchronized关键字相似的方式实现Lock接口。除了Lock接口的实现之外，ReentrantLock还包含一些实用方法来获取持有锁的线程，等待获取锁的线程等。

   `synchronized`同步方法本质上是可重入的，即，如果一个线程在监视对象上具有锁，并且如果另一个同步块需要在同一监视对象上具有锁，则线程可以输入该代码块。我认为这是类名称为ReentrantLock的原因。让我们通过一个简单的示例来了解此功能。

   ```java
   /**
    * 线程安全的类，使用synchronized关键字来保证线程安全
    * @author Mr.zxb
    * @date 2020-09-12 11:53:37
    */
   public class SafeThreadTest {
       public synchronized void foo() {
           // do something
           bar();
       }
   
       public synchronized void bar() {
           // do some more
       }
   }
   ```

   如果线程输入foo()，则它具有`SafeThreadTest`对象的锁定，因此，当它尝试执行bar()方法时，由于该线程已经持有`SafeThreadTest`对象的锁定，因此允许该线程执行bar()方法。


### Java 中的  ReentrantLock

`ReentrantLock`是一个可重入的互斥锁，所谓可重入是线程可以重复获取已经持有的锁。锁基本上都是要支持可重入性，否则很容易出现死锁问题。

`ReentrantLock`内部实现主要通过`AbstractQueuedSynchronizer`类实现的，`AbstractQueuedSynchronizer`是抽象类，在`ReentrantLock`类中有两个实现类：`NonfairSync`和`FairSync`，分别对应非公平锁和公平锁的实现。

ReentrantLock类内部持有一个Sync类型的变量，主要实现基本上都是调用Sync的实现机制，默认构建的是NonfairSync，即非公平锁，也可以通过带Boolean类型的构造函数构建公平锁，源码如下：

```java
/**
* 1、默认创建的非公平锁，性能更高，等价于ReentrantLock(false)
*/
public ReentrantLock() {
    sync = new NonfairSync();
}

/**
* @param fair true:公平锁    false:非公平锁
*/
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

现在，让我们看一个简单的示例，其中将用Java Lock API替换`synchronized`关键字。

假设我们有一个Resource类，其中包含一些我们希望它是线程安全的操作，以及一些不需要线程安全的方法。

```java
public class Resource {
    public void doSomething() {
        // do some operation, DB read, write etc
    }

    public void doLogging() {
        // logging, no need for thread safety
    }
}
```

现在，我们有一个`Runnable`类，在其中我们将使用`Resouce`方法。

```java
/**
 * synchronized 代码块来保证线程安全
 * @author Mr.zxb
 * @date 2020-09-12 12:01:24
 */
public class SynchronizedLockExample implements Runnable {

    private final Resource resource;

    public SynchronizedLockExample(Resouce resource) {
        this.resource = resource;
    }

    @Override
    public void run() {
        // 数据库读写操作需要线程安全
        synchronized (resouce) {
            resource.doSomething();
        }
        // 日志操作，无须线程安全
        resource.doLogging();
    }
}
```

请注意，我们正在使用同步代码块来获取对Resource对象的锁定，我们可以在类中创建一个虚拟对象，并将其用于锁定目的。

现在让我们看看如何使用`Java Lock API`并在不使用`synchronized`关键字的情况下重写上述程序。我们将在类中使用`ReentrantLock`。

```java
/**
 * 基于Java Lock API {@link java.util.concurrent.locks.ReentrantLock} 来实现线程安全的示例
 *
 * @author Mr.zxb
 * @date 2020-09-12 12:05:44
 */
public class ConcurrencyLockExample implements Runnable {
    private final Resource resource;

    private Lock lock;

    public ConcurrencyLockExample(Resource resource) {
        this.resource = resource;
        // 默认创建非公平锁
        this.lock = new ReentrantLock();
    }

    @Override
    public void run() {
        try {
            // 等待获取锁
            if (lock.tryLock(10, TimeUnit.SECONDS)) {
                resource.doSomething();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            // 释放锁
            lock.unlock();
        }
        resource.doLogging();
    }
}
```

如您所见，我正在使用tryLock() 方法来确保我的线程仅等待一定的时间，并且如果它没有获得对象的锁，那么它只是在记录并退出。需要注意的另一个重要点是，即使doSomething() 方法调用引发任何异常，也要使用`try-finally`块来确保释放锁。

### Java Lock 与 synchronized的区别

基于以上程序，我们可以轻松得出Java Lock与`synchronized`之间的差异：

1. Java Lock API提供了更多的锁定可见性和选项，与同步机制不同，同步机制可能导致线程无限期地等待锁定，因此我们可以使用tryLock() 来确保线程仅在特定时间等待。
2. 同步代码更简洁，易于维护，而使用Lock时，即使在lock() 和unlock() 方法调用之间引发了某些异常，我们也不得不尝试进行最后锁定，以确保释放Lock。
3. 同步块或方法只能覆盖一个方法，而我们可以使用Lock API在一个方法中获取锁并在另一方法中释放锁。
4. `synchronized`关键字不提供公平性，而我们在创建`ReentrantLock`对象时可以将公平性设置为true，以便等待时间最长的线程首先获得该锁。
5. 我们可以为Lock创建不同的条件，并且不同的线程可以为不同的条件使用await() 。

### 总结

Lock机制的核心就是通过cas原子操作AQS中的state属性，state=0表示锁资源可用，获取锁就是通过cas原子操作将state从0设置成1，成功就表示获取锁成功，如果state>0，cas操作将会失败，即表示锁已被占用，当前获取锁失败。获取锁失败，根据是否是可中断、可超时等特性，处理的逻辑不太一致，但大致为：

1. 将获取锁失败的线程封装成Node，封装成Node一方面是要构建双向队列，另一方面是Node中额外添加状态信息对节点进行控制。
2.  在一个for无线循环中通过Lock.park()让线程休眠，当有锁资源被释放发生时，会从队列头到尾的顺序依次唤醒线程(会跳过CANCELLED标记的节点，因为这些节点代表的线程已经无效了)，注意这里只会唤醒一个线程，唤醒的线程只表示该线程具有竞争锁资源的资格，还需要和新申请但还没有放入到Queue中的线程进行竞争该锁资源，这就是非公平锁的特性，这样设计主要是从性能方面考虑，如果竞争成功则退出for循环返回，否则继续进入休眠状态。