---
title: JavaScript类型转换（一） 常见数据类型
date: 2018-08-19 19:31:43
tags: [JavaScript, Type]
categories: [JavaScript]
description: JavaScript中的类型介绍
---

**_莫逆于心，遂相与为友。——庄子_**

[JavaScript 数据类型（一） 常见数据类型](/blog/javascript/javascript-Type-conversion.html)
[JavaScript 数据类型（二） 类型转换](/blog/javascript/javascript-type-one-question.html)
[JavaScript 数据类型（三）常见的面试题](/blog/javascript/javascript-type-one-questionone.html)
[JavaScript 数据类型（四）IF 转换规则](/blog/javascript/javascript-IF-False-options.html)
[JavaScript 数据类型（五）== 混乱的转换规则](/blog/javascript/javascript-false-true.html)
[JavaScript 数据类型（六）多种数据类型判断方法](/blog/javascript/javascript-bool-type.html)

## 简述

---

JavaScript 中的内置类型，七中类型中的又分为两大类：**基本类型（值类型）和引用类型**
基本类型有六种：**null**、**number**、**string**、**undefined**、**boolean**、**symbol**
引用类型：**object**
所有基本类型的值都是**不可改变**的。但需要注意的是，基本类型本身和一个赋值为基本类型的变量的区别。
变量会被赋予一个新值，而原值不能像数组、对象以及函数那样被改变。

**基本类型** 是储存在`栈`中
**引用类型** 引用数据类型的`值`是保存在`堆内存`中的对象。JavaScript`不允许`直接访问堆内存中的位置，堆内存中的`值地址引用`保存在`栈`中，我们都是操作`栈中`的地址引用。

**如果想看数据结构如堆、栈、链表等等，可以在本站搜索。**

如下图所示：

<img src="../../images/javascript/javascript-type.png" alt="javascript-type" width="60%" style="margin: 0 auto;"/>

如果不知道怎么判断数据类型的请看另一篇文章 [JavaScript 类型判断](/blog/javascript/javascript-bool-type.html)

### 基本类型

像基本类型如果**String、Boolean、Number**我们就不细写了，讲一下里面比较特殊的。如 **null、undefined、NaN、symbol**等等。

#### null

值`null`是一个字面量，他不像**undefined**是一个**全局对象**的一个**属性**。**null**是表示缺少的标识，知识变量未被指向任何对象，也可以看做是一个**空指针对象**。

如果我们在浏览器中赋值 **null**,他会报错如下：

```javascript
// chrome Google Chrome 已是最新版本
// 版本 75.0.3770.100（正式版本） （64 位）
null = '111';
// Uncaught ReferenceError: Invalid left-hand side in assignment
```

#### undefined

**undefined**它是一个 JavaScript**基本类型**。它也是一个**全局的属性**undefined 表示 undefined，undefined 也可以表示一个被**声明**没有被**赋值**的**变量**。

| undefined 属性的属性特性： |       |
| :------------------------: | :---: |
|          writable          | false |
|         enumerable         | false |
|        configurable        | false |

在现代浏览器（JavaScript 1.8.5/Firefox 4+），自 ECMAscript5 标准以来 undefined 是一个不能被配置（non-configurable），不能被重写（non-writable）的属性。即便事实并非如此，也要避免去重写它。

如果我们在浏览器中赋值 **undefined**, 因为他的 writable 是为 false,所以我们的赋值没有生效，如下所示：

```javascript
// chrome Google Chrome 已是最新版本
// 版本 75.0.3770.100（正式版本） （64 位）
undefined = '111';
undefined === undefined; // true
```

#### NaN

**全局属性** NaN 的值表示**不是**一个数字（Not-A-Number），`NaN`是一种特殊的`Number`类型.
`NaN` 属性的初始值就是 `NaN`，和 `Number.NaN` 的值一样。在现代浏览器中（ES5 中），`NaN` 属性是一个不可配置（non-configurable），不可写（non-writable）的属性。但在 ES3 中，这个属性的值是可以被更改的，但是也应该避免覆盖。

| **NaN** 属性的属性特性： |       |
| :----------------------: | :---: |
|         writable         | false |
|        enumerable        | false |
|       configurable       | false |

如果我们在浏览器中赋值 **NaN**, 因为他的`writable`是为`false`,所以我们的赋值没有生效，如下所示：

```javascript
// chrome Google Chrome 已是最新版本
// 版本 75.0.3770.100（正式版本） （64 位）
NaN = '111';
NaN === NaN; // false
```

**判断 NaN**
我们必须使用 **Number.isNaN()** 或 **isNaN()** 函数和比较中**不等于**它**自己**来判断是否为 NaN，为什么 NaN 不等于 NaN 自己，这是因为 NaN 它使表示一个**集合**

```javascript
NaN == NaN; // false
NaN === NaN; // false
isNaN(NaN); // true
Number.isNaN(NaN); // true
```

