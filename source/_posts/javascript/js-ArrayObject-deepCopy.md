---
title: Javascript中Array、Object深度复制、数据类型
date: 2017-08-02 15:25:33
tags: [JavaScript]
categories: [JavaScript]
description: 在Javascript中复制String、Number等基础类型是很简单的，但是复制Array、Object就不是那么简单的了，JavaScript中数据类型的区分
---
## 类型
在JavaScript中类型分为两类：基本类型(值类型)、引用类型
### 基本类型
Undefind、Null、Boolean、Number、String
#### 实现复制
```javascript
    var str = '小伙子,25278';
    var str1 = str;
    str1 = "大伙子,5967";
    console.log(str); // 小伙子,25278
    console.log(str1); // 大伙子,5967

    var num = 25278;
    var num1 = num;
    num1 = 5967;
    console.log(num); // 25278
    console.log(num1); // 5967
```
#### 注意事项
javascript里的<font color="#ff502c">基本类型（值类型）</font>是不可以改变的，javascript也没有提供任何一个改变字符串的方法和语法.<font color="#ff502c">他不是能当成对象来使用的。</font> 
示例：
```javascript
    var str = "myobject";
    str.name = "myname";
    console.log(str.name); // undefined

    var num = 123;
    num.name = "myname";
    console.log(num.name); // undefined
```
### 引用类型
如Array、Object
javascript引用数据类型是保存在堆内存中的对象，与其它语言不同的是，你不可以直接访问堆内存空间中的位置和操作堆内存空间。只能通过操作对象的在栈内存中的引用地址。所以引用类型的数据，在栈内存中保存的实际上是对象在堆内存中的引用地址。
#### 示例
```javascript
    var obj1 = new Object();
    var obj2 = obj1;
    obj2.name = "我有名字了";
    console.log(obj1.name); // 我有名字了
```
示例图：![目录结构](../images/Array_Object.jpg)
自然，给obj2添加name属性，实际上是给堆内存中的对象添加了name属性，obj2和obj1在栈内存中保存的只是堆内存对象的引用地址，虽然也是拷贝了一份，但指向的对象却是同一个。故而改变obj2引起了obj1的改变。

__Array实现复制__
```javascript
    var arr = [1, 2, 3, 4, 5];
    var arr1 = arr;
    arr1.push(6);
    console.log(arr); // [1, 2, 3, 4, 5, 6]
    console.log(arr1); // [1, 2, 3, 4, 5, 6]
```
<font color="#ff502c">改为：</font>
```javascript
    var arr = [1, 2, 3, 4, 5];
    // Array.prototype.slice() 返回一个新的数组，可以切断他们之间的联系
    var arr1 = arr.slice(); // 根据数组下标改变数组
    // Array.prototype.splice() 返回一个新的数组，可以切断他们之间的联系
    var arr2 = arr.splice(); // 根据数组下标改变数组
    var arr3 = arr.concat();
    arr1.push(6);
    arr2.push(6);
    arr3.push(6);
    console.log(arr); // [1, 2, 3, 4, 5]
    console.log(arr1); // [1, 2, 3, 4, 5, 6]
    console.log(arr2); // [6]
    console.log(arr3); // [1, 2, 3, 4, 5, 6]
```
<font color='red'>Array.prototype.slice()</font><http://asyncnode.com/blog/es6-Array1.html>查看详情
<font color='red'>Array.prototype.splice()</font><http://asyncnode.com/blog/es6-Array1.html>查看详情
<font color='red'>Array.prototype.concat()</font><http://asyncnode.com/blog/es6-Array.html>查看详情
__Object实现复制__
```javascript
    var obj = {
        name: '111'
    }
    var obj1 = obj;
    obj1.age = 12;
    console.log(obj); // Object {name: "111", age: 12}
    console.log(obj1); // Object {name: "111", age: 12}
```
<font color="#ff502c">改为：</font>、
```javascript
    var obj = {
        name: '111'
    }
    var obj1 = JSON.parse(JSON.stringify(obj)); // 先把对象通过浏览器对象转为 JSON字符串，再转为 JSON对象
    obj1.age = 12;
    console.log(obj); // Object {name: "111"}
    console.log(obj1); // Object {name: "111", age: 12}
```
<font color="#ff502c">同时Object也是有合并方法的为assgin(),但是他是不能项Array.prototype.concat()实现深度复制</font> 
```javascript
   var obj = {
        name: '111'
    }
    var obj1 = Object.assgin(obj, {}); // Object的合并对象方法
    obj1.age = 12;
    console.log(obj); // Object {name: "111", age: 12} 
    console.log(obj1); // Object {name: "111", age: 12} 
```
<font color="#ff502c">最后一种也是比较官方的方法</font>
```javascript
    function copy(obj) {
    var copy = Object.create(Object.getPrototypeOf(obj));
    var propNames = Object.getOwnPropertyNames(obj);

    propNames.forEach(function(name) {
        var desc = Object.getOwnPropertyDescriptor(obj, name);
        Object.defineProperty(copy, name, desc);
    });

    return copy;
    }

    var obj1 = { a: 1, b: 2 };
    var obj2 = copy(obj1); // obj2 looks like obj1 now
    obj2.c = 3;
    console.log(obj1); // Object {a: 1, b: 2}
    console.log(obj2); // Object {a: 1, b: 2, c: 3}
```
下面的代码会创建一个给定对象的副本。 创建对象的副本有不同的方法，以下是只是一种方法，并解释了Array.prototype.forEach() 是如何使用ECMAScript 5 Object.* 元属性（meta property ）函数工作的。
