---
layout:      post
title:       "ThreadLocal Analysis"
subtitle:    " \"Source Code Analysis\""
date:        2019-04-18 10:45:26
author:      "Ming"
catalog: true
header-img:  "img/about-bg-walle.jpg"
tags:
    - JAVA
    - Knowledge
---

> "I figure life is a gift. I don’t intend on wasting it. To make each day count!"

#### ThreadLocal是什么?

ThreadLocal(线程局部变量),是一个本地线程副本变量工具类，其作用是将私有线程和该线程存放的副本对象做一个映射，各个线程之间的变量互不干扰，在高并发场景下，可以实现无状态的调用，特别适用于各个线程依赖不同的变量值完成操作的场景。( **官方文档介绍**： ThreadLocal类提供了线程局部 (thread-local) 变量。这些变量与普通变量不同，每个线程都可以通过其 get 或 set方法来访问自己的独立初始化的变量副本。ThreadLocal 实例通常是类中的 private static 字段，它们希望将状态与某一个线程（例如，用户 ID 或事务 ID）相关联。)

通俗点说，ThreadLocal为变量在每一个线程中都创建了一个副本，每个线程都可以访问自己内部的副本变量。当我们在进行并发编程时，成员变量如果不做任何处理其实是线程不安全的，各个线程都在操作同一个变量,并且voliatile这个关键字也是不能保证线程安全的。

#### 为什么要使用ThreadLocal?

不妨考虑下面这样一个这样一个应用场景：

```java
class ConnectionUtil{
    private static Connection conn = null;

    public static Connection openConnection(){
        if(conn == null){
            conn = DriverManager.getConnection();
        }
        return conn;
    }

    public static void closeConnection(){
        if(conn != null){
            conn.close();
        }
    }
}

```
假如有这样一个数据库链接类，很显然，在多线程的使用情景下使用conn变量就会存在线程安全问题：第一，openConnection和closeConnection方法都没有同步，很可能在openConnection方法中会多次创建connect；第二，由于conn是共享变量，那么必然在调用conn的地方需要使用同步来保障线程安全(因为很可能一个线程在使用conn进行数据库操作，而另外一个线程调用closeConnection关闭链接)。

所以出于线程安全的考虑，必须将ConnectionUtil的两个方法进行同步处理，并且在调用conn的地方进行同步处理。虽然这是一种可行的技术方案，但是这会大大影响程序的执行效率。

仔细分析一下这个问题，在这里我们到底需不需要将conn变量进行共享？事实上，假如每个线程中都有一个conn变量，各个线程之间对conn变量的访问实际上是没有依赖关系的(即一个线程不需要关心其他线程是否对这个conn进行了修改)。可能，有人会说既然不需要共享变量，那么可以如下面这样处理：

```java
class ConnectionUtil{
    private Connection conn = null;

    public Connection openConnection(){
        if(conn == null){
            conn = DriverManager.getConnection();
        }
        return conn;
    }

    public void closeConnection(){
        if(conn != null){
            conn.close();
        }
    }
}

class Dao{
    public void insert(){
    ConnectionUtil connectionUtil = new ConnectionUtil();
    Connection conn = connectionUtil.openConnection();
    //进行数据库操作
    ...
    conn.insert();
    ...
    ConnectionUtil.closeConnection();
    }
}
```
这样处理确实也没有任何问题，由于每次都是在方法内部创建的连接，那么线程之间自然不存在线程安全问题。但是这样会有一个致命的影响：导致服务器压力非常大，并且严重影响程序执行性能。由于在方法中需要频繁地开启和关闭数据库连接，这样不仅严重影响程序执行效率，还可能导致服务器压力巨大。

在面对这种情况下，使用ThreadLocal就很合适了。因为ThreadLocal在每个线程中对该变量会创建一个副本，即每个线程内部都会有一个该变量，且在线程内部任何地方都可以使用，线程之间互不影响，这样一来就不存在线程安全问题，也不会严重影响程序执行性能。

**重点补充**

