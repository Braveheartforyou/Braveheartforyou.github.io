---
title: ES系列 Object.defineProperty和Proxy的对比
date: 2019-09-23 09:12:32
tags: [ECMAScript6]
categories: [ECMAScript6]
description: Object.defineProperty和Proxy分别是什么，他们之间的区别和优缺点，vue源码中的为什么把Object.defineProperty用proxy重写。
---

## 简介

在前两篇文章中分别介绍了`Object.defineProperty`和`Proxy`两个新的特性，其实看起来`Proxy`更像是对`Object.defineProperty`的一种补充和完善