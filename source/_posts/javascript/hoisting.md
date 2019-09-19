---
title: JavaScript中的变量提升
date: 2019-03-21 11:32:13
tags: [JavaScript]
categories: [JavaScript]
description: JavaScript中的变量提升，一般人都能说出来一点，我这里总结一下自己比较常见的，值的声明提升，还有函数的声明提升
---

## 产生变量提升的原因

在 ES6 之前，JavaScript 没有块级作用域(一对花括号{}即为一个块级作用域)，只有全局作用域和函数作用域。变量提升即将变量声明提升到它所在作用域的最开始的部分。
在 JavaScript 代码运行之前其实是有一个编译阶段的。编译之后才是从上到下，一行一行解释执行。变量提升就发生在编译阶段，它把变量和函数的声明提升至作用域的顶端。（编译阶段的工作之一就是将变量与其作用域进行关联）。
我先分开介绍变量提升和函数提升到后面再放到一起比较。

### 变量提升

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

### 函数提升

和变量提升一样存在函数提升，但是他们还有点不同。如下代码

```javascript
console.log(a); // function a () {console.log(1);}
function a() {
  console.log(1);
}
```

他会提前声明并且赋值，还有一种函数声明方式，代码如下

```javascript
console.log(a); // undefined
var a = function() {
  console.log(1);
};
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

为什么会输出 一个 <font color="#ff502c">1</font> 和一个<font color="#ff502c">undefined</font>,我来解释一下,首先 console.log(a())，他会<font color="#ff502c">先执行 a()</font>方法，<font color="#ff502c">输出 1</font>,但是 a 方法没有返回值，默认返回<font color="#ff502c">undefined</font>。

```javascript
console.log(a); // undefined
console.log(a()); // Uncaught TypeError: a is not a function
var a = function() {
  console.log(1);
};
```

这个看一下就行。

### 结合变量提升和函数提升

如果变量提升遇到函数提升，那个优先级更高呢，看下面的代码。

```javascript
console.log(a); // function a () {console.log(1);}
var a = 1;
function a() {
  console.log(1);
}
console.log(a); // 1
```

看上面的代码知道<font color="#ff502c">函数提升</font><font color="#ff502c">是<font color="#ff502c">高于变量提升</font>的，因为在 javascript 中函数是一等公民，<font color="#ff502c">并且不会被变量声明覆盖</font>，但是会被<font color="#ff502c">变量赋值覆盖</font>。其实代码如下

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
