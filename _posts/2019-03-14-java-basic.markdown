---
layout:      post
title:       "Java Basic Knowledge 1"
subtitle:    " \"Java core technology\""
date:        2019-03-14 16:18:37
author:      "Ming"
catalog: true
header-img:  "img/post-bg-alitrip.jpg"
tags:
    - JAVA
    - Knowledge
---

> "The most difficult stage in life is not when no one understands you, but when you don't understand yourself."

### 1.Java基本程序设计结构

#### 1.1 带标签的break语句

Java提供了一种带标签的break语句，用于跳出多重嵌套的循环语句。下面给出一个示例说明break语句的工作状态。(eg: 标签必须放在希望跳出的最外层循环之前，并且必须紧跟一个冒号)

```java
Scanner in = new Scanner(System.in);
int n;
read_data:
while(...){
    ...
    for(...){
        n = in.nextInt();
        if(n < 0){
            break read_data;
        }
        ...
    }
}
```
> 事实上,可以将标签应用到任何语句中，甚至可以应用到if语句或者块语句中。

#### 1.2 返回可变对象的访问器方法

当如果需要返回一个可变对象的引用，应该首先对它进行克隆(clone)。对象clone是指存放在另一个位置上的对象副本。

```java
class Employee{
    ...
    public Date getHireDay(){
        return (Date)hireDay.clone();
    }
    ...
}
``` 

#### 1.3 System.out的特殊之处

首先，System.out是一个静态常量，为此它不允许再将其他打印流赋给它。然而，如果查看一下System类，就会发现有一个setOut方法，它可以将System.out设置为不同的流。

```java

//System.java
public static void setOut(PrintStream out) {
        checkIO();
        setOut0(out);
    }


   private static void checkIO() {
        SecurityManager sm = getSecurityManager();
        if (sm != null) {
            sm.checkPermission(new RuntimePermission("setIO"));
        }
    }

    private static native void setIn0(InputStream in);
    private static native void setOut0(PrintStream out);
    private static native void setErr0(PrintStream err);
```

一般来说，final变量的值是不可以改变的。但是，这里的原因在于,setOut方法是一个native方法，而不是用java语言实现的。本地方法可以绕过java语言的存储控制机制，进行修改，这是一种特殊的方法。

#### 1.4 编写一个完美的equals方法的建议

- 显示参数命名为 _otherObject_，稍后需要将它转换成另一个叫做 _other_的变量
- 检测 _this_ 与 _otherObject_ 是否引用同一个对象:
```java
if(this == otherObject) return true;
```
这条语句只是一个优化。实际上，这是一种经常采用的形式。因为计算这个等式要比一个个地比较类中的域所付出的代价小的多。
- 检测 _otherObject_ 是否为null,如果为null,返回false。这项检测很有必要。
```java
if (otherObject == null) return false
```
- 比较 _this_ 与 _otherObject_ 是否属于同一个类。如果equals的语义在每个子类中有所改变，就使用getClass检测：
```java
if(getClass() != otherObject.getClass()) return false;
```
如果所有的子类都拥有统一的语义，就使用instanceof检测：
```java
if(!otherObject instanceof ClassName) return false;
```
- 将 _otherObject_ 转换为相应的类类型变量:
```java
ClassName other = (ClassName) otherObject;
```
- 现在开始对所有需要比较的域进行比较了。使用 == 比较基本类型域，使用equals比较对象与。如果所有的域都匹配，就返回true;否则返回false。
```java
return field1 == other.field1
     && Objects.equals(field2, other.field2)
     && ...;
```
如果在子类中重新定义equals，就需要在其中包含调用super.equals(other).

#### 1.5 Java protected关键字详解

大部分书籍上都对protected介绍的比较简单，基本上都是一句话，就是：**被protected修饰的成员对于本包及其子类可见。**这种说法有点过于含糊，常常对大家造成误解。实际上，protected的可见性在于两点：

- 基类的protected成员是包内可见的，并且对子类可见；
- 若子类与基类不在同一包中，那么在子类中，子类实例可以访问其从基类继承而来的protected方法，而不能访问基类实例的protected方法。

在碰到涉及protected成员的调用时，首先要确定该protected成员来自何方，其可见性范围是什么，然后就可以判断出当前用法是否可行。

##### 示例一

```java
package p1;
public class Father1 {
    protected void f() {}    // 父类Father1中的protected方法
}

package p1;
public class Son1 extends Father1 {}

package p11;
public class Son11 extends Father1{}

package p1;
public class Test1 {
    public static void main(String[] args) {
        Son1 son1 = new Son1();
        son1.f(); // Compile OK     ----（1）
        son1.clone(); // Compile Error     ----（2）
 
        Son11 son = new Son11();    
        son11.f(); // Compile OK     ----（3）
        son11.clone(); // Compile Error     ----（4）
    }
}
```
**分析**：首先看(1)(3),其中的f()方法是从类Father1继承而来，其可见性是包p1及其子类Son1和Son11，而由于调用f()方法的类Test1所在的也是p1，因此(1)(3)处编译通过。**也就是说，如果我们换一个包，比如Test1.java在p11下，那么将都不可访问。**如下所示：

```java
package p11;
public class Test1 {
    public static void main(String[] args) {
        Son1 son1 = new Son1();
        son1.f(); // Compile Error     ----（1）
        son1.clone(); // Compile Error     ----（2）
 
        Son11 son = new Son11();    
        son11.f(); // Compile Error     ----（3）
        son11.clone(); // Compile Error     ----（4）
    }
}
```
其次，看一下(2)(4),其中的clone()方法的可见性是java.lang包及其所有子类，对于语句_son1.clone()_和_son11.clone()_，两者的clone()在类Son1,Son11中是可见的，但是对于Test1是不可见的，因此(2)(4)处编译不通过。**也就是说，如果在Son1或Son11这两个类中调用clone()方法，则是可以编译通过的。**

```java
package p1;
public class Son1 extends Father1 {

    public Son1() throws CloneNotSupportException(){
        clone();   //Compile OK
    }
}
```

##### 示例二

