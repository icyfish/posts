---
title: JavaScript Inside 2 - dig v8 engine
date: 2019-06-30
permalink: JavaScript-Inside-2
---

[How JavaScript works: inside the V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)

### 概述

JavaScript 引擎, 实际上是一个程序, 或者说 JavaScript 的解释器. A JavaScript engine can be implemented as a standard interpreter, or just-in-time compiler that compiles JavaScript to bytecode in some form.

下面是一些热门的 JavaScript 引擎:

- V8 -- Google用 C++ 开发的开源项目
- Rhino -- Mozilla Foundation 用 Java 开发的开源项目
- ...

### 为何开发了 V8 引擎

除了Chrome浏览器, Node.js 内部也使用了 V8 引擎. 

V8引擎设计的初衷是增强浏览器环境中 JavaScript 执行的性能. 为了速度考虑, V8 引擎直接将 JavaScript 代码编译为效率更高的机器码, 而不是使用解释器. V8引擎通过实现 **JIT(Just-In-Time)** 编译器将 JavaScript 代码在执行时直接编译成机器码, 许多其他的 JavaScript 引擎, 如 SpiderMonkey, Rhino 也是这样做的. 最主要的区别是: V8 引擎不生成 bytecode 或者 任何其他中间码.

### V8 引擎使用了两种编译器

在5.9版本的V8出现之前, 该引擎使用了两种编译器: 

    - full-codegen: 编译过程简单且高效, 但生成的机器码相对比较慢. 
    - Crankshaft: 编译过程相对复杂(Just-In-Time), 但生成的机器码较为高效.

V8引擎本身是多线程的:

    - 主线程获取, 编译, 执行代码
    - 其他线程在主线程执行时帮助编译优化代码
    - 调节统计程序运行状态的线程(Profiler Thread)告诉运行环境哪些方法耗时长, Crankshaft根据数据对相关方法进行优化
    - 还有一些线程处理垃圾收集

最开始执行 JavaScript 代码的时候, V8 引擎分配 **full-codegen** 直接将解析过的 JavaScript 翻译成机器码. 这一步使得机器码的执行迅速开始. V8 不使用中间码(intermediate bytecode), 因此解释器也没有必要存在.

当代码执行一段时间之后, profiler 线程收集到了足够的数据, 就能判断初哪些方法需要被优化.

之后, **Crankshaft** 开始在另一个线程中执行代码的优化, 它将 JavaScript 抽象的语法树编译成高级的静态单赋值形式(static single-assignment (SSA))的**Hydrogen**, 然后优化Hydrogen表. 大多数优化是在这一步实现的.

### Inlining

The first optimization is inlining as much code as possible in advance. Inlining is the process of replacing a call site (the line of code where the function is called) with the body of the called function. This simple step allows following optimizations to be more meaningful.

![-w606](/images/15618988743363.jpg)


#### Hidden class (隐藏类)
JavaScript 是基于原型的编程语言, 在克隆的阶段并没有类或对象生成. 同时, JavaScript 还是动态的编程语言, 这代表这在其实例化之后, 对象的属性可以被轻易地添加或删除.

大多数 JavaScript 解释器使用类字典的结构(基于哈希函数)来存储对象属性在内存中的位置. 相对其他的非动态编程语言(Java, C#)来说, 这种结构使得在 JavaScript 中获取属性值变得尤其费力. 在 Java 中, 所有的对象属性在被编译之前, 均由固定的对象布局决定, 而且不可以在执行时动态添加或删除属性. (与 C# 的区别是: C#是动态类型的, 但这又是另一个话题了). 这种机制的结果是, 属性的值(或者指向属性的指针) 能够在内存中被存储为有固定偏移量的连续 buffer. 偏移量的长度则基于属性的类型. 而这在 JavaScript, 是不可能实现的, 因为属性的类型在执行时有可能被改变.

由于使用字典方式在内存中查询对象属性的位置效率极低, V8使用了另一种方式: **hidden classes**. 这种方式与在 Java 中所使用的固定对象布局方式类似, 唯一的区别是, hidden classes是在运行时生成的. 现在我们来看一下实际的例子:

```js
function Point(x, y) {
    this.x = x;
    this.y = y;
}
var p1 = new Point(1, 2);
```

当 `new Point(1, 2)` 这段代码执行时, V8引擎会创建一个名为"C0"的隐藏类(hidden class).

![-w412](/images/15618999150250.jpg)

当前, Point对象没有属性, 因此"C0"是空的. 

一旦第一个赋值语句"this.x = x"被执行, V8 会基于第一个隐藏类"C0" 创建第二个隐藏类"C1". "C1" 描述了属性 x 相对于对象指针在内存中所在的位置. 此时, "x"被存储于 [offset](http://en.wikipedia.org/wiki/Offset_%28computer_science%29) 0 的位置. 



