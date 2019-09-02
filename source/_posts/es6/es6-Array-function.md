---
title: ES6 Array系列(四) Array常用的方法和实现reduce、map、filter、forEach
date: 2019-08-14 11:12:21
tags: [JavaScript]
categories: [JavaScript]
description: Array常用的方法和实现reduce、map、filter、forEach
---

> [ES6 Array 系列(一) 一些常用array的扩展方法](/blog/es6/es6-Array.html)
> [ES6 Array 系列(二) 一些常用array的扩展方法（二）](/blog/es6/es6-Array1.html)
> [ES6 Array系列(三) Array中的forEach方法可以用break、continue跳出循环？](/blog/es6/es6-Array-break-continue.html)
> [ES6 Array系列(四) Array常用的方法和实现reduce、map、filter、forEach](/blog/es6/es6-Array-function.html)

## 简介

* * *
`Array.prototype`上有很多方法，可以很方便的实现各种循环、过滤对数组做很多的处理，这里主要记录自己怎么实现几个方法`map`、`forEach`、`filter`、`reduce`，怎么使用就不多做讲解了因为在[mdn 中 Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)或者别人的文章中有很多的讲解了。

## 实现reduce

* * *
首先要了解`reduce`它有两个参数，第一个参数是一个回调方法`callback`，第二个参数是一个初始值`initialValue`。
大致实现步骤如下：

- 判断参数，判断调用方法本身是否为Array
- 声明要用的变量
- 判断是否有初始值，如果没有则从本身数组中取，取到直接跳出循环
- 循环调用callback

```javascript
    Array.prototype.selfReduce = function (callback, initalValue) {
        // 不能是null调用方法
        if (this === null) {
            throw new TypeError('Array.prototype.reduce' + 'called on null or undefined');
        }
        // 第一个参数必须要为function
        if (typeof callback !== 'function') {
             throw new TypeError(callback + ' is not a function');
        }
        // 声明要用到的变量
        // 要循环的数组
        let arr = Array.prototype.slice.call(this);
        let _len = arr.length;
        // 结果数组
        let res = [];
        // 开始的数组索引 默认为 0
        let startIndex = 0;

        // 判断是否为有初始化值 initalValue
        if (initalValue === undefined) { // 如果初始值为 undefined 循环从数组中找到有值，并且退出循环
            // 过滤稀疏值
            for(let i = 0; i < _len; i++) {
                if (!arr.hasOwnProperty(i)) {
                    continue;
                } else {
                    startIndex = i;
                    res = arr[i];
                    break;
                }
            }
        } else {
            res = initalValue;
        }

        // 在上一步拿到初始值，循环调用传入的回调函数，并且过滤松散值
        for (let i = startIndex++; i < arr.length; i++) {
            if (!arr.hasOwnProperty(i)) {
                continue;
            }
            res = callback.call(null, res, arr[i], i, this)
        }
        return res;
    }
```

测试一下`selfReduce`和`reduce`方法是否表现一致，代码如下：

```javascript
    var aTest = [1, 2, 3, 4, 5];
    arr.reduce((prev, next) => {
        return prev + next;
    }, 0); // 15
    arr.selfReduce((prev, next) => {
        return prev + next;
    }, 0); // 15
```

其实也可以去看一下官方的[mdn reduce polyfill](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)。

## 实现map

* * *
`map`的使用这里不多做赘述，只记录它的两种实现方式一种通过`for`循环实现，另一种通过`reduce`实现。

### for循环实现

```javascript
    Array.prototype.selfMap = function (callback, context) {
        // 不能是null调用方法
        if (this === null) {
            throw new TypeError('Array.prototype.reduce' + 'called on null or undefined');
        }
        // 第一个参数必须要为function
        if (typeof callback !== 'function') {
             throw new TypeError(callback + ' is not a function');
        }
        // 声明要用到的变量
        let arr = Array.prototype.slice.call(this);
        let _len = arr.length;
        let aMap = [];
        // 循环调用
        for (let i = 0; i < _len; i++) {
            // 过滤稀疏值
            if (!arr.hasOwnProperty(i)) {
                continue;
            }
            aMap[i] = callback.call(context, arr[i], i, this);
        }
        return aMap;
    }
```

测试实现`selfMap`和`map`是否一致，代码如下：

```javascript
    var aTest = [1, 2, 3, 4, 5];
    aTest.map((item) => {
        return item * 2;
    }); // [2, 4, 6, 8, 10]
    aTest.selfMap((item) => {
        return item * 2;
    }); // [2, 4, 6, 8, 10]
```

