---
title: 浏览器执行解析的过程
date: 2019-07-03
tags:
  - Browser
---

[How Browser Works](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)


浏览器执行解析的过程

<!-- more -->

HTML 解析文档时, 遇到如下标签会先执行解析操作, 完成之后再开始解析文档.

```js
<html>
<head>
    <link rel="stylesheet" type="text/css" href="/style.css">
    <script type="text/javascript" src="/header.js"></script>
</head>
<body>
  <p>Text</p>
  <script type="text/javascript" src="/main.js"></script>
</body>
</html>
```

以上的 `script` 和 `link` 标签就属于会中断解析文档过程的标签类型.


## 优化JS的加载

[Prefetching, preloading, prebrowsing](https://css-tricks.com/prefetching-preloading-prebrowsing/)


### DNS prefetching

```html
<link rel="dns-prefetch" href="//example.com">
```

在head中插入以上代码之后, 浏览器会预先执行网站 example.com 的 DNS 查询, 后续引用的资源如果是从 `example.com` 获取时, 就不需要再在 DNS 查询这一步上耗费多余的时间. 

**More About Prefetch:** [Front-end performance for web designers and front-end developers](https://csswizardry.com/2013/01/front-end-performance-for-web-designers-and-front-end-developers/#section:dns-prefetching)

### Preconnect

```html
<link rel="preconnect" href="https://css-tricks.com">
```

Preconnect: 执行对某个域名的 DNS 预先查询之后, 建立 TCP 握手连接, TLS 协商. 

**More About Preconnect:** [Eliminating Roundtrips with Preconnect](https://www.igvita.com/2015/08/17/eliminating-roundtrips-with-preconnect/)

### Prefetching

```html
<link rel="prefetch" href="xxx.com/image.png">
```

以上标签会预先加载`xxx.com/image.png`的资源并且存在缓存中以便后续使用.

和 DNS 预先查询不同的是, 这种方式实际上已经执行了对资源的请求和加载. 但某些预先加载的行为会被浏览器忽略. 比如说, 弱网情况下浏览器不会去预先加载size比较大的字体文件. 