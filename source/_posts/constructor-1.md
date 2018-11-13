---
title: constructor的作用
date: 2017-07-23 17:53:53
tags: [JavaScript]
categories: [JavaScript]
description: js中的constructor介绍和作用
---
## 概述
所有对象都有一个  constructor属性。在不明确使用构造函数（即对象和数组文字）的情况下创建的对象将具有constructor指向该对象的基础对象构造函数类型的属性。
```javascript
	var o = [];
	o.constructor === Object; // true
	var o = new Object;
	o.constructor === Object; // true
	var a = [];
	a.constructor === Array; // true
	var a = new Array;
	a.constructor === Array // true
	var n = new Number(3);
	n.constructor === Number; // true
```
## 例子
### 显示对象的构造函数
以下示例创建一个原型，Tree以及该类型的对象theTree。该示例然后显示constructor该对象的属性theTree。
```javascript
	function Tree(name){
		this.name = name;
	}
	var theTree = new Tree('Redwood');
	console.log('theTree.constructor is' + theTree.constructor);
	//theTree.constructor is function Tree(name) {
	//  this.name = name;
	//}
```
### 更改对象的构造函数
以下示例显示如何修改泛型对象的构造函数值。只有true，1并且"test" 不会受到影响，因为他们有只读的本地构造函数。这个例子表明，依靠constructor对象的属性并不总是安全的。
```javascript
	function Type () {}

	var types = [
	  new Array(),
	  [],
	  new Boolean(),
	  true,             // remains unchanged
	  new Date(),
	  new Error(),
	  new Function(),
	  function () {},
	  Math,
	  new Number(),
	  1,                // remains unchanged
	  new Object(),
	  {},
	  new RegExp(),
	  /(?:)/,
	  new String(),
	  'test'            // remains unchanged
	];

	for (var i = 0; i < types.length; i++) {
	  types[i].constructor = Type;
	  types[i] = [types[i].constructor, types[i] instanceof Type, types[i].toString()];
	}

	console.log(types.join('\n'));
```
此示例显示以下输出：
```javascript
	function Type() {},false,
	function Type() {},false,
	function Type() {},false,false
	function Boolean() {
	    [native code]
	},false,true
	function Type() {},false,Mon Sep 01 2014 16:03:49 GMT+0600
	function Type() {},false,Error
	function Type() {},false,function anonymous() {

	}
	function Type() {},false,function () {}
	function Type() {},false,[object Math]
	function Type() {},false,0
	function Number() {
	    [native code]
	},false,1
	function Type() {},false,[object Object]
	function Type() {},false,[object Object]
	function Type() {},false,/(?:)/
	function Type() {},false,/(?:)/
	function Type() {},false,
	function String() {
	    [native code]
	},false,test
```