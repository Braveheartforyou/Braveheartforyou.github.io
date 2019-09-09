---
title: Promise系列（二） 实现一个自己的Promise
date: 2019-02-18 18:23:26
tags: [ECMAScript6]
categories: [ECMAScript6]
description: Promise是我们用来解决地狱回调，我在这篇博客中实现自己的一个Promise.
---

## 简介

一个 `Promise` 就是一个对象，它代表了一个异步操作的最终完成或者失败。大多数人仅仅是使用已创建的`Promise`实例对象，因此本教程将首先说明怎样使用 `Promise`，之后说明如何创建`Promise`。
本质上，`Promise` 是一个绑定了回调的对象，而不是将回调传进函数内部。
原生提供了`Promise`对象。本篇不注重讲解`promise`的用法，关于用法，可以看阮一峰老师的`ECMAScript 6`系列里面的`Promise`部分：
[ECMAScript 6 : Promise对象](http://es6.ruanyifeng.com/#docs/promise)
我们在最后实现一个`es2015`版本的`PromiseA`.
注：在本代码中有很多不完善的地方，最后会给出一个`es2015`的版本，那个版本是比较完善的。
本篇博客逐步实现，最终使其符合`Promises/A+`规范

## 代码实现

逐步实现：

1. **基础版本**
2. **支持同步任务**
3. **支持状态**
4. **支持链式操作**
5. **支持串行异步任务**
6. **达到PromiseA+规范**
7. **实现Promise方法all、resolve、reject、race等方法**
8. **实现promiseify方法**

> 注意事项：这边建议不要使用 `setTimeout`作为 `Promise` 的实现。因为 `setTimeout` 属于 宏任务， 而 `Promise` 属于 微任务。
<!-- 不理知道宏任务和微任务请看量一篇博客：[evenloop](http://asyncnode.com/blog/evenloop.html) -->

### 基础版本(异步回调)

目标

- 可以通过`new` 关键字创建一个 `Promise`实例。
- `Promise`实例传入的异步方法执行成功就执行注册的成功回调函数，失败就执行注册的失败回调函数。

首先实现两个判断函数：

```javascript
// 判断当前传入的参数是否是function
const isFunction = variable => typeof variable === 'function';
// 另一个是判断当前执行环境如果实在node中首先使用process.nextTick如果没有使用setImmediate,如果还没有就使用setTimeout
let executeAsync = undefined;
if (typeof process === 'object' && process.nextTick) {
    executeAsync = process.nextTick;
} else if (typeof setImmediate === 'function') {
    executeAsync = setImmediate;
} else {
    executeAsync = function(fn){setTimeout(fn, 0)};
};
// 封装我们要调用的callAsync函数
function callAsync (fn, arg, callback, onError) {
    executeAsync(function () {
        try {
            callback ? callback(fn(arg)) : fn(arg);
        } catch (e) {
            onError(e);
        };
    });
};
```

注：判断传入参数是否为`function`, 根据当前环境降级实现**微任务或宏任务**

**代码实现**

```javascript
// 判断当前传入的参数是否是function
const isFunction = variable => typeof variable === 'function';
function PromiseA (handle) {
  if (!isFunction(handle)){
      throw new Error('Promise resolver handle is not a function');
  }
  var self = this;
  self.value = null; // 成功时的值
  self.error = null; // 失败时的原因
  self.onFulfilled = function (value) { (value) }; // 成功的回调函数
  self.onRejected = function (value) { (value) };; // 失败的回调函数
  
  function resolve (value) {
      self.value = value;
      self.onFulfilled(self.value);
  }

  function reject (error) {
      self.error = error;
      self.onRejected(self.error);
  }
  //捕获callback是否报错
  try {
    handle(resolve, reject);
  } catch (error) {
    reject(error);
  }
}
PromiseA.prototype.then = function (onFulfilled, onRejected) {
    //在这里给promise实例注册成功和失败回调
    console.log(this.onFulfilled);
    console.log(onFulfilled);
    this.onFulfilled = onFulfilled;
    this.onRejected = onRejected;
}
// 测试代码
new PromiseA((resolve, reject) => {
    setTimeout(function () {
        resolve(111);
    }, 10);
}).then(data => {
    console.log(data);
}); // 111
new PromiseA([]);
```

判断实例传入的参数是否为function，在then中注册了这个promise实例的成功回调和失败回调，当promise resolve时，当promise resolve时，就把异步执行结果赋值给promise实例的value，并把这个值传入成功回调中执行，失败就把异步执行失败原因赋值给promise实例的error，并把这个值传入失败回调并执行。
### 支持同步代码

我们执行

```javascript
new PromiseA((resolve, reject) => {
    resolve(111);
}).then(data => {
    console.log(data);
});
```

现在如果我们同步执行resolve(111)的话，我们的then函数还没有被执行，所以后续的then中得回调函数也不会被执行，简单来说就是then函数在resolve(111)的函数之后执行，所以then中得回调也不会被执行。
**目标**
- 使promise支持同步方法

**代码**
```javascript
// 判断当前传入的参数是否是function
const isFunction = variable => typeof variable === 'function';
// 另一个是判断当前执行环境如果实在node中首先使用process.nextTick如果没有使用setImmediate,如果还没有就使用setTimeout
let executeAsync = undefined;
if (typeof process === 'object' && process.nextTick) {
    executeAsync = process.nextTick;
} else if (typeof setImmediate === 'function') {
    executeAsync = setImmediate;
} else {
    executeAsync = function(fn){setTimeout(fn, 0)};
};
function PromiseA (handle) {
    if (!isFunction(handle)){
        throw new Error('Promise resolver handle is not a function');
    }
    var self = this;
    self.value = null; // 成功时的值
    self.error = null; // 失败时的原因
    self.onFulfilled = function (value) { (value) }; // 成功的回调函数
    self.onRejected = function (value) { (value) };; // 失败的回调函数
    
    function resolve (value) {
        executeAsync(() => {
            self.value = value;
            console.log(self.onFulfilled(self.value));
            self.onFulfilled(self.value);
        }, 0);
    }

    function reject () {
        executeAsync(() => {
            self.error = error;
            self.onRejected(self.error);
        }, 0);
    }
    
    handle(resolve, reject);
}
PromiseA.prototype.then = function (onFulfilled, onRejected) {
    //在这里给promise实例注册成功和失败回调
    console.log(this.onFulfilled);
    console.log(onFulfilled);
    this.onFulfilled = onFulfilled;
    this.onRejected = onRejected;
}
// 测试代码
new PromiseA((resolve, reject) => {
    resolve(111);
}).then(data => {
    console.log(data);
}); // 111
```
就是在resolve和reject里面用setTimeout进行包裹，使其到then方法执行之后再去执行，这样我们就让promise支持传入同步方法。

注：setTimeout其实是最后一种方法，要看环境，
    要是支持的话，优先使用MutationObserver(微任务)
    再是MessageChannel（优先级比定时器高的宏任务）
    再是setImmediate（这个兼容性太差，不建议用）
    再不行就退化到setTimeout了

### 支持三种状态
我们知道在使用promise时，promise有三种状态：pending(进行中)、fulfilled(已成功)、rejected(已失效)。
1、只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
2、一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从pending变为fulfilled和从pending变为rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 resolved（已定型）。如果改变已经发生了，你再对Promise对象添加回调函数，也会立即得到这个结果。

**目标**
- 实现promise的三种状态
- 实现promise对象的状态改变，改变只有两种可能：从pending变为fulfilled和从pending变为rejected。
- 实现一旦promise状态改变，再对promise对象添加回调函数，也会立即得到这个结果。

**代码实现**
```javascript
    // 判断当前传入的参数是否是function
    const isFunction = variable => typeof variable === 'function';
    // 另一个是判断当前执行环境如果实在node中首先使用process.nextTick如果没有使用setImmediate,如果还没有就使用setTimeout
    let executeAsync = undefined;
    if (typeof process === 'object' && process.nextTick) {
        executeAsync = process.nextTick;
    } else if (typeof setImmediate === 'function') {
        executeAsync = setImmediate;
    } else {
        executeAsync = function(fn){setTimeout(fn, 0)};
    };
    // 定义Promise的三种状态常量
    const PENDING = 'pending'
    const FULFILLED = 'fulfilled'
    const REJECTED = 'rejected'
    function PromiseA (handle) {
        let self = this;
        self.value = undefined;
        self.error = undefined;
        self.status = PENDING;
        self.onFulfilled = function (value) { (value) };
        self.onRejected = function (value) { (value) }

        function resolve (value) {
            if (self.status === PENDING) {
                executeAsync(() => {
                    self.status = FULFILLED;
                    self.value = value;
                    self.onFulfilled(self.value);
                });
            }
        }

        function reject (error) {
            if (self.status === PENDING) {
                executeAsync(() => {
                    self.status = REJECTED;
                    self.error = value;
                    self.onRejected(self.error);
                });
            }
        }

        handle(resolve, reject);
        //捕获callback是否报错
        // try {
        // } catch (error) {
        //     reject(error);
        // }
    }
    PromiseA.prototype.then = function (onFulfilled, onRejected) {
        if (this.status === PENDING) {
            this.onFulfilled = onFulfilled;
            this.onRejected = onRejected;
        } else if (this.status === FULFILLED) {
            onFulfilled(this.value);
        } else if (this.status === REJECTED) {
            onRejected(this.value);
        }
    }
    // 测试代码
    new PromiseA((resolve, reject) => {
        resolve(111);
    }).then((data, error) => {
        console.log(data);
        console.log(error);
    }); // 111
```
为了实现上面的目标我们建立了三种状态：pending、fulfilled、rejected，如果是peding状态，我们才会改变promise的状态，并且执行相关状态的操作，并且现在的promise的状态是不可改变的。在the那种我们判断promise的状态已经从pending转换为'fulfilled'或者’rejected‘就会立刻执行他的状态的回调，并且把结果传入。

### 支持链式调用（同步）
大家都知道jquery的链式调用，promise也是支持链式调用。
我们首先在这一步实现同步的链式调用。

**目标**
- 使promise支持链式调用

注：我们把then中的回调存入数组中
```javascript
    self.onFulfilledCallbacks = [];
    self.onRejectedCallbacks = [];
```
当我们执行回调时，也要改成遍历回调数组执行回调函数
```javascript
self.onFulfilledCallbacks.forEach((callback) => callback(self.value));
self.onRejectedCallbacks.forEach((callback) => callback(self.value));
```
最后，then方法也要改一下,只需要在最后一行加一个return this即可，这其实和jQuery链式操作的原理一致，每次调用完方法都返回自身实例，后面的方法也是实例的方法，所以可以继续执行。
**代码实现**
```javascript
        // 判断当前传入的参数是否是function
    const isFunction = variable => typeof variable === 'function';
    // 另一个是判断当前执行环境如果实在node中首先使用process.nextTick如果没有使用setImmediate,如果还没有就使用setTimeout
    let executeAsync = undefined;
    if (typeof process === 'object' && process.nextTick) {
        executeAsync = process.nextTick;
    } else if (typeof setImmediate === 'function') {
        executeAsync = setImmediate;
    } else {
        executeAsync = function(fn){setTimeout(fn, 0)};
    };
    // 定义Promise的三种状态常量
    const PENDING = 'pending'
    const FULFILLED = 'fulfilled'
    const REJECTED = 'rejected'
    function PromiseA (handle) {
        let self = this;
        self.value = undefined;
        self.error = undefined;
        self.onFulfilledCallbacks = [];
        self.onRejectedCallbacks = [];
        self.status = PENDING;
        // self.onFulfilled = function (value) { (value) };
        // self.onRejected = function (value) { (value) }

        function resolve (value) {
            if (self.status === PENDING) {
                executeAsync(() => {
                    self.status = FULFILLED;
                    self.value = value;
                    // self.onFulfilled(self.value);
                    self.onFulfilledCallbacks.forEach((callback) => callback(self.value));
                });
            }
        }

        function reject (error) {
            if (self.status === PENDING) {
                executeAsync(() => {
                    self.status = REJECTED;
                    self.value = error;
                    // self.onRejected(self.value);
                    self.onRejectedCallbacks.forEach((callback) => callback(self.error));
                });
            }
        }

        handle(resolve, reject);
        //捕获callback是否报错
        // try {
        // } catch (error) {
        //     reject(error);
        // }
    }
    PromiseA.prototype.then = function (onFulfilled, onRejected) {
        if (this.status === PENDING) {
            // this.onFulfilled = onFulfilled;
            // this.onRejected = onRejected;
            this.onFulfilledCallbacks.push(onFulfilled);
            this.onRejectedCallbacks.push(onRejected);
        } else if (this.status === FULFILLED) {
            onFulfilled(this.value);
        } else if (this.status === REJECTED) {
            onRejected(this.value);
        }
        return this;
    }
    // 测试代码
    new PromiseA((reslove, reject) => {
    setTimeout(() => {
        reslove(111);
    }, 10);
    }).then(data => {
        console.log('第一' + data);
    }).then(data => {
        console.log('第二' + data);
    }); // 第一 第二
```
总结： 这个就是最简单的同步then的回调用一个内部数组来储存，最后循环调用。

### 支持串行异步任务
我们一般都是用promise.then来写异步任务，在下面完善一下代码

**目标**
- 使promise支持串行异步操作
- 支持传入promise