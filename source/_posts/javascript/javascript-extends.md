---
title: javascript中实现一个自己的new
date: 2019-06-16 10:55:36
tags: [JavaScript]
categories: [JavaScript]
description: 首先我们要知道new 关键字做了什么事情，才能写一个自己的new方法。
---
## 简述
传统的javascript中只有对象，没有类的概念。它是基于原型的面向对象语言。原型对象特点就是将自身的属性共享给新对象。
先看一下new关键字是他都做了什么，让我们通过new实现继承的。
下面请看代码：
```javascript
function Preson (name, age) {
    this.name = name;
    this.age = age;
}

```