### reduce实现

```javascript
    Array.prototype.reduceMap = function (callback, context) {
        // 不能是null调用方法
        if (this === null) {
            throw new TypeError('Array.prototype.reduce' + 'called on null or undefined');
        }
        // 第一个参数必须要为function
        if (typeof callback !== 'function') {
             throw new TypeError(callback + ' is not a function');
        }
        // 获取数组
        let aMap = Array.prototype.slice.call(this);
        // 使用reduce实现循环
        return aMap.reduce((pre, cur, index) => {
            // 拼接上次循环结果和当前结果
            // 循环调用callback
            return [...pre, callback.call(context, cur, index, this)];
        }, []);
    }
```

测试实现`reduceMap`和`map`是否一致，代码如下：

```javascript
    var aTest = [1, 2, 3, 4, 5];
    aTest.map((item) => {
        return item * 2;
    }); // [2, 4, 6, 8, 10]
    aTest.reduceMap((item) => {
        return item * 2;
    }); // [2, 4, 6, 8, 10]
```

## filter实现

* * *
filter也用到很多次，这里也不多做赘述直接看两种实现方法：

### for循环实现

```javascript
    Array.prototype.selfFilter = function (callback, context) {
        // 不能是null调用方法
        if (this === null) {
            throw new TypeError('Array.prototype.reduce' + 'called on null or undefined');
        }
        // 第一个参数必须要为function
        if (typeof callback !== 'function') {
             throw new TypeError(callback + ' is not a function');
        }
        // 获取数组
        let aArr = Array.prototype.slice.call(this);
        let _len = aArr.length;
        let aFArr = [];
        // 循环调用callback
        for (let i = 0; i < _len; i++) {
            if (!aArr.hasOwnProperty(i)) {
                continue;
            }
            callback.call(context, aArr[i], i, this) && aFArr.push(i);
        }
        return aFArr;
    }
```

### reduce实现

```javascript
    Array.prototype.reduceFilter = function (callback, context) {
        // 不能是null调用方法
        if (this === null) {
            throw new TypeError('Array.prototype.reduce' + 'called on null or undefined');
        }
        // 第一个参数必须要为function
        if (typeof callback !== 'function') {
             throw new TypeError(callback + ' is not a function');
        }
        // 获取数组
        let aArr = Array.prototype.slice.call(this);

        // 循环调用callback
        aArr.reduce((pre, cur, index) => {
            return callback.call(context, cur, index, this) ? [...pre, cur] : [...pre]
        }, []);

        return aArr;
    }
```

### 测试代码

```javascript
    var aTest = [1, 2, 3, 4, 5, 6];
    aTest.filter((item) => {
        return item > 2;
    });
    aTest.selfFilter((item) => {
        return item > 2;
    });
    aTest.reduceFilter((item) => {
        return item > 2;
    });
```

## 实现forEach

* * *
for循环实现

```javascript
    Array.prototype.selfForeach = function (callback, context) {
        // 不能是null调用方法
        if (this === null) {
            throw new TypeError('Array.prototype.reduce' + 'called on null or undefined');
        }
        // 第一个参数必须要为function
        if (typeof callback !== 'function') {
             throw new TypeError(callback + ' is not a function');
        }
        // 获取数组
        let arr = Array.prototype.slice.call(this);
        let _len = arr.length;
        for (let i = 0; i < _len; i++) {
            callback.call(context, arr[i], i, arr);
        }
        return arr;
    }
```

reduce循环实现

```javascript
    Array.prototype.reduceForeach = function (callback, context) {
        // 不能是null调用方法
        if (this === null) {
            throw new TypeError('Array.prototype.reduce' + 'called on null or undefined');
        }
        // 第一个参数必须要为function
        if (typeof callback !== 'function') {
             throw new TypeError(callback + ' is not a function');
        }
        // 获取数组
        let arr = Array.prototype.slice.call(this);
        arr.reduce((pre, cur, index) => {
            return [...pre, callback.call(context, cur, index, this)];
        }, []);
    }
```

## 总结

基本上理解了怎么实现，无论是使用for来实现还是用reduce实现，基本上没有太大的差别。

## 参考

> [Array.prototype.reduce()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)
> [一个合格的中级前端工程师需要掌握的 28 个 JavaScript 技巧](https://juejin.im/post/5cef46226fb9a07eaf2b7516#heading-4)
> [JS Array.reduce 实现 Array.map 和 Array.filter](https://juejin.im/post/5c0b7f03e51d452eec725729)