---
layout:     post
title:      "sun.misc.Unsafe详解"
subtitle:   " \"魔法类之Unsafe\""
date:       2019-09-20 15:29:07
author:     "ming"
catalog: true
header-img: "img/post-bg-alone.jpg"
tags:
    - 并发编程
    - JAVA
---

> "If you think you can, you can. And if you think you can't, you're right."

### 1.Unsafe类了解

我们会发现在看源码的过程，有很多地方都用到了sun.misc.Unsafe这个类，为此，下面对这个类做一个详细的了解：

1. java 生态圈。 几乎每个使用 java开发的工具、软件基础设施、高性能开发库都在底层使用了 sun.misc.Unsafe 。
2. 这就是SUN未开源的sun.misc.Unsafe的类，该类功能很强大，涉及到类加载机制，其实例一般情况是获取不到的，源码中的设计是采用单例模式，不是系统加载初始化就会抛出SecurityException异常。
3. 查阅一些资料后发现，Unsafe类官方并不对外开放，因为Unsafe这个类提供了一些绕开JVM的更底层功能，基于它的实现可以提高效率。
4. 在jdk 1.9版本中对Unsafe提供了公开的API。

### 2.Unsafe的大部分方法

其主要分为以下几类：

- Info:主要返回某些低级别的内存信息。

```java
public native int addressSize();
public native int pageSize();
```

- Objects: 主要提供Object和它的域操作办法：

```java
public native Object allocateInstance(Class<?> var1) throws InstantiationException;
public native long objectFieldOffset(Field var1);
```

- Class: 主要提供Class和它的静态域操作方法：

```java
public native long staticFieldOffset(Field var1);
public native Class<?> defineClass(String var1, byte[] var2, int var3, int var4, ClassLoader var5, ProtectionDomain var6);
public native Class<?> defineAnonymousClass(Class<?> var1, byte[] var2, Object[] var3);
public native void ensureClassInitialized(Class<?> var1);
```

- Arrays: 数组操作办法

```java
public native int arrayBaseOffset(Class<?> var1);
public native int arrayIndexScale(Class<?> var1);
```

- Synchronization: 主要提供低级别同步原语

```java
/** @deprecated */
@Deprecated
public native void monitorEnter(Object var1);

/** @deprecated */
@Deprecated
public native void monitorExit(Object var1);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public native void putOrderedInt(Object var1, long var2, int var4);

```

- Memory: 直接内存访问方法(绕过JVM直接操作本地内存)

```java
public native long allocateMemory(long var1);
public native long reallocateMemory(long var1, long var3);
public native void setMemory(Object var1, long var2, long var4, byte var6);
public native void copyMemory(Object var1, long var2, Object var4, long var5, long var7);
```

### 3.Unsafe类的实例的获取

查看Unsafe的源码我们可以发现，它类设计只提供给JVM信任的启动类加载器所使用，是一个典型的单例模式类：

```java
private Unsafe() {
}

@CallerSensitive
public static Unsafe getUnsafe() {
    Class var0 = Reflection.getCallerClass();
    if(!VM.isSystemDomainLoader(var0.getClassLoader())) {
        throw new SecurityException("Unsafe");
    } else {
        return theUnsafe;
    }
}
```
虽然，它提供了一个getUnsafe()的静态方法，但是，如果直接调用这个方法会抛出一个SecurityException异常，这是因为Unsafe仅供java内部类使用，外部类不应该使用它。虽然这不可行，但是我们可以通过反射拿到它，如下所示：

```java
Field f = Unsafe.class.getDeclaredField("theUnsafe");
f.setAccessible(true);
Unsafe unsafe = (Unsafe) f.get(null);
```

### 4. Unsafe类的使用

#### 4.1 创建实例

通过allocateInstance()方法，你可以创建一个类的实例，但是却不需要调用它的构造函数、初始化代码、各种JVM安全检查以及其它的一些底层的东西。即使构造函数是私有，我们也可以通过这个方法创建它的实例。