```java
package p2;
class MyObject2 {
    protected Object clone() throws CloneNotSupportedException{
       return super.clone();
    }
}
 
package p22;
public class Test2 extends MyObject2 {
    public static void main(String args[]) {
       MyObject2 obj = new MyObject2();
       obj.clone(); // Compile Error         ----（1）
 
       Test2 tobj = new Test2();
       tobj.clone(); // Complie OK         ----（2）
    }
}
```
**分析**：对于(1)而言，clone()方法来自于类MyObject2本身，因此其可见性为包p2及MyObject2的子类，虽然Test2是MyObject2的子类，**但是在Test2中不能访问父类MyObject2的protected方法clone()**,因此编译不通过；对于(2)而言，由于在Test2中访问的是其本身实例的从父类MyObject2继承而来的clone()，因此编译通过。此处，就很好的诠释了上面所给出的第二条结论。

> 若子类与父类**不在同一包中**，那么**在子类中**，子类实例可以访问从父类继承而来的protected方法，而不能访问父类实例的protected方法。

##### 示例三
```java
package p3;
class MyObject3 extends Test3 {
}
 
package p33;
public class Test3 {
  public static void main(String args[]) {
    MyObject3 obj = new MyObject3();
    obj.clone();   // Compile OK     ------（1）
  }
}
```
**分析**：对于(1)而言，clone()方法来自于Test3,因此其可见性为包p33及其子类MyObject3，而(1)正是在p33的类Test3中调用，属于同一包，编译通过。

##### 示例四
```java
package p4;
class MyObject4 extends Test4 {
  protected Object clone() throws CloneNotSupportedException {
    return super.clone();
  }
}
 
package p44;
public class Test4 {
  public static void main(String args[]) {
    MyObject4 obj = new MyObject4();
    obj.clone(); // Compile Error      -----（1）
  }
}
```
**分析**：对于(1)而言，clone()方法来自于类MyObject4，因此其可见性为包p4及其子类(此处没有子类)，而类Test4却在包p44中，因此不满足可见性，编译不通过。

##### 示例五
```java
package p5;
 
class MyObject5 {
    protected Object clone() throws CloneNotSupportedException{
       return super.clone();
    }
}
public class Test5 {
    public static void main(String[] args) throws CloneNotSupportedException {
       MyObject5 obj = new MyObject5();
       obj.clone(); // Compile OK        ----(1)
    }
}
```
**分析**：对于(1)而言，clone()方法来自于类MyObject5，因此其可见性为包p5及其子类(此处没有子类)，而类Test5也在包p5中，因此满足可见性，编译通过。

##### 示例六
```java
package p6;
 
class MyObject6 extends Test6{}
public class Test6 {
  public static void main(String[] args) {
    MyObject6 obj = new MyObject6();
    obj.clone();        // Compile OK   -------（1）
  }
}
```
**分析**：对于(1)而言，clone()方法来自于类Test6，因此其可见性为包p6及其子类MyObject6，而类Test6也在包p6中，因此满足可见性，编译通过。

##### 示例七
```java
package p7;
 
class MyObject7 extends Test7 {
    public static void main(String[] args) {
        Test7 test = new Test7();
        test.clone(); // Compile Error   ----- (1)
  }
}
 
public class Test7 {
}
```
**分析**：对于(1)而言，clone()方法来自于类Object，因此该clone()方法可见性为包java.lang及其子类Test7，由于类MyObject7不在此范围内，因此不满足可见性，编译不通过。

#### 1.6 java Comparator为何是函数式接口?

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
    boolean equals(Object obj);
    ...
}
```

函数式接口，@FunctionalInterface，简单的说，就是指仅仅含有一个抽象方法的接口，以@FunctionalInterface注解标记。**注意**，这里的抽象方法指的是该接口自己特有的抽象方法，而不包含它从其上级继承过来的抽象方法。

函数式接口可以额外定义多个抽象方法，但这些抽象方法签名必须和Object的public方法一样。接口最终有确定的类实现，而类的最终父类是Object。因此函数式接口可以定义Object的public方法。例如下面的接口依然是函数式接口：

```java
@FunctionalInterface
public interface ObjectMethodFunctionalInterface {
   void count(int i);
   String toString(); //same to Object.toString
   int hashCode(); //same to Object.hashCode
   boolean equals(Object obj); //same to Object.equals
}
```

为什么限定public类型的方法呢？因为接口中定义的方法都是public类型的。举个例子，下面的接口就不是函数式接口:
```java
interface WrongObjectMethodFunctionalInterface {
    void count(int i);
    Object clone(); //Object.clone is protected
} 
```
**解释**: 因为Object.clone()方法是protected类型。

#### 1.7 Java泛型中extends和super的区别

_<? extends T>_ 和 _<? super T>_ 是Java泛型中的"通配符(Willdcards)"和"边界(Bounds)"的概念。

- <? extends T>:是指"上界通配符(Upper Bounds Willdcards)"
- <? super T>:是指"下界通配符(Lower Bounds Willdcards)"

**为什么会要使用通配符和边界？**

使用泛型的过程中，经常会出现一种很变扭的情况。例如，我们有以下类，Fruit类和它的子类Apple.

```java
class Fruit{}
class Apple extends Fruit{}
```

然后有一个最简单的容器：Plate类。盘子里可以放一个泛型的东西。我们可以对这个东西做最简单的"放"和"取"的动作：set()和get()方法。

```java
class Plate<T>{
    private T item;
    public Plate(T t){
        item = t;
    }
    public void set(T t){item = t;}
    public T get(){ return item;}
}
```
现在我定义一个“水果盘子”，逻辑上水果盘子当然可以装苹果。

```java
Plate<Fruit> p = new Plate<Apple>(new Apple());  //编译失败
```

但实际上Java编译器不允许这个操作。会报错：( _Plate< Apple >_ 是无法转换成 _Plate< Fruit >_ ,即装"苹果的盘子"是无法转换成"装水果的盘子")

```java
error: incompatible types: Plate<Apple> cannot be converted to Plate<Fruit>
```

实际上，在编译器认定的逻辑如下：
- 苹果 IS-A 水果
- 装苹果的盘子 NOT-IS-A 装水果的盘子

**所以，就算容器里装的东西之间有继承关系，但容器之间是没有继承关系的**。为此，为了解决这样一个问题，_<? extends T>_ 和 _<? super T>_ 就被提出来用于让“水果盘子”和“苹果盘子”之间发生关系。

**什么是上界？**

下面的代码就是上界通配符：

```java
Plate<? extends Fruit>
```

翻译成人话就是：一个能放水果以及一切是水果子类的盘子。 _Plate<? extends Fruit>_ 和  _Plate< Apple >_ 最大的区别就是： _Plate<? extends Fruit>_ 是 _Plate< Fruit >_ 以及 _Plate< Apple >_ 的基类。直接的好处，就是可以用苹果盘子给水果盘子赋值了。

```java
Plate<? extends Fruit> p = new Plate<Apple>(new Apple()); //编译成功
```

如果把Fruit和Apple的例子扩展一下，食物分为水果和肉类，水果有香蕉和苹果，肉类有猪肉和牛肉，苹果还有两种青苹果和红苹果。

```java
//Lev 1
class Food{}

