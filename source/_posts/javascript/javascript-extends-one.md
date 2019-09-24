---
title: js继承实现
date: 2017-07-23 16:09:42
tags: [JavaScript]
categories: [JavaScript]
description: js几种继承方式
---

## 创建 js 样例：

```javascript
/*
 * 约定
 */
function Fun() {
  // 私有属性
  var val = 1; // 私有基本属性
  var arr = [1]; // 私有引用属性
  function fun() {} // 私有函数（引用属性）

  // 实例属性
  this.val = 1; // 实例基本属性
  this.arr = [1]; // 实例引用属性
  this.fun = function() {}; // 实例函数（引用属性）
}

// 原型属性
Fun.prototype.val = 1; // 原型基本属性
Fun.prototype.arr = [1]; // 原型引用属性
Fun.prototype.fun = function() {}; // 原型函数（引用属性）
```

这样创建比较合理

## 原型链继承

这是实现继承最简单的方式，核心就是一句话

### 实例

```javascript
function Super() {
  this.val = 1;
  this.arr = [1];
}
function Sub() {}
Sub.prototype = new Super(); // 核心
var sub1 = new Sub();
var sub2 = new Sub();
sub1.val = 2;
sub1.arr.push(2);
console.log(sub1.val); // 2 基本类型 不共享
console.log(sub2.val); // 1

console.log(sub1.arr); // [1, 2] 引用 array类型 共享
console.log(sub2.arr); // [1, 2]
```

### 核心

<font color="#ff502c">拿父类实例充当子类原型对象</font>

### 优缺点

#### 优点:

简单，易于实现

#### 缺点：

修改 sub1.arr 后 sub2.arr 也变了，因为来自原型对象的引用树形是所有实例共享的。
执行顺序：执行 sub1.arr.push(2);先对 sub1 进行属性查找，找遍了实例属性（在本例中没有实例属性），没找到，就开始顺着原型链向上找，拿到了 sub1 的原型对象，一搜身，发现有 arr 属性。于是给 arr 末尾插入了 2，所以 sub2.arr 也变了

创建子类实例时，无法向父类构造函数传参

## 借用构造函数

简单原型链真够简单，可是存在 2 个致命缺点简直不能用，于是上个世纪末的 jsers 就想办法 fix 这 2 个缺陷，然后出现了借用构造函数方式

### 实例

```javascript
function Super(val) {
  this.val = val;
  this.arr = [1];
  this.fun = function() {
    console.log(this.val);
  };
}
function Sub(val) {
  Super.call(this, val); // 核心
}
var sub1 = new Sub(1);
var sub2 = new Sub(2);
sub1.arr.push(2);
console.log(sub1.val); // 1
console.log(sub2.val); // 2

console.log(sub1.arr); // [1, 2]
console.log(sub2.arr); // [1]
console.log(sub1.fun === sub2.fun); // false
```

### 核心

<font color="#ff502c">借父类的构造函数来增强子类实例</font>，等于是把父类的实例属性复制了一份给子类实例装上了（完全没有用到原型）

### 优缺点

#### 优点

解决了子类实例共享父类引用属性的问题
创建子类实例时，可以向父类构造函数传参

#### 缺点

无法实现函数复用，每个子类实例都持有一个新的 fun 函数，太多了就会影响性能

## 组合继承（常用）

目前我们的借用构造函数方式还是有问题（无法实现函数复用），没关系，接着修复，jsers 吭哧吭哧又搞出了组合继承

### 实例

```javascript
function Super() {
  // 只在此处声明基本属性和引用属性
  this.val = 1;
  this.arr = [1];
}
//  在此处声明函数
Super.prototype.fun1 = function() {};
Super.prototype.fun2 = function() {};
//Super.prototype.fun3...
function Sub() {
  Super.call(this); // 核心
  // ...
}
Sub.prototype = new Super(); // 核心

var sub1 = new Sub(1);
var sub2 = new Sub(2);
console.log(sub1.fun === sub2.fun); // true
```

### 核心

<font color="red>把实例函数都放在原型对象上，以实现函数复用。同时还要保留借用构造函数方式的优点</font>，通过 Super.call(this);继承父类的基本属性和引用属性并保留能传参的优点；通过 Sub.prototype = new Super();继承父类函数，实现函数复用

### 优缺点

#### 优点

不存在引用属性共享问题
可传参
函数可复用

#### 缺点

子类原型上有一份多余的父类实例属性，因为父类构造函数被调用了两次，生成了两份，而子类实例上的那一份屏蔽了子类原型上的

## 寄生组合继承（最佳方式）

从名字就能看出又是对组合继承的优化

### 实例

```javascript
function beget(obj) {
  // 生孩子函数 beget：龙beget龙，凤beget凤。
  var F = function() {};
  F.prototype = obj;
  return new F();
}
function Super() {
  // 只在此处声明基本属性和引用属性
  this.val = 1;
  this.arr = [1];
}
//  在此处声明函数
Super.prototype.fun1 = function() {};
Super.prototype.fun2 = function() {};
//Super.prototype.fun3...
function Sub() {
  Super.call(this); // 核心
  // ...
}
var proto = beget(Super.prototype); // 核心
console.log(proto); // F {}
console.log(proto.constructor);
// function Super(){
// 只在此处声明基本属性和引用属性
// this.val = 1;
// this.arr = [1];
// }
proto.constructor = Sub; // 核心
console.log(proto.constructor);
// function Sub(){
// Super.call(this);   // 核心
// ...
// }
Sub.prototype = proto; // 核心
console.log(Sub.prototype);
// Super {constructor: function}
var sub = new Sub();
console.log(sub.val);
console.log(sub.arr);
```

construcotr 可参考：

### 核心

用 beget(Super.prototype);<font color="red>切掉了原型对象上多余的那份父类实例属性</font>
寄生组合式继承，这名字不是很贴切，和寄生式继承关系并不是特别大

## 原型式

其实介绍完上面的完美方案就可以结束了，但从组合继承到完美方案好像有一段不小的思维跳跃，有必要把故事说清楚

### 实例

```javascript
function beget(obj) {
  // return 一个新的 function 原型复制为参数原型
  var F = function() {};
  F.prototype = obj;
  return new F();
}
function Super() {
  this.val = 1;
  this.arr = [1];
}

// 拿到父类对象
var sup = new Super();
// 生孩子
var sub = beget(sup); // 核心
// 增强
sub.attr1 = 1;
sub.attr2 = 2;
//sub.attr3...

console.log(sub.val); // 1
console.log(sub.arr); // 1
console.log(sub.attr1); // 1
```
