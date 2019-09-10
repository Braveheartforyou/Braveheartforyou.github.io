---
title: es6中let、const和var之间的关联、区别
date: 2017-07-17 15:14:25
tags: [ECMAScript6]
categories: [ECMAScript6]
description: ECMAScript6 let && const && var
---
## let、const、var的区别（let、const 为es5新添加的）

 let 允许你声明一个作用域被限制在块级中的变量、语句或者表达式。
 var 声明的变量只能是全局或者整个函数块的。
 const 伪常量 本身的值不能改变 d
 ```javascript
  {
    let a = 10;
    var b = 1;
  }

  a // ReferenceError: a is not defined.
  b // 1
 ```
<!--more-->
### let const 暂时性死区 （temporal dead zone，简称 TDZ）

 #### 只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。
 ```javascript
 var tmp = 123;
 if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
 }
 ```
 #### 必须在声明以后才能使用

 ES6 规定暂时性死区和let、const语句不出现变量提升，主要是为了减少运行时错误，防止在变量声明前就使用这个变量，从而导致意料之外的行为。
 参考 > <https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let> 兼容性查询
 
### let 不允许被重复声明
  ```javascript
  //Uncaught SyntaxError: Identifier 'q' has already been declared
  function b() {
  	var a = 10;
  	let a = 10;
  }
  ```

### 三点注意
	允许在块级作用域内声明函数。
	函数声明类似于var，即会提升到全局作用域或函数作用域的头部。
	同时，函数声明还会提升到所在的块级作用域的头部。
	上面的代码在符合 ES6 的浏览器中，都会报错，因为实际运行的是下面的代码。
	```javascript
	// 浏览器的 ES6 环境
	function f() { console.log('I am outside!'); }
	(function () {
	  var f = undefined;
	  if (false) {
	    function f() { console.log('I am inside!'); }
	  }

	  f();
	}());
	// Uncaught TypeError: f is not a function
	```