#### symbol

这个技术术语页面同时描述了一种称为 “**symbol**” 的数据类型，还有一个像类的函数 “**Symbol()**”，用来创建 **symbol** 数据类型实例。

数据类型 “**symbol**” 是一种原始数据类型，该类型的性质在于这个类型的值可以用来创建匿名的对象属性。该数据类型通常被用作一个对象属性的键值——当你想让它是私有的时候。
`Symbol` 是 JavaScript 的 原始数据类型 ，Symbol 实例是唯一且不可改变的.
符号类型是唯一的并且是不可修改的, 并且也可以用来作为 Object 的 key 的值(如下). 在某些语言当中也有类似的原子类型(Atoms).  
我们只能通过`Symbol('ssss')`声明`symbol`，不能通过`new`声明`Symbol`.

### 引用类型

#### 对象

在计算机科学中, 对象是指内存中的可以被 标识符引用的一块区域.
在 Javascript 里，对象可以被看作是一组属性的集合。用对象字面量语法来定义一个对象时，会自动初始化一组属性。
ECMAScript 定义的对象中有两种属性：数据属性和访问器属性。

**数据属性**
数据属性的特性(Attributes of a data property)

|       特性       |       数据类型       |                                                描述                                                |  默认值   |
| :--------------: | :------------------: | :------------------------------------------------------------------------------------------------: | :-------: |
|    [[Value]]     | 任何 Javascript 类型 |                                       包含这个属性的数据值。                                       | undefined |
|   [[Writable]]   |       Boolean        |                      如果该值为 false，则该属性的 [[Value]] 特性 不能被改变。                      |   true    |
|  [[Enumerable]]  |       Boolean        |                       如果该值为 true，则该属性可以用 for...in 循环来枚举。                        |   true    |
| [[Configurable]] |       Boolean        | 如果该值为 false，则该属性不能被删除，并且 除了 [[Value]] 和 [[Writable]] 以外的特性都不能被改变。 |   true    |

还有一些过时的数据属性**Read-only**、**DontEnum**、**DontDelete**
**访问器属性**

|       特性       |        数据类型        |                                                描述                                                |  默认值   |
| :--------------: | :--------------------: | :------------------------------------------------------------------------------------------------: | :-------: |
|     [[Get]]      | 函数对象或者 undefined |              该函数使用一个空的参数列表，能够在有权访问的情况下读取属性值。另见 get。              | undefined |
|     [[Set]]      | 函数对象或者 undefined |                            该函数有一个参数，用来写入属性值，另见 set。                            | undefined |
|  [[Enumerable]]  |        Boolean         |                       如果该值为 true，则该属性可以用 for...in 循环来枚举。                        |   true    |
| [[Configurable]] |        Boolean         | 如果该值为 false，则该属性不能被删除，并且 除了 [[Value]] 和 [[Writable]] 以外的特性都不能被改变。 |   true    |

> - 注意：这些特性只有 JavaScript 引擎才用到，因此你不能直接访问它们。所以特性被放在两对方括号中，而不是一对。

#### "标准的" 对象, 和函数

**对象**
一个 Javascript 对象就是键和值之间的映射.。键是一个字符串（或者 Symbol） ，值可以是任意类型的值。 这使得对象非常符合 哈希表。
**函数**
函数是一个附带可被调用功能的常规对象。
**日期**
当你想要显示日期时，毋庸置疑，使用内建的 Date 对象。
**数组和类型数组**
数组是一种使用整数作为键(integer-key-ed)属性和长度(length)属性之间关联的常规对象。
Int8Array、Uint8Array、Uint8ClampedArray 、Int16Array、Uint16Array、Int32Array、Uint32Array、Float32Array、Float64Array
**键控集: Maps, Sets, WeakMaps, WeakSets**
Map, Set, WeakMap, WeakSet

## 动态类型

JavaScript 是一种**弱类型**或者说**动态**语言。这意味着我们不用提前声明变量的类型，在程序运行过程中，类型会被自动确定。

```javascript
var one = 'one'; // one is a String now
one = 456; // one is a Number now
```

## 总结

JavaScript 是一种**弱类型**或者说**动态**语言。
JavaScript 中的内置类型，七中类型中的又分为两大类：基本类型（值类型）和引用类型

基本类型有六种：**null**、**number**、**string**、**undefined**、**boolean**、**symbol**
引用类型：**object**

所有基本类型的值都是**不可改变**的。但需要注意的是，基本类型本身和一个赋值为基本类型的变量的区别。

**基本类型** 是储存在`栈`中

**引用类型** 引用数据类型的`值`是保存在`堆内存`中的对象。JavaScript`不允许`直接访问堆内存中的位置，堆内存中的`值地址引用`保存在`栈`中，我们都是操作`栈中`的地址引用。
