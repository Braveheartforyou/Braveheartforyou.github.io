---
title: 前端性能优化中为什么style放在顶部，javascript放在底部
date: 2018-10-15 17:43:09
tags: [Html]
categories: [Html]
description: 前端性能优化中为什么style放在顶部，javascript放在底部，内部原理讲解
---
## 引子
我们在看一些前端优化规则的时候，比如雅虎军规等等，都有看到style写在head中，但是外链script写在body的最后，以优化性能，都知道应该怎么做，但是不知道其中的原理。
如果还不知道浏览器渲染的原理的，看一看[浏览器渲染原理](http://asyncnode/blog/html/html-style-javascript.html)这一篇文章。其实这个就是考验大家对html中的css、javascript、dom之间的解析和相互阻塞关系。
### JavaScript会阻塞CSS、DOM吗？
当我们把script标签写到页面的顶部时，dom树在解析的时候检测到script标签是，会加载script里面的内容并且执行，他会阻塞dom的解析和渲染，阻塞css的解析和加载