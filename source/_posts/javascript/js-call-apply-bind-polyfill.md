---
title: 自己手动实现call、apply、bind this（三）
date: 2019-01-15 15:31:37
tags: [JavaScript]
categories: [JavaScript]
description: 自己手动实现一个 call、apply 并且实现 bind的polyfill
---
## 实现思路
1. <font color="red">将函数设为对象的属性</font>
2. <font color="red">执行该函数</font>
3. <font color="red">删除该函数</font>
```javascript
  // 第一步
  foo.fn = bar
  // 第二步
  foo.fn()
  // 第三步
  delete foo.fn
```
## 注意事项：
1. <font color="red">传入的参数并不确定</font>
2. <font color="red">this 参数可以传 null，当为 null 的时候，视为指向 window</font>
3. <font color="red">函数是可以有返回值的！</font>
## call 实现代码
```javascript
  Function.prototype.call = function (context) {
    var context = context || window;
    context.fn = this;
    var args = [];
    var len = arguments.length;
    // arguments是类数组对象，遍历之前需要保存长度，过滤出第一个传参
    for (var i = 1; i < len; i++) {
      // 避免object之类传入
      args.push('arguments[' + i + ']');
    }
    var result = eval('context.fn(' + args + ')');
    delete context.fn;
    return result;  
  }
  // 测试
  var aArr = [1, 2, 3];
  var oObj = {
    firstName: 'joy',
    lastName: 'tony'
  };
  function callArr () {
    // console.log(this);
    return this;
  }
  callArr.call(aArr); // [1, 2, 3]
  
  function calloObj (age) {
    return 'firstName：' + this.firstName + '/ lastName：' + this.lastName + '/ age：' + age;
  }
  calloObj.call(oObj, 25); // "firstName：joy/ lastName：tony/ age：25"
```
## apply 实现代码
```javascript
  Function.prototype.apply = function (context, arr) {
    var context = context || window;
    context.fn = this;

    var result;
    if (!arr) {
        result = context.fn();
    }
    else {
        var args = [];
        for (var i = 0, len = arr.length; i < len; i++) {
            args.push('arr[' + i + ']');
        }
        result = eval('context.fn(' + args + ')')
    }

    delete context.fn
    return result;
  }
  function applyArr () {
    // console.log(this);
    return this;
  }
  applyArr.apply(aArr); // [1, 2, 3]

  function applyoObj (age) {
    return 'firstName：' + this.firstName + '/ lastName：' + this.lastName + '/ age：' + age;
  }
  calloObj.call(oObj, [25]); // "firstName：joy/ lastName：tony/ age：25"
```
## bind 实现polyfill
参考：<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind>
```javascript
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var aArgs   = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP    = function() {},
        fBound  = function() {
          // this instanceof fBound === true时,说明返回的fBound被当做new的构造函数调用
          return fToBind.apply(this instanceof fBound
                 ? this
                 : oThis,
                 // 获取调用时(fBound)的传参.bind 返回的函数入参往往是这么传递的
                 aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    // 维护原型关系
    if (this.prototype) {
      // Function.prototype doesn't have a prototype property
      fNOP.prototype = this.prototype; 
    }
    // 下行的代码使fBound.prototype是fNOP的实例,因此
    // 返回的fBound若作为new的构造函数,new生成的新对象作为this传入fBound,新对象的__proto__就是fNOP的实例
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```
