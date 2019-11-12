---
layout:     post
title:      "JS基础学习"
subtitle:   " \"JavaScript学习\""
date:       2019-11-06 20:05:00
author:     "Ming"
catalog: true
header-img: "img/post-bg-girl.jpg"
tags:
    - JavaScript
    - 前端
---

> "We must accept finite disappointment, but we must never lose infinite hope."

## 1. 基础语法

### 1.1 区块

JavaScript使用大括号，将多个相关的语句组合在一起，称为“区块”(block)。对于`var`命令来说，JavaScript的区块不构成单独的作用域。

```js
{
    var a = 1;
}
a // 1
```
上面代码在区块内部，使用`var`命令声明并赋值了变量a，然后在区块外部，变量a依然有效，区块对于`var`命令不构成单独的作用域，与不使用区块的情况没有任何区别。在JavaScript语言中，单独使用区块并不常见，区块往往用来构成其他更复杂的语法结构，比如`for`、`if`、`while`、`function`等。

### 1.2 with语句

`with`语句的格式如下：

```js
with(对象){
    语句;
}
```
它的作用是操作同一个对象的多个属性时，提供一些书写的方便。

```js
// 例一
var obj = {
  p1: 1,
  p2: 2,
};
with (obj) {
  p1 = 4;
  p2 = 5;
}
// 等同于
obj.p1 = 4;
obj.p2 = 5;

// 例二
with (document.links[0]){
  console.log(href);
  console.log(title);
  console.log(style);
}
// 等同于
console.log(document.links[0].href);
console.log(document.links[0].title);
console.log(document.links[0].style);
```

注意，如果`with`区块内部有变量的赋值操作，必须是当前对象已经存在的属性，否则会创造一个当前作用域的全局变量。

### 1.3 闭包

闭包（closure）是 JavaScript 语言的一个难点，也是它的特色，很多高级应用都要依靠闭包实现。理解闭包，首先必须理解变量作用域。JavaScript 有两种作用域：全局作用域和函数作用域。函数内部可以直接读取全局变量。

```js
var n = 999;

function f1(){
    console.log(n);
}

f1() // 999
```
上面代码中，函数f1可以读取全局变量n。但是，函数外部无法读取到函数内部声明的变量。

```js
function f1() {
  var n = 999;
}

console.log(n)
// Uncaught ReferenceError: n is not defined(
```

上面代码中，函数f1内部声明的变量n，函数外是无法读取的。如果出于种种原因，需要得到函数内的局部变量。正常情况下，这是办不到的，只有通过变通方法才能实现。那就是在函数的内部，再定义一个函数。

```js
function f1() {
  var n = 999;
  function f2() {
　　console.log(n); // 999
  }
}
```
上面代码中，函数`f2`就在函数`f1`的内部，这时`f1`内部的所有局部变量，对`f2`都是可见的。但是反过来就不行，`f2`内部的局部变量，对`f1`就是不可见的。这就是`JavaScript`语言特有的"链式作用域"结构（chain scope），子对象会一级一级地向上寻找所有父对象的变量。所以，父对象的所有变量，对子对象都是可见的，反之则不成立。

既然`f2`可以读取`f1`的局部变量，那么只要把`f2`作为返回值，我们不就可以在`f1`外部读取它的内部变量了吗！

```js
function f1() {
  var n = 999;
  function f2() {
    console.log(n);
  }
  return f2;
}

var result = f1();
result(); // 999
```

上面代码中，函数`f1`的返回值就是函数`f2`，由于`f2`可以读取`f1`的内部变量，所以就可以在外部获得`f1`的内部变量了。

闭包就是函数f2，即能够读取其他函数内部变量的函数。由于在 JavaScript 语言中，只有函数内部的子函数才能读取内部变量，因此可以把闭包简单理解成“定义在一个函数内部的函数”。闭包最大的特点，就是它可以“记住”诞生的环境，比如f2记住了它诞生的环境f1，所以从f2可以得到f1的内部变量。在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。

闭包的最大用处有两个，一个是可以读取函数内部的变量，另一个就是让这些变量始终保持在内存中，即闭包可以使得它诞生环境一直存在。请看下面的例子，闭包使得内部变量记住上一次调用时的运算结果。

```js
function createIncrementor(start) {
  return function () {
    return start++;
  };
}

var inc = createIncrementor(5);

inc() // 5
inc() // 6
inc() // 7
```

上面代码中，start是函数createIncrementor的内部变量。通过闭包，start的状态被保留了，每一次调用都是在上一次调用的基础上进行计算。从中可以看到，闭包inc使得函数createIncrementor的内部环境，一直存在。所以，闭包可以看作是函数内部作用域的一个接口。