> 首先，**ThreadLocal不是用来解决共享对象的多线程访问问题的**。一般情况下，通过ThreadLocal.set()到线程中的对象是该线程自己使用的对象，其他线程是不需要访问的，也访问不到的(因为各个线程中访问的是不同的对象，这也是所说副本的真实含义)。另外，说ThreadLocal使得各线程能够保持各自独立的一个对象，并不是通过ThreadLocal.set()来实现的，**而是通过每个线程中的new对象的操作来创建的对象，每个线程创建一个，不是什么对象的拷贝或副本**。如果ThreadLocal.set()进去的东西本来就是多个线程共享的一个对象，那么多个线程的ThreadLocal.get()取得的还是这个共享对象本身，还是有并发访问的问题的。**其实，说白了，同步与ThreadLocal是解决多线程中数据访问问题的两种思路，同步是数据共享的思路，以时间换空间思想；而后者是数据隔离思路，以空间换时间的思想。**

#### 从数据结构入手

![ThreadLocal内部结构图](https://ws1.sinaimg.cn/large/005CDUpdly1g26uybwclbj30me0n9q5h.jpg)

从上面的结构图，我们可以初步窥见ThreadLocal的核心思想：
- 每个Thread线程内部有一个Map
- Map里面存储线程本地对象(key)和线程变量副本(value)
- 但是，Thread内部的Map是由ThreadLocal维护的，由ThreadLocal负责向map获取和设置线程的变量值。

所以，对于不同的线程，每次获取副本值时，别的线程并不能获取到当前线程的副本值，形成了副本的隔离，互不干扰。

![UML类图](https://ws1.sinaimg.cn/large/005CDUpdly1g26vg3wvmnj30o20iiq3q.jpg)

ThreadLocal中的嵌套内部类ThreadLocalMap,这个类本质上是一个Map,和HashMap的实现相似，依然是key-value的形式，其中有一个内部类Entry,其中key可以看做是ThreadLocal实例，但是本质是持有ThreadLocal实例的弱引用。

了解一下ThreadLocalMap,在ThreadLocal中并没有对于ThreadLocalMap的引用。其ThreadLocalMap的引用是在Thread类中。每个线程再向ThreadLocal里塞值的时候，其实都是向自己所持有的ThreadLocalMap里面塞入数据。读取数值的时候，首先从当前线程中取出自己持有的ThreadLocalMap,然后再根据ThreadLocal引用作为key取出value.

```java
public class Thread implements Runnable {
    ...

    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    ...
}
```

#### ThreadLocalMap源码分析

我们从ThreadLocalMap源码入手，来一步步介绍ThreadLocal,基本上分析完ThreadLocalMap，ThreadLocal也就搞定了。

##### 成员变量

```java
     /**
     * 初始容量 —— 必须是2的冥
     */
    private static final int INITIAL_CAPACITY = 16;

    /**
     * 存放数据的table，Entry类的定义在下面分析
     * 同样，数组长度必须是2的冥。
     */
    private Entry[] table;

    /**
     * 数组里面entrys的个数，可以用于判断table当前使用量是否超过负因子。
     */
    private int size = 0;

    /**
     * 进行扩容的阈值，表使用量大于它的时候进行扩容。
     */
    private int threshold; // Default to 0
    
    /**
     * 定义为长度的2/3
     */
    private void setThreshold(int len) {
        threshold = len * 2 / 3;
    }
```

##### 存储结构——Entry

```java
/**
 * Entry继承WeakReference，并且用ThreadLocal作为key.如果key为null
 * (entry.get() == null)表示key不再被引用，表示ThreadLocal对象被回收
 * 因此这时候entry也可以从table从清除。
 */
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```
Entry继承WeakReference,使用弱引用，可以将ThreadLocal对象的生命周期和线程生命周期解绑，持有对ThreadLocal的弱引用，可以使得ThreadLocal在没有其他强引用的时候被回收，这样避免因为线程得不到销毁导致ThreadLocal对象无法被回收。

##### ThreadLocalMap的set()方法和Hash映射

要了解ThreadLocalMap中Hash映射的方式，首先我们从ThreadLocal的set()方法入手，逐层深入。

**ThreadLocal的set()**

```java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocal.ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
    
    ThreadLocal.ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocal.ThreadLocalMap(this, firstValue);
    }
```
- 获取当前当前线程，并获取当前线程的ThreadLocalMap实例
- 判断这个ThreadLocalMap实例是否为空，不为空则调用map.set()方法，否则调用构造函数ThreadLocal.ThreadLocalMap(this,firstValue)进行初始化ThreadLocalMap

下面来看看ThreadLocalMap的构造函数：

```java
    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        //初始化table
        table = new ThreadLocal.ThreadLocalMap.Entry[INITIAL_CAPACITY];
        //计算索引
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        //设置值
        table[i] = new ThreadLocal.ThreadLocalMap.Entry(firstKey, firstValue);
        size = 1;
        //设置阈值
        setThreshold(INITIAL_CAPACITY);
    }
```
关于计算索引(firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1)),我们以两个数学问题的剖析进行解答。

1. ThreadLocal.ThreadLocalMap规定了table的大小必须是2的N次幂

```java
     /**
     * The initial capacity -- MUST be a power of two.
     */
    private static final int INITIAL_CAPACITY = 16;

    /**
    * The table, resized as necessary.
    * table.length MUST always be a power of two.
    */
    private Entry[] table;
```
因为从计算机的角度讲，对位操作的效率要比数学运算效率高。比如说当前table的长度为16，那么16 -1 = 15，也就是二进制的1111，现在有一个数字是23，也就是二进制的00010111,23%16=7，看一下与 (&) 运算:

00010111 & 00001111 = 00000111, 也就是7，和取模运算结果一样，效率反而高。

所以，关于 _& (INITIAL_CAPACITY - 1)_ ,这是取模的一种方式，对于2的幂作为模数取模，用以代替 _%(2^n)_ 。

2. 神奇的 0x61c88647

关于firstKey.threadLocalHashCode,看一下源码：

```java
    private final int threadLocalHashCode = nextHashCode();
    
    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }
    private static AtomicInteger nextHashCode =
            new AtomicInteger();

    /**
    * The difference between successively generated hash codes - turns
    * implicit sequential thread-local IDs into near-optimally spread
    * multiplicative hash values for power-of-two-sized tables.
    */        
    private static final int HASH_INCREMENT = 0x61c88647;
```
注意到实例变量threadLocalHashCode，定义了一个AtomicInteger类型，每当创建ThreadLocal实例时这个值都会累加0x61c88647(HASH_INCREMENT)，其目的就是为了能够让哈希码能均匀地分布在2的N次方的数组里，也就是Entry[] table中。为什么这样就能产生(firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1))均匀分布的数组下标呢？我们来用python做个测试：

