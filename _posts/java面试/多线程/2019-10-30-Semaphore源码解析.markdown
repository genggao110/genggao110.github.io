---
layout:     post
title:      "Semaphore源码解析"
subtitle:   " \"Semaphore详解\""
date:       2019-10-30 19:37:07
author:     "ming"
catalog: true
header-img: "img/host-bg-mylove.jpg"
tags:
    - 并发编程
    - JAVA
---

> "Even the weariest river winds somewhere safe to sea."

### 1. 简介

Semaphore，信号量，它保存了一系列的许可(permits)，每次调用acquire()都将消耗一个许可，每次调用release()都将归还一个许可。

Semaphore通常用于限制同一时间对共享资源的访问次数上，也就是常说的限流。接下来，将具体来看看Java中是如何实现Semaphore的。

### 2. 源码分析

#### 2.1 类结构

![类结构](https://ws1.sinaimg.cn/large/005CDUpdgy1g8ggq3ht1dj30ds0ardfy.jpg)

Semaphore中包含了一个实现AQS的同步器Sync，以及它的两个子类FairSync和NonFairSync，这说明Semaphore也是区分公平模式和非公平模式的。

我们直接来看看内部类Sync的实现：

```java
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 1192457210091910933L;

        // 构造函数，传入许可次数，放入到state中
        Sync(int permits) {
            setState(permits);
        }

        // 获取许可次数
        final int getPermits() {
            return getState();
        }

        // 非公平模式尝试获取许可
        final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                // 获取还有几个许可
                int available = getState();
                // 减去这次需要获取的许可还剩几个许可
                int remaining = available - acquires;
                // 如果剩余许可小于0了则直接返回
                // 如果剩余许可不小于0则尝试原子更新state的值，成功了则返回剩余许可
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
        
        // 释放许可
        protected final boolean tryReleaseShared(int releases) {
            for (;;) {
                // 获取还有几个许可
                int current = getState();
                // 加上这次释放的许可
                int next = current + releases;
                //检测溢出
                if (next < current) // overflow
                    throw new Error("Maximum permit count exceeded");
                // 如果原子更新state的值成功，则说明释放许可成功，返回true
                if (compareAndSetState(current, next))
                    return true;
            }
        }

        // 减少许可
        final void reducePermits(int reductions) {
            for (;;) {
                // 获取还有几个许可
                int current = getState();
                // 减去将要减少的许可
                int next = current - reductions;
                // 检查溢出
                if (next > current) // underflow
                    throw new Error("Permit count underflow");
                // 如果原子更新state的值成功，成功了返回
                if (compareAndSetState(current, next))
                    return;
            }
        }
        
        // 销毁许可
        final int drainPermits() {
            for (;;) {
                // 获取还有几个许可
                int current = getState();
                //如果为0直接返回
                // 如果不为0，把state原子更新为0
                if (current == 0 || compareAndSetState(current, 0))
                    return current;
            }
        }
    }
```
通过Sync的几个实现方法，我们获取到以下几点信息：
1. 许可是在构造方法时传入的；
2. 许可存放在状态变量state中；
3. 尝试获取一个许可的时候，则state的值减去1；
4. 当state的值为0的时候，则无法再获取许可；
5. 释放一个许可的时候，则state的值加1；
6. 许可的个数可以动态改变。

下面，`我们来分析一下内部类NonfairSync的具体实现`：

```java
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = -2694183684443567898L;

        // 构造函数，调用父类的构造方法
        NonfairSync(int permits) {
            super(permits);
        }

        // 尝试获取许可，调用父类的nonfairTryAcquireShared()方法
        protected int tryAcquireShared(int acquires) {
            return nonfairTryAcquireShared(acquires);
        }
    }
```
非公平模式下，直接调用父类的nonfairTryAcquireShared()尝试获取许可。

`我们来分析一下内部类FairSync的具体实现：`

```java
    static final class FairSync extends Sync {
        private static final long serialVersionUID = 2014338818796000944L;

        // 构造方法，调用父类的构造方法
        FairSync(int permits) {
            super(permits);
        }

        // 尝试获取许可
        protected int tryAcquireShared(int acquires) {
            for (;;) {
                // 公平模式需要检测是否前面有排队的
                // 如果有排队的直接返回失败
                if (hasQueuedPredecessors())
                    return -1;
                // 没有排队的再尝试更新state的值
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
    }
```
公平模式下，会首先去检查前面是否有排队的，如果有排队的则获取许可失败，进入队列排队，否则尝试原子更新state的值。

#### 2.2 构造方法

```java
    // 构造方法，创建时要传入许可次数，默认使用非公平模式
    public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }

    // 构造方法，需要传入许可次数，以及是否公平模式
    public Semaphore(int permits, boolean fair) {
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
    }
```
创建Semaphore时需要传入许可次数。Semaphore默认也是非公平模式，但是可以调用第二个构造方法声明为公平模式。

下面，我们简要地介绍下非公平模式下的几个重要方法。

#### 2.3 常用方法介绍

**1. acquire()方法**

```java
    // Semaphore.acquire
    public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }

    // AbstractQueuedSynchronizer.acquireSharedInterruptibly
    public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
```
获取一个许可，默认使用的是可中断方式，如果尝试获取许可失败，会进入AQS的队列中进行排队。

**2. acquireUniterruptibly()方法**

```java
    public void acquireUninterruptibly() {
        sync.acquireShared(1);
    }
```
获取一个许可，非中断方式，如果尝试获取许可失败，会进入AQS的队列中进行排队。

**3. tryAcquire()方法**

```java
    public boolean tryAcquire() {
        return sync.nonfairTryAcquireShared(1) >= 0;
    }
```
尝试获取一个许可，使用Sync的非公平模式尝试获取许可方法，不论是否获取到许可都返回，只尝试一次，不会进入队列排队。

**4. tryAcquire(long timeout, TimeUnit unit)方法**

```java
    public boolean tryAcquire(long timeout, TimeUnit unit)
        throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }
```
尝试获取一个许可，先尝试一次获取许可，如果失败则会等待TimeOut时间，这段时间内都没有获取到许可，则返回false，否则返回true。

**5. release()方法**

```java
    public void release() {
        sync.releaseShared(1);
    }
```
释放一个许可，释放一个许可时state的值会加1，并且会唤醒下一个等待获取许可的线程。

**6. acquire(int permits)方法**

```java
    public void acquire(int permits) throws InterruptedException {
        if (permits < 0) throw new IllegalArgumentException();
        sync.acquireSharedInterruptibly(permits);
    }
```
一次获取多个许可，可中断方式。

**7. acquireUninterruptibly(int permits)方法**

```java
    public void acquireUninterruptibly(int permits) {
        if (permits < 0) throw new IllegalArgumentException();
        sync.acquireShared(permits);
    }
```
一次获取多个许可，非中断方式。

**8. tryAcquire(int permits)方法**

```java
   public boolean tryAcquire(int permits) {
        if (permits < 0) throw new IllegalArgumentException();
        return sync.nonfairTryAcquireShared(permits) >= 0;
    }
```
一次尝试获取多个许可，只尝试一次。

**9. tryAcquire(int permits, long timeout, TimeUnit unit)方法**

```java
    public boolean tryAcquire(int permits, long timeout, TimeUnit unit)
        throws InterruptedException {
        if (permits < 0) throw new IllegalArgumentException();
        return sync.tryAcquireSharedNanos(permits, unit.toNanos(timeout));
    }
```
尝试获取多个许可，并会等待timeout时间，这段时间没获取到许可则返回false，否则返回true。

**10. release(int permits)方法**

```java
    public void release(int permits) {
        if (permits < 0) throw new IllegalArgumentException();
        sync.releaseShared(permits);
    }
```
一次释放多个许可，state的值会相应增加permits的数量。

**11. availablePermits()方法**

```java
    public int availablePermits() {
        return sync.getPermits();
    }
```
获取可用的许可次数。

**12. drainPermits()方法**

```java
    public int drainPermits() {
        return sync.drainPermits();
    }
```
销毁当前可用的许可次数，对于已经获取的许可没有影响，会把当前剩余的许可全部销毁。

**13. reducePermits(int reduction)方法**

```java
    protected void reducePermits(int reduction) {
        if (reduction < 0) throw new IllegalArgumentException();
        sync.reducePermits(reduction);
    }
```
减少许可的次数。

#### 2.4 总结

- Semaphore，也叫信号量，通常用于控制同一时刻对共享资源的访问，也就是限流场景；
- Semaphore的内部实现是基于AQS的共享锁来实现的；
- Semaphore初始化的时候需要指定许可的次数，许可的次数是存储在state中；
- 获取一个许可时，则state的值减去1；
- 释放一个许可时，则state的值加1；
- 可以动态减少n个许可；
- 可以动态增加n个许可。

### 3. 补充

**1. 如何动态增加n个许可？**

调用release(int permits)即可。我们知道释放许可的时候state的值会相应增加，再回头看看释放许可的源码，发现与ReentrantLock的释放锁有些区别，Semaphore释放许可的时候并不会检查线程有没有获取过许可，所以可以调用释放许可的方法动态增加一些许可。

**2. 如何实现限流？**

限流，即在流量突然增大的时候，上层要能够限制住突然的大流量对下游服务的冲击，在分布式系统中限流一般都做在网关层，当然在个别功能中也可以自己简单地来实现限流，比如秒杀场景，假如只有10个商品需要秒杀，那么，服务本身可以限制同时只能进来100个请求，其他请求全部作废，这样服务的压力也不会太大。

使用Semaphore就可以直接针对这个功能来限流，以下是代码实现：

```java
public class SemaphoreTest {

    public static final Semaphore SEMAPHORE = new Semaphore(100);
    public static final AtomicInteger failCount = new AtomicInteger(0);
    public static final AtomicInteger successCount = new AtomicInteger(0);

    public static void main(String[] args) {
        for (int i = 0; i < 1000; i++){
            new Thread(()-> seckill()).start();
        }
    }

    public static boolean seckill(){
        if (!SEMAPHORE.tryAcquire()){
            System.out.println("no permits, count=" + failCount.incrementAndGet());
            return false;
        }

        try {
            Thread.sleep(2000);
            System.out.println("seckill success, count=" + successCount.incrementAndGet());
        }catch (InterruptedException e){
            e.printStackTrace();
        }finally {
            SEMAPHORE.release();
        }
        return true;
    }
}
```