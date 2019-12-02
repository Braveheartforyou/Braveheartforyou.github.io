---
title: JavaScript中的this（三）自己实现绑定相关方法bind、call、apply、new、Object.create等等
date: 2018-12-13 23:43:35
tags: [JavaScript]
categories: [JavaScript]
description: 在上一篇文章中我们知道了多种绑定方式，这篇文章中我们自己实现bind、call、apply、new、Object.create等函数
---

## 简介

在上面我们已经了解了多种绑定方式，**默认绑定**、**隐式绑定**、**显示绑定**、**new绑定、Object.create绑定**、**箭头函数**。
我们本篇文章通过实现多种显示绑定的方法，来加深对`this`的指向理解和应对开发时不同的场景使用不同的绑定方法。其实还有一部分原因就是在面试的时候，一般来说一面都会笔试或者电话面试都会遇到，让你手写一个`bind/new/Object.create`等等。后面的文章会越来越深入的去理解`JavaScript`这门语言，等再后面的框架，架构能让我们更好的晋升。

## call、apply

