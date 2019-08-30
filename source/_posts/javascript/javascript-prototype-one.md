---
title: JavaScript原型系列（二）什么是原型继承
date: 2018-07-04 09:42:23
tags: [JavaScript, Prototype]
categories: [JavaScript]
description: JavaScript原型系列（二）什么是原型继承
---
> [JavaScript原型系列（一）构造函数、原型和原型链](/blog/javascript/javascript-prototype.html)
> [JavaScript原型系列（二）什么是原型继承](/blog/javascript/javascript-prototype-one.html)
> [JavaScript原型系列（三）Function、Object、Null等等的关系](/blog/javascript/javascript-prototype-two.html)

## 简介

* * *
<img src="../../images/javascript/javascript-prototype-1-3.jpg" width="50%" alt="JavaScript-prototype"/>

在上一节上面介绍了原型和原型链，即每个对象拥有一个**原型对象**，通过 `__proto__` 指针指向上一个**原型** ，并从中**继承方法和属性**，同时原型对象也可能拥有原型，这样一层一层，最终指向 `null`，这种关系被称为`原型链(prototype chain)`。

`继承`是面向对象编程语的一大核心功能点，虽然`JavaScript`
