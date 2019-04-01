---
layout:      post
title:       "Java Basic Knowledge 1"
subtitle:    " \"Java core technology\""
date:        2019-03-14 16:18:37
author:      "Ming"
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
**分析**：首先看(1)(3),其中的f()方法是从类Father1继承而来，其可见性是包p1及其子类Son1和Son11，而由于调用f()方法的类Test1所在的也是p1，因此(1)(3)处编译通过。**也就是说，如果我们换一个包，比如Test11.java在p11下，那么将都不可访问。**如下所示：

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





