---
title: 运算符优先级
date: 2017-11-28 15:56:23
tags: [JavaScript]
categories: [JavaScript]
description: 在JavaScript中运算符是有明确的有限级的，如*运算符比+运算符的优先级高，如果想让+运算符的优先级超过*运算符，可以在+外层嵌套一个()符号，他的优先级就是高于*运算符的
---
## 简述
运算符的优先级决定了表达式中运算执行的先后顺序，优先级高的运算符最先被执行。
```javascript
    1 + 2 * 3 // 7
```
<font color="red"></font>
乘法运算符 ("<font color="red">*</font>")比起加法运算符("<font color="red">+</font>")有着更高的优先级，所以它会被最先执行。
### 结合性
结合性决定了拥有相同优先级的运算符的执行顺序。考虑下面这个表达式：
- 左结合
左结合(从左到右计算)相当于把左边的子表达式加上小括号(a OP b) OP c
- 右关联
(从右到左计算)相当于a OP (b OP c)
```javascript
    a = b = 5;
```
结果 a 和 b 的值都会成为5。这是因为赋值运算符的返回结果就是赋值运算符右边的那个值，具体过程是：b被赋值为5，然后a也被赋值为 b=5 的返回值，也就是5。
### 汇总表
下面的表将所有运算符按照优先级的不同从高到低排列。
可以查看 mdn中的 table来分他的等级。

mdn运算符优先级参考表 <https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence#Table>