```python
>>> HASH_INCREMENT = 0x61c88647
>>> def magic_hash(n):
...     for i in range(n):
...         nextHashCode = i * HASH_INCREMENT + HASH_INCREMENT
...         print nextHashCode & (n - 1),
...     print
... 
>>> magic_hash(16)
7 14 5 12 3 10 1 8 15 6 13 4 11 2 9 0
>>> magic_hash(32)
7 14 21 28 3 10 17 24 31 6 13 20 27 2 9 16 23 30 5 12 19 26 1 8 15 22 29 4 11 18 25 0
```

可以看出，通过这种方式很好地避免了Hash冲突，二来相邻的两个数字都比较分散。关于这个数字与(斐波那契散列法)以及黄金分割有关，有兴趣的读者可以自行搜索。

##### Hash冲突如何解决

和HashMap的最大的不同在于，ThreadLocalMap结构非常简单，没有next引用，也就是ThreadLocalMap中解决Hash冲突的方式并非链表的方式，而是才采用开地址法且是线性探测的方式。(线性探测就是根据初始key的hashcode的值确定元素在table数组中的位置，如果发现这个位置上已经有其他key值的元素被占用，则利用固定的算法寻找一定步长的下个位置，依次判断，直至找到能够存放的位置。)

先看一下ThreadLocalMap线性探测相关的代码，从中也可以看出来table实际是一个环：

```java
     /**
     * 获取环形数组的下一个索引
     */
    private static int nextIndex(int i, int len) {
        return ((i + 1 < len) ? i + 1 : 0);
    }

    /**
     * 获取环形数组的上一个索引
     */
    private static int prevIndex(int i, int len) {
        return ((i - 1 >= 0) ? i - 1 : len - 1);
    }
```
显然ThreadLocalMap采用线性探测的方式解决Hash冲突的效率很低，如果有大量不同的ThreadLocal对象放入map中时发送冲突，或者发生二次冲突，则效率很低。

