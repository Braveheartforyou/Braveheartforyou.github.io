---
title: javascript中实现一个自己的new
date: 2019-06-16 10:55:36
tags: [JavaScript]
categories: [JavaScript]
description: 首先我们要知道new 关键字做了什么事情，才能写一个自己的new方法。
---
## 简述
传统的javascript中只有对象，没有类的概念。它是基于原型的面向对象语言。原型对象特点就是将自身的属性共享给新对象。
先看一下new关键字是他都做了什么，让我们通过new实现继承的。
下面请看代码：
```javascript
function Car(name, year, model) {
    this.name = name;
    this.year = year;
    this.model = model;
}
Car.prototype.getV = function () {
    return this.name + '--' + this.year + '--' + this.model;
}
var carOne = new Car('jeep', '2018', 'Wrangler');
console.log(carOne.getV());
console.log(carOne);
<font color="blue"></font>
```
new关键字进行如下的操作：
> 1. 创建一个空的JavaScript对象；
> 2. 链接该对象（即设置该对象的构造函数）到另一个对象；
> 3. 将步骤1新创建的对象作为this的上下文；
> 4. 如果该函数没有返回对象，则返回this。

同时，我们自己写的这个函数接收的<font color="blue">第一个参数</font>就是我们要<font color="blue">继承</font>的对象。
下面我们就一步一步实现一个自己的new 关键字
## 第一步
第一步比较简单我们要首先定义一个create方法在方法内创建一个<font color="blue">空对象</font>
```javascript
function create () {
    // 创建一个空对象
    let obj = new Object();
}
```
这个对象会在后面用到，经过后面的处理，如果<font color="blue">没有</font>返回值，就会<font color="blue">返回</font>我们创建的这个<font color="blue">空对象</font>。
## 第二步
第二步比较关键，用到了我们基于prototype继承的知识。就是把我们新创建的这个<font color="blue">空对象</font>的<font color="blue">__proto__</font>，指向我们要<font color="blue">继承对象</font>的<font color="blue">prototype</font>。
下面我们就在第一步代码的基础上实现
```javascript
    function create () {
        // 创建一个空对象
        let obj = new Object();
        // 获取第一个参数，构造函数
        let Preson = [].shfit.call(arguments);
        // 链接该对象（即设置该对象的构造函数）到另一个对象；
        obj.__proto__ = Preson.prototype;
    }
```
在这一步我们就是通过<font color="blue">__proto__</font>关联了我们创建的空对象的prototype到我们要继承的<font color="blue">另一个</font>对象。
## 第三步
第三步，将步骤1新创建的对象作为<font color="blue">this的上下文</font>，我们通过<font color="blue">apply执行构造函数</font>并且改变this指向。
下面我们在第二步的基础上实现
```javascript
    function create () {
        // 创建一个空对象
        let obj = new Object();
        // 获取第一个参数，构造函数
        let Preson = [].shfit.call(arguments);
        // 链接该对象（即设置该对象的构造函数）到另一个对象；
        obj.__proto__ = Preson.prototype;
        // 绑定this指向，执行构造函数
        let result = Preson.apply(obj, arguments);
    }
```
## 第四步
判断是否有返回值，如果该函数没有返回对象，则<font color="blue">返回this</font>。
```javascript
    function create () {
        // 创建一个空对象
        let obj = new Object();
        // 获取第一个参数，构造函数
        let Preson = [].shfit.call(arguments);
        // 链接该对象（即设置该对象的构造函数）到另一个对象；
        obj.__proto__ = Preson.prototype;
        // 绑定this指向，执行构造函数
        let result = Preson.apply(obj, arguments);
        
        return typeof result === 'object' ? result : obj;
    }
```
## 测试
```javascript
function create () {
    // 创建一个空对象
    let obj = new Object();
    // 获取第一个参数，构造函数
    let Preson = [].shift.call(arguments);
    // 链接该对象（即设置该对象的构造函数）到另一个对象；
    obj.__proto__ = Preson.prototype;
    // 绑定this指向，执行构造函数
    let result = Preson.apply(obj, arguments);
    
    return typeof result === 'object' ? result : obj;
}
function Car(name, year, model) {
    this.name = name;
    this.year = year;
    this.model = model;
}
var carOne = create(Car, 'jeep', '2018', 'Wrangler');
var carTwo = new Car('jeep', '2018', 'Wrangler');
console.log(carOne);
console.log(carTwo);
console.log(carOne.__proto__ === Car.prototype); // true
console.log(carTwo.__proto__ === Car.prototype); // true
```

**一道面试题**
一道有关new的面试题，其实一部还是考察运算符优先级。
```javascript
function Foo () {
    return this;
}
Foo.getName = function () {
    console.log(1);
};
Foo.prototype.getName = function () {
    console.log(2);
}
new Foo.getName();
new Foo().getName();
```
其实这个是考察运算优先级和new的面试题，通过mdn上的[运算优先级](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)
new Foo() 的优先级大于 new Foo ，所以对于上述代码来说可以这样划分执行顺序
```javascript
new (Foo.getName());
(new Foo()).getName();
```
对于第一个函数来说，先执行了 Foo.getName() ，所以结果为 1；对于后者来说，先执行 new Foo() 产生了一个实例，然后通过原型链找到了 Foo 上的 getName 函数，所以结果为 2。

> [mdn new 运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)
> [大佬 new的实现](https://yuchengkai.cn/docs/frontend/#new)