为什么会这样呢？原因就在于inc始终在内存中，而inc的存在依赖于createIncrementor，因此也始终在内存中，不会在调用结束后，被垃圾回收机制回收。

`闭包的另一个用处，是封装对象的私有属性和私有方法。`

```java
function Person(name) {
  var _age;
  function setAge(n) {
    _age = n;
  }
  function getAge() {
    return _age;
  }

  return {
    name: name,
    getAge: getAge,
    setAge: setAge
  };
}

var p1 = Person('张三');
p1.setAge(25);
p1.getAge() // 25
```
上面代码中，函数Person的内部变量`_age`，通过闭包`getAge`和`setAge`，变成了返回对象p1的私有变量。

### 1.4 原生错误类型

- SyntaxError对象： 是解析代码时发生的语法错误。
- ReferenceError对象： 引用一个不存在的变量时发生的错误。
- RangeError对象：是一个值超过有效范围时发生的错误。主要有几种情况，一是数组长度为负数，二是Number对象的方法参数超出范围，以及函数堆栈超出最大值。
- TypeError对象：是变量或者参数不是预期类型时发生的错误。比如，对字符串、布尔值、数值等原始类型的值使用`new`命令，就会抛出这种错误。因为`new`命令的参数应该是一个构造函数。
- URIError对象：是URI相关函数的参数不正确的时候抛出的错误，主要涉及`encodeURI()`、`decodeURI()` 、`encodeURIComponent()`、`decodeURIComponent()`、`escape()`和`unescape()`这六个函数。
- EvalError对象： `eval`函数未能正确执行的时候，会抛出`EvalError`错误。该错误类型已经不再使用了，只是为了保证与以前代码兼容，才继续保留。

### 1.5 属性描述对象

JavaScript提供了一个内部数据结构，用来描述对象的属性，控制对象的行为，比如该属性是否可写、可遍历等等。这个内部数据结构称为“属性描述对象”。每个属性都有自己对应的属性描述对象，保存该属性的一些元信息。(给出一个例子如下所示)

```js
{
  value: 123,
  writable: false,
  enumerable: true,
  configurable: false,
  get: undefined,
  set: undefined
}
```
属性描述对象提供6个元信息。
1. value: 是该属性的属性值，默认为`undefined`
2. writable: 是一个布尔值，表示属性(value)是否可以改变(即是否可写)，默认为true。
3. enumerable: 是一个布尔值，表示该属性是否可以遍历，默认为true。如果设置为false，会使得某些操作(比如for...in循环、Object.keys())跳过该属性。
4. configurable: 是一个布尔值，表示可配置性，默认为true。如果设为false，将阻止某些操作改写该属性，比如无法删除该属性，也不得改变该属性的属性描述对象(value属性除外)。也就是说，`configurable`属性控制了属性描述对象的可写性。
5. get: 是一个函数，表示该属性的取值函数(getter)，默认值为`undefined`。
6. set: 是一个函数，表示该属性的存值函数(setter),默认是为`undefined`。

### 1.6 面向对象编程

#### 1. 构造函数

构造函数就是一个普通的函数，但是其有自己的特征和用法：

```js
var Vehicle = function () {
  this.price = 1000;
};
```
上面的代码中，`Vehicle`就是构造函数。为了和普通函数进行区别，构造函数名字的第一个字母通常大写。构造函数的特点主要有两个：
1. 函数体内部使用了`this`关键字，代表所有生成的对象实例。
2. 生成对象的时候，必须使用`new`命令。

> 有一个很自然的问题，如果忘记了使用`new`命令，直接调用构造函数会发生什么事情？

在这种情况下，构造函数就变成了普通函数，并不会生成实例对象。`this`这时将会代表全局对象，将造成一些意想不到的结果。

```js
var Vehicle = function (){
  this.price = 1000;
};

var v = Vehicle();
v // undefined
price // 1000
```
上面代码中，调用Vehicle构造函数时，忘了加上new命令。结果，变量v变成了undefined，而price属性变成了全局变量。因此，应该非常小心，避免不使用new命令、直接调用构造函数。

**解决方法**： 一个解决方法就是构造函数内部使用严格模式，即第一行加上use strict。这样的话，一旦忘了使用new命令，直接调用构造函数就会报错。

