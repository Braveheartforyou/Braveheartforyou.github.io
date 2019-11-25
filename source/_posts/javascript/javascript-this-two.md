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
- bind、call、apply绑定
- new绑定、Object.create()绑定
- 箭头函数

我们都知道`this`的值是在运行时绑定的，并且谁调用它，它指向谁。下面我们就开始一步一步的了解`this`绑定的细节和实现。

### 默认绑定

**默认绑定**其实就是在**全局中声明函数**，并且在**全局中调用函数**，这样它不会受到任何调用对象和修饰符的干扰。代码如下：

```js
    function defalutsFunc () {
        console.log(this);
    }
    defalutsFunc(); // windows

    function defalutsFuncTwo () {
        // use strict
        console.log(this);
    }
    defalutsFuncTwo(); // undefined
```

在普通模式下，如果我们是在浏览器端运行代码，`this`指向`window`。它在**严格模式**下`this`会是`undefined`。

### 隐式绑定

**隐式绑定**，其实就是当前调用函数的`this`会指向当前调用该函数的执行上下文。隐式绑定`this`是不可靠的，他会因为调用者的不同而不同。

```js
    function globalFunc () {
        console.log(this.name);
    }

    var scopeObj = {
        name: 'scopeObj',
        scopeFunc: globalFunc
    }

    var name = 'globalFunc';

    scopeObj.scopeFunc(); // scopeObj
    globalFunc(); // globalFunc
```

在`scopeObj.scopeFunc()`我们其实是在`scopeObj`这个`作用域`中调用`scopeFunc`，这样`scopeFunc`的`this`指向`scopeObj`创建的上下文。 但是**隐式绑定**在传递过程中会丢失`this`，其实还是看调用它的执行上下文是那个，它的`this`就会指向当前**执行上下文**。修改代码如下：

```js
    var funcObj = scopeObj.scopeFunc;
    funcObj(); // globalFunc
```

我们又把`scopeObj.scopeFunc`赋值给了一个普通变量`funcObj`，在全局作用域中调用了赋值的这个`funcObj`，所以`funcObj`的this指向了`全局`的执行上下文。

> 其实`参数`也是一样的效果。

### 显示绑定

因为隐式绑定的丢失问题，所以有了后面的显示绑定`call`、`apply`、`bind`等等的方式。

我们可以通过`call`、`apply`它们可以显示的改变`this`的绑定.

- `call`: `fun.call(thisArg, arg1, arg2, ...)`第一个参数`this`要绑定的值，后面多个参数是要传入方法的参数。
- `apply`: `func.apply(thisArg, [argsArray])` 第一个参数`this`要绑定的值，可选的。一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 `func` 函数。

```js
    function globalFunc (age) {
        console.log(this.name, age);
    }

    var scopeObj = {
        name: 'scopeObj'
    }
    var name = 'globalFunc';

    globalFunc.call(scopeObj, 18); // scopeObj, 18
    globalFunc.apply(scopeObj, [18]); // scopeObj, 18

    globalFunc(18); // globalFunc, 18
```

但是即使通过`call`or`apply`改变的`this`值也是会丢失的，在传递的过程中，其实通过显示绑定也并不能保证我们的`this`一直是绑定的一个值。