---
title: Promise系列（三） 实现一个自己的Promise
date: 2019-02-20 21:44:35
tags: [Polyfill, Promise]
categories: [ECMAScript6]
description: Promise是我们用来解决地狱回调，我在这篇博客中实现自己的一个Promise.
---

## 简介

接上文的`Promsie`文章，在这篇文章中实现`Promsie`常用的方法，并且是实现`PromsieA+`的规范等等。

## 实现 Promise 方法 catch、resolve、reject、all、race 等方法

`catch`方法：

> 相当于调用 `then` 方法, 但只传入 `Rejected` 状态的回调函数

```javascript
// 添加catch方法
PromiseA.prototype.catch = function (onRejected) {
  return this.then(undefined, onRejected);
};
// 测试代码
new PromiseA((resolve, reject) => {
  reject(111);
}).catch((err) => {
  console.log(err); // 111
});
```

`resolve`方法：

> 静态 `resolve` 方法

```javascript
// 如果参数是MyPromise实例或thenable对象，直接返回value
PromiseA.resolve = function (value) {
  return value instanceof PromiseA || (value && isFunction(value.then))
    ? value
    : new PromiseA((resolve) => resolve(value));
};
// 测试代码
PromiseA.resolve(111).then((res) => {
  console.log(res); // 111
});
```

`reject`方法：

> 静态 `reject` 方法

```javascript
// 添加静态reject方法
PromiseA.reject = function (value) {
  return new PromiseA((resolve, reject) => reject(value));
};
// 测试代码
PromiseA.reject(111).then((res) => {
  console.log(res); // 111
});
```

`all` 方法:

> 静态 `all` 方法

```javascript
// 添加静态all方法
PromiseA.all = function (list) {
  return new PromiseA((resolve, reject) => {
    let values = [],
      count = list.length;
    for (let i in list) {
      // 数组参数如果不是PromiseA实例，先调用PromiseA.resolve
      this.resolve(list[i]).then((res) => {
        values[i] = res;
        // 所有状态都变成fulfilled时返回的PromiseA状态就变成fulfilled
        --count < 1 && resolve(values);
      }, reject);
    }
  });
};
// 测试代码
PromiseA.all([
  new PromiseA((resolve, reject) => {
    resolve(111);
  }),
  new PromiseA((resolve, reject) => {
    resolve(222);
  }),
  new PromiseA((resolve, reject) => {
    resolve(333);
  })
]).then((res) => {
  console.log(res);
});
// [111, 222, 333]
```

`race` 方法：

> 静态 `race` 方法

```javascript
// 添加静态race方法
PromiseA.race = function (list) {
  return new PromiseA((resolve, reject) => {
    for (let p of list) {
      // 只要有一个实例率先改变状态，新的PromiseA的状态就跟着改变
      this.resolve(p).then((res) => {
        resolve(res);
      }, reject);
    }
  });
};
// 测试代码
PromiseA.race([
  new PromiseA((resolve, reject) => {
    resolve(111);
  }),
  new PromiseA((resolve, reject) => {
    resolve(222);
  }),
  new PromiseA((resolve, reject) => {
    resolve(333);
  })
]).then((res) => {
  console.log(res);
});
// 111
```

## 实现 promiseify 方法

`promiseify`是将异步回调函数`api`转换为`promise`形式。代码实现如下：

```javascript
// 静态promisify
PromiseA.promisify = function (fn) {
  return function () {
    var args = Array.from(arguments);
    return new PromiseA(function (resolve, reject) {
      fn.apply(
        null,
        args.concat(function (err) {
          err ? reject(err) : resolve(arguments[1]);
        })
      );
    });
  };
};
```

其实就是把不同异步函数转为`Promise`的实现。

## 达到 PromiseA+规范标准
