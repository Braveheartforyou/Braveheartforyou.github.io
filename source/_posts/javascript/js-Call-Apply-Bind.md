---
title: JavaScript中的call、apply、bind的应用场景（二）
date: 2017-07-25 15:38:39
tags: [JavaScript, This]
categories: [JavaScript]
description: apply()方法调用一个函数，其具有一种个指定的this值，以及作为一个数组（或类似数组的对象）提供的参数。call()方法的作用和apply()方法类似，只有一个区别，就是call()方法接受的是若干个参数列表，而apply()方法接受的是一个包含多个参数的数组。
---
## 概述
apply()方法吊用一个函数，其具有一种个指定的this值，<font color="#ff502c"> 以及作为一个数组（或类似数组的对象）提供的参数</font>。call()方法的作用和apply()方法类似，只有一个区别，就是call()方法<font color="#ff502c">接受的是若干个参数列表</font>，而apply()方法接受的是一个包含<font color="#ff502c">多个参数的数组</font>
bind() 函数会创建一个新函数（称为绑定函数），新函数与被调函数（绑定函数的目标函数）具有相同的函数体（在 ECMAScript 5 规范中内置的call属性）。当目标函数被调用时 this 值绑定到 bind() 的第一个参数，该参数不能被重写。绑定函数被调用时，bind() 也接受预设的参数提供给原函数。一个绑定函数也能使用new操作符创建对象：这种行为就像把原函数当成构造器。提供的 this 值被忽略，同时调用时的参数被提供给模拟函数。
## apply语法
```javascript
fun.apply(thisArg, [argsArray])
```
### 参数
#### thisArg
在 fun 函数运行时指定的 this 值。需要注意的是，指定的 this 值并不一定是该函数执行时真正的 this 值，如果这个函数处于非严格模式下，则指定为 null 或 undefined 时会自动指向全局对象（浏览器中就是window对象），同时值为原始值（数字，字符串，布尔值）的 this 会指向该原始值的自动包装对象
#### argsArray
一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 fun 函数。如果该参数的值为null 或 undefined，则表示不需要传入任何参数。从ECMAScript 5 开始可以使用类数组对象。浏览器兼容性请参阅本文底部内容。

### 描述
在调用一个存在的函数时，你可以为其指定一个 this 对象。 this 指当前对象，也就是正在调用这个函数的对象。 使用 apply， 你可以只写一次这个方法然后在另一个对象中继承它，而不用在新对象中重复写该方法。

apply 与 call() 非常相似，不同之处在于提供参数的方式。apply 使用参数数组而不是一组参数列表（原文：a named set of parameters）。apply 可以使用数组字面量（array literal），如 fun.apply(this, ['eat', 'bananas'])，或数组对象， 如  fun.apply(this, new Array('eat', 'bananas'))。

你也可以使用 arguments  对象作为 argsArray 参数。 arguments 是一个函数的局部变量。 它可以被用作被调用对象的所有未指定的参数。 这样，你在使用apply函数的时候就不需要知道被调用对象的所有参数。 你可以使用arguments来把所有的参数传递给被调用对象。 被调用对象接下来就负责处理这些参数。

从 ECMAScript 第5版开始，可以使用任何种类的类数组对象，就是说只要有一个 length 属性和[0...length) 范围的整数属性。例如现在可以使用 NodeList 或一个自己定义的类似 {'length': 2, '0': 'eat', '1': 'bananas'} 形式的对象。

需要注意：Chrome 14 以及 Internet Explorer 9 仍然不接受类数组对象。如果传入类数组对象，它们会抛出异常。

### 实例
```javascript
  var array = ['a', 'b'];
  var elements = [0, 1, 2];
  array.push.apply(array, elements);
  console.log(array);
```
### 使用apply来链接构造器

你可以使用apply来链接一个对象<font color="#ff502c">构造器</font>，类似于Java。在接下来的例子中我们会创建一个全局Function 对象的construct方法 ，来使你能够在构造器中使用一个类数组对象而非参数列表。
```javascript
  Function.prototype.construct = function (aArgs) {
    var oNew = Object.create(this.prototype);
    this.apply(oNew, aArgs);
    return oNew;
  };
```
注意: 上面使用的Object.create()方法相对来说比较新。另一种可选的方法，请考虑如下替代方法：
```javascript
  Function.prototype.construct = function (aArgs) {
    var oNew = {};
    oNew.__proto__ = this.prototype;
    this.apply(oNew, aArgs);
    return oNew;
  };
```
使用闭包：
```javascript
  Function.prototype.construct = function(aArgs) {
    var fConstructor = this, fNewConstr = function() {
      fConstructor.apply(this, aArgs);
    };
    fNewConstr.prototype = fConstructor.prototype;
    return new fNewConstr();
  };
```
使用 Function 构造器：
```javascript
  Function.prototype.construct = function(aArgs) {
    var fConstructor = this, fNewConstr = function() {
      fConstructor.apply(this, aArgs);
    };
    fNewConstr.prototype = fConstructor.prototype;
    return new fNewConstr();
  };
```
## call语法
```javascript
fun.call(thisArg, arg1, arg2, ...)
```
### 参数
#### thisArg
在fun函数运行时指定的this值。需要注意的是，指定的this值并不一定是该函数执行时真正的this值，如果这个函数处于non-strict mode，则指定为null和undefined的this值会自动指向全局对象(浏览器中就是window对象)，同时值为原始值(数字，字符串，布尔值)的this会指向该原始值的自动包装对象
#### arg1, arg2, ...
指定的参数列表。
#### 示例
[参考地址](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

## bind语法
```javascript
function.bind(thisArg[, arg1[, arg2[, ...]]])
```
### 参数
#### thisArg
调用绑定函数时作为this参数传递给目标函数的值。 如果使用new运算符构造绑定函数，则忽略该值。当使用bind在setTimeout中创建一个函数（作为回调提供）时，作为thisArg传递的任何原始值都将转换为object。如果bind函数的参数列表为空，执行作用域的this将被视为新函数的thisArg。
#### arg1, arg2, ...
#### 示例
[参考地址](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)