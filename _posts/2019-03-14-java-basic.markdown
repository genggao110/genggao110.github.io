---
layout:      post
title:       "Java Basic Knowledge"
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

#### 1.7
