---
title: var和let、const对比
date: 2019-08-06 12:03:54
tags: [ECMAScript6, InterviewQuestion]
categories: [ECMAScript6]
description: var和let、const对比
---

## 简介

<hr/>

在**ECMAScript6**中新增两个变量声明的指令`let`和`const`，以前经常用的`var`有什么区别。

## var 声明

<hr/>

`var` **声明语句**声明一个变量，并可选地将其初始化为一个值。

- 可重复声明
- 变量提升
- 声明变量的作用域限制在其声明位置的上下文中，而非声明变量总是全局的。

```javascript
var count = 1;
```

通过`var`关键字声明一个`count`变量，并且`count`的值为 1。

### 可重复声明

可以重复声明`count`变量，也可以重新赋值，代码如下：

```javascript
var count = 1;
var count = 2;
count = 3;
console.log(count); // 3
```

### 变量提升

`var`声明的变量是存在**变量提升**的 ，由于变量声明（以及其他声明）总是在任意代码执行之前处理的，所以在代码中的任意位置声明变量总是等效于在代码开头声明。代码如下：

```javascript
console.log(count); // undefined
var count = 1;
```

**提升将影响变量声明，而不会影响其值的初始化**。

### 作用域

`var`声明变量的作用域限制在其声明位置的**上下文中**，而非声明变量总是**全局**的。代码如下：

```javascript
function x() {
  var count = 1;
  sum = 2;
}
x();
console.log(count); // undefined
console.log(sum); // 2
console.log(window.sum); // 2
```

> - 建议始终声明变量，无论它们是在函数还是全局作用域内。

## let 声明

<hr/>

`let`允许你声明一个作用域被限制在 `块级中的变量`、`语句或者表达式`。与 `var` 关键字不同的是， `var`声明的变量只能是`全局`或者`整个函数块`的。

- 不可重复声明，可以重复赋值
- 块作用域
- 暂存死区
- 不存在变量提升

### 不可重复声明，可以重复赋值

**`let`不可以重复同样名称的变量，但是可以重复给同一个变量多次赋值**。代码如下：

```javascript
let count = 1;
let count = 2; // Identifier 'count' has already been declared
count = 2;
console.log(count); // 2
```

### 块作用域

`let`声明的变量只在其声明的**块或子块中**可用，这一点，与`var`相似。二者之间最主要的区别在于`var`声明的变量的作用域是**整个封闭函数**。

```javascript
function varTest() {
  var x = 1;
  if (true) {
    var x = 2; // 同样的变量!
    console.log(x); // 2
  }
  console.log(x); // 2
}

function letTest() {
  let x = 1;
  if (true) {
    let x = 2; // 不同的变量
    console.log(x); // 2
  }
  console.log(x); // 1
}
```

**let 声明**的变量不会挂载顶层对象下面，会临时创建一个`scope`来储存**let 声明**，执行完成清除，示例代码如下：

```html
<script>
  debugger;
  let count = 1;
  var sum = 2;
</script>
```

在执行完`let count = 1;`，效果如下图所示：
![let](../../images/es/es-let.png)
当**javascript**执行完成后，`scope`也会被清空。

### 暂存死区/不存在变量提升

`let` 被创建在包含该声明的（块）作用域顶部，**一般被称为“提升”**。与通过 `var` 声明的有初始化值 `undefined` 的变量不同，通过 `let` 声明的变量直到它们的定义被执行时才初始化。在变量初始化前访问该变量会导致 `ReferenceError`。该变量处在一个自块顶部到初始化处理的“暂存死区”中。
代码如下：

```javascript
console.log(count); // Uncaught ReferenceError: Cannot access 'count' before initialization
console.log(sum); // undefined
let count = 1;
var sum = 2;
```

## const 声明

<hr/>

常量是块级作用域，很像使用 `let` 语句定义的变量。常量的值**不能**通过重新赋值来改变，并且**不能**重新声明。

- 不能重复声明
- 不能重复赋值
- 块级作用域
- 暂存死区
- 不存在变量提升
- 一旦声明，必须马上赋值

`不能重复声明`、`块级作用域`、`块级作用域`、`不存在变量提升`在这里不再赘述和`let`中的表现相同，请看上文。

### 一旦声明，必须马上赋值

`const` 声明之后必须马上赋值，否则会报错，代码如下：

```javascript
    const count; // Uncaught SyntaxError: Missing initializer in const declaration
    const sum = 2;
```

### 不能重复赋值

`const`声明创建一个值的**只读引用**。但这并不意味着它所持有的值是**不可变**的，只是**变量标识符**不能重新分配。
**JavaScript**中的数据类型分为两大类：`值类型`、`引用类型`。

**值类型**

用`const`声明的变量被首次被赋值为**值类型**，它的值就不能改变了，代码如下：

```javascript
const count = 1;
count = 2; // Uncaught TypeError: Assignment to constant variable.
```

**引用类型**

只要不改变量的**指针地址**就不会报错，代码如下：

```javascript
const user = {
  name: 'nihao',
  age: 17
};
user.name = '大家好';
```

## 总结

总结对比如下面表格所示：

|   /   | 能否重复声明 | 能否重复赋值 |        作用域         |   变量提升   |    暂存死区    | 声明是否需要立即赋值 |
| :---: | :----------: | :----------: | :-------------------: | :----------: | :------------: | :------------------: |
|  var  |      能      |      能      | 函数作用域/全局作用域 | 存在变量提升 | 不存在暂存死区 |    不需要立即赋值    |
|  let  |     不能     |      能      | 块级作用域/全局作用域 |    不存在    |  存在暂存死区  |    不需要立即赋值    |
| const |     不能     |     不能     | 块级作用域/全局作用域 |    不存在    |  存在暂存死区  |     需要立即赋值     |

他们的特性基本上如上面表格所示，可以根据各个不同的需要，选择 var、let、const 来声明变量。

> [JavaScript 类型转换（一） 常见数据类型](http://asyncnode.com/blog/javascript/javascript-Type-conversion.html)

## 参考

> [var 声明语句](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/var) > [let 声明语句](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let) > [const 声明语句](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/const) > [JS 系列一：var、let、const、解构、展开、函数](https://mp.weixin.qq.com/s?__biz=MzUzNjk5MTE1OQ==&mid=2247483812&idx=1&sn=9bab06614e079bd9cc533a3b2cd02a75&chksm=faec857ccd9b0c6a9b58e49f747651ffdf484acdd6fc82318a0964e4c339dbda6586e340ca4d&mpshare=1&scene=1&srcid=082024th073paIFjxG2PXq8C&sharer_sharetime=1566313518851&sharer_shareid=491f5e3b572f21d39b90888df1c8829b#rd) > [ES6 中 let、const 与 var 的区别](https://mp.weixin.qq.com/s?__biz=Mzg4MjAyMzY1OQ==&mid=2247483768&idx=1&sn=328166a7f78be132b77215060e96128b&chksm=cf5c4cfcf82bc5ea327e302b81401165663f3ad91d35270ee4e537f7b27081cad4fecffc4776&mpshare=1&scene=1&srcid=0820mIdzuPo1JIpEQKyVpaGY&sharer_sharetime=1566313451138&sharer_shareid=491f5e3b572f21d39b90888df1c8829b#rd)
