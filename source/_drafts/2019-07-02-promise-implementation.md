---
title: 70行代码实现Promise
date: 2019-07-02
tags:
  - JavaScript
---

[70 行代码实现 Promise](https://hackernoon.com/implementing-javascript-promise-in-70-lines-of-code-b3592565af0f)

<!--more-->

实现 Promise, 我们首先需要构造`new Promise()`, 我们的 Promise 对象, 关联了`resolve` 和 `reject` 方法. 将我们的 Promise 命名为 `Nancy`.

现在 Promise 执行的时候如果抛错:

```js
new Nancy(resolve => {
  throw new Error();
});
```

会被 `reject` 捕获. 我们还能执行如下方法:

```js
new Nancy(resolve => resolve(42));
```

此时 `this.state`的值会变成`states.resolved`, 与此同时, `this.value`的值会编程`42`. 现在我们定义一下`resolve`与`reject`:

定义一个高阶函数`getCallback`, 以减少重复代码. 现在`resolve(42)`结果如预期.

现在看看`then`是怎么实现的. `then`的作用是允许 Promise 被链式调用, 这意味这 `then` 方法需要返回一个 Promise. 我们首先实现`Nancy.resolve`与`Nancy.reject`语法糖, 用`Nancy.resolve`替代`new Nancy(resolve => resolve(42))`. 现在我们看看可以从

This allows us to write our new Nancy(resolve => resolve(42)) as Nancy.resolve(42). Now let’s see what we expect from then: