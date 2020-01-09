---
title: JavaScript中的this（三）自己实现绑定相关方法bind、call、apply、new、Object.create等等
date: 2018-12-13 23:43:35
tags: [JavaScript]
categories: [JavaScript]
description: 在上一篇文章中我们知道了多种绑定方式，这篇文章中我们自己实现bind、call、apply、new、Object.create等函数
---

## 简介

在上面我们已经了解了多种绑定方式，**默认绑定**、**隐式绑定**、**显示绑定**、**new绑定、Object.create绑定**、**箭头函数**。
我们本篇文章通过实现多种显示绑定的方法，来加深对`this`的指向理解和应对开发时不同的场景使用不同的绑定方法。其实还有一部分原因就是在面试的时候，一般来说一面都会笔试或者电话面试都会遇到，让你手写一个`call/apply/bind/new/Object.create`等等。后面的文章会越来越深入的去理解`JavaScript`这门语言，等再后面的框架，架构能让我们更好的晋升。

我们首先要实现一个`call/apply`，因为无论是后面的`bind`还是`new/Object.create`都会用到`call/apply`。

文章大致章节：

- `call/apply`: `call/apply`的使用场景，它们两个运行速度，谁更快一点，自己实现`call/apply`
- `bind`: `bind`的使用场景，实现一个`bind`
- `new/Object.create`: `new/Object.create`使用场景，分别实现他们

## call/apply

`call()/apply()` 方法使用一个指定的 `this` 值和单独给出的一个或多个参数来调用一个函数。

> 注意：该方法的语法和作用与 `apply()` 方法类似，只有一个区别，就是 `call()` 方法接受的是一个参数列表，而 `apply()` 方法接受的是一个包含多个参数的数组。

`call/apply`是怎么使用，简单使用如下：

```js
    var testFunc = function (name, age) {
        console.log(this.name);
        console.log(this.age);
    }

    testFunc.call({name: 'call', age: 20}, 'call', 20) // call、20
    testFunc.apply({name: 'apply', age: 20}, ['apply', 20]) // apply 20
```

## new和Object.create

`new 运算符`创建一个用户定义的`对象类型的实例`或具有构造函数的`内置对象的实例`。会在`new`的内部改变新生成的`对象实例`的`this`，我们可以通过下面的样例看一下：

```js
function Car(make, model, year) {
    this.make = make;
    this.model = model;
    this.year = year;
}

var car1 = new Car('Eagle', 'Talon TSi', 1993);

console.log(car1.make);
// expected output: "Eagle"

car1.__proto__ === Car.prototype // true
```

我们可以看到在通过`new`关键字生成的新的对象，它复刻了`Car`中声明了的多个变量，并且这些变量在`Car`中都是绑定在`Car`中的`this`上。而新生成的**实例对象**同时也拥有所有的属性。我们可以知道`new`运行的时候会新创建一个对象，这个对象包含所有`Car`中的`this`上的对象。



