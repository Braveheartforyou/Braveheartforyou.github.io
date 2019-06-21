---
title: 解构赋值（es6）(未完待续)
date: 2017-07-22 23:08:32
tags: [ECMAScript6]
categories: [ECMAScript6]
description: ECMAScript6 解构赋值(未完待续)
---
## <font color="red">解构赋值</font> 语法是一个Javascript表达式，这使得可以将值从**数组或属性**从对象提取到不同的变量中。
<!--more-->
## 语法
```javascript
	let a, b, rest;

	/* array 解构赋值 */
	[a, b] = [1, 2]
	console.log(a); // 1
	console.log(b); // 2

	[a, b, ...rest] = [1, 2, 3, 4, 5];
	console.log(a); // 1
	console.log(b); // 2
	console.log(rest); // 3, 4, 5

	/* object 解构赋值 */
	({a, b} = {a:1, b:2});
	console.log(a); // 1
	console.log(b); // 2

	// ES7 - 试验性 (尚未标准化)
	// Uncaught SyntaxError: Unexpected token ...
	({a, b, ...rest} = {a:1, b:2, c:3, d:4});
```
**剩余元素必须是解构赋值表达式中的最后一个元素**
## 简述
 对象字面量和数组字面量提供了一种简单的定义一个特定的数据组的方法。
 ```javascript
 var x = [1, 2, 3, 4, 5];
 ```
 结构赋值使用了相同的语法，不同的是在表达式左边定义了要从原变量中取出什么变量。
 ```javascript
 var x = [1, 2, 3, 4, 5];
 var [y, z] = x;
 console.log(y) // 1
 console.log(z) // 2
 ```
 解构赋值的作用类似于perl和Python语言中的相似特性。
 ## 解构数组
 ### 基本变量赋值
 ```javascript
 var foo = ["one", "two", "three"];
 var [one, two, three] = foo;
 console.log(one); // "one"
 console.log(two); // "two"
 console.log(three); // "three"
 ```
 ### 声明赋值分离
 通过解构分离变量的声明，可以为一个变量赋值
 ```javascript
 var a, b;
 [a, b] = [1, 2];
 console.log(a); // 1
 console.log(b); // 2
 ```
 ### 默认值
 为了防止从数组中去一个值为undefined的对象，可以为这个对象设置默认值
 ```javascript
 var a, b;
 [a=5, b=7] = [1];
 console.log(a); // 1
 console.log(b); // 7
 ```
 ### 交换变量
 在一个解构表达式中可以交换两个变量的值。
 没有解构值情况下，交换两个变量需要一个临时变量
 ```javascript
 var a = 1;
 var b = 3;
 [a, b] = [b, a];
 console.log(a); // 3
 console.log(b); // 1
 ```
 ### 解析一个从函数返回的数组
 从一个函数返回一个数组是十分常见的情况.。解构使得处理返回值为数组时更加方便。
 在下面例子中，[1, 2] 作为函数的 f() 的输出值，可以使用解构用一句话完成解析。
 ```javascript
 function f() {
  return [1, 2];
 }
 var a, b;
 [a, b] = f();
 console.log(a); // 1
 console.log(b); // 2
 ```
 感谢解构赋值，函数现在可以返回多个值了。尽管函数一直都可以返回一个数组，但现在这样做有更多的灵活性。
 ### 忽略某些返回值
 你也可以忽略你不感兴趣的返回值：
 ```javascript
 function f() {
  return [1, 2, 3];
 }
 var [a, , b] = f();
 console.log(a); // 1
 console.log(b); // 3
 ```
你也可以忽略全部返回值。例如：
 ```javascript
 [,,] = f();
 ```
 ### 将剩余数组赋值给一个变量
 当解构一个数组时，可以使用剩余模式，将数组剩余部分赋值给一个变量。
 ```javascript
 var [a, ...b] = [1, 2, 3];
 console.log(a); // 1
 console.log(b); // [2, 3]
 ```
 注意：如果剩余元素右侧有一个逗号，会抛出语法错误的异常，因为剩余元素必须是数组的最后一个元素。
 ```javascript
 var [a, ...b,] = [1, 2, 3];
 // SyntaxError: rest element may not have a trailing comma
 ```
 ## 解构对象
 ### 简单示例
 ```javascript
 var o = {p: 42, q: true};
 var {p, q} = o;
 console.log(p); // 42
 console.log(q); // true 
 // 用新变量名赋值
 var {p: foo, q: bar} = o;
 console.log(foo); // 42
 console.log(bar); // true  
 ```