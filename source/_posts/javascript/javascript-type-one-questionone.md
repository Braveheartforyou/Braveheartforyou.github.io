---
title: javascript的中的类型转换（三）
date: 2019-04-28 19:23:43
tags: [JavaScript]
categories: [JavaScript]
description: 我们在这篇中记录两个比较经典的面试题
---
## 简述
在另一篇文章中有表述大致显示转化规则，隐式转换规则，着篇文章中会收集一些前端比较经典的关于类型转换的问题。下面就开始比较经典的面试题。
## ![] == []
这个题在另一篇面试中有记录过，[![] == []](/blog/javascript/javascript-false-true.html).
## (a === 1 && a === 2 && a === 3)
这个真的有很多种办法，这里只记录我知道的方法，如果有好的方法，请在下方留言，大家一起进步。
首先要知道==和===的区别。
**==**
宽松匹配 ==会先将左右两两边的值转化成相同的原始类型，然后再去比较他们是否相等。
**===**
他是不转化直接比较，如果类型不同直接就是false，如果类型相同，原始值相同就为true。
这里主要讲解(a === 1 && a === 2 && a === 3)相等的多种解法，如下：
1. 重写Object的valueOf
2. 重写Object的toString
3. 重写ToPrimitive，es6 smybol('')
4. 通过劫持obj的getter方法
5. 数组join、shift
6. 字符串骚操作

### 重写Object的valueOf、重写Object的toString
重写代码的valueOf方法、和toString方法，如果不知道ToPrimitive(obj, type)规则，请看[ToPrimitive规则](/blog/javascript/javascript-type-one-question.html)我的另一篇博客。代码如下：
```javascript
const a = { value: 0 }
a.valueOf = function () {
    return  this.value += 1
}
console.log(a == 1 && a == 2 && a == 3); // true

const a1 = { value: 0 }
a1.valueOf = function () {
    return  {};
}
a1.toString = function () {
    return this.value += 1
}
console.log(a1 == 1 && a1 == 2 && a1 == 3); // true

```
###