//Lev 2
class Fruit extends Food{}
class Meat extends Food{}

//Lev 3
class Apple extends Fruit{}
class Banana extends Fruit{}
class Pork extends Meat{}
class Beef extends Meat{}

//Lev 4
class RedApple extends Apple{}
class GreenApple extends Apple{}
```

在这个体系中，上界通配符_Plate<? extends Fruit>_覆盖下图中蓝色的区域：

![上界通配符](https://ws1.sinaimg.cn/large/005CDUpdly1g1hbp58jbyj317s0l8dlz.jpg)

**什么是下界？**

相对应的，下界通配符如下：

```java
Plate<? super Fruit>
```

表达的就是相反的概念：一个能放水果以及一切是水果基类的盘子。 _Plate<? super Fruit>_ 是 _Plate< Fruit >_ 的基类，但不是 _Plate< Apple >_ 的基类。对应刚才那个例子， _Plate<? super Fruit>_ 覆盖下图中红色的区域。

![下界通配符](https://ws1.sinaimg.cn/large/005CDUpdly1g1hbus470gj317o0mg43g.jpg)

**上下界通配符的副作用**

> 边界让Java不同泛型之间的转换更容易了。但是，这样的转换也存在着一定的副作用。那就是容器的部分功能可能失效。

以刚才的Plate为例。我们可以对盘子做两件事，向盘子里set()新东西，以及从盘子里get()东西。

```java
class Plate<T>{
    private T item;
    public Plate(T t){item=t;}
    public void set(T t){item=t;}
    public T get(){return item;}
}
```

**上界 _<? extends T>_ 不能往里面存，只能往外取**

_<? extends T>_ 会使往盘子里放东西的set()方法失效。但取东西get()方法还有效。例如下面例子里两个set()方法，插入Apple和Fruit都报错。

```java
Plate<? extends Fruit> p = new Plate<Apple>(new Apple());

//不能存入任何元素
p.set(new Fruit());  //Error
p.set(new Apple());  //Error

//读取出来的东西只能存放在Fruit或它的基类里。
Fruit newFruit1 = p.get();
Object newFruit2 = p.get();
Apple newFruit3 = p.get();  //Error
```

原因是编译器只知道容器内是Fruit或者它的派生类，但具体是什么类型不知道。可能是Fruit,可能是Apple，也可能是Banana。编译器在看到后面用Plate赋值之后，盘子里没有被标上有“苹果”。而是标记上一个占位符： CAP#1, 来表示捕获一个Fruit或Fruit的子类，具体是什么类不知道，代号CAP#1。然后无论是是想往里插入Apple或者Meat或者Fruit编译器都不知道能不能和这个CAP#1匹配，所以就都不允许。

所以通配符<?>和类型参数的区别就在于，对编译器来说所有的T都代表同一种类型。比如下面这个泛型方法里，三个T都指代同一个类型，要么都是String，要么都是Integer。

```java
public <T> List<T> fill(T... t);
```
但通配符<?>没有这种约束，Plate<?>单纯的就表示：盘子里放了一个东西，是什么我不知道。

**下界 _<? super T>_ 不影响往里面存，但往外取只能放在Object对象里**

使用下界 _<? super T>_ 会使从盘子里取东西的get()方法部分失效，只能存放到Object对象里。set()方法正常。

```java
Plate<? super Fruit> p=new Plate<Fruit>(new Fruit());

//存入元素正常
p.set(new Fruit());
p.set(new Apple());

//读取出来的东西只能存放在Object类里。
Apple newFruit3=p.get();    //Error
Fruit newFruit1=p.get();    //Error
Object newFruit2=p.get();
```

因为下界规定了元素的最小粒度的下限，实际上放松了容器元素的类型控制。既然元素是Fruit的基类，那往里存粒度比Fruit小的都可以。但往外读取元素就费劲了，只有所有类的基类Object对象才能装下。但这样的话，元素的类型信息就全部丢失。

**PESC原则**

PECS（Producer Extends Consumer Super）原则：
- 频繁往外读取内容的，适合用上界Extends。
- 经常往里插入的，适合用下界Super。

#### 1.8 Java基础之String,StringBuffer与StringBuilder


##### 区别
- String对象是常量，它的值创建后不能被改变，StringBuilder和StringBuffer可以被改变
- StringBuilder非线程安全(单线程使用),String和StringBuffer线程安全(多线程使用)
- 如果程序不是多线程的，那么使用StringBuilder效率高于StringBuffer

##### String对象

**1. String类的基本认知**

String类的包含如下定义：

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -6849794470754667710L;
}
```

String及几个基本知识点如下：
- String类是引用类型，它是值不可变的常量，是线程安全的；
- String类重写了Object类的equals()和hashCode()，用于比较内容是否相等，而非引用地址;
- “==”运算符，对基本数据类型比较的是字面值，对引用类型比较的则是引用地址。
- Sring类使用了final修饰符，String类是不可继承的。

