---
title: arrow-functions（箭头函数）和普通的函数的区别
date: 2019-04-10 16:53:14
tags: [ECMAScript6]
categories: [ECMAScript6]
description: ECMAScript6中的箭头我们基本都是非常的常用，总结一下他与普通函数的区别和优点、确定。
---

## 简介

在用 vue 框架、或者 react 框架中我们会用到很多 es6 中的新特性，比较多的就是<font color="#ff502c">箭头函数</font>.都会知道一点普通函数和箭头函数的区别,这里总结一下**箭头函数和普通函数的区别**：

|        对比        |                  普通函数                   |                                                                                           箭头函数                                                                                            |
| :----------------: | :-----------------------------------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|   this 指向规则    |        this 总是指向调用它的那个对象        | 1.所有箭头函数本身没有 this </br>2.箭头函数的 this 在定义的时候捕获自外层第一个普通函数的 this </br> 3.如果箭头函数外层没有普通函数,严格模式和非严格模式下它的 this 都会指向 window(全局对象) |
|   有无 prototype   |                     有                      |                                                                   箭头函数没有<font color="#ff502c">prototype</font>(原型)                                                                    |
|      可否 new      |                    可以                     |                                                箭头函数作为匿名函数,是不能作为构造函数的(因为箭头函数没有 constructor),不能使用 new,不然会报错                                                |
|   有无 arguments   |                     有                      |                       1.箭头函数的 this 指向全局,使用会报未声明的错误 </br> 2.箭头函数的 this 指向普通函数时,它的<font color="#ff502c">argumens</font>继承于改普通函数                        |
|      可否 new      |                    可以                     |                                                箭头函数作为匿名函数,是不能作为构造函数的(因为箭头函数没有 constructor),不能使用 new,不然会报错                                                |
| 可否改变 this 指向 | 可以通过 call、apply、bind 改变 this 的指向 |                                                              箭头函数本身的 this 指向不能改变,但是可以修改它要捕获的对象的 this                                                               |

## 箭头函数 this 指向的规则

### 1.箭头函数的 this 在定义的时候捕获自外层第一个普通函数的 this

```javascript
var a = 20;
var obj = {
  a: 10,
  b: function () {
    console.log(this.a); //10
  },
  c: () => {
    console.log(this.a); //20
  }
};
obj.b();
obj.c();
```

**箭头函数外层没有普通函数，严格模式和非严格模式下它的 this 都会指向 window(全局对象)**
**this 对象的指向是可变的，但是在箭头函数中，它是固定的。**

### 2.不能直接修改箭头函数的 this 指向,但是可以通过修改外层函数的 this

**不能通过 call、apply、bind 改变 this**

```javascript
var a = 20;
var obj = {
  a: 10,
  b: function () {
    console.log(this.a);
  },
  c: () => {
    console.log(this.a); // 20
  }
};
obj.b.call({ a: 30 }); // 30
obj.c.call({ a: 40 }); // 20
```

由于箭头函数**没有**自己的`this`，所以当然也就不能用`call()`、`apply()`、`bind()`这些方法去改变`this`的指向。
**但是可以改变它外部普通函数的 this 指向，箭头函数也会跟着改变。**

<font color="#ff502c">函数体内的 this 对象，就是定义时所在的对象，而不是使用时所在的对象。</font>

### 3.箭头函数没有<font color="#ff502c">prototype</font>(原型)，所以箭头函数本身没有 this

```javascript
var bb = () => {};
console.log(bb.prototype); // undefined
```

### 4.箭头函数是匿名函数，不能作为构造函数，不能使用 new

如果

```javascript
var aa = () => {
  console.log(this);
};
var bb = new aa(); // Uncaught TypeError: aa is not a constructor
```

### 5.箭头函数的 arguments

- 箭头函数的 this 指向全局，使用 arguments 会报未声明的错误
- 箭头函数的 this 指向普通函数时,它的 argumens 继承于该普通函数

```javascript
var bb = () => {
  console.log(arguments); // Uncaught ReferenceError: arguments is not defined
};
var name = 'window';
var aa = function () {
  var cc = () => {
    console.log(this.name);
    console.log(arguments); // Arguments [callee: ƒ, Symbol(Symbol.iterator): ƒ]
  };
  cc();
};
aa.call({ name: 'aa' });
bb();
```

不可以使用 arguments 对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。

### 箭头函数不能当做 Generator 函数,不能使用 yield 关键字

## 总结

**箭头函数注意事项**

- **箭头函数不支持重命名函数参数,普通函数的函数参数支持重命名**
- **箭头函数一条语句返回对象字面量，需要加括号**
- **箭头函数在参数和箭头之间不能换行！**
- **箭头函数不支持 new.target**
- **箭头函数的 this 意外指向和代码的可读性。**