```js
function Fubar(foo, bar){
  'use strict';
  this._foo = foo;
  this._bar = bar;
}

Fubar()
// TypeError: Cannot set property '_foo' of undefined
```
上面的代码的`Fubar`为构造函数，`use strict`命令保证了该函数在严格模式下运行。由于严格模式中，函数内部的`this`不能指向全局对象，默认等于`undefined`，导致不加`new`调用会报错。(JavaScript 不允许对undefined添加属性)。

另外一种解决办法是构造函数内部判断是否使用new命令，如果发现没有使用，则直接返回一个实例对象。

```js
function Fubar(foo, bar) {
  if (!(this instanceof Fubar)) {
    return new Fubar(foo, bar);
  }

  this._foo = foo;
  this._bar = bar;
}

Fubar(1, 2)._foo // 1
(new Fubar(1, 2))._foo // 1
```

#### 2. new命令的原理

使用`new`命令时，它后面的函数依次执行下面的步骤：
1. 创建一个空对象，作为将要返回的对象实例。
2. 将这个空对象的原型，指向构造函数的`prototype`属性。
3. 将这个空对象赋值给函数内部的`this`关键字。
4. 开始执行构造函数内部的代码。

也就是说，构造函数内部，this指的是一个新生成的空对象，所有针对this的操作，都会发生在这个空对象上。构造函数之所以叫“构造函数”，就是说这个函数的目的，就是操作一个空对象（即`this`对象），将其“构造”为需要的样子。

如果构造函数内部有return语句，而且return后面跟着一个对象，new命令会返回return语句指定的对象；否则，就会不管return语句，返回this对象。

```js
var Vehicle = function () {
  this.price = 1000;
  return 1000;
};

(new Vehicle()) === 1000
// false
```

但是，如果return语句返回的是一个跟this无关的新对象，new命令会返回这个新对象，而不是this对象。这一点需要特别引起注意。

```js
var Vehicle = function (){
  this.price = 1000;
  return { price: 2000 };
};

(new Vehicle()).price
// 2000
```
上面代码中，构造函数Vehicle的return语句，返回的是一个新对象。new命令会返回这个对象，而不是this对象。

new命令可以使用`Object.setPrototypeOf`方法模拟。

```js
var F = function () {
  this.foo = 'bar';
};

var f = new F();
// 等同于
var f = Object.setPrototypeOf({}, F.prototype);
F.call(f);
```

上面代码中，new命令新建实例对象，其实可以分成两步。第一步，将一个空对象的原型设为构造函数的`prototype`属性(上例是`F.prototype`);第二步，将构造函数内部的`this`绑定到这个空对象，然后执行构造函数，使得定义在`this`上面的属性和方法(上例是`this.foo`)，都转移到这个空对象上。

#### 3. this的本质

JavaScript语言之所以有this的设计，主要是跟内存里面的数据结构有关系。

```js
var obj = { foo:  5 };
```
上面的代码将一个对象赋值给变量obj。JavaScript 引擎会先在内存里面，生成一个对象{ foo: 5 }，然后把这个对象的内存地址赋值给变量obj。也就是说，变量obj是一个地址（reference）。后面如果要读取obj.foo，引擎先从obj拿到内存地址，然后再从该地址读出原始的对象，返回它的foo属性。

原始的对象以字典结构保存，每个属性名都有一个对应的属性描述对象。举例来说，上面例子的`foo`属性，实际上是以下面的形式保存的。

```js
{
  foo: {
    [[value]]: 5
    [[writable]]: true
    [[enumerable]]: true
    [[configurable]]: true
  }
}
```
注意，foo属性的值保存在属性描述对象的value属性里面。这样的结构是很清晰的，但是问题就在于属性的值可能是一个函数。

```js
var obj = { foo: function () {} };
```
这时，引擎会将函数单独保存在内存中，然后再将函数的地址赋值给`foo`属性的`value`属性。

```js
{
  foo: {
    [[value]]: 函数的地址
    ...
  }
}
```
由于函数是一个单独的值，所以它可以在不同的环境(上下文)执行。

```js
var f = function () {};
var obj = { f: f };

// 单独执行
f()

// obj 环境执行
obj.f()
```
JavaScript允许在函数体内部，引用当前环境的其他变量。

```js
var f = function () {
  console.log(x);
};
```
上面代码中，函数体里面使用了变量x。该变量由运行环境提供。现在问题就来了，由于函数可以在不同的运行环境执行，所以需要有一种机制，能够在函数体内部获得当前的运行环境（context）。所以，`this`就出现了，它的设计目的就是在函数体内部，指代函数当前的运行环境。

```js
var f = function () {
  console.log(this.x);
}
```
上面代码中，函数体里面的`this.x`就是指当前运行环境中的`x`。

