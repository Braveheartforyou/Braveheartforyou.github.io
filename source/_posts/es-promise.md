---
title: 实现一个自己的Promise
date: 2019-02-27 21:44:35
tags: [ECMAScript6]
categories: [ECMAScript6]
description: Promise是我们用来解决地狱回调，我在这篇博客中实现自己的一个Promise.
---
## 简介
一个 Promise 就是一个对象，它代表了一个异步操作的最终完成或者失败。大多数人仅仅是使用已创建的Promise实例对象，因此本教程将首先说明怎样使用 Promise，之后说明如何创建Promise。
本质上，Promise 是一个绑定了回调的对象，而不是将回调传进函数内部。
原生提供了Promise对象。本篇不注重讲解promise的用法，关于用法，可以看阮一峰老师的ECMAScript 6系列里面的Promise部分：
[ECMAScript 6 : Promise对象](http://es6.ruanyifeng.com/#docs/promise)

本篇博客逐步实现，最终使其符合Promises/A+规范
## 基础版本(异步回调)
目标
- 可以通过new 关键字创建一个 Promise实例。
- Promise实例传入的异步方法执行成功就执行注册的成功回调函数，失败就执行注册的失败回调函数。
代码实现
```javascript
function PromiseA (handle) {
  if (!)
  // const self = this;
  // sel
}
```