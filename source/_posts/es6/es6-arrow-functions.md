---
title: arrow-functions（箭头函数）和普通的函数的区别
date: 2019-04-10 16:53:14
tags: [ECMAScript6]
categories: [ECMAScript6]
description: ECMAScript6中的箭头我们基本都是非常的常用，总结一下他与普通函数的区别和优点、确定。
---
在用vue框架、或者react框架中我们会用到很多es6中的新特性，比较多的就是<font color="blue">箭头函数</font>.都会知道一点普通函数和箭头函数的区别,这里总结一下箭头函数和普通函数的区别：

|      对比     |      普通函数      |     箭头函数     |
|:------------:|:-------------:|:-------------:|
| this指向规则 |  this总是指向调用它的那个对象| 1.所有箭头函数本身没有this </br>2.箭头函数的this在定义的时候捕获自外层第一个普通函数的this </br> 3.如果箭头函数外层没有普通函数,严格模式和非严格模式下它的this都会指向window(全局对象) |
| 有无prototype |   有   | 箭头函数没有<font color="blue">prototype</font>(原型) |
| 可否new |   可以   | 箭头函数作为匿名函数,是不能作为构造函数的(因为箭头函数没有constructor),不能使用new,不然会报错 |
| 有无arguments |   有   | 1.箭头函数的this指向全局,使用会报未声明的错误 </br> 2.箭头函数的this指向普通函数时,它的<font color="blue">argumens</font>继承于改普通函数 |
| 可否new |   可以   | 箭头函数作为匿名函数,是不能作为构造函数的(因为箭头函数没有constructor),不能使用new,不然会报错 |
| 可否改变this指向 |   可以通过call、apply、bind改变this的指向  | 箭头函数本身的this指向不能改变,但是可以修改它要捕获的对象的this |

## 箭头函数this指向的规则

### 1.箭头函数的this在定义的时候捕获自外层第一个普通函数的this
```javascript
    var a = 20;
    var obj = {
        a: 10,
        b: function(){
            console.log(this.a); //10
        },
        c: () => {
            console.log(this.a); //20
        }
    }
    obj.b(); 
    obj.c();
```
箭头函数外层没有普通函数，严格模式和非严格模式下它的this都会指向window(全局对象)
this对象的指向是可变的，但是在箭头函数中，它是固定的。
### 2.不能直接修改箭头函数的this指向,但是可以通过修改外层函数的this
** 不能通过call、apply、bind改变this **
```javascript 
    var a = 20;
    var obj = {
        a: 10,
        b: function(){
            console.log(this.a); 
        },
        c: () => {
            console.log(this.a); // 20
        }
    }
    obj.b.call({a: 30}); // 30
    obj.c.call({a: 40}); // 20
```
由于箭头函数没有自己的this，所以当然也就不能用call()、apply()、bind()这些方法去改变this的指向。
但是可以改变它外部普通函数的this指向，箭头函数也会跟着改变。
<font color="blue">函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。</font>
### 3.箭头函数没有<font color="blue">prototype</font>(原型)，所以箭头函数本身没有this
```javascript
    var bb = () => {};
    console.log(bb.prototype); // undefined
```
### 4.箭头函数是匿名函数，不能作为构造函数，不能使用new
如果
```javascript
    var aa = () => {
        console.log(this);
    }
    var bb = new aa(); // Uncaught TypeError: aa is not a constructor
```
### 5.箭头函数的arguments
- 箭头函数的this指向全局，使用arguments会报未声明的错误
- 箭头函数的this指向普通函数时,它的argumens继承于该普通函数
```javascript
    var bb = () => {
        console.log(arguments); // Uncaught ReferenceError: arguments is not defined
    }
    var name = 'window';
    var aa = function () {
        var cc = () => {
            console.log(this.name);
            console.log(arguments); // Arguments [callee: ƒ, Symbol(Symbol.iterator): ƒ]
        }
        cc();
    }
    aa.call({name: 'aa'});
    bb();
```
不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。
### 箭头函数不能当做Generator函数,不能使用yield关键字

## 总结
** 箭头函数注意事项 **
- 箭头函数不支持重命名函数参数,普通函数的函数参数支持重命名
- 箭头函数一条语句返回对象字面量，需要加括号
- 箭头函数在参数和箭头之间不能换行！
- 箭头函数不支持new.target
- 箭头函数的this意外指向和代码的可读性。

