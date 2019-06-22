---
title: js中的this指向问题（-）
date: 2017-07-25 09:25:37
tags: [JavaScript]
categories: [JavaScript]
description: 基本的判断是, this的指向在函数定义的时候是确定不了的，只有函数执行的时候才能确定this到底指向谁，实际上this的最终指向的是那个调用它的对象
---
## 概述
 首先必须要说的是，this的指向在函数定义的时候是确定不了的，只有函数执行的时候才能确定this到底指向谁，实际上this的最终指向的是那个调用它的对象,那么接下来我会深入的探讨这个问题。
 <font color="red">function执行默认返回undefined</font>
 ```javascript
 function a () {
	console.log(1111);
}
console.log(a()); // undefined
 ```
### 在一般函数方法中使用 this 指代全局对象
```javascript
    var ale = '外部';
    function test() {
        var ale = '内部';
        console.log(this.ale); // 外部
        console.log(this); // window
    }
    test(); // 因为在全局中执行，所以this指向window
    // window.a(); 与上面其实是一致的
```
### 作为对象方法调用，this纸袋上级对象
```javascript
    var ale = '外部'
    var oObeject = {
        ale: '内部',
        fn: function () {
            console.log(this.ale); // 内部
            console.log(this); // oObeject
        }
    }
    oObeject.fn();
```
这里的this指向oObeject,因为这个fn是通过oObeject调用的,所以指向oObeject,this在函数创建的时候是决定不了的,在调用的时候才能决定，<font color="red">谁调用this指像谁</font> 

### 作为构造函数调用，this 指代new 出的对象
```javascript
    var ale = '外部'
    function Test () {
        this.ale = '内部';
        console.log(this.ale); // 内部
        console.log(this); // Test
        return this; // 默认 return 当前this
    }
    var test1 = new Test();
    console.log(test1.ale); // 内部
```
new关键字可以改变this的指向,将这个this指向对象test1,此时仅仅只是创建，并没有执行，而调用这个函数Fn的是对象test1，那么this指向的自然是对象test1
#### 当函数中有return时
```javascript
    function fn() {  
        this.user = '内部';  
        return {}; // 因为返回的是空对象像 or return function () {}
    }
    var a = new fn;  
    console.log(a.user); //undefined
    console.log(a); // Object {}
```
<font color="red">如果返回值是一个对象，那么this指向的就是那个返回的对象，如果返回值不是一个对象那么this还是指向函数的实例。</font>
```javascript
    function fn() {  
        this.user = '内部';  
        return 1; // 因为返回的是空对象像 or return undefined
    }
    var a = new fn;  
    console.log(a.user); //undefined
    console.log(a); // Object {user}
```
需要注意的是 <font color="red">还有一点就是虽然null也是对象，但是在这里this还是指向那个函数的实例，因为null比较特殊。</font>
### apply、call、bind改变函数的调用对象，此方法的第一个参数为改变后调用这个函数的对象，this指代第一个参数
```javascript
    var x = '外部';
    function test() {
        console.log(this.x);
    }
    var o = {
        x: '内部',
        m: test
    }
    o.m.apply(); // 外部
    //apply()的参数为空时，默认调用全局对象。因此，这时的运行结果为0，证明this指的是全局对象。如果把最后一行代码修改为
    o.m.apply(o); // 内部
```