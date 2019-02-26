---
title: 为什么![] == []为true, [] == false为true,  ![] == false为true, !![] == false 为false
date: 2019-02-26 17:00:31
tags: [JavaScript]
categories: [JavaScript]
description: 在JavaScript中==和===的区别,为什么![] == []为true, [] == false为true, ![] == false为true, !![] == false为false.
---
## 简介
JavaScript 有两种比较方式：严格比较运算符和转换类型比较运算符。对于严格比较运算符（===）来说，仅当两个操作数的类型相同且值相等为 true，而对于被广泛使用的比较运算符（==）来说，会在进行比较之前，将两个操作数转换成相同的类型。
比较的特点:
- 对于两个拥有相同字符顺序，相同长度，并且每个字符的位置都匹配的字符串，应该使用严格比较运算符。
- 对于两个数值相同的数字应该使用严格比较运算符，NaN和任何值不相等，包括其自身，正数零等于负数零。
- 对于两个同为true或同为false的布尔操作数，应使用严格比较运算符。
- 不要使用严格比较运算符或比较运算符来比较两个不相等的对象。
- 当比较一个表达式和一个对象时，仅当两个操作数引用相同的对象（指针指向相同对象）。
- 对于Null 和 Undefined 类型而言，应使用严格比较运算符比较其自身，使用比较运算符进行互相比较。
### 一致/严格相等 (===)
一致运算符不会进行类型转换，仅当操作数严格相等时返回true
### 相等(==)
比较操作符会为两个不同类型的操作数转换类型，然后进行严格比较。当两个操作数都是对象时，JavaScript会比较其内部引用，当且仅当他们的引用指向内存中的相同对象（区域）时才相等，即他们在栈内存中的引用地址相同。
### [] == false or ![] == [] or ![] = [] 为true
#### [] == false
mdn运算符优先级参考表 > https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence#Table