```js
var f = function () {
  console.log(this.x);
}

var x = 1;
var obj = {
  f: f,
  x: 2,
};

// 单独执行
f() // 1

// obj 环境执行
obj.f() // 2
```
上面代码中，函数`f`在全局环境执行，`this.x`指向全局环境的x；在`obj`环境执行，`this.x`指向`obj.x`。

#### 4. this的使用注意点

**1.避免多层this**

由于this的指向是不确定的，所以切勿在函数中包含多层的`this`。

```js
var o = {
  f1: function () {
    console.log(this);
    var f2 = function () {
      console.log(this);
    }();
  }
}

o.f1()
// Object
// Window
```
上面代码包含两层`this`，结果运行后，第一层指向对象`o`，第二层指向全局对象，因为实际执行的是下面的代码。

```js
var temp = function(){
  console.log(this);
}

var o = {
  f1: function () {
    console.log(this);
    var f2 = temp();
  }
}
```
一个解决方法是在第二层改用一个指向外层this的变量。

```js
var o = {
  f1: function() {
    console.log(this);
    var that = this;
    var f2 = function() {
      console.log(that);
    }();
  }
}

o.f1()
// Object
// Object
```
上面代码定义了变量that，固定指向外层的this，然后在内层使用that，就不会发生this指向的改变。另外一种方法就是JavaScript 提供了严格模式，也可以硬性避免这种问题。严格模式下，如果函数内部的this指向顶层对象，就会报错。

```js
var counter = {
  count: 0
};
counter.inc = function () {
  'use strict';
  this.count++
};
var f = counter.inc;
f()
// TypeError: Cannot read property 'count' of undefined
```
上面代码中，inc方法通过'use strict'声明采用严格模式，这时内部的this一旦指向顶层对象，就会报错。

**2. 避免数组处理方法中的this**

数组的`map`和`foreach`方法，允许提供一个函数作为参数。这个函数内部不应该使用`this`。

```js
var o = {
  v: 'hello',
  p: [ 'a1', 'a2' ],
  f: function f() {
    this.p.forEach(function (item) {
      console.log(this.v + ' ' + item);
    });
  }
}

o.f()
// undefined a1
// undefined a2
```

上面代码中，foreach方法的回调函数中的this，其实是指向window对象，因此取不到o.v的值。原因跟上一段的多层this是一样的，就是内层的this不指向外部，而指向顶层对象。

解决这个问题的一种方法，就是前面提到的，使用中间变量固定`this`。

```js
var o = {
  v: 'hello',
  p: [ 'a1', 'a2' ],
  f: function f() {
    var that = this;
    this.p.forEach(function (item) {
      console.log(that.v+' '+item);
    });
  }
}

o.f()
// hello a1
// hello a2
```
另一种方法是将this当作foreach方法的第二个参数，固定它的运行环境。

```js
var o = {
  v: 'hello',
  p: [ 'a1', 'a2' ],
  f: function f() {
    this.p.forEach(function (item) {
      console.log(this.v + ' ' + item);
    }, this);
  }
}

o.f()
// hello a1
// hello a2
```

**3. 避免回调函数中的this**

回调函数中的`this`往往会改变指向，最后避免使用。

```js
var o = new Object();
o.f = function () {
  console.log(this === o);
}

// jQuery 的写法
$('#button').on('click', o.f);
```
上面代码中，点击按钮以后，控制台会显示false。原因是此时this不再指向o对象，而是指向按钮的 DOM 对象，因为f方法是在按钮对象的环境中被调用的。这种细微的差别，很容易在编程中忽视，导致难以察觉的错误。

#### 5. JavaScript多重继承

JavaScript不提供多重继承功能，即不允许一个对象同时继承多个对象。但是，可以通过变通方法，实现这个功能。

```js
function M1() {
  this.hello = 'hello';
}

function M2() {
  this.world = 'world';
}

function S() {
  M1.call(this);
  M2.call(this);
}

// 继承 M1
S.prototype = Object.create(M1.prototype);
// 继承链上加入 M2
Object.assign(S.prototype, M2.prototype);

// 指定构造函数
S.prototype.constructor = S;

var s = new S();
s.hello // 'hello'
s.world // 'world'
```
上面代码中，子类S同时继承了父类`M1`和`M2`。这种模式又称为 `Mixin`（混入）。

#### 6. prototype、__proto__与constructor详解

首先，在介绍之前，说明一下，`__proto__`属性的两边是各自由两个下划线构成，实际上，该属性在ES标准定义的名字是`[[Prototype]]`，具体实现由浏览器代理自己实现，谷歌浏览器就是将`[[Prototype]]`命名为`__proto__`。

