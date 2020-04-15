---
title: JavaScript中的变量提升
date: 2019-03-21 11:32:13
tags: [JavaScript]
categories: [JavaScript]
description: 我们首先了解一下JavaScript中的变量提升，后面再来两道面试题加深自己的记忆。
---

## 产生变量提升的原因

在 ES6 之前，JavaScript `没有块级作用域(一对花括号{}即为一个块级作用域)`，大致分为`全局作用域`和`函数作用域`。变量提升即将变量声明提升到它所在`作用域`的`最开始`的部分。
在 JavaScript 代码运行之前其实是有一个`编译阶段`的。编译之后才是`从上到下`，一行一行解释执行。变量提升就发生在`编译阶段`，它把`变量`和`函数`的声明提升至作用域的顶端。（编译阶段的工作之一就是将变量与其作用域进行关联）。我先分开介绍变量提升和函数提升到后面再放到一起比较。

**`注意`**

1. 同一个变量只会`声明一次`，其它的会被覆盖掉。
2. `变量提升/函数提升`是提升到`当前作用域`的顶部，如果遇到特殊的`if(){}/try-cache`作用域，同时也会把也会提升到`特殊作用域`的外部。
3. `函数提升`的优先级是高于`变量提升`的优先级，并且`函数声明`和`函数定义`的部分一起被提升。

## 变量提升

我们直接从代码从最基础的开始

```javascript
console.log(a); // undefined
var a = 2;
```

相信这个大家知道，上面代码其实就是

```javascript
var a;
console.log(a); // undefined
a = 2;
```

他会提前声明 a,但是不会给 a 赋值。
但是如下代码会怎么执行呢？

```javascript
console.log(a); // Uncaught ReferenceError: a is not defined
a = 2;
```

如果没有通过 var 声明值类型的就不会存在变量提升，而是会报错。

## 函数提升

`声明函数`有两种方式: 一种是`函数表达式`，另一种是`函数声明`。

### 函数表达式

```js
  console.log(aa) // undefined
  var aa = function () {};

  /** 代码分解 ***/
  var aa;
  console.log(aa);
  aa = function () {};
```

`函数表达式`和`变量`的提升效果基本上是一致的，它会输出`undefined`。

### 函数声明

它和`函数表达式`是有点不一样的，在没有`{}作用域`时它们表现是一致的。表现一致的例子

```javascript
console.log(a); // function a () {}
function a() { };

/** 代码分解 ***/
function a() { };
console.log(a);
```

那如果`变量提升`和`函数提升`同时存在，谁先谁后呢? 我们根据上面的注意事项`1`和`3`可以得出结果，根据实例来分析一下。 请看下面的例子：

```js
  console.log(aa); // function aa () {}
  var aa = 'aaaa';
  function aa () {};
  console.log(aa); // aaaa

  /** 代码分解 ***/
  var aa; // 只会声明一次的变量
  function aa () {}; // 变量别覆盖为 aa 字面量函数
  console.log(aa); // function aa () {} 输出字面量函数
  aa = 'aaaa'; // aa 重新被覆盖为 'aaaa'
  console.log(aa); // aaaa 输出最后的覆盖值
```

其实我们可以通过`chrome`浏览器调试效果大致如下图所示：
![hosting-debugger](../../images/javascript/javascirpt-hosting-1.png)

到这里就大致知道`变量提升`、`函数提升`它们的大致过程和它们之间的`优先级`。下面我们来说一下它们和`块级作用域`和`函数作用域`的关系。

## 作用域

在`ES6`出现之后作用域变得很复杂，有太多种了，这里只说和本篇文章相关的几种作用域。我们只看`全局作用域`、`词法作用域`、`块级作用域`、`函数作用域`这四种作用域。
`全局作用域`基本上没什么好说的，上面的样例基本上都是`全局作用域`，这里就不做多的赘述。

### 词法作用域

词法作用域：`函数在定义它们的作用域里运行，而不是在执行它们的作用域里运行。`
我们直接通过一个例子来分析一下：

```js
  var aa 
  function bb () {}
```

`let`作用域
**`函数声明和作用域相关`**

在有`作用域`时，我们来看一下`函数声明`的表现，还是通过一个实例来分析一下，代码如下：

```js
  console.log(aa); // 如果直接输入 会报错 VM1778:1 Uncaught ReferenceError: a is not defined
```

下面修改代码来分析在`特殊的作用域`中`函数声明`的特殊表现。

```js
  console.log(aa); //这个会输出什么？
  if (true) {
    console.log(aa); //这个会输出什么？
    function aa () {};
  }
  console.log(aa); //这个会输出什么？
```

这种写法就和变量提升相同了。
还有一种我感觉要说一下，其实不仔细思考也很容易犯错。

```javascript
console.log(a());
function a() {
  console.log(1);
}
// 会输出如下：
// 1 这个 a function 输出的
// undefined 因为 a function没有 return出来任何值，默认return  undefined。
```

为什么会输出 一个 `1` 和一个`undefined`,我来解释一下,首先 console.log(a())，他会`先执行 a()`方法，`输出 1`,但是 a 方法没有返回值，默认返回`undefined`。

```javascript
console.log(a); // undefined
console.log(a()); // Uncaught TypeError: a is not a function
var a = function() {
  console.log(1);
};
```

这个看一下就行。

**`重要的属性`**


## 结合变量提升和函数提升

如果变量提升遇到函数提升，那个优先级更高呢，看下面的代码。

```javascript
console.log(a); // function a () {console.log(1);}
var a = 1;
function a() {
  console.log(1);
}
console.log(a); // 1
```

看上面的代码知道`函数提升`是`高于变量提升`的，因为在 javascript 中函数是一等公民，`并且不会被变量声明覆盖`，但是会被`变量赋值覆盖`。其实代码如下

```javascript
var a = function() {
  console.log(1);
};
var a;
console.log(a); // function a () {console.log(1);}
a = 1;
console.log(a); // 1
```

我们再来一个稍微复杂一点的，代码如下：

```javascript
console.log(a); // function a () {console.log(2);}
var a = 1;
function a() {
  console.log(1);
}
console.log(a); // 1
var a = 2;
function a() {
  console.log(2);
}
console.log(a); // 2
console.log(typeof a); // number
```

在多次函数提升的会后一个覆盖前一个，然后才是变量提升，其实代码如下：

```javascript
var a = function() {
  console.log(1);
};
var a = function() {
  console.log(2);
};
var a;
console.log(a); // function a () {console.log(2);}
a = 1;
console.log(a); // 1
a = 2;
console.log(a); // 2
console.log(typeof a); // number
```

基本上这个上面基本上包含了 javascript 的变量提升和函数提升。其实还有一个作用域的我在这边就不介绍了，我会开篇再好好说一下作用域。
