---
title: JavaScript类型转换（三）常见的面试题
date: 2019-04-28 19:23:43
tags: [JavaScript]
categories: [JavaScript]
description: 我们在这篇中记录两个比较经典的面试题
---
## 简述

> [JavaScript类型转换（一） 常见数据类型](/blog/javascript/javascript-Type-conversion.html)
> [JavaScript类型转换（二） 类型转换](/blog/javascript/javascript-type-one-question.html)
> [JavaScript类型转换（三）常见的面试题](/blog/javascript/javascript-type-one-questionone.html)
> [JavaScript类型转换（四）IF 转换规则](/blog/javascript/javascript-IF-False-options.html)
> [JavaScript类型转换（五）== 混乱的转换规则 ](/blog/javascript/javascript-false-true.html)

在另一篇文章中有表述大致显示转化规则，隐式转换规则，着篇文章中会收集一些前端比较经典的关于类型转换的问题。下面就开始比较经典的面试题。
## ![] == []
这个题在另一篇面试中有记录过，[![] == []](/blog/javascript/javascript-false-true.html).
## (a == 1 && a == 2 && a == 3)
这个真的有很多种办法，这里只记录我知道的方法，如果有好的方法，请在下方留言，大家一起进步。
首先要知道==和===的区别。
**==**
宽松匹配 ==会先将左右两两边的值转化成相同的原始类型，然后再去比较他们是否相等。
**===**
他是不转化直接比较，如果类型不同直接就是false，如果类型相同，原始值相同就为true。

这里主要讲解(a == 1 && a == 2 && a == 3)相等的多种解法，如下：
1. 重写Object的valueOf
2. 重写Object的toString
3. 重写ToPrimitive，es6 smybol('')
4. 通过劫持obj的getter方法
5. 数组join、shift
6. 字符串骚操作

### 重写Object的valueOf、重写Object的toString
**重写代码的valueOf方法、和toString方法**，如果不知道ToPrimitive(obj, type)规则，请看[ToPrimitive规则](/blog/javascript/javascript-type-one-question.html)我的另一篇博客。代码如下：
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
### 重写ToPrimitive，es6 smybol('')
**在 ES6 之后，还允许对象通过显式指定 @@toPrimitive Symbol来覆盖原有的行为，得到我们想要的结果**。看代码：
```javascript
    const a = { value: 0 }
    a[Symbol.toPrimitive] = function () {
        return this.value += 1
    }
    console.log(a == 1 && a == 2 && a == 3); // true
```
### Obj.defineProperty
**通过劫持对象的属性值的getter操作**，让他累加来做到我们想要的。
```javascript
    var value = 0;
    Object.defineProperty(window, 'a', {
        get: function () {
            return this.value += 1;
        }
    })
    console.log(a == 1 && a == 2 && a == 3); // true
```
### 通过数组的方式实现
**通过改变数组的join方法实现**。
```javascript
    var a = [1, 2, 3];
    a.join = a.shift;
    console.log(a == 1 && a == 2 && a == 3); // true
```
### 字符串的骚操作
**通过定义变量是包含空格**，实现视觉上的相等。
```javascript
    var aﾠ = 1;
    var a = 2;
    var ﾠa = 3;
    console.log(aﾠ == 1 && a == 2 && ﾠa== 3 );
```
上面就是知道的可以让(aﾠ == 1 && a == 2 && ﾠa== 3 )的六种方法，如果还有后面会接着补充，下面来说一下(a === 1 && a === 2 && a === 3)。
## (a === 1 && a === 2 && a === 3)
在上面介绍过==与===的区别， ===是不会进行类型转换的，所以==的很多规则并不适用，那只有**劫持方法**和**字符串**的操作可以实现，其他方法都不能实现。
### Obj.defineProperty
**通过劫持对象的属性值的getter操作**，让他累加来做到我们想要的。
```javascript
    var value = 0;
    Object.defineProperty(window, 'a', {
        get: function () {
            return this.value += 1;
        }
    })
    console.log(a === 1 && a === 2 && a === 3); // true
```
### 字符串的骚操作
**通过定义变量是包含空格**，实现视觉上的相等。
```javascript
    var aﾠ = 1;
    var a = 2;
    var ﾠa = 3;
    console.log(aﾠ === 1 && a === 2 && ﾠa=== 3 );
```

未完待续。。。。