我们先来从一个简单的例子入手：

```js
function Foo() {...};
let f1 = new Foo();
```

以上代码表示创建一个构造函数`Foo()`，并用new关键字实例化该构造函数得到一个实例化对象f1。(new操作符将函数作为构造器的过程原理上面已经介绍了)虽然这是简简单单的两行代码，其背后的关系是错综复杂的，如下图所示：

![new操作背后原理](https://ws1.sinaimg.cn/large/005CDUpdgy1g8ty9etzygj31d40rk43g.jpg)

**图的说明：**右下角给出了图例，其中红色箭头表示`__proto__`属性指向、绿色箭头表示`prototype`属性的指向、棕色实线箭头表示本身`具有的constructor属性`的指向，棕色虚线箭头表示继承而来的`constructor`属性的指向；蓝色方块表示对象，浅绿色方块表示函数(这里为了更好看清，Foo()仅代表是函数，并不是指执行函数Foo后得到的结果，图中的其他函数同理)。图的中间部分即为它们之间的联系，图的最左边即为例子代码。

**1. __proto__属性**

首先，我们需要了解的是：
1. `__proto__`和`constructor`属性是对象所独有的；
2. `prototype`属性是函数所独有的。

但是，由于JS中函数也是一种对象，所以函数也拥有着`__proto__`和`constructor`属性，这点是致使我们产生困惑的很大原因之一。我们来对上面的图按照属性分别拆开，然后进行分析：

![__proto__属性分析](https://ws1.sinaimg.cn/large/005CDUpdgy1g8u5csgbmnj31dq0qgdiz.jpg)

这里我们仅仅留下`__proto__`属性，它是`对象所独有的`，可以看到`__proto__`属性都是由一个对象指向一个对象，即指向他们的原型对象(也可以理解成为父对象)。这个属性的作用就是当访问一个对象的属性时，如果该对象内部不存在这个属性，那么就会去它的`__proto__`属性所指向的那个对象(可以理解为父对象)里找，如果父对象也不存在这个属性，则继续往父对象的`__proto__`属性所指向的那个对象里找，如果还没有找到，则继续往上找......直到原型链顶端`null`。再往上找就相当于在null上取值，会报错，以上这种通过`__proto__`属性来连接对象直到`null`的一条链即为我们所谓的`原型链`。

**2. prototype属性**

第二，接下来我们看`prototype`属性：

![prototype属性分析](https://ws1.sinaimg.cn/large/005CDUpdgy1g8u69cdxiaj31bf0oowha.jpg)

`prototype`属性，别忘了一点，就是我们前面所说的需要了解的第二点，它是`函数所独有的`，它是从一个函数指向一个对象。它的含义是`函数的原型对象`，也就是这个函数(其实所有函数都可以作为构造函数)所创建的实例的原型对象，由此可知：`f1.__proto__ === Foo.prototype`，它们两个完全一样。那么`prototype`属性的作用又是什么呢？它的作用就是包含可以由特定类型的所有实例共享的属性和方法，也就是让该函数所实例化的对象们都可以找到公用的属性和方法。`任何函数在创建的时候，其实会默认同时创建该函数的prototype对象`。

**3. constructor属性**

最后，我们来看一下`constructor`属性：

![constructor属性分析](https://ws1.sinaimg.cn/large/005CDUpdgy1g8u6h7gsw5j31c80q90w1.jpg)

`constructor`属性也是对象才拥有的，它是指从一个对象指向一个函数，含义就是`指向该对象的构造函数`。“每个对象都有构造函数”（本身拥有或者继承而来，继承而来的要结合`__proto__`属性查看会更清晰的表达，如下所示）。从上图可以看出`Function`对象比较特殊，它的构造函数就是它自己(因为Function可以看成是一个函数，也可以是一个对象)，所有函数和对象最终都是由Function构造函数得来，所以`constructor`属性的终点就是`Function`这个函数。

![constructor分析](https://ws1.sinaimg.cn/large/005CDUpdgy1g8u6mged71j31cx0qwtd4.jpg)

这里还是来解释一下上一段中的`每个对象都有构造函数`这句话。这里的意思是每个对象都可以找到其对应的`constructor`，因为创建对象的前提是需要有`constructor`，而这个`constructor`可能是对象本身显示定义的或者通过`__proto__`在原型链中找到的。`而单单从constructor这个属性来讲，只有prototype对象才有`。每个函数在创建的时候，JS会同时创建一个该函数对应的prototype对象，而`函数创建的对象.__proto__ === 该函数.prototype, 该函数.prototype.constructor === 该函数本身`，故通过函数创建的对象即使自己没有constructor属性，它也能够通过`__proto__`找到对应的constructor，所以任何对象最终都可以找到其构造函数(null如果当成对象的话，将null除外)。如下所示：

![constructor解释](https://ws1.sinaimg.cn/large/005CDUpdgy1g8u6ui1vkpj30sw0gr0uv.jpg)

**4. 总结**

1. 我们需要牢记两点：`__proto__`和`constructor`属性是对象所独有的；`prototype`属性是函数所独有的，因为函数也是一种对象，所以函数也拥有`__proto__`和`constructor`属性。
2. `__proto__`属性的作用就是当访问一个对象的属性时，如果该对象内部不存在这个属性，那么就会去它的`__proto__`属性所指向的那个对象（父对象）里找，一直找，直到`__proto__`属性的终点null，再往上找就相当于在`null`上取值，会报错。通过`__proto__`属性将对象连接起来的这条链路即我们所谓的原型链。
3. `prototype`属性的作用就是让该函数所实例化的对象们都可以找到公用的属性和方法，即`f1.__proto__ === Foo.prototype`。
4. `constructor`属性的含义就是指向该对象的构造函数，所有函数（此时看成对象了）最终的构造函数都指向`Function`。

**参考文章**：

[帮你彻底搞懂JS中的prototype、__proto__与constructor（图解）](https://blog.csdn.net/cc18868876837/article/details/81211729)

[prototype和__proto__的关系是什么？](https://www.cnblogs.com/Narcotic/p/6899088.html)

[对JavaScript中原型模式的理解](https://blog.csdn.net/qq_30904985/article/details/81240791)

[js的面试与笔试--JavaScript prototype原型和原型链详解](https://blog.csdn.net/qq_30904985/article/details/81248613)

[一张图理解prototype、proto和constructor的三角关系](https://www.cnblogs.com/xiaohuochai/p/5721552.html)

#### 7. 异步操作的流程控制

如果有多个异步操作，就存在一个流程控制的问题：如何确定异步操作执行的顺序，以及如何保证遵守这种顺序。

```js
function async(arg, callback) {
  console.log('参数为 ' + arg +' , 1秒后返回结果');
  setTimeout(function () { callback(arg * 2); }, 1000);
}
```
上面代码的async函数是一个异步任务，非常耗时，每次执行需要1秒才能完成，然后再调用回调函数。如果有六个这样的异步任务，需要全部完成后，才能执行最后的final函数。请问应该如何安排操作流程？或许首先给出回调函数的一种形式：

```js
function final(value) {
  console.log('完成: ', value);
}

async(1, function (value) {
  async(2, function (value) {
    async(3, function (value) {
      async(4, function (value) {
        async(5, function (value) {
          async(6, final);
        });
      });
    });
  });
});
// 参数为 1 , 1秒后返回结果
// 参数为 2 , 1秒后返回结果
// 参数为 3 , 1秒后返回结果
// 参数为 4 , 1秒后返回结果
// 参数为 5 , 1秒后返回结果
// 参数为 6 , 1秒后返回结果
// 完成:  12
```
上面代码中，六个回调函数的嵌套，不仅写起来麻烦，容易出错，而且难以维护。

**1. 串行执行**

我们可以编写一个流程控制函数，让它来控制异步任务，一个任务完成以后，再执行另一个。这就叫串行执行：

```js
var items = [ 1, 2, 3, 4, 5, 6 ];
var results = [];

function async(arg, callback) {
  console.log('参数为 ' + arg +' , 1秒后返回结果');
  setTimeout(function () { callback(arg * 2); }, 1000);
}

function final(value) {
  console.log('完成: ', value);
}

function series(item) {
  if(item) {
    async( item, function(result) {
      results.push(result);
      return series(items.shift());
    });
  } else {
    return final(results[results.length - 1]);
  }
}

series(items.shift());
```
上面代码中，函数series就是串行函数，它会依次执行异步任务，所有任务都完成后，才会执行final函数。items数组保存每一个异步任务的参数，results数组保存每一个异步任务的运行结果。注意，上面的写法需要六秒，才能完成整个脚本。

**2. 并行执行**

流程控制函数也可以是并行执行，即所有异步任务同时执行，等到全部完成以后，才执行`final`函数。

```js
var items = [ 1, 2, 3, 4, 5, 6 ];
var results = [];

function async(arg, callback) {
  console.log('参数为 ' + arg +' , 1秒后返回结果');
  setTimeout(function () { callback(arg * 2); }, 1000);
}

function final(value) {
  console.log('完成: ', value);
}

items.forEach(function(item) {
  async(item, function(result){
    results.push(result);
    if(results.length === items.length) {
      final(results[results.length - 1]);
    }
  })
});
```
上面代码中，forEach方法会同时发起六个异步任务，等到它们全部完成以后，才会执行final函数。

相比而言，上面的写法只要一秒，就能完成整个脚本。这就是说，并行执行的效率较高，比起串行执行一次只能执行一个任务，较为节约时间。但是问题在于如果并行的任务较多，很容易耗尽系统资源，拖慢运行速度。因此有了第三种流程控制方式。

**3. 并行与串行的结合**

所谓并行与串行的结合，就是设置一个门槛，每次最多只能并行执行n个异步任务，这样就避免了过分占用系统资源。

```js
var items = [ 1, 2, 3, 4, 5, 6 ];
var results = [];
var running = 0;
var limit = 2;

function async(arg, callback) {
  console.log('参数为 ' + arg +' , 1秒后返回结果');
  setTimeout(function () { callback(arg * 2); }, 1000);
}

function final(value) {
  console.log('完成: ', value);
}

function launcher() {
  while(running < limit && items.length > 0) {
    var item = items.shift();
    async(item, function(result) {
      results.push(result);
      running--;
      if(items.length > 0) {
        launcher();
      } else if(running == 0) {
        final(results);
      }
    });
    running++;
  }
}

launcher();
```
上面代码中，最多只能同时运行两个异步任务。变量`running`记录当前正在运行的任务数，只要低于门槛值，就再启动一个新的任务，如果等于0，就表示所有任务都执行完了，这时就执行`final`函数。

这段代码需要三秒完成整个脚本，处在串行执行和并行执行之间。通过调节limit变量，达到效率和资源的最佳平衡。

#### 8. setTimeout和setInterval运行机制

`setTimeout`和`setInterval`的运行机制，是将指定的代码移出本轮事件循环，等到下一轮事件循环，再检查是否到了指定时间。如果到了，就执行对应的代码；如果不到，就继续等待。

这意味着，`setTimeout`和`setInterval`指定的回调函数，必须等到本轮事件循环的所有同步任务都执行完，才会开始执行。由于前面的任务到底需要多少时间执行完，是不确定的，所以没有办法保证，`setTimeout`和`setInterval`指定的任务，一定会按照预定时间执行。

```js
setTimeout(someTask, 100);
veryLongTask();
```
上面代码的`setTimeout`，指定100毫秒以后运行一个任务。但是，如果后面的`veryLongTask`函数（同步任务）运行时间非常长，过了100毫秒还无法结束，那么被推迟运行的`someTask`就只有等着，等到`veryLongTask`运行结束，才轮到它执行。

再看一个`setInterval`的例子。

```js
setInterval(function () {
  console.log(2);
}, 1000);

sleep(3000);

function sleep(ms) {
  var start = Date.now();
  while ((Date.now() - start) < ms) {
  }
}
```
上面代码中，`setInterval`要求每隔1000毫秒，就输出一个2。但是，紧接着的sleep语句需要3000毫秒才能完成，那么`setInterva`l就必须推迟到3000毫秒之后才开始生效。注意，生效后`setInterval`不会产生累积效应，即不会一下子输出三个2，而是只会输出一个2。

#### 9. setTimeout(f,0)的妙用

`setTimeout`的作用是将代码推迟到指定时间执行，如果指定时间为0，即`setTimeout(f, 0)`，那么会立刻执行吗？

答案是不会。因为上一节说过，必须要等到当前脚本的同步任务，全部处理完以后，才会执行`setTimeout`指定的回调函数f。也就是说，`setTimeout(f, 0)`会在下一轮事件循环一开始就执行。

```js
setTimeout(function () {
  console.log(1);
}, 0);
console.log(2);
// 2
// 1
```
上面代码先输出2，再输出1。因为2是同步任务，在本轮事件循环执行，而1是下一轮事件循环执行。

总之，setTimeout(f, 0)这种写法的目的是，尽可能早地执行f，但是并不能保证立刻就执行f。

实际上，setTimeout(f, 0)不会真的在0毫秒之后运行，不同的浏览器有不同的实现。以 Edge 浏览器为例，会等到4毫秒之后运行。如果电脑正在使用电池供电，会等到16毫秒之后运行；如果网页不在当前 Tab 页，会推迟到1000毫秒（1秒）之后运行。这样是为了节省系统资源。

**setTimeout(f,0)的应用**

`setTimeout(f, 0)`有几个非常重要的用途。它的一大应用是，可以调整事件的发生顺序。比如，网页开发中，某个事件先发生在子元素，然后冒泡到父元素，即子元素的事件回调函数，会早于父元素的事件回调函数触发。如果，想让父元素的事件回调函数先发生，就要用到`setTimeout(f, 0)`。

```js
// HTML 代码如下
// <input type="button" id="myButton" value="click">

var input = document.getElementById('myButton');

input.onclick = function A() {
  setTimeout(function B() {
    input.value +=' input';
  }, 0)
};

document.body.onclick = function C() {
  input.value += ' body'
};
```
上面代码在点击按钮后，先触发回调函数A，然后触发函数C。函数A中，setTimeout将函数B推迟到下一轮事件循环执行，这样就起到了，先触发父元素的回调函数C的目的了。

另一个应用是，用户自定义的回调函数，通常在浏览器的默认动作之前触发。比如，用户在输入框输入文本，`keypress`事件会在浏览器接收文本之前触发。因此，下面的回调函数是达不到目的的。

```js
// HTML 代码如下
// <input type="text" id="input-box">

document.getElementById('input-box').onkeypress = function (event) {
  this.value = this.value.toUpperCase();
}
```
上面代码想在用户每次输入文本后，立即将字符转为大写。但是实际上，它只能将本次输入前的字符转为大写，因为浏览器此时还没接收到新的文本，所以`this.value`取不到最新输入的那个字符。只有用`setTimeout`改写，上面的代码才能发挥作用。

```js
document.getElementById('input-box').onkeypress = function() {
  var self = this;
  setTimeout(function() {
    self.value = self.value.toUpperCase();
  }, 0);
}
```

上面代码将代码放入setTimeout之中，就能使得它在浏览器接收到文本之后触发。

由于setTimeout(f, 0)实际上意味着，将任务放到浏览器最早可得的空闲时段执行，所以那些计算量大、耗时长的任务，常常会被放到几个小部分，分别放到setTimeout(f, 0)里面执行。

```js
var div = document.getElementsByTagName('div')[0];

// 写法一
for (var i = 0xA00000; i < 0xFFFFFF; i++) {
  div.style.backgroundColor = '#' + i.toString(16);
}

// 写法二
var timer;
var i=0x100000;

function func() {
  timer = setTimeout(func, 0);
  div.style.backgroundColor = '#' + i.toString(16);
  if (i++ == 0xFFFFFF) clearTimeout(timer);
}

timer = setTimeout(func, 0);
```
上面代码有两种写法，都是改变一个网页元素的背景色。写法一会造成浏览器“堵塞”，因为 JavaScript 执行速度远高于 DOM，会造成大量 DOM 操作“堆积”，而写法二就不会，这就是`setTimeout(f, 0)`的好处。

此外，另一个使用这种技巧的例子是代码高亮的处理。如果代码块很大，一次性处理，可能会对性能造成很大的压力，那么将其分成一个个小块，一次处理一块，比如写成`setTimeout(highlightNext, 50)`的样子，性能压力就会减轻。

#### 8. 寄生组合式继承

所谓寄生组合式继承，即通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。其背后的基本思路是：**不必为了指定子类型的原型而调用超类型的构造函数，我们所需要的无非就是超类型原型的一个副本而已。本质上，就是使用寄生式继承来继承超类型的原型，然后再将结果指定给子类型的原型。**基本模式如下所示：

```js
//或者不用Object.create()方法，采用这种

function object(o){
  function F(){}'
  F.prototype = o;
  return new F();
}

function inhertiPrototype(subType, superType){
  var prototype = Object.create(superType.prototype); //或者调用object()方法
  prototype.constructor = subType;
  subType.prototype = prototype;
}

//继承方式如下

function SuperType(name){
  this.name = name;
  this.colors = ['red', 'blue', 'green'];
}

SuperType.prototype.sayName = function(){
  alert(this.name);
};

function SubType(name, age){
  SuperType.call(this, name);
  this.age = age;
}

inhertiPrototype(SubType, SuperType);

SubType.prototype.sayName = function(){
  alert(this.name);
}
```
这种方式的高效性体现在它只是调用了一次`SuperType`的构造函数，并且因此避免了在`SubType.prototype`上面创建不必要、多余的属性。与此同时，原型链还能保持不变，因此能够正常地使用`instanceof()`和`isPrototypeof()`。