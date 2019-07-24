---
layout:      post
title:       "Nodejs整理"
subtitle:    " \"nodejs Analysis\""
date:        2019-05-27 20:31:26
author:      "Ming"
catalog: true
header-img:  "img/home-bg-o.jpg"
tags:
    - Nodejs
    - Knowledge
---

> A great poem is a fountain forever overflowing with the waters of wisdom and delight.

### 1.异步式I/O与事件式编程

> Node.js最大的特点就是异步式I/O(或者非阻塞式I/O)与事件紧密结合的编程模式。这种模式与传统的同步式I/O线性的编程思路有很大的不同，因为控制流很大程度上要靠事件和回调函数来组织，一个逻辑要拆分为若干个单元。

**异步式I/O**

当线程遇到I/O操作时，不会以阻塞的方式等待I/O操作的完成或数据的返回，而只是将I/O请求发送给操作系统，继续执行下一条语句。当操作系统完成I/O操作时，以事件的形式通知执行I/O操作的线程，线程会在特定时候处理这个事件。为了处理异步I/O，线程必须有事件循环，不断地检查有没有未处理的事件，依次予以处理。

> 相比较于传统的多线程阻塞式I/O，异步式I/O其好处就是在于减少了多线程的开销。对操作系统而言，创建一个线程的代价是十分昂贵的，需要给它分配内存、列入调度，同时在线程切换的时候还要执行内存换页，CPU的缓存被清空，切换回来的时候还要重新从内存中读取信息，破坏了数据的局部性。

**回调函数**

在Nodejs中，异步式I/O是通过回调函数来实现的。以fs.readFile为例：

```js
fs.readFile('file.txt', 'utf-8', function(err,data){
    if(err){
        console.error(err);
    }else{
        console.log(data);
    }
})
console.log('finish');
```

fs.readFile调用时所做的工作只是将异步式I/O请求发送给了操作系统，然后立即返回并执行后面的语句，执行完之后进入事件循环监听事件。当fs接收到I/O请求完成的事件时，事件循环会主动调用回调函数以完成后续工作。

**Nodejs的事件循环机制**

Nodejs程序由事件循环开始，到事件循环结束，所有的逻辑都是事件的回调函数，所以Nodejs始终在事件循环中，程序入口就是事件循环第一个事件的回调函数。事件的回调函数在执行的过程中，可能会发出I/O请求或直接发射事件，执行完毕后再返回事件循环，事件循环会检查事件队列中有没有未处理的事件，直到程序结束。

