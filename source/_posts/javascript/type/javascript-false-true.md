---
title: JavaScript数据类型（五）== 混乱的转换规则
date: 2019-02-26 17:00:31
tags: [JavaScript, Type]
categories: [JavaScript]
description: 在JavaScript中==和===的区别,为什么![] == []为true, [] == false为true, ![] == false为true, !![] == false为false.
---

**_察见渊鱼者不详，智料隐匿者有殃。——《列子·说符》_**

[JavaScript 数据类型（一） 常见数据类型](/blog/javascript/javascript-Type-conversion.html)
[JavaScript 数据类型（二） 类型转换](/blog/javascript/javascript-type-one-question.html)
[JavaScript 数据类型（三）常见的面试题](/blog/javascript/javascript-type-one-questionone.html)
[JavaScript 数据类型（四）IF 转换规则](/blog/javascript/javascript-IF-False-options.html)
[JavaScript 数据类型（五）== 混乱的转换规则](/blog/javascript/javascript-false-true.html)
[JavaScript 数据类型（六）多种数据类型判断方法](/blog/javascript/javascript-bool-type.html)

## 简介

---

JavaScript 有两种比较方式：**严格比较运算符**和**转换类型比较运算符**。对于严格比较运算符（===）来说，仅当两个操作数的**类型相同且值相等**为 `true`，而对于被广泛使用的比较运算符（==）来说，会在进行比较之前，将两个操作数**转换成相同的类型**。

比较的特点:

- **对于两个拥有相同字符顺序，相同长度，并且每个字符的位置都匹配的字符串，应该使用严格比较运算符。**
- **对于两个数值相同的数字应该使用严格比较运算符，NaN 和任何值不相等，包括其自身，正数零等于负数零。**
- **对于两个同为 true 或同为 false 的布尔操作数，应使用严格比较运算符。**
- **不要使用严格比较运算符或比较运算符来比较两个不相等的对象。**
- **当比较一个表达式和一个对象时，仅当两个操作数引用相同的对象（指针指向相同对象）。**
- **对于 Null 和 Undefined 类型而言，应使用严格比较运算符比较其自身，使用比较运算符进行互相比较。**

[](https://www.h5jun.com/post/why-false-why-true.html)

### 一致/严格相等 (===)

一致运算符不会进行类型转换，仅当操作数**严格相等**时返回`true`

### 相等(==)

比较操作符会为两个不同类型的操作数**转换类型**，然后进行**严格比较**。当两个操作数都是对象时，JavaScript 会比较其**内部引用**，当且仅当他们的引用指向内存中的相同对象（区域）时才相等，即他们在栈内存中的**引用地址相同**。

非严格比较操作符 == 是会做强制类型转换的，那么根据 ECMA 262 它的规则是：

**图 1-1**
![! ==](../../images/false-true/1.png)
**<font color="#ff502c">ToPrimitive：</font>**

**图 1-2**
![! ==](../../images/false-true/2.png)

**图 1-3**
![! ==](../../images/false-true/3.png)

**图 1-4**
![! ==](../../images/false-true/4.png)
**<font color="#ff502c">ToBoolean: </font>**

**图 1-5**

![! ==](../../images/false-true/5.png)
**图 1-6**
![! ==](../../images/false-true/6.png)

[ecma 规则](http://www.ecma-international.org)

## [] == false or ![] == [] or ![] == false 为 true

mdn 运算符优先级参考表
![! ==](../../images/false-true/false-true-1.png)
![! ==](../../images/false-true/false-true-2.png)

<font color="#ff502c">==</font>的优先级 16
<font color="#ff502c">!</font>的优先级 10

### [] == false 结果为 true

根据图 1-1 可知

- 第 7 条：**If Type(y) is Boolean, return the result of the comparison x == ToNumber(y).**
- 第 9 条：**If Type(x) is Object and Type(y) is either String, Number, or Symbol, return the result of the comparison ToPrimitive(x) == y.**

所以 `[] == false` 的比较是对 x 执行 `ToPrimitive(x)`，然后和 `ToNumber(false)` （为 0）进行比较。

**看一下 ToPrimitive：**
根据上图 1-2、1-3、1-4 的规则对于`ToPrimitive([])`，先执行`[].valueOf()`，返回 result 的是'[]'，因为 Type(result)是 Object，所以继续执行`[].toString()`，返回””。
因此实际上最终是比较`"" == 0`，结果为`true`。

### ![] == [] 结果为 true

按照优先级，先执行 ![]，根据规范，实际上是 `!(ToBoolean([]))`：
根据上图 1-5、1-6 可看出，实际上 `ToBoolean([])` 会`return`出`true`, `![]` 就是 `false`.
[] 上文已经讲过了 是 ""。
所以对比就是 `false == ""`，结果为`true`。

### ![] == false 结果为 true

按照优先级，先执行 `![]`，根据规范，实际上是 `!(ToBoolean([]))`：
根据上图可看出，实际上 `ToBoolean([])` 会 return 出 true, ![] 就是 false.
`false == false` ，结果为`true`。

### !![] == false 结果为 false

按照优先级，先执行 !![]，根据规范，实际上是 `!!(ToBoolean([]))`：
根据上图可看出，实际上 `ToBoolean([])`会 return 出 true, !![] 就是 true.
`true == false` ，结果为 `false`。

## 总结

基本上一个**对象转为原始值**的大致过程再进行对比，如果不太清楚可以看另一篇博客[JavaScript 数据类型（二） 类型转换](/blog/javascript/javascript-type-one-question.html)，但是在本篇博客中最重要的时`ToBoolean([])`的转换比较不容易理解。