String设计成不可变类的好处：
- 线程安全。不可变类表明类的属性初始化后不可改变，即对象的不可变；那么当多线程来访问此对象时不用担心线程安全问题，所以String类的对象是线程安全的；
- 符合字符串常量池的设计。当我们声明一个字符串对象时，编译器首先会去字符串常量池中查找是否存在，若存在则把字符串引用直接指向池中的对象，避免重复声明；若不存在则生成一个对象并放入到池中，供后续使用；虽然String不是Java语言的基本数据类型，但使用非常频繁，避免重复声明可以节省一大部分的内存开销，从而减少GC的时间。
- 确保hashCode的唯一性。hashMap中是以key的hashCode为存储地址，地址当然是不变的好，不然如何查找可变的地址；不可变String对象的hashCode是唯一的，所以能很好的适用hashMap中的key。

常量池的概念：
- 常量池是一个内存空间，不同于使用new关键字创建的对象所在的堆空间。
- 常量池是为了避免频繁的创建和销毁对象而影响系统性能，其实现了对象的共享。当需要一个对象时，就可以从池中取一个出来(比如池中没有则创建一个)，则在需要重复创建相等变量时节省了时间。
- 在编译器被确定，并被保存在已编译的.class文件中的一些数据，包括类、方法、接口等中的常量和字符串常量。常量池还具备动态性，运行期间可以将新的常量放入池中。Java中基本类型的包装类大部分都实现了常量池技术，即Byte,Short,Integer,Long,Character,Boolean.

给出一个试题，下面代码创建了几个String对象？

```java
String s1 = new String("s1");
String s2 = new String("s1");
```

```java
// 3个,编译期间在常量池中创建1个，即"s1"常量对象，运行期间堆中创建2个，即s1和s2对象
```

为了更为详细的讲述我们来看一下下面的代码：

```java
public class StringTest {
 public static void main(String[] args) {
     String str = "main";
     String newStr = new String("main");

     String newStr1 = new String(str);
     String str1 = "main";
     String str2 = newStr;
     String str3 = newStr1;

     System.out.print(str == str1);             // t
     System.out.println(str.equals(str1));      // t

     System.out.print(str == newStr);           // f
     System.out.println(str.equals(newStr));    // t

     System.out.print(str == newStr1);          // f
     System.out.println(str.equals(newStr1));   // t

     System.out.print(str == str2);             // f
     System.out.println(str.equals(str2));      // t

     System.out.print(newStr == newStr1);       // f
     System.out.println(newStr.equals(newStr1));// t

     System.out.print(newStr == str2);          // t
     System.out.println(newStr.equals(str2));   // t

     System.out.print(newStr == str3);          // f
     System.out.println(newStr.equals(str3));   // t
 }
}
```

**第一种情况**：str和str1的比较

JVM加载StringTest类并执行静态的main方法，str变量的声明方法使得在方法区的运行时常量池生成一个“main”值(jdk1.6版本),str引用指向该值的地址；str1变量在创建的过程中，首先会去运行时常量池检验是否已经有相同的变量，如果有则直接指向该值的地址，否则新建；因此str和str1变量都指向运行时常量池中的同一个地址，所以“==“运算符和equals()方法的运行结果都是true.

**第二种情况**：str和newStr的比较

str指向的是方法区运行时常量池中的内容，而newStr对象声明的方式并不会去常量池检测，而直接在堆上生成一个新的对象，因此str和newStr引用指向的地址不相等，但地址内存储的内容相等，所以“==”运算符返回false，equals()方法返回true。

**第三种情况**：str和newStr1的比较

newStr1引用变量的声明方式与newStr类似，只不过通过str变量给newStr1引用的内容赋值，newStr1引用指向的对象还是在堆上，因此str和newStr1引用指向的地址不相等，但地址内存储的内容相等，所以“==”运算符返回false，equals()方法返回true。

**第四种情况**：str和str2的比较

str引用指向方法区运行时常量池，而str2引用指向引用newStr指向的堆上的对象，因此str和str2指向的地址不同，但是常量池和对象的内容一样都是“main”，所以“==”运算符返回false，equals()方法返回true。

**第五种情况**: newStr和newStr1

这种情况最明了，newStr和newStr1是两个完全不同的引用，分别指向堆上不同的地址，但堆上内存存储的内容都是“main”，所以“==”运算符返回false，equals()方法返回true。

**第六种情况**：newStr和str2的比较

代码中把引用newStr赋值给str2，表明引用str2指向引用newStr指向的内存地址，所以“==”运算符和equals()方法的运行结果都是true。

**第七种情况**：newStr和str3的比较

引用str3实际指向引用newStr1的内存地址，str3与newStr的比较等价于newStr1与newStr之间的比较；所以“==”运算符返回false，equals()方法返回true。


**2. String类的常规操作**

```java
public class StringTest {
    public static void main(String[] args) {
        String str = "hello", str1 = "world";

        String str3 = str + str1;
        String str4 = str + "world";
        String str5 = "hello" + "world";

        System.out.println(str3 == str4);   // f
        System.out.println(str3 == str5);   // f
        System.out.println(str4 == str5);   // f
    }
}
```

这里通过javap命令查看StringTest类的字节码：