```java
public class UnsafePlayer {
    public static void main(String[] args) throws Exception {
        //通过反射实例化Unsafe
        Field f = Unsafe.class.getDeclaredField("theUnsafe");
        f.setAccessible(true);
        Unsafe unsafe = (Unsafe) f.get(null);

        //实例化私有的构造函数
        Player player = (Player) unsafe.allocateInstance(Player.class);
        player.setName("jack");
        System.out.println(player.getName());
    }
}

public static class Player{
    private int age;
    private String name;
    private Player() {
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }

    }
}

```

#### 4.2 可以使用堆外内存

如果进程在运行过程中JVM上的内存不足了，会导致频繁的进行GC。理想情况下，我们可以考虑使用堆外内存，这是一块不受JVM管理的内存。

使用Unsafe的allocateMemory()我们可以直接在堆外分配内存，这可能非常有用，但我们要记住，这个内存不受JVM管理，因此我们要调用freeMemory()方法手动释放它。

假设我们要在堆外创建一个巨大的int数组，我们可以使用allocateMemory()方法来实现：

```java
public class OffHeapArray {

    //一个int等于4个字节
    private static final int INT = 4;
    private long size;
    private long address;

    private static Unsafe unsafe;

    static {
        try{
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            unsafe = (Unsafe) f.get(null);
        }catch (NoSuchFieldException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }
    }

    //构造方法，分配内存
    public OffHeapArray(long size){
        this.size = size;
        //参数字节数
        address = unsafe.allocateMemory(size * INT);
    }

    //获取指定索引处的元素
    public int get(long i){
        return unsafe.getInt(address + i * INT);
    }

    //设置指定索引处的元素
    public void set(long i, int value){
        unsafe.putInt(address + i * INT,value);
    }

    //元素个数
    public long size(){
        return size;
    }
    //释放堆外内存
    public void freeMemory(){
        unsafe.freeMemory(address);
    }
}
```

在构造方法中调用allocateMemory()分配内存，在使用完成后调用freeMemory()释放内存。使用方式如下：

```java
    public static void main(String[] args) {
        OffHeapArray offHeapArray = new OffHeapArray(4);
        offHeapArray.set(0,1);
        offHeapArray.set(1,2);
        offHeapArray.set(2,3);
        offHeapArray.set(3,4);
        offHeapArray.set(2,5); //在索引2的位置重新放入元素

        int sum = 0;
        for (int i = 0; i < offHeapArray.size(); i++){
            sum += offHeapArray.get(i);
        }
        //打印12
        System.out.println(sum);
        offHeapArray.freeMemory();
    }
```
**注意**：最后，一定要记得调用freeMemory()将内存释放回操作系统。

#### 4.3 通过内存偏移地址修改变量值

**public native long objectFieldOffset(Field field);**

返回指定静态field的内存地址偏移量，在这个类的其他方法中这个值只是被用作一个访问特定field的一个方式。这个值对于给定的field是唯一的，并且后续对该方法的调用都应该返回相同的值。

**public native int arrayBaseOffset(Class arrayClass);**

获取给定数组中第一个元素的偏移地址。为了存取数组中的元素，这个偏移地址与arrayIndexScale方法的非0返回值一起被使用。
public native int arrayIndexScale(Class arrayClass)
获取用户给定数组寻址的换算因子。如果不能返回一个合适的换算因子的时候就会返回0。这个返回值能够与arrayBaseOffset一起使用去存取这个数组class中的元素

**public native boolean compareAndSwapInt(Object obj, long offset,int expect, int update);**

在obj的offset位置比较integer field和期望的值，如果相同则更新。这个方法的操作应该是原子的，因此提供了一种不可中断的方式更新integer field。当然还有与Object、Long对应的compareAndSwapObject和compareAndSwapLong方法。

**public native void putOrderedInt(Object obj, long offset, int value);**

