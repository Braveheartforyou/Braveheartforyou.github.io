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

## Object

**Object** 构造函数创建一个对象包装器。**JavaScript**中的所有对象都来自 `Object`；所有对象从`Object.prototype`继承方法和属性，尽管它们可能被覆盖。

***Object 作为构造函数时，其 [[Prototype]] 内部属性值指向 Function.prototype***

```javascript
    Object.__proto__ === Function.prototype; // true
```

![JavaScript-prototype](../../images/javascript/javascript-prototype-1-9.png)

### Object.prototype

`Object.prototype` 表示 `Object` 的原型对象，其 `[[Prototype]]` 属性是 `null`，访问器属性 `__proto__` 暴露了一个对象的内部 `[[Prototype]]` 。`Object.prototype`是浏览器底层根据 ECMAScript 规范创造的一个对象。

### object

通过**字面量**实例化一个`object`，它的`__proto__`指向`Object.prototype`。

```javascript
    var obj = {};
    obj.__proto__ === Object.prototype; // true
```

而通过**new Object**实例化一个`object`，它的`__proto__`指向`Object.prototype`。

```javascript
    var obj = new Object;
    obj.__proto__ === Object.prototype; // true
```

## Function

[摘录来自ECMAScript 5.1规范](http://www.ecma-international.org/ecma-262/5.1/#sec-15.3.4)
> 对象类型的成员，标准内置构造器 Function的一个实例，并且可做为子程序被调用。
> 注： 函数除了拥有命名的属性，还包含可执行代码、状态，用来确定被调用时的行为。函数的代码不限于 ECMAScript。

**Function构造函数**创建一个新的`Function`对象。在**JavaScript**中每个函数实际上都是一个`Function`对象。

### Function.prototype

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

```javascript
    Function.prototype.__proto__ === Object.prototype; // true
```

### Function.__proto__

`Function` 构造函数是一个函数对象，其 `[[Class]]` 属性是 `Function`。`Function` 的 `[[Prototype]]` 属性指向了 `Function.prototype`，即

```javascript
    Function.__proto__ === Function.prototype; // true
```

![JavaScript-prototype](../../images/javascript/javascript-prototype-1-8.png)

### function

实例化一个`Function`，它的`__proto__`指向`Function.prototype`。

```javascript
    function foo () {}
    foo.__proto__ === Function.prototype; // true
    // foo.__proto__ => Function.prototype => Function.prototype.__proto__ => Object.prototype => Object.prototype.__proto__ => null
```

## Object和Function的鸡和蛋的问题

经过上面对`Object`和`Function`的阐述，延伸出来几个问题如下：

- 在忽滤`null`在原型链上时，原型链的尽头（root）是`Object.prototype`。所有对象均从`Object.prototype`继承属性。
![JavaScript-prototype](../../images/javascript/javascript-prototype-1-10.png)

- `Function.prototype`和`Function.__proto__`为同一对象。
![JavaScript-prototype](../../images/javascript/javascript-prototype-1-11.png)
这意味着： `Object/Array/String`等等**构造函数**本质上和`Function`一样，均继承于`Function.prototype`。

第一个问题不需要太多的解释，但是第二个问题`Function.prototype === Function.__proto__; // true`比较重要。其实可以这么