![ZgYMdS.jpg](https://s2.ax1x.com/2019/07/10/ZgYMdS.jpg)

上面字节码主要看0-50行，后面都是打印比较的字节码。第0/2行存储字符串hello，第3/5行存储字符串world；第6行new一个StringBuilder的对象，第10行到21行都是对该StringBuilder对象进行操作，包括< init >和append操作，最后调用toString()方法返回字符串；那么这个过程实际上是**str3变量生成的过程**。

第25行又重新new一个StringBuilder对象，29行和30行表示从常量池获取str引用的值，33行基于获取的String初始化StringBuilder对象，36行加载常量“word”到操作数栈，注意因为常量池已经有”world”，所以此处不会重新声明；38行调用StringBuilder的append方法，连接str引用和“world”,41行调用toString()方法生成信息的String对象。第25行到44行实际上是**变量str4生成的过程.**

第46行直接把常量值helloworld加载到操作数栈并打印，没有生成任何StringBuilder对象；这就是**变量str5生成的过程。**

从上面字节码可以分析出：
- 对String类型的引用进行拼接操作，实际都会通过StringBuilder对象来实现，最后通过toString()方法返回一个新的对象；
- 直接对字符串进行拼接操作(而非引用)，与直接声明一样，过程中不会生成新的对象。

##### StringBuilder对象

StringBuilder类提供append()方法来改变自身的值，方法返回的是对象本身而非新的StringBuilder对象。因此，StringBuilder类完美的解决String类不可变的问题。下面看两段代码比较下String类和StringBuilder类带来的差异：

```java
public class StringTest {
    public static void main(String[] args) {
        method();
        method1();
    }

    public static void method() {
        String str = "";
        for (int i = 0; i < 1000; i++) {
            str += "+";
        }
    }

    public static void method1() {
        StringBuilder sb = new StringBuilder("");
        for (int i = 0; i < 1000; i++) {
            sb.append("+");
        }
    }
}
```
直接查看字节码：(method方法的字节码)
![ZgYOw8.jpg](https://s2.ax1x.com/2019/07/10/ZgYOw8.jpg)

第5行到31行是循环体，第8行表明生成一个StringBuilder类型的对象，意味着循环1000次要生成1000个StringBuilder对象；把循环体 str += “+” 操作解读成以下几个步骤：

```java
StringBuilder stringBuilder = new StringBuilder(str);  // str是每次从常量池获取的新值
stringBuilder.append("+");
stringBuidler.toString();
```

在循环体内会不断的生成StringBuilder和String类型的对象，从而造成不必要的空间浪费。

method1()方法的字节码：

![Zgt6hQ.jpg](https://s2.ax1x.com/2019/07/10/Zgt6hQ.jpg)

第12行到25行是循环体，在一开始就会new一个StringBuilder对象，循环体内只会执行对该StringBuilder对象的append()方法而不会生成额外的对象，所以StringBuilder类的字符串拼接占用的内存更小。

既然String类对象不可变的问题已经通过StringBuilder类解决了，还需要StringBuffer类干嘛。
既然StringBuilder类对象可变，那么当其声明成全局变量，必然会带来线程安全问题（一个类是否线程安全取决于类的全局变量状态是否可以改变，能改变则说明该类线程不安全，否则线程安全。来自《Java并发编程的艺术》）。

为了解决StringBuilder类线程不安全的问题，StringBuffer类就出来了。除了这一点外，StringBuffer类与StringBuilder类完全一样。StringBuffer类线程安全的实现方式是用synchronized关键字修饰方法，即同步方法的方式。

##### String\StringBuilder\StringBuffer类的性能

按照从快到满的顺序：
StringBuilder > StringBuffer > String

当然，这是一般情况下的顺序，也有特殊的场景，如：
```java
String string = "hello" + "world";
```
就会优于
```java
StringBuilder sBuilder = new StringBuilder();
sBuilder.append("hellow");
sBuilder.append("world");
```
**需要考虑线程安全，优先考虑StringBuffer；需要考虑到字符串的拼接操作，优先考虑StringBuilder；而对于常量优先考虑String。**

##### String使用中的陷阱

1. final修饰的String类型变量
```java
public class StringTest {
    public static void main(String[] args) {

        String str = "hello";
        final String str1 = "hello";

        String str2 = "helloworld";
        String str3 = str1 + "world";
        String str4 = str + "world";

        System.out.println(str == str1);   // t
        System.out.println(str2 == str3);   // t
        System.out.println(str2 == str4);   // f
    }
}
```
想必很多人对三个运行结果都很吃惊，啥也不说先看字节码。
![ZgNkjI.png](https://s2.ax1x.com/2019/07/10/ZgNkjI.png)

第9行的字节码是str3引用的生成过程，可见在编译阶段str1引用的值会参与拼接生成str3引用；其实，对于JVM来说，被final修饰的str1引用不会被改变，即生命周期内始终指向保存“hello”内容的内存地址，为了**避免执行过程中再耗费时间去常量池中取值，就会被编译器提前优化**。

2. 字符串的复合运算

```java
public class StringTest {
    public static void main(String[] args) {
        String str = "hello";

        str += " world " + "!";         // a
        str = str + " world " + "!";   // b
    }
}
```
![字符串的复合运算](https://ws1.sinaimg.cn/large/005CDUpdgy1g5595h9cvoj30t80bowg9.jpg)

因为涉及到字符串拼接，所以运算a和运算b都会生成一个StringBuilder对象，但运算a只会调用一次append()方法，直接把字符串“ world !”与引用str拼接，而运算b需要调用两次append()方法，分别把str引用先后与字符串“ world ”和“!”拼接；表明复合运算“+=”使得编译器在编译阶段会优化字符串“ world ”和“!”的拼接。所以运算a的效率会高于运算b。

参考：

[String/StringBuilder/StringBuffer 之间的区别](https://mp.weixin.qq.com/s/SKqNBSZ0kuLjU3oL2vQjcA)

[Java基础之String、StringBuffer与StringBuilder](https://blog.csdn.net/chenliguan/article/details/51911906)

[浅谈Java字符串（String, StringBuffer, StringBuilder）](https://segmentfault.com/a/1190000002683782)

[Java中的字符串常量池](https://droidyue.com/blog/2014/12/21/string-literal-pool-in-java/)

#### 1.9 Java自动装箱与拆箱

##### 定义
Java中基础数据类型与它们的包装类进行运算时，编译器会自动帮我们进行转换，转换过程对于我们来说是透明的。自动装箱就是Java自动将原始类型值转换成对应的对象，比如将int的变量转换成Integer对象，这个过程叫做装箱，反之将Integer对象转换成int类型值，这个过程叫做拆箱。因为这里的装箱和拆箱是自动进行的非人为转换，所以就称作为自动装箱和拆箱。

Java中基本类型与它们对应的包装类如下表所示：

原始类型 | 包装类型
--- | ---
boolean | Boolean
byte | Byte
char | Character
float | Float
int | Integer
long | Long
short | Short
double | Double

当表格中左边的基础类型与它们的包装类有如下几种情况时，编译器会自动帮我们进行装箱或拆箱。
- 进行=赋值操作(装箱或拆箱)
- 进行+，-，*，/混合运算时(拆箱)
- 进行>,<,==比较运算(拆箱)
- 调用equals进行比较(装箱)
- ArrayList,HashMap等集合类添加基础类型数据时(装箱)

##### 自动装箱与拆箱使如何实现的

来看这样一段代码：
```java
public void testAutoBox() {
    List<Float> list = new ArrayList<>();
    list.add(1.0f);
    float firstElement = list.get(0);
}
```
list集合存储的是Float包装类型，我传入的是float基础类型，所以需要进行装箱，而最后的get方法返回的是Float包装类型，我们赋值给float基础类型，所以需要进行拆箱.

那么，编译器到底做了些什么事情呢，直接上编译器编译之后的字节码：
```java
 public testAutoBox()V
   L0
    LINENUMBER 15 L0
    NEW java/util/ArrayList
    DUP
    INVOKESPECIAL java/util/ArrayList.<init> ()V
    ASTORE 1
   L1
    LINENUMBER 16 L1
    ALOAD 1
    FCONST_1
    INVOKESTATIC java/lang/Float.valueOf (F)Ljava/lang/Float;
    INVOKEINTERFACE java/util/List.add (Ljava/lang/Object;)Z
    POP
   L2
    LINENUMBER 17 L2
    ALOAD 1
    ICONST_0
    INVOKEINTERFACE java/util/List.get (I)Ljava/lang/Object;
    CHECKCAST java/lang/Float
    INVOKEVIRTUAL java/lang/Float.floatValue ()F
    FSTORE 2
   L3
    LINENUMBER 18 L3
    RETURN
```
- L0，对应我们代码的第一行，new了一个ArrayList，并赋值给了1号引用（就是list）。
- L1，先加载list到栈顶，然后FCONST_1指令就是从常量池加载1.0f浮点数并压入栈顶（这一块知识，见附录1），然后调用了Float类的静态 valueOf方法，进行装箱 ，然后调用list的add方法。
- L2，先加载list到栈顶，从常量池获取0（float，int，long，double等基础类型初始值都是0），调用list的get方法，检查是否能转换，调用了Float的floatValue方法，进行拆箱，存储得到的浮点数。

结论：很明显，以float和Float为例，装箱就是调用Float的valueOf方法new一个Float并赋值，拆箱就是调用Float对象的floatValue方法并赋值返回给float。

##### 自动装箱与拆箱中的坑

如下面的代码所示：

```java
public void testAutoBox2() {
     //1
     int a = 100;
     Integer b = 100;
     System.out.println(a == b); //true
     
     //2
     Integer c = 100;
     Integer d = 100;
     System.out.println(c == d); //true
     
     //3   
     c = 200;
     d = 200;

     System.out.println(c == d); //false
}
```

分析：
- 第1段代码，基础类型a与包装类b进行比较，这时b会拆箱，直接比较值，所以打印true;
- 第二段代码，二个包装类型，都被赋值了100，所以根据我们之前的解析，这时会进行装箱，调用Integer的valueOf方法，生成2个Integer对象，引用类型==比较，直接比较对象指针，这里我们先给出结论，最后会分析原因，打印 true
- 跟上面第2段代码类似，只不过赋值变成了200，直接说结论，打印 false。

第二种情况的诡异之处在于Java中会对-128到127的Integer对象进行缓存，当创建新的Integer对象时，如果符合这个范围，并且已经存在相同值得对象，则返回这个对象，否则创建新的Integer对象。

直接上源码来解释，看一下Integer类的valueOf方法的实现(JDK8实现)：

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```
可以看到，这里的实现并不是简单的new Integer,而是用IntegerCache做一个cache，cache的range是可以配置的。

```java
private static class IntegerCache {
  static final int low = -128;
  static final int high;
  static final Integer cache[];

  static {
  int h = 127;
  String integerCacheHighPropValue =
          sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
  if (integerCacheHighPropValue != null) {
     try {
         int i = parseInt(integerCacheHighPropValue);
         i = Math.max(i, 127);
         // Maximum array size is Integer.MAX_VALUE
         h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
        } catch( NumberFormatException nfe) {
         // If the property cannot be parsed into an int, ignore it.
       }
     }
     high = h;

     cache = new Integer[(high - low) + 1];
     int j = low;
     for(int k = 0; k < cache.length; k++)
        cache[k] = new Integer(j++);
  ....
```
这是IntegerCache静态代码块中的一段，默认Integer cache 的下限是-128，上限默认127，可以配置，所以到这里就清楚了，我们上面当赋值100给Integer时，刚好在这个range内，所以从cache中取对应的Integer并返回，所以二次返回的是同一个对象，所以==比较是相等的，当赋值200给Integer时，不在cache 的范围内，所以会new Integer并返回，当然==比较的结果是不相等的。

**附录1**：JVM字节码整型的入栈指令有4个，分别是：
- iconst（0~5分别对应iconst_0、iconst_1、iconst_2、iconst_3、iconst_4、iconst_5，-1对应iconst_m1）
- bipush （-128~127）
- sipush （-32768~32767）
- ldc （-2147483648~2147483647）

**参考博文**：

[彻底理解-Java自动装箱、拆箱](https://www.jianshu.com/p/bbe6bffcb03b)

[Java中的自动装箱与拆箱](https://blog.csdn.net/paul_544425851/article/details/79679700)

[Java Integer的内存存储在堆和常量池中，及String的内存存储](https://www.jianshu.com/p/4e28ae29c8d3)

#### 1.10 理解Java中的软、弱引用

[理解Java中的弱引用](https://droidyue.com/blog/2014/10/12/understanding-weakreference-in-java/)

#### 1.11 java二进制按位运算符、移位运算符、原码、反码、补码

首先，来讲述一下计算机原码、反码、补码的关系。

> 注意：在计算机中没有原码，反码的存在，只有补码。

##### 原码是什么

原码就是早期用来表示数字的一种方式：一个正数，转换为二进制位就是这个正数的原码。负数的绝对值转换成二进制位然后在高位补1就是这个负数的原码。

**举例说明**：

int类型的3的原码是11B(B代表二进制位)，在32位机器上占四个字节，那么高位补零就得：

00000000 00000000 00000000 00000011

int类型的 -3 的绝对值的二进制位就是上面的 11B 展开后高位补零就得：

10000000 00000000 00000000 00000011　

但原码有几个缺点，零分为两种+0和-0.还有，在进行不同符号的加法运算或者同符号的减法运算的时候，不能直接判断出结果的正负。你需要将两个值的绝对值进行比较，然后进行加减操作 ，最后符号位由绝对值大的决定。于是反码就产生了。

##### 反码是什么

正数的反码就是原码，负数的反码等于原码除符号位以外所有的位取反。

**举例说明**：

int类型的 3 的反码是：

00000000 00000000 00000000 00000011

int 类型的 -3 的反码是：

11111111 11111111 11111111 11111100

除开符号位，所有位取反。解决了加减运算的问题，但是还有+0与-0之分，为此，补码就产生了。

##### 补码是什么

正数的补码与原码相同，负数的补码为其原码除符号位外所有位取反(得到反码了)，然后最低位加1.

**举例说明**：

int类型的 3 的补码是：

00000000 00000000 00000000 00000011

int类型的 -3的补码是：

11111111 11111111 1111111 11111101

就是反码的最低位+1.

##### 原码、反码、补码总结

对于正整数而言： 其原码、反码和补码都是一样的；

对于负数部分而言：
- 原码和反码的相互转换： 符号位不变，数值位按位取反。
- 原码和补码的相互转换： 符号位不变，数值按位取反，末位再加1.(补码的补码等于原码)
- 已知补码，求原码的负数的补码：符号位和数值位都取反，末位再加1.

在介绍完二进制在计算机中的存储形式，我们来了解一下按位运算符：

1. ^(异或运算符)：针对二进制，相同则为0，不同则为1
2. &(与运算)：针对二进制，只要有一个是0，就为0，两者都为1才是1
3. |(或运算)：针对二进制，只要有一个是1，就为1
4. ~(按位取反)：按位取反，0->1, 1->0

```
2 ========> 0010
3 ========> 0011

2^3 则为 0001，结果就是1
2&3 则为 0010 ，结果是2
2|3 则为 0011， 结果是3
~2 的补码则为11111101，转换成原码则为10000011，结果就是-3
```

最后，来了解一下移位运算符：

运算符 | 含义 | 例子
--- | --- | ---
"<<" | 左移运算符，将运算符左边的对象向左移动运算符右边指定的位数(在低位补0) | x<<3
">>" | "有符号"右移运算符，将运算符左边的对象向右移动运算符右边指定的位数。使用符号扩展机制，也就是说，如果值为正，则在高位补0，如果值为负，则在高位补1 | x>>3
">>>" | "无符号"右移运算符，将运算符左边的对象向右移动运算符右边指定的位数。采用0扩展机制，也就是说，无论值的正负，都在高位补0 | x>>>3

```
//都是补码形式
2 ========> 0000 0010
3 ========> 0000 0011
16 ========> 0001 0000
-16 ========> 1111 0000

2<<3 则为 0001 0000 ，结果就是16
2>>3 则为 0000 0000， 结果就是0
16>>>2 则为 0000 0100， 结果为4
-16>>2 则为 1111 1100，换算成原码则为 1000 0100，结果为-4
```
**注：** x<<y就相当于 x*2的y次方 ；x>>y相当于x/2的y次方。

##### Java位运算技巧

**1. 获取int型最大值**

```java
    public static void main(String[] args) {
        int maxInt = (1 << 31) - 1;
        int maxInt1 = ~(1 << 31);
        int maxInt2 = (1 << -1) - 1;
        int maxInt3 = (-1>>>1);
        System.out.println("十进制: "+ maxInt +" ,二进制: " + Integer.toBinaryString(maxInt));
        System.out.println("十进制: "+ maxInt1 +" ,二进制: " + Integer.toBinaryString(maxInt1));
        System.out.println("十进制: "+ maxInt2 +" ,二进制: " + Integer.toBinaryString(maxInt2));
        System.out.println("十进制: "+ maxInt3 +" ,二进制: " + Integer.toBinaryString(maxInt3));
    }

    //都输出
    // 十进制: 2147483647 ,二进制: 1111111111111111111111111111111
```

exp: int类型为32位，要获得int的最大值，只需要最高位为0(正数)，其余位为1，便可得到最大数[01111111 11111111 11111111 11111111](2进制)、[0xFFFFFFF](16进制)。

**2.获取int最小值**

```java
    public static void main(String[] args) {
        int minInt = 1 << 31;
        int minInt1 = -1 << 31;
        int minInt2 = 1 << -1;
        System.out.println("十进制: "+ minInt +" ,二进制: " + Integer.toBinaryString(minInt));
        System.out.println("十进制: "+  minInt1 +" ,二进制: " + Integer.toBinaryString(minInt1));
        System.out.println("十进制: "+  minInt2 +" ,二进制: " + Integer.toBinaryString(minInt2));
        System.out.println("十进制: "+  0x80000000 +" ,二进制: " + Integer.toBinaryString(0x80000000));
    }
    /** ~output~
     十进制: -2147483648 ,二进制: 10000000000000000000000000000000
     十进制: -2147483648 ,二进制: 10000000000000000000000000000000
     十进制: -2147483648 ,二进制: 10000000000000000000000000000000
     十进制: -2147483648 ,二进制: 10000000000000000000000000000000
     */
```
exp：int类型为32位，要获得int的最大值，只需要最高位为1(负数)，其余位为0，便可得到最大数[10000000 00000000 00000000 00000000](2进制)、[0x80000000](16进制)。

**3.n乘以2的m次方或除以2的m次方**

```java
	// 计算n*(2^m)
    public static int mulTwoPower(int n,int m){
        return n << m;
    }
 
    // 计算n/(2^m)
    public static int divTwoPower(int n,int m){
        return n >> m;
    }
    public static void main(String[] args) {
        System.out.println("5 * 2 * 2 * 2 = "+ mulTwoPower(5, 3) );
        System.out.println("6 / 2 / 2 = "+ divTwoPower(6, 2) );
    }
    /** ~output~
        5 * 2 * 2 * 2 = 40
        6 / 2 / 2 = 1
    */
```
我们知道十进制是逢十进一，二进制是逢二进一 ，十进制*10，扩大原来的10倍，尾部多一个0，二进制*2，扩大原来的2倍，尾数也多一个0，这便相当于二进制数左移<<1位，0000 1111 * 2 = 0001 1110， 除以2则反之。

**4.判断奇偶数**

```java
    // true 奇数   false 偶数
    public static boolean isOddNumber(int n){
        return (n & 1) == 1;
    }
 
    public static void main(String[] args) {
        System.out.println("5 是 "+ isOddNumber(5) );
        System.out.println("1234 是 "+ isOddNumber(1234) );
    }
    /** ~output~
         5 是 true
        1234 是 false
     */
```
为什么与1能判断奇偶？所谓的二进制就是满2进1，那么好了，偶数的最低位肯定是0（恰好满2，对不对？），同理，奇数的最低位肯定是1.int类型的1，前31位都是0，无论是1&0还是0&0结果都是0，那么有区别的就是1的最低位上的1了，若n的二进制最低位是1（奇数）与上1，结果为1，反则结果为0.

**5.对2的n次方取余**

```java
    public static int indexFor(int m, int n){
        return  m & (n - 1);
    }
 
    public static void main(String[] args) {
        System.out.println("19 与 16 求余 = "+ indexFor(19, 16) );
        System.out.println("19 与 16 求余 = "+ 19 % 16 );
    }
    /** ~output~
         19 与 16 求余 = 3
         19 与 16 求余 = 3
     */
```
此方法中n为2的指数值，则其二进制形式的表示中只存在一个1，其余位都为0，例如: 0000 1000、0100 0000、0010 0000等等。

则n-1的二进制形式就为1的位数变为0，其右边位全变为1，例如16的二进制  0001 0000 -1 = 0000 1111

测试m为19的二进制 0001 0011 & 0000 1111 = 0000 0011 = 3，低位保留的结果便是余数。

`此位运算也是HashMap中确定元素键(key)值所在哈希数组下标位置的核心方法，此位运算(hash & (length - 1))的效率极高于hash % length的求余, 所以也解释了为什么HashMap的扩容始终为2的倍数(2的指数值)。`

**6.快速幂算法，求n的m次方**

```java
    // 快速幂算法求n的m次方
    public static int power(int n, int m) {
        int temp = 1, base = n;
        while (m != 0)  {
            if ((m & 1) == 1) {  // 判断奇偶,
                temp = temp * base;
            }
            base = base * base;
            m >>= 1;   // 舍弃尾部位
        }
        return temp;
    }
 
    public static void main(String[] args) {
        System.out.println("3的3次方 = " + power(3,3));
        System.out.println("5的7次方 = " + power(5,7));
    }
    /** ~output~
     3的3次方 = 27
     5的7次方 = 78125
     */
```
在数学中，存在等式n^m = n^(m1+m2+m3+.....+mk) = n^m1 * n^m2 * n^m3 * ...* n^mk， 且m1 + m2 + m3 +....+mk = m

我们计算m的二进制，如上示例幂数7的二进制=0000 0111，他的十进制计算为： 1+2+4，所以5^7 = 5^(1+2+4) = 5^1 * 5^2 * 5^4，可以看出时间复杂度为f（n）= O(lgn)

**7. 取绝对值**

```java
int abs(int n){  
return (n ^ (n >> 31)) - (n >> 31);  
/* n>>31 取得n的符号，若n为正数，n>>31等于0，若n为负数，n>>31等于-1 
若n为正数 n^0=0,数不变，若n为负数有n^-1 需要计算n和-1的补码，然后进行异或运算， 
结果n变号并且为n的绝对值减1，再减去-1就是绝对值 */  
}  
```
若a为正数，则不变，需要用异或0保持的特点；若a为负数，则其补码为源码翻转每一位后+1，先求其源码，补码-1后再翻转每一位，此时需要使用异或1具有翻转的特点.（**已知补码，求原码的负数的补码：符号位和数值位都取反，末位再加1.**）

任何正数右移31后只剩符号位0，最终结果为0，任何负数右移31后也只剩符号位1，溢出的31位截断，空出的31位补符号位1，最终结果为-1.右移31操作可以取得任何整数的符号位。

那么综合上面的步骤，可得到公式。a>>31取得a的符号，若a为正数，a>>31等于0，a^0=a，不变；若a为负数,a>>31等于-1 ，a^-1翻转每一位.

**8.不用临时变量交换两个数**

在int[]数组首尾互换中，是不是见过这样的代码：

```java

public static int[] reverse(int[] nums){
        int i = 0;
        int j = nums.length-1;
        while(j>i){
            nums[i]= nums[i]^nums[j];
            nums[j] = nums[j]^nums[i];
            nums[i] = nums[i]^nums[j];
            j--;
            i++;
        }
        return nums;
}

//交换a,b值
int a= 3;
int b = 4;
a ^= b;  
b ^= a;  
a ^= b; 
//此时a = 4, b = 3

```
上面的计算主要遵循了一个计算公式：b^(a^b)=a，推导如下：

任何数异或本身结果为0.且有定理a^b=b^a。异或是一个无顺序的运算符，则b^a^b=b^b^a，结果为0^a。根据异或运算定义(相同为0，不同为1)，可以知道异或0具有保持的特点，而异或1具有翻转的特点。使用这些特点可以进行取数的操作。

那么那么0^a，使用异或0具有保持的特点，最终结果就是a。
