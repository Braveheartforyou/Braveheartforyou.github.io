---
title: JavaScript中的this（二）this的多种绑定方式
date: 2018-12-10 14:23:43
tags: [JavaScript]
categories: [JavaScript]
description: this的绑定
---

## 简介

在[上一篇文章](/blog/javascript/javascript-this-one.html)中我们记录了`执行栈`、`执行上下文`、`执行上下文生命周期`、`this的产生`等等，在这一篇文章中我们来记录一下`this的绑定`也就是`this`的值确定。
`this`在`创建阶段`被创建(确定默认值)，但是在`执行阶段`会改变`this`的值。所以一般我们都会说`确定this`是在`执行阶段`。

大致过程分为：

- 多种绑定`this`方式
- 改变`this`方式 `new`、`Object.create`
- 另外三种改变`this`的方式`bind`、`call`、`apply`
- 异类`箭头函数`

下面我们就慢慢开始一步一步了解`this`。

## 多种绑定`this`方式

无论是默认的绑定`this`的规则，还是后面改变`this`的方法，我们尽量深入的记录，大致目录如下：

- 默认绑定
- 显示绑定
- 隐式绑定
- new绑定
- Object.create()绑定
- 箭头函数
- bind、call、apply改变this


