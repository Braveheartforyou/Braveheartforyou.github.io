---
title: JavaScript原型系列（三）Function、Object、null等等的关系
date: 2018-07-08 16:23:63
tags: [JavaScript, Prototype]
categories: [JavaScript]
description: JavaScript原型系列（三）Function.prototype、Object.prototype、null、Function.prototype.__proto__、Object.prototype.__proto__、function、object之间的关系
---
> [JavaScript原型系列（一）构造函数、原型和原型链](/blog/javascript/javascript-prototype.html)
> [JavaScript原型系列（二）什么是原型继承](/blog/javascript/javascript-prototype-one.html)
> [JavaScript原型系列（三）Function、Object、Null等等的关系](/blog/javascript/javascript-prototype-two.html)

## 简介

* * *
<img src="../../images/javascript/javascript-prototype-1-3.jpg" width="50%" alt="JavaScript-prototype"/>

基本上都知道原型链的尽头指向`null`，那么`Function.prototype`、`Object.prototype`、`null`、`Function.prototype.__proto__`、`Object.prototype.__proto__`、`function、object`之间的关系是什么，下面慢慢来记录一下。

## Function

[摘录来自ECMAScript 5.1规范](http://www.ecma-international.org/ecma-262/5.1/#sec-15.3.4)
> 对象类型的成员，标准内置构造器 Function的一个实例，并且可做为子程序被调用。
> 注： 函数除了拥有命名的属性，还包含可执行代码、状态，用来确定被调用时的行为。函数的代码不限于 ECMAScript。

**Function构造函数**创建一个新的`Function`对象。在**JavaScript**中每个函数实际上都是一个`Function`对象。

全局的`Function`对象**没有**自己的属性和方法, 但是因为它本身也是**函数**，所以它也会通过原型链从`Function.prototype`上继承部分属性和方法。`Function.prototype`也是一个“函数对象“，其`[[prototype]]`内部属性值指向`Object.prototype`。

Function.prototype 的 [[Class]] 属性是 Function，所以这是一个函数，但又不大一样。

```javascript
    Function.prototype; // ƒ () { [native code] }
    Function.prototype.prototype; // undefined
```

**用 Function.prototype.bind 创建的函数对象没有 prototype 属性。**

```javascript
    let foo = Function.prototype.bind();
    foo.prototype; // undefined
```

`Function.prototype` 是引擎创建出来的函数，引擎认为不需要给这个函数对象添加 `prototype` 属性，不然 `Function.prototype.prototype…` 将无休无止并且没有存在的意义。

`Function.prototype.__proto__`指向`Object.prototype`。

## function

