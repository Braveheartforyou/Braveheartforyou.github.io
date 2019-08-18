---
title: Array中的forEach方法可以用break、continue跳出循环？
date: 2019-08-17 09:24:31
tags: [ECMAScript6]
categories: [ECMAScript6]
description: Array中的forEach方法可以用break、continue跳出循环？
---
## 简介
在`Array.prototype`上有很多方法，比较常用的就是`every`、`filter`、`forEach`、`map`、`some`这些循环方法，可以通过`break`、`comtinue`、`return`跳出循环？
现在基本上都是通过`forEach`、`every`来代替`for`循环，`for`循环可以通过`break`、`comtinue`、`return`跳出循环。而forEach可以不可以呢，下面一步一步的验证一下。