设置obj对象中offset偏移地址对应的整型field的值为指定值。这是一个有序或者有延迟的putIntVolatile方法，并且不保证值的改变被其他线程立即看到。只有在field被volatile修饰并且期望被意外修改的时候使用才有用。当然还有与Object、Long对应的putOrderedObject和putOrderedLong方法。

**public native void putObjectVolatile(Object obj, long offset, Object value);**

设置obj对象中offset偏移地址对应的object型field的值为指定值。支持volatile store语义。
与这个方法对应的get方法为：

**public native Object getObjectVolatile(Object obj, long offset);**

获取obj对象中offset偏移地址对应的object型field的值,支持volatile load语义
这两个方法还有与Int、Boolean、Byte、Short、Char、Long、Float、Double等类型对应的相关方法.

**public native void putObject(Object obj, long offset, Object value);**

设置obj对象中offset偏移地址对应的object型field的值为指定值。与putObject方法对应的是getObject方法。Int、Boolean、Byte、Short、Char、Long、Float、Double等类型都有getXXX和putXXX形式的方法。

下面通过一个组合示例来了解一下如何使用它们，详细如下：

```java
public class UnsafePlayerCAS {
    public static void main(String[] args) throws Exception{
        //通过反射实例化Unsafe
        Field f = Unsafe.class.getDeclaredField("theUnsafe");
        f.setAccessible(true);
        Unsafe unsafe = (Unsafe) f.get(null);

        // 实例化Player
        Player player = (Player) unsafe.allocateInstance(Player.class);
        player.setAge(18);
        player.setName("li lei");
        for (Field field : Player.class.getDeclaredFields()) {
            System.out.println(field.getName() + ":对应的内存偏移地址" + unsafe.objectFieldOffset(field));
        }

        System.out.println("-------------------");
        // unsafe.compareAndSwapInt(arg0, arg1, arg2, arg3)
        // arg0, arg1, arg2, arg3 分别是目标对象实例，目标对象属性偏移量，当前预期值，要设的值

        int ageOffset = 12;
        // 修改内存偏移地址为12的值（age）,返回true,说明通过内存偏移地址修改age的值成功
        System.out.println(unsafe.compareAndSwapInt(player, ageOffset, 18, 20));
        System.out.println("age修改后的值：" + player.getAge());
        System.out.println("-------------------");

        // 修改内存偏移地址为12的值，但是修改后不保证立马能被其他的线程看到。
        unsafe.putOrderedInt(player, 12, 33);
        System.out.println("age修改后的值：" + player.getAge());
        System.out.println("-------------------");

        // 修改内存偏移地址为16的值，volatile修饰，修改能立马对其他线程可见
        unsafe.putObjectVolatile(player, 16, "han mei");
        System.out.println("name修改后的值：" + unsafe.getObjectVolatile(player, 16));
    }

    public static class Player{
        @Getter @Setter private String name;
        @Getter @Setter private int age;
        private Player(){}
    }
}
```

输出结果：

```
age:对应的内存偏移地址12
name:对应的内存偏移地址16
-------------------
true
age修改后的值：20
-------------------
age修改后的值：33
-------------------
name修改后的值：han mei
```

#### 4.4 park/unpark

JVM在上下文切换的时候使用了Unsafe中的两个非常牛逼的方法park()和unpark()。
- 当一个线程正在等待某个操作时，JVM调用Unsafe的park()方法来阻塞此线程。
- 当阻塞中的线程需要再次运行时，JVM调用Unsafe的unpark()方法来唤醒此线程。

#### 4.5 CompareAndSwap操作

JUC下面大量使用了CAS操作，它们的底层是调用的Unsafe的CompareAndSwapXXX()方法。这种方式广泛运用于无锁算法，与java中标准的悲观锁机制相比，它可以利用CAS处理器指令提供极大的加速。

比如，我们可以基于Unsafe的compareAndSwapInt()方法构建线程安全的计数器。

