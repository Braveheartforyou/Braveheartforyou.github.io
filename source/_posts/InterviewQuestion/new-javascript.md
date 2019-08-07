---
title: javascript中new相关的面试题
date: 2019-08-03 23:13:45
tags: [InterviewQuestion, JavaScript]
categories: [InterviewQuestion]
description: 这一篇记录二个new相关的面试题，会详细分析答案
---
## 第一题
```javascript
    // 声明一个Foo 函数
    function Foo () {
        getName = function () {
            console.log(1);
        }
        return this;
    }
    // Foo创建了一个叫getName的静态属性存储了一个匿名函数
    Foo.getName = function () { console.log(2); };
    // Foo的原型对象新创建了一个叫getName的匿名函数
    Foo.prototype.getName = function () { console.log(3); };
    // 声明一个匿名，并且赋值给全局变量getName (函数表达式)
    var getName = function () { console.log(4); };
    // 声明一个全局函数getName (函数声明)
    function getName () { console.log(5); }

    Foo.getName(); // 第一问
    getName(); // 第二问
    Foo().getName(); // 第三问
    getName(); // 第四问
    new Foo.getName(); // 第五问
    new Foo().getName(); // 第六问
    new new Foo().getName(); // 第七问
```
**如果这几个问题能直接回答出来，后面就没必要看了**。

## 第一问

