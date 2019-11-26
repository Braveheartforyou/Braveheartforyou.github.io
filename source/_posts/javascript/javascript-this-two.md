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

我们可以通过在外层包裹一层函数来绑定`this`，示例代码如下：

```js
    function scopeFunc () {
        console.log(this.name, arguments);
    }

    var globalObj = {
        name: 'globalObj'
    };

    function simpleBind (fn, obj) {
        return function () {
            return fn.call(obj, arguments);
        }
    }

    var func = simpleBind(scopeFunc, globalObj);

    func(3); // globalObj Arguments [Arguments(1), callee: ƒ, Symbol(Symbol.iterator): ƒ]

    func.call(null, 3); // globalObj Arguments [Arguments(1), callee: ƒ, Symbol(Symbol.iterator): ƒ]
```

可以看到我们通过`simpleBind`中返回一个匿名函数，这样通过`call`或`apply`它也只能改变外部匿名函数的`this`，在**匿名函数内部**我们通过`fn.call(obj)`给方法默认绑定一个`this`，这个只是一个简单的`bind`实现。

也可以直接通过`Function.prototype.bind`来实现，`bind`返回一个硬绑定的函数。

```js
    function scopeFunc (args) {
        console.log(this.name, args);
    }
    var globalObj = {
        name: 'globalObj'
    };
    var func = scopeFunc.bind(globalObj);

    func(3); // globalObj 3

    func.call(null, 3); // globalObj 3
```

还有一种方式就是高阶函数，我们传入一下函数来获取当前执行上下文，比如`map`、`forEach`等等。

```js
    var aData = [{name: 'firstName', age: 18}, {name: 'lastName', age: 20}];

    aData.forEach(item => {
        console.log(item);
    });
```

它的内部也是使用了`call`或者`apply`来改变传入函数的`this`。如果有兴趣去看一下另一篇博客[Array常用的方法和实现reduce、map、filter、forEach](https://juejin.im/post/5d786452e51d4561eb0b2719)来深入了解一下。

### new绑定、Object.create()绑定

`new运算符`和`Object.create()`方法也是可以改变`this`的指向的。
首先我们要理解`new运算符`它具体做了什么操作，大致过程如下：

- 创建一个空的简单JavaScript对象（即`{}`）；
- 链接该对象（即设置该对象的构造函数）到另一个对象 ；
- 将步骤1新创建的对象作为`this`的上下文 ；
- 如果该函数没有返回对象，则返回`this`。

可以看到在`new 运算符`中有修改过`this`的指向，下面我们通过一个示例代码来了解一下。

```js
    function globalFunc () {
        this.name = 'globalFunc';
    }

    var exampleFunc = new globalFunc();

    console.log(exampleFunc); // globalFunc {name: "globalFunc"}
```

我们通过`new 运算符`调用`globalFunc`时，我们可以看到`globalFunc`中的`name`在`exampleFunc`中也可以访问到相同的`name`。

其实我们也可以通过`Object.create()`来也可以实现一样的效果，代码如下：

```js
    function globalFunc () {
        this.name = 'globalFunc';
    }

    var exampleFunc = Object.create(globalFunc);

    console.log(exampleFunc); // globalFunc {name: "globalFunc"}
```

其实`Object.create`内部和new有点类似，但是`Object.create`它调用的是`new Func()`用于生成一个对象实例。

### 箭头函数

**箭头函数外层没有普通函数，严格模式和非严格模式下它的this都会指向window(全局对象)**
**this对象的指向是可变的，但是在箭头函数中，它是固定的。**

普通函数与箭头函数的对比如下表所示：

|      对比     |      普通函数      |     箭头函数     |
|:------------:|:-------------:|:-------------:|
| `this`指向规则 |  `this`总是指向调用它的那个对象| 1.所有箭头函数本身没有`this` </br>2.箭头函数的this在定义的时候捕获自外层第一个普通函数的`this` </br> 3.如果箭头函数外层没有普通函数,严格模式和非严格模式下它的`this`都会指向`window`(全局对象) |
| 有无`prototype` |   有   | 箭头函数没有<font color="#ff502c">prototype</font>(原型) |
| 可否`new` |   可以   | 箭头函数作为匿名函数,是不能作为构造函数的(因为箭头函数没有`constructor`),不能使用new,不然会报错 |
| 有无`arguments` |   有   | 1.箭头函数的`this`指向全局,使用会报未声明的错误 </br> 2.箭头函数的`this`指向普通函数时,它的<font color="#ff502c">argumens</font>继承于改普通函数 |
| 可否`new` |   可以   | 箭头函数作为匿名函数,是不能作为构造函数的(因为箭头函数没有`constructor`),不能使用new,不然会报错 |
| 可否改变`this`指向 |   可以通过`call、apply、bind`改变`this`的指向  | 箭头函数本身的`this`指向不能改变,但是可以修改它要捕获的对象的`this` |

如果有兴趣的话可以去看另一篇[arrow-functions（箭头函数）和普通的函数的区别 this（二）](/blog/es6/es6-arrow-functions.html)

## 注意事项

### new 注意事项

`new`可以很方便构造调用一个函数并且声称一个实例，但是如果我们使用不太小心的话也会带来很多不必要的麻烦。

**忘记写new**运算符，那样我们就得不到我们想要的结果，如果实在全局环境中，那么**函数**的`this`会绑定到全局。如果实在**严格模式**`this`会绑定为`undefined`。

### bind/call、bind注意事项

把`null`或者`undefined`作为`this`的绑定对象传入`call、apply`或者`bind`，这些值在调用时会被忽略，实际应用的是默认规则。

### 软绑定

硬绑定可以把`this`强制绑定到指定的对象（`new`除外），防止函数调用应用**默认绑定规则**。但是会降低函数的灵活性，使用**硬绑定之后就无法使用隐式绑定或者显式绑定来修改**`this`。

如果给**默认绑定指定一个全局对象和undefined以外的值**，那就可以实现和硬绑定相同的效果，同时**保留隐式绑定或者显示绑定修改this的能力**。

```js
    if (!Function.prototype.softBind) {
        Function.prototype.softBind = function (obj) {
            var fn = this;

            var currArgs = Array.prototype.slice.call(arguments, 1);

            var bound = function () {
                return fn.apply()
            }
        }
    }
```