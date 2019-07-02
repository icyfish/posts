---
title: 浏览器执行解析的过程
date: 2019-07-03
tags:
  - Browser
---

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


### DNS prefetching

```html
<link rel="dns-prefetch" href="//example.com">
```

在head中插入以上代码之后, 浏览器会预先执行网站 example.com 的 DNS 查询, 后续引用的资源如果是从 `example.com` 获取时, 就不需要再在 DNS 查询这一步上耗费多余的时间. 

[Front-end performance for web designers and front-end developers](https://csswizardry.com/2013/01/front-end-performance-for-web-designers-and-front-end-developers/#section:dns-prefetching)

### Preconnect

```html
<link rel="preconnect" href="https://css-tricks.com">
```

执行了 DNS 预先查询之后, 还会建立 TCP 握手连接, TLS 协商. 

[Eliminating Roundtrips with Preconnect](https://www.igvita.com/2015/08/17/eliminating-roundtrips-with-preconnect/)

```js


```