```java
public class Counter {

    private volatile int count = 0;
    private static long offset;
    private static Unsafe unsafe;

    static {
        try{
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            unsafe = (Unsafe) f.get(null);
            offset = unsafe.objectFieldOffset(Counter.class.getDeclaredField("count"));
        }catch (NoSuchFieldException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }
    }

    public void increment(){
        int before = count;
        while (!unsafe.compareAndSwapInt(this,offset,before,before+1)){
            before = count;
        }
    }

    public int getCount(){
        return count;
    }
}
```
我们定义了一个volatile的字段count，以便对它的修改所有线程都可见，并在类加载的时候获取count在类中的偏移地址。

在increment()方法中，我们通过调用Unsafe的compareAndSwapInt()方法来尝试更新之前获取到的count的值，如果它没有被其它线程更新过，则更新成功，否则不断重试直到成功为止。

我们可以通过使用多个线程来测试我们的代码：

```java
   public static void main(String[] args)throws Exception {
        Counter counter = new Counter();
        ExecutorService threadPool = Executors.newFixedThreadPool(100);

        //起100个线程，每个线程自增10000次
        IntStream.range(0,100)
                .forEach(i ->threadPool.submit(()-> IntStream.range(0,10000).forEach(j->counter.increment())));
        threadPool.shutdown();
        Thread.sleep(2000);
        // 打印1000000
        System.out.println(counter.getCount());
    }
```

#### 4.6 抛出checked异常

我们知道如果代码抛出了checked异常，要不就使用try...catch捕获它，要不就在方法签名上定义这个异常，但是，通过Unsafe我们可以抛出一个checked异常，同时却不用捕获或在方法签名上定义它。

```java
// 使用正常方式抛出IOException需要定义在方法签名上往外抛
public static void readFile() throws IOException {
    throw new IOException();
}
// 使用Unsafe抛出异常不需要定义在方法签名上往外抛
public static void readFileUnsafe() {
    unsafe.throwException(new IOException());
}
```

### 5.总结

使用Unsafe几乎可以操作一切：
- 实例化一个类
- 修改私有字段的值
- 抛出checked异常
- 使用堆外内存
- CAS操作；
- 阻塞/唤醒线程；

**补充：实例化一个类的方式？**

1. 通过构造方法实例化与一个类
2. 通过Class实例化一个类
3. 通过反射实例化一个类(获取Constructor对象)
4. 通过克隆实例化与一个类
5. 通过反序列化实例化一个类
6. 通过Unsafe实例化一个类

```java
public class InstantialTest {

    private static Unsafe unsafe;

    static {
        try{
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            unsafe = (Unsafe) f.get(null);
        }catch (NoSuchFieldException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args)throws Exception {
        //1. 构造方法
        User user1 = new User();
        //Class,里面也是反射
        User user2 = User.class.newInstance();
        //3. 反射
        User user3 = User.class.getConstructor().newInstance();
        //4. 克隆
        User user4 = (User)user1.clone();
        //5. 反序列化
        User user5 = unserialize(user1);
        //6. unsafe
        User user6 = (User) unsafe.allocateInstance(User.class);

        System.out.println(user1.age);
        System.out.println(user2.age);
        System.out.println(user3.age);
        System.out.println(user4.age);
        System.out.println(user5.age);
        System.out.println(user6.age);
    }

    private static User unserialize(User user1)throws Exception{
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("D://object.txt"));
        oos.writeObject(user1);
        oos.close();

        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("D://object.txt"));
        User user5 = (User)ois.readObject();
        ois.close();
        return user5;
    }

    static class User implements Cloneable, Serializable{
        private int age;

        public User(){
            this.age = 10;
        }

        @Override
        protected  Object clone()throws CloneNotSupportedException{
            return super.clone();
        }
    }
}
```
输出结果如下：

```
10
10
10
10
10
0
```

通过Unsafe实例化的类，里面的age的值竟然是0，而不是10或者20。这是因为调用Unsafe的allocateInstance()方法只会给对象分配内存，并不会初始化对象中的属性，所以int类型的默认值就是0。
