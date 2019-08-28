---
title: JavaScript系列（一）构造函数、原型和原型链
date: 2018-07-01 23:12:43
tags: [JavaScript]
categories: [JavaScript]
description: JavaScript系列（一）构造函数、原型和原型链
---
## 简介
首先要了解几个属性`constructor`、`prototype`、`[[prototype]]`、`__proto__`分别作用是什么，还要理解几个概念**原型**、**原型链**、**构造函数**。
结合代码先解释一下概念，再解释属性的作用和用法。

## 构造函数、constructor

### 构造函数
`构造函数`本身就是一个函数，与普通函数`没有`任何区别，不过为了规范一般将其`首字母`大写。`构造函数`和`普通函数`的区别在于，使用 `new` 生成实例的函数就是`构造函数`，直接调用的就是`普通函数`。下面示例代码：
```javascript
    function ConstructorFun () {
        this.name = 
    }
```