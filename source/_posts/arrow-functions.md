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
| 有无argumens |   有   | 1.箭头函数的this指向全局,使用argunens会报未声明的错误 </br> 2.箭头函数的this指向普通函数时,它的<font color="blue">argumens</font>继承于改普通函数 |
| 可否new |   可以   | 箭头函数作为匿名函数,是不能作为构造函数的(因为箭头函数没有constructor),不能使用new,不然会报错 |
| 可否改变this指向 |   可以通过call、apply、bind改变this的指向  | 箭头函数本身的this指向不能改变,但是可以修改它要捕获的对象的this |

## 箭头函数this指向的规则

### 1.箭头函数的this在定义的时候捕获自外层第一个普通函数的this
```javascript
    var aa = () => {
        console.log(this);
    }
    aa(); // window
```
### 2.不能直接修改箭头函数的this指向,但是可以通过修改外层函数的this
```javascript 
    var aa = () => {
        console.log(this);
    }
    aa(); // window
```