![事件循环](https://ws1.sinaimg.cn/large/005CDUpdgy1g3gtodedixj30ec0ckwer.jpg)

### 2.Nodejs HTTP服务器与客户端

> Nodejs标准库提供了http模块，其中封装了一个高效的HTTP服务器和一个简易的HTTP客户端。http.Server是一个基于事件的HTTP服务器，它的核心由Nodejs下层C++部分实现，而接口由JavaScript封装，兼顾了高性能与简易性。http.request则是一个HTTP客户端工具，用于向HTTP服务器发起请求。

#### 2.1 http.Server的事件

http.Server是一个基于事件的HTTP服务器，所有的请求都被封装为独立的事件，开发者只需要对它的事件编写响应函数即可实现HTTP服务器的所有功能。其继承自EventEmitter，提供以下几个事件：

- request:当客户端请求到来时，该事件被触发，提供两个参数req和res，分别是http.ServerRequest和http.ServerResponse的实例，表示请求和响应信息。
- connection:当TCP连接建立时，该事件被触发，提供一个参数socket，为net.Socket的实例。connection事件的粒度要大于request，因为客户端在Keep-Alive模式下可能会在同一个连接内发送多次请求。
- close: 当服务器关闭，该事件被触发。

#### 2.2 http.ServerRequest

http.ServerRequest是HTTP请求的信息。HTTP 请求一般可以分为两部分：请求头（Request Header）和请求体（Requset Body）。
以上内容由于长度较短都可以在请求头解析完成后立即读取。而请求体可能相对较长，需要一定的时间传输，因此 http.ServerRequest 提供了以下3个事件用于控制请求体传输。

- data:当请求体数据到来时，该事件被触发。该事件提供一个参数 chunk，表示接收到的数据。如果该事件没有被监听，那么请求体将会被抛弃。该事件可能会被调用多次。
- end: 当请求体数据传输完成时，该事件被触发，此后将不会再有数据到来。
- close: 用户当前请求结束时，该事件被触发。不同于 end，如果用户强制终止了传输，也还是调用close。

#### 2.3 http.ServerResponse

http.ServerResponse 是返回给客户端的信息，决定了用户最终能看到的结果。它也是由 http.Server 的 request 事件发送的，作为第二个参数传递，一般简称为response 或 res。

- response.writeHead(statusCode, [headers])：向请求的客户端发送响应头。statusCode 是 HTTP 状态码，如 200 （请求成功）、404 （未找到）等。headers是一个类似关联数组的对象，表示响应头的每个属性。该函数在一个请求内最多只能调用一次，如果不调用，则会自动生成一个响应头。
- response.write(data, [encoding])：向请求的客户端发送响应内容。data 是一个 Buffer 或字符串，表示要发送的内容。如果 data 是字符串，那么需要指定encoding 来说明它的编码方式，默认是 utf-8。在 response.end 调用之前，response.write 可以被多次调用。
- response.end([data], [encoding])：结束响应，告知客户端所有发送已经完成。当所有要返回的内容发送完毕的时候，该函数 必须 被调用一次。它接受两个可选参数，意义和 response.write 相同。如果不调用该函数，客户端将永远处于等待状态。

### 3.Nodejs进阶问题

#### 3.1 控制流

基于异步I/O的事件式编程容易将程序的逻辑拆的七零八落，给控制流的梳理制造障碍。

**循环的陷阱**

看下面这样一个案例：

```js
var fs = require('fs');
var files = ['a.txt', 'b.txt', 'c.txt'];

for(var i = 0; i < files.length; i++){
    fs.readFile(files[i], 'utf-8', function(err, contents){
        console.log(files[i] + ': ' + contents);
    })
}
```

很直观，代码目的就是想依次读取文件a.txt,b.txt,c.txt，并输出文件名和内容。然而运行结果如下：

```js
undefined: AAA
undefined: BBB
undefined: CCC
```

很明显，这是一个闭包问题。3次读取文件的回调函数事实上是同一个实例，其中引用到的i值是上面循环执行结束后的值(即 i = 3)。因此不能分辨。前端中常用的解决方案利用函数式编程的特性，手动构建一个闭包：

```js
var fs = require('fs');
var files = ['a.txt', 'b.txt', 'c.txt'];

for(var i = 0; i < files.length; i++){
    (function(i){
        fs.readFile(files[i], 'utf-8', function(err, contents){
            console.log(files[i] + ': ' + contents);
        });
    })(i);  
}
```

for 循环体中建立了一个匿名函数，将循环迭代变量 i 作为函数的参数传递并调用。由于运行时闭包的存在，该匿名函数中定义的变量（包括参数表）在它内部的函
数（ fs.readFile 的回调函数）执行完毕之前都不会释放，因此我们在其中访问到的 i 就分别是不同的闭包实例，这个实例是在循环体执行的过程中创建的，保留了不同的值。

然而，这种写法并不常见，大多数情况下我们可以使用数组的foreach方法解决问题.（当然let的使用也可以解决问题）。

```js
var fs = require('fs');
var files = ['a.txt', 'b.txt', 'c.txt'];
files.forEach(function(filename) {
    fs.readFile(filename, 'utf-8', function(err, contents) {
        console.log(filename + ': ' + contents);
    });
});
```

#### 3.2 部署优化

- 1.日志功能
- 2.使用cluster模块，充分利用多核CPU的资源
- 3.启动脚本
- 4.共享端口，反向代理等等