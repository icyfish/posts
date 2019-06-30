---
title: JavaScript Inside 1 - engine, runtime, callstack
date: 2019-06-30
permalink: JavaScript-Inside-1
---

[How JavaScript works: an overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)

<!-- more -->

### JS引擎
JavaScript 引擎: 最出名的是 V8 引擎, Chrome 和 Node.js 内部使用的就是 V8 引擎. V8 引擎大概长这样:
![-w450](/posts/images/15618888972406.jpg)

从图片中可以看出, V8引擎由两部分组成: 

- 内存堆 -- 主要控制内存的分配
- 调用栈 -- 代码执行时栈是如何被构造的

### 执行环境 (The Runtime)

浏览器提供了一些API (如: setTimeout). 这些API不是由JS引擎提供的. 
![-w744](/posts/images/15618892125075.jpg)

因此, 编写浏览器环境下的JS代码时, 不仅依赖JS引擎, 还依赖 Web API (如 DOM, AJAX, setTimeout) 提供给我们的能力. 

于是, 我们需要 **Event Loop** 与 **callback queue**.

### 调用栈

JavaScript 是单线程的编程语言, 这意味着它只有一个调用栈, 因此一次只能做一件事. 

调用栈是一种数据结构, 记录了我们的程序执行到了哪一个位置. 当执行到某个函数的时候, 这个函数就位于执行栈的顶部. 从函数中`return`意味着该函数从执行栈顶部弹出. 下面看一个代码实例: 

```js
function multiply(x, y) {
    return x * y;
}
function printSquare(x) {
    var s = multiply(x, x);
    console.log(s);
}
printSquare(5);
```

当引擎开始执行上述代码时, 调用栈首先是空的, 之后的步骤则如下图所示: 

![-w686](/posts/images/15618896502672.jpg)


执行栈的每一个入口被叫做栈帧(**Stack Frame**).

当函数抛出错误时, 引擎会根据调用栈的顺序抛出错误:

```js
function foo() {
    throw new Error('SessionStack will help you resolve crashes :)');
}
function bar() {
    foo();
}
function start() {
    bar();
}
start();
```

![-w570](/posts/images/15618899992439.jpg)

爆栈指的是超出了调用栈可以接受的最大size. 这种场景很容易发生, 特别是当我们在代码中使用递归时:

```js
function foo() {
    foo();
}
foo();
```

当引擎开始执行这段代码时, `foo`函数被调用. 因为`foo`函数内部调用了本身, 同时并没有终止条件. 因此每一次`foo`被调用, 它就被加调用栈中, 超过一定次数之后就会爆栈.

![-w675](/posts/images/15618902608386.jpg)

单线程语言, 处理代码相对比较简单, 因为我们不需要处理多线程语言中才会出现的复杂情景, 比如: 死锁(deadlocks).

然而单线程带来的限制也比较多, 当在主线程的函数运行太慢怎么办呢? 

### 并发与事件循环(Concurrency & the Event Loop)

当调用栈中的函数执行事件太长, 阻住了下一个函数的执行, 怎么办呢? 比如, 当我们使用JavaScript在浏览器环境下执行复杂的图片转换程序.

你也许会问, 这哪里算是问题? 但实际情况是, 在执行图片转换程序时, 浏览器的主线程被阻塞, 无法处理其他事件. 这意味着浏览器进行渲染, 无法执行其他代码, 体验差到极致. 如果你对产品的流畅性有比较高的要求, 这就是个很大的问题了. 

当然这并不是唯一的问题. 一旦调用栈中有过多的任务需要执行, 会有很长一段时间, 浏览器无法处理用户交互. 大多数浏览器会采取相应的抛异常措施, 询问用户是否需要关闭网页. 

那么要如何解决这个阻塞的问题呢? 答案是**异步回调**.