**所以这里引出的良好建议是：每个线程只存一个变量，这样的话所有的线程存放到map中的Key都是相同的ThreadLocal，如果一个线程要保存多个变量，就需要创建多个ThreadLocal，多个ThreadLocal放入Map中时会极大的增加Hash冲突的可能。**

##### ThreadLocalMap中的set()

ThreadLocalMap的set()及其set()相关代码如下：(代码来自于：https://www.jianshu.com/p/80866ca6c424)

```java
 private void set(ThreadLocal<?> key, Object value) {
        ThreadLocal.ThreadLocalMap.Entry[] tab = table;
        int len = tab.length;
        //计算索引，上面已经有说过。
        int i = key.threadLocalHashCode & (len-1);

        /**
         * 根据获取到的索引进行循环，如果当前索引上的table[i]不为空，在没有return的情况下，
         * 就使用nextIndex()获取下一个（上面提到到线性探测法）。
         */
        for (ThreadLocal.ThreadLocalMap.Entry e = tab[i];
             e != null;
             e = tab[i = nextIndex(i, len)]) {
            ThreadLocal<?> k = e.get();
            //table[i]上key不为空，并且和当前key相同，更新value
            if (k == key) {
                e.value = value;
                return;
            }
            /**
             * table[i]上的key为空，说明被回收了（上面的弱引用中提到过）。
             * 这个时候说明改table[i]可以重新使用，用新的key-value将其替换,并删除其他无效的entry
             */
            if (k == null) {
                replaceStaleEntry(key, value, i);
                return;
            }
        }

        //找到为空的插入位置，插入值，在为空的位置插入需要对size进行加1操作
        tab[i] = new ThreadLocal.ThreadLocalMap.Entry(key, value);
        int sz = ++size;

        /**
         * cleanSomeSlots用于清除那些e.get()==null，也就是table[index] != null && table[index].get()==null
         * 之前提到过，这种数据key关联的对象已经被回收，所以这个Entry(table[index])可以被置null。
         * 如果没有清除任何entry,并且当前使用量达到了负载因子所定义(长度的2/3)，那么进行rehash()
         */
        if (!cleanSomeSlots(i, sz) && sz >= threshold)
            rehash();
    }

     /**
     * 替换无效entry
     */
    private void replaceStaleEntry(ThreadLocal<?> key, Object value,
                                   int staleSlot) {
        ThreadLocal.ThreadLocalMap.Entry[] tab = table;
        int len = tab.length;
        ThreadLocal.ThreadLocalMap.Entry e;

        /**
         * 根据传入的无效entry的位置（staleSlot）,向前扫描
         * 一段连续的entry(这里的连续是指一段相邻的entry并且table[i] != null),
         * 直到找到一个无效entry，或者扫描完也没找到
         */
        int slotToExpunge = staleSlot;//之后用于清理的起点
        for (int i = prevIndex(staleSlot, len);
             (e = tab[i]) != null;
             i = prevIndex(i, len))
            if (e.get() == null)
                slotToExpunge = i;

        /**
         * 向后扫描一段连续的entry
         */
        for (int i = nextIndex(staleSlot, len);
             (e = tab[i]) != null;
             i = nextIndex(i, len)) {
            ThreadLocal<?> k = e.get();

            /**
             * 如果找到了key，将其与传入的无效entry替换，也就是与table[staleSlot]进行替换
             */
            if (k == key) {
                e.value = value;

                tab[i] = tab[staleSlot];
                tab[staleSlot] = e;

                //如果向前查找没有找到无效entry，则更新slotToExpunge为当前值i
                if (slotToExpunge == staleSlot)
                    slotToExpunge = i;
                cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
                return;
            }

            /**
             * 如果向前查找没有找到无效entry，并且当前向后扫描的entry无效，则更新slotToExpunge为当前值i
             */
            if (k == null && slotToExpunge == staleSlot)
                slotToExpunge = i;
        }

        /**
         * 如果没有找到key,也就是说key之前不存在table中
         * 就直接最开始的无效entry——tab[staleSlot]上直接新增即可
         */
        tab[staleSlot].value = null;
        tab[staleSlot] = new ThreadLocal.ThreadLocalMap.Entry(key, value);

        /**
         * slotToExpunge != staleSlot,说明存在其他的无效entry需要进行清理。
         */
        if (slotToExpunge != staleSlot)
            cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
    }

    /**
     * 连续段清除
     * 根据传入的staleSlot,清理对应的无效entry——table[staleSlot],
     * 并且根据当前传入的staleSlot,向后扫描一段连续的entry(这里的连续是指一段相邻的entry并且table[i] != null),
     * 对可能存在hash冲突的entry进行rehash，并且清理遇到的无效entry.
     *
     * @param staleSlot key为null,需要无效entry所在的table中的索引
     * @return 返回下一个为空的solt的索引。
     */
    private int expungeStaleEntry(int staleSlot) {
        ThreadLocal.ThreadLocalMap.Entry[] tab = table;
        int len = tab.length;

        // 清理无效entry，置空
        tab[staleSlot].value = null;
        tab[staleSlot] = null;
        //size减1，置空后table的被使用量减1
        size--;

        ThreadLocal.ThreadLocalMap.Entry e;
        int i;
        /**
         * 从staleSlot开始向后扫描一段连续的entry
         */
        for (i = nextIndex(staleSlot, len);
             (e = tab[i]) != null;
             i = nextIndex(i, len)) {
            ThreadLocal<?> k = e.get();
            //如果遇到key为null,表示无效entry，进行清理.
            if (k == null) {
                e.value = null;
                tab[i] = null;
                size--;
            } else {
                //如果key不为null,计算索引
                int h = k.threadLocalHashCode & (len - 1);
                /**
                 * 计算出来的索引——h，与其现在所在位置的索引——i不一致，置空当前的table[i]
                 * 从h开始向后线性探测到第一个空的slot，把当前的entry挪过去。
                 */
                if (h != i) {
                    tab[i] = null;
                    while (tab[h] != null)
                        h = nextIndex(h, len);
                    tab[h] = e;
                }
            }
        }
        //下一个为空的solt的索引。
        return i;
    }

    /**
     * 启发式的扫描清除，扫描次数由传入的参数n决定
     *
     * @param i 从i向后开始扫描（不包括i，因为索引为i的Slot肯定为null）
     *
     * @param n 控制扫描次数，正常情况下为 log2(n) ，
     * 如果找到了无效entry，会将n重置为table的长度len,进行段清除。
     *
     * map.set()点用的时候传入的是元素个数，replaceStaleEntry()调用的时候传入的是table的长度len
     *
     * @return true if any stale entries have been removed.
     */
    private boolean cleanSomeSlots(int i, int n) {
        boolean removed = false;
        ThreadLocal.ThreadLocalMap.Entry[] tab = table;
        int len = tab.length;
        do {
            i = nextIndex(i, len);
            ThreadLocal.ThreadLocalMap.Entry e = tab[i];
            if (e != null && e.get() == null) {
                //重置n为len
                n = len;
                removed = true;
                //依然调用expungeStaleEntry来进行无效entry的清除
                i = expungeStaleEntry(i);
            }
        } while ( (n >>>= 1) != 0);//无符号的右移动，可以用于控制扫描次数在log2(n)
        return removed;
    }


    private void rehash() {
        //全清理
        expungeStaleEntries();

        /**
         * threshold = 2/3 * len
         * 所以threshold - threshold / 4 = 1en/2
         * 这里主要是因为上面做了一次全清理所以size减小，需要进行判断。
         * 判断的时候把阈值调低了。
         */
        if (size >= threshold - threshold / 4)
            resize();
    }

    /**
     * 扩容，扩大为原来的2倍（这样保证了长度为2的冥）
     */
    private void resize() {
        ThreadLocal.ThreadLocalMap.Entry[] oldTab = table;
        int oldLen = oldTab.length;
        int newLen = oldLen * 2;
        ThreadLocal.ThreadLocalMap.Entry[] newTab = new ThreadLocal.ThreadLocalMap.Entry[newLen];
        int count = 0;

        for (int j = 0; j < oldLen; ++j) {
            ThreadLocal.ThreadLocalMap.Entry e = oldTab[j];
            if (e != null) {
                ThreadLocal<?> k = e.get();
                //虽然做过一次清理，但在扩容的时候可能会又存在key==null的情况。
                if (k == null) {
                    //这里试试将e.value设置为null
                    e.value = null; // Help the GC
                } else {
                    //同样适用线性探测来设置值。
                    int h = k.threadLocalHashCode & (newLen - 1);
                    while (newTab[h] != null)
                        h = nextIndex(h, newLen);
                    newTab[h] = e;
                    count++;
                }
            }
        }

        //设置新的阈值
        setThreshold(newLen);
        size = count;
        table = newTab;
    }

    /**
     * 全清理，清理所有无效entry
     */
    private void expungeStaleEntries() {
        ThreadLocal.ThreadLocalMap.Entry[] tab = table;
        int len = tab.length;
        for (int j = 0; j < len; j++) {
            ThreadLocal.ThreadLocalMap.Entry e = tab[j];
            if (e != null && e.get() == null)
                //使用连续段清理
                expungeStaleEntry(j);
        }
    }


```

还是给出一个简单的步骤描述：
1. 先对ThreadLocal里面的threadLocalHashCode取模得到一个table中的位置；
2. 这个位置上如果有数据，获取这个位置上的ThreadLocal
- 判断一下位置上的ThreadLocal和我本身这个ThreadLocal是不是同一个，是的话就覆盖数据，返回
- 不是同一个ThreadLocal,再判断一下位置上的ThreadLocal是不是null,因为是弱引用，是null说明被垃圾回收了,这个时候把新设置的value替换到当前位置上，返回
- 上面都没有返回，给模加1，看看模加1后的table位置上是不是空的，一直循环，直到找到一个table上的位置为空为止，往里面塞value。也就是说：**当table位置上有数据的时候，ThreadLocal采取的是找最近的一个空位置设置数据**。


##### ThreadLocalMap中的getEntry()及其相关

同样的，对于ThreadLocalMap中的getEntry()也从ThreadLocal的get()入手。

```java
public T get() {
    //同set方法类似获取对应线程中的ThreadLocalMap实例
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    //为空返回初始化值
    return setInitialValue();
}
/**
 * 初始化设值的方法，可以被子类覆盖。
 */
protected T initialValue() {
   return null;
}

private T setInitialValue() {
    //获取初始化值，默认为null(如果没有子类进行覆盖)
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    //不为空不用再初始化，直接调用set操作设值
    if (map != null)
        map.set(this, value);
    else
        //第一次初始化，createMap在上面介绍set()的时候有介绍过。
        createMap(t, value);
    return value;
}
```
ThreadLocalMap中的getEntry():

```java
    private ThreadLocal.ThreadLocalMap.Entry getEntry(ThreadLocal<?> key) {
        //根据key计算索引，获取entry
        int i = key.threadLocalHashCode & (table.length - 1);
        ThreadLocal.ThreadLocalMap.Entry e = table[i];
        if (e != null && e.get() == key)
            return e;
        else
            return getEntryAfterMiss(key, i, e);
    }

    /**
     * 通过直接计算出来的key找不到对于的value的时候适用这个方法.
     */
    private ThreadLocal.ThreadLocalMap.Entry getEntryAfterMiss(ThreadLocal<?> key, int i, ThreadLocal.ThreadLocalMap.Entry e) {
        ThreadLocal.ThreadLocalMap.Entry[] tab = table;
        int len = tab.length;

        while (e != null) {
            ThreadLocal<?> k = e.get();
            if (k == key)
                return e;
            if (k == null)
                //清除无效的entry
                expungeStaleEntry(i);
            else
                //基于线性探测法向后扫描
                i = nextIndex(i, len);
            e = tab[i];
        }
        return null;
    }
```
整理一下这个流程：
1. 获取当前线程
2. 尝试去当前线程中拿它的ThreadLocal.ThreadLocalMap
3. 当前线程中判断是否有ThreadLocal.ThreadLocalMap
  - 有就尝试根据当前ThreadLocal的ThreadLocalHashCode取模去table中取值，有就返回，没有模加1继续找
  - 没有就调用setInitalValue()给当前线程ThreadLocal.ThreadLocalMap设置一个初始值

##### ThreadLocalMap中的remove()

```java
    private void remove(ThreadLocal<?> key) {
        ThreadLocal.ThreadLocalMap.Entry[] tab = table;
        int len = tab.length;
        //计算索引
        int i = key.threadLocalHashCode & (len-1);
        //进行线性探测，查找正确的key
        for (ThreadLocal.ThreadLocalMap.Entry e = tab[i];
             e != null;
             e = tab[i = nextIndex(i, len)]) {
            if (e.get() == key) {
                //调用weakrefrence的clear()清除引用
                e.clear();
                //连续段清除
                expungeStaleEntry(i);
                return;
            }
        }
    }
```

remove()方法就很简单了，就是找到对应的table[],调用weakreference的clear()清除引用，然后再调用expungStaleEntry(i)进行清除。

##### ThreadLocalMap的问题

由于ThreadLocalMap的key是弱引用，而Value是强引用。这就导致了一个问题，ThreadLocal在没有外部对象强引用时，发生GC时弱引用Key会被回收，而Value不会回收，如果创建ThreadLocal的线程一直持续运行，那么这个Entry对象中的value就有可能一直得不到回收，发生内存泄露。

**如何避免内存泄漏**

既然Key是弱引用，那么我们要做的事，就是在调用ThreadLocal的get()、set()方法时完成后再调用remove方法，将Entry节点和Map的引用关系移除，这样整个Entry对象在GC Roots分析后就变成不可达了，下次GC的时候就可以被回收。

如果使用ThreadLocal的set方法之后，没有显示的调用remove方法，就有可能发生内存泄露，所以养成良好的编程习惯十分重要，使用完ThreadLocal之后，记得调用remove方法。

```java
ThreadLocal<Session> threadLocal = new ThreadLocal<Session>();
try {
    threadLocal.set(new Session(1, "test"));
    // 其它业务逻辑
} finally {
    threadLocal.remove();
}
```

#### 应用情景

以Hibernate的session获取场景为例子：

```java
private static final ThreadLocal<Session> threadLocal = new ThreadLocal<Session>();

//获取Session
public static Session getCurrentSession(){
    Session session =  threadLocal.get();
    //判断Session是否为空，如果为空，将创建一个session，并设置到本地线程变量中
    try {
        if(session ==null&&!session.isOpen()){
            if(sessionFactory==null){
                rbuildSessionFactory();// 创建Hibernate的SessionFactory
            }else{
                session = sessionFactory.openSession();
            }
        }
        threadLocal.set(session);
    } catch (Exception e) {
        // TODO: handle exception
    }

    return session;
}
```

每个线程访问数据库都应当是一个独立的Session会话，如果多个线程共享同一个Session会话，有可能其他线程关闭连接了，当前线程再执行提交时就会出现会话已关闭的异常，导致系统异常。此方式能避免线程争抢Session，提高并发下的安全性。

使用ThreadLocal的典型场景正如上面的数据库连接管理，线程会话管理等场景，只适用于独立变量副本的情况，如果变量为全局共享的，则不适用在高并发下使用。

#### 总结

1. 每个ThreadLocal只能保存一个变量副本，如果想要一个线程能够保存多个副本以上，就需要创建多个ThreadLocal。
2. ThreadLocal不是集合，它不存储任何内容，真正存储数据的集合在Thread中。ThreadLocal只是一个工具，一个往各个线程的ThreadLocal.ThreadLocalMap中table的某一个位置set一个值的工具而已。
3. ThreadLocal不需要key，因为线程里面自己的ThreadLocal.ThreadLocalMap不是通过链表法实现的，而是通过开地址法实现的。
4. 适用于无状态，副本变量独立后不影响业务逻辑的高并发场景。如果如果业务逻辑强依赖于副本变量，则不适合用ThreadLocal解决，需要另寻解决方案。

相关参考资料：

[ThreadLocal-面试必问深度解析](https://www.jianshu.com/p/98b68c97df9b)

[Java并发编程：深入剖析ThreadLocal](http://www.cnblogs.com/dolphin0520/p/3920407.html)

[ThreadLocal详解](https://www.cnblogs.com/dreamroute/p/5034726.html)

[ThreadLocal](http://www.cnblogs.com/mingforyou/archive/2012/03/10/2389284.html)

[Java多线程9：ThreadLocal源码剖析](https://www.cnblogs.com/xrq730/p/4854813.html)

[Java多线程10:ThreadLocal的作用及使用](https://www.cnblogs.com/xrq730/p/4854820.html)

[ThreadLocal源码分析](https://www.jianshu.com/p/80866ca6c424)

[ThreadLocal和神奇的数字0x61c88647](https://www.cnblogs.com/ilellen/p/4135266.html)