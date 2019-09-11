---
title: Promise系列（二） 实现一个自己的Promise
date: 2019-02-18 18:23:26
tags: [ECMAScript6]
categories: [ECMAScript6]
description: Promise是我们用来解决地狱回调，我在这篇博客中实现自己的一个Promise.
---

> [Promise系列（二） 实现一个自己的Promise](/blog/es6/es6-promise.html)
> [Promise系列（三） 实现一个自己的Promise](/blog/es6/es-promise-two.html)

## 简介

一个 `Promise` 就是一个对象，它代表了一个异步操作的最终完成或者失败。大多数人仅仅是使用已创建的`Promise`实例对象，因此本教程将首先说明怎样使用 `Promise`，之后说明如何创建`Promise`。
本质上，`Promise` 是一个绑定了回调的对象，而不是将回调传进函数内部。
原生提供了`Promise`对象。本篇不注重讲解`promise`的用法，关于用法，可以看阮一峰老师的`ECMAScript 6`系列里面的`Promise`部分：
[ECMAScript 6 : Promise 对象](http://es6.ruanyifeng.com/#docs/promise)
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
6. **实现 Promise 方法 all、resolve、reject、race 等方法**
7. **实现 promiseify 方法**
8. **达到 PromiseA+规范**

> 注意事项：这边建议不要使用 `setTimeout`作为 `Promise` 的实现。因为 `setTimeout` 属于 宏任务， 而 `Promise` 属于 微任务。

<!-- 不理知道宏任务和微任务请看量一篇博客：[evenloop](http://asyncnode.com/blog/evenloop.html) -->

显示代码请看

## 基础版本(异步回调)

目标

- 可以通过`new` 关键字创建一个 `Promise`实例。
- `Promise`实例传入的异步方法执行成功就执行注册的成功回调函数，失败就执行注册的失败回调函数。

首先实现两个判断函数：

```javascript
// 判断当前传入的参数是否是function
const isFunction = variable => typeof variable === "function";
// 另一个是判断当前执行环境如果实在node中首先使用process.nextTick如果没有使用setImmediate,如果还没有就使用setTimeout
let executeAsync = undefined;
if (typeof process === "object" && process.nextTick) {
  executeAsync = process.nextTick;
} else if (typeof setImmediate === "function") {
  executeAsync = setImmediate;
} else {
  executeAsync = function(fn) {
    setTimeout(fn, 0);
  };
}
// 封装我们要调用的callAsync函数
function callAsync(fn, arg, callback, onError) {
  executeAsync(function() {
    try {
      callback ? callback(fn(arg)) : fn(arg);
    } catch (e) {
      onError(e);
    }
  });
}
```

注：判断传入参数是否为`function`, 根据当前环境降级实现**微任务或宏任务**

**代码实现**

```javascript
// 判断当前传入的参数是否是function
const isFunction = variable => typeof variable === "function";
function PromiseA(handle) {
  if (!isFunction(handle)) {
    throw new Error("Promise resolver handle is not a function");
  }
  var self = this;
  self.value = null; // 成功时的值
  self.error = null; // 失败时的原因
  self.onFulfilled = function(value) {
    value;
  }; // 成功的回调函数
  self.onRejected = function(value) {
    value;
  }; // 失败的回调函数

  function resolve(value) {
    self.value = value;
    self.onFulfilled(self.value);
  }

  function reject(error) {
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
PromiseA.prototype.then = function(onFulfilled, onRejected) {
  //在这里给promise实例注册成功和失败回调
  console.log(this.onFulfilled);
  console.log(onFulfilled);
  this.onFulfilled = onFulfilled;
  this.onRejected = onRejected;
};
// 测试代码
new PromiseA((resolve, reject) => {
  setTimeout(function() {
    resolve(111);
  }, 10);
}).then(data => {
  console.log(data);
}); // 111
new PromiseA([]);
```

判断实例传入的参数是否为`function`，在`then`中注册了这个`promise`实例的成功回调和失败回调，当`promise resolve`时，当`promise resolve`时，就把异步执行结果赋值给`promise`实例的`value`，并把这个值传入成功回调中执行，失败就把异步执行失败原因赋值给`promise`实例的`error`，并把这个值传入失败回调并执行。

## 支持同步代码

我们执行

```javascript
new PromiseA((resolve, reject) => {
  resolve(111);
}).then(data => {
  console.log(data);
});
```

现在如果我们同步执行`resolve(111)`的话，我们的`then`函数还没有被执行，所以后续的`then`中得回调函数也不会被执行，简单来说就是`then`函数在`resolve(111)`的函数之后执行，所以`then`中得回调也不会被执行。

**目标**

- **使`promise`支持同步方法**

**代码**

```javascript
// 判断当前传入的参数是否是function
const isFunction = variable => typeof variable === "function";
// 另一个是判断当前执行环境如果实在node中首先使用process.nextTick如果没有使用setImmediate,如果还没有就使用setTimeout
let executeAsync = undefined;
if (typeof process === "object" && process.nextTick) {
  executeAsync = process.nextTick;
} else if (typeof setImmediate === "function") {
  executeAsync = setImmediate;
} else {
  executeAsync = function(fn) {
    setTimeout(fn, 0);
  };
}
function PromiseA(handle) {
  if (!isFunction(handle)) {
    throw new Error("Promise resolver handle is not a function");
  }
  var self = this;
  self.value = null; // 成功时的值
  self.error = null; // 失败时的原因
  self.onFulfilled = function(value) {
    value;
  }; // 成功的回调函数
  self.onRejected = function(value) {
    value;
  }; // 失败的回调函数
  function resolve(value) {
    executeAsync(() => {
      self.value = value;
      console.log(self.onFulfilled(self.value));
      self.onFulfilled(self.value);
    }, 0);
  }

  function reject() {
    executeAsync(() => {
      self.error = error;
      self.onRejected(self.error);
    }, 0);
  }
  handle(resolve, reject);
}
PromiseA.prototype.then = function(onFulfilled, onRejected) {
  //在这里给promise实例注册成功和失败回调
  console.log(this.onFulfilled);
  console.log(onFulfilled);
  this.onFulfilled = onFulfilled;
  this.onRejected = onRejected;
};
// 测试代码
new PromiseA((resolve, reject) => {
  resolve(111);
}).then(data => {
  console.log(data);
}); // 111
```

就是在`resolve`和`reject`里面用`setTimeout`进行包裹，使其到`then`方法执行之后再去执行，这样我们就让`Promise`支持传入同步方法。

> 注：`setTimeout`其实是最后一种方法，要看环境, 要是支持的话，优先使用`MutationObserver`(微任务),再是`MessageChannel`（优先级比定时器高的宏任务）,再是`setImmediate`（这个兼容性太差，不建议用）再不行就降级为`setTimeout`了

## 支持三种状态

我们知道在使用`Ppromise`时，`Promise`有三种状态：`pending(进行中)`、`fulfilled(已成功)`、`rejected(已失效)`。

1. 只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果。`Promise`对象的状态改变，

只有两种可能：从`pending`变为`fulfilled`和从`pending`变为`rejected`。只要这两种情况发生，状态就**凝固**了，不会再变了，会一直保持这个结果，这时就称为 `resolved（已定型）`。如果改变已经发生了，你再对`Promise`对象添加回调函数，也会立即得到这个结果。

**目标**

- 实现`promise`的三种状态
- 实现`promise`对象的状态改变，改变只有两种可能：从`pending`变为`fulfilled`和从`pending`变为`rejected`。
- 实现一旦`promise`状态改变，再对`promise`对象添加回调函数，也会立即得到这个结果。

**代码实现**

```javascript
// 判断当前传入的参数是否是function
const isFunction = variable => typeof variable === "function";
// 另一个是判断当前执行环境如果实在node中首先使用process.nextTick如果没有使用setImmediate,如果还没有就使用setTimeout
let executeAsync = undefined;
if (typeof process === "object" && process.nextTick) {
  executeAsync = process.nextTick;
} else if (typeof setImmediate === "function") {
  executeAsync = setImmediate;
} else {
  executeAsync = function(fn) {
    setTimeout(fn, 0);
  };
}
// 定义Promise的三种状态常量
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";
function PromiseA(handle) {
  if (!isFunction(handle)) {
    throw new Error("Promise resolver handle is not a function");
  }
  let self = this;
  self.value = undefined;
  self.error = undefined;
  self.status = PENDING;
  self.onFulfilled = function(value) {
    value;
  };
  self.onRejected = function(value) {
    value;
  };

  function resolve(value) {
    if (self.status === PENDING) {
      executeAsync(() => {
        self.status = FULFILLED;
        self.value = value;
        self.onFulfilled(self.value);
      });
    }
  }

  function reject(error) {
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
PromiseA.prototype.then = function(onFulfilled, onRejected) {
  if (this.status === PENDING) {
    this.onFulfilled = onFulfilled;
    this.onRejected = onRejected;
  } else if (this.status === FULFILLED) {
    onFulfilled(this.value);
  } else if (this.status === REJECTED) {
    onRejected(this.value);
  }
};
// 测试代码
new PromiseA((resolve, reject) => {
  resolve(111);
}).then((data, error) => {
  console.log(data);
  console.log(error);
}); // 111
```

为了实现上面的目标我们建立了三种状态：`pending`、`fulfilled`、`rejected`，如果是`peding`状态，我们才会改变`promise`的状态，并且执行相关状态的操作，并且现在的`promise`的状态是不可改变的。在`then`那种我们判断`promise`的状态已经从`pending`转换为`fulfilled`或者`rejected`就会立刻执行他的状态的回调，并且把结果传入。

## 支持链式调用（同步）

大家都知道`jquery`的链式调用，`promise`也是支持链式调用。
我们首先在这一步实现同步的链式调用。

**目标**

- **使`promise`支持链式调用**

> 注：我们把`then`中的回调存入数组中

```javascript
self.onFulfilledCallbacks = [];
self.onRejectedCallbacks = [];
```

当我们执行回调时，也要改成遍历回调数组执行回调函数

```javascript
self.onFulfilledCallbacks.forEach(callback => callback(self.value));
self.onRejectedCallbacks.forEach(callback => callback(self.value));
```

最后，`then`方法也要改一下,只需要在最后一行加一个`return this`即可，这其实和`jQuery`链式操作的原理一致，每次调用完方法都返回自身实例，后面的方法也是实例的方法，所以可以继续执行。

**代码实现**

```javascript
// 判断当前传入的参数是否是function
const isFunction = variable => typeof variable === "function";
// 另一个是判断当前执行环境如果实在node中首先使用process.nextTick如果没有使用setImmediate,如果还没有就使用setTimeout
let executeAsync = undefined;
if (typeof process === "object" && process.nextTick) {
  executeAsync = process.nextTick;
} else if (typeof setImmediate === "function") {
  executeAsync = setImmediate;
} else {
  executeAsync = function(fn) {
    setTimeout(fn, 0);
  };
}
// 定义Promise的三种状态常量
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";
function PromiseA(handle) {
  if (!isFunction(handle)) {
    throw new Error("Promise resolver handle is not a function");
  }

  let self = this;
  self.value = undefined;
  self.error = undefined;
  self.onFulfilledCallbacks = [];
  self.onRejectedCallbacks = [];
  self.status = PENDING;
  // self.onFulfilled = function (value) { (value) };
  // self.onRejected = function (value) { (value) }

  function resolve(value) {
    if (self.status === PENDING) {
      executeAsync(() => {
        self.status = FULFILLED;
        self.value = value;
        // self.onFulfilled(self.value);
        self.onFulfilledCallbacks.forEach(callback => callback(self.value));
      });
    }
  }

  function reject(error) {
    if (self.status === PENDING) {
      executeAsync(() => {
        self.status = REJECTED;
        self.value = error;
        // self.onRejected(self.value);
        self.onRejectedCallbacks.forEach(callback => callback(self.error));
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
// 原型上的then方法
PromiseA.prototype.then = function(onFulfilled, onRejected) {
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
};
// 测试代码
new PromiseA((reslove, reject) => {
  setTimeout(() => {
    reslove(111);
  }, 10);
})
  .then(data => {
    console.log("第一" + data);
  })
  .then(data => {
    console.log("第二" + data);
  }); // 第一 第二
```

总结： 这个就是最简单的同步`then`的回调用一个内部数组来储存，最后循环调用。

## 支持串行异步任务

我们一般都是用`promise.then`来写异步任务，在下面完善一下代码

**目标**

- **使`PromiseA`支持串行异步操作**
- **支持传入`PromiseA`对象**

### 实现

在上一步已经实现可以链式调用，但是只支持**同步**的链式调用，现在要实现支持一步串行调用。

代码如下：

```javascript
// 第一个promise实例
new PromiseA((resolve, reject) => {
  setTimeout(() => {
    resolve("first------" + new Date());
  }, 1000);
})
  .then(first => {
    console.log(first);
    // 第二个promise实例
    return new PromiseA((resolve, reject) => {
      setTimeout(() => {
        resolve("second------" + new Date());
      }, 2000);
    });
  })
  .then(second => {
    console.log(second);
    // 第三个promise实例
    return new PromiseA((resolve, reject) => {
      setTimeout(() => {
        resolve("third------" + new Date());
      }, 3000);
    });
  })
  .then(res => {
    console.log(res);
  });
// 异步串行
// "first------" + new Date()
// "first------" + new Date()
// "first------" + new Date()
```

想要的结果是在**1s**之后输出`'first------' + new Date()`结果，再经过**2s**之后执行`'second------' + new Date()`，再等**3s**之后才会执行`'third------' + new Date()`，但是结果和预想的结果不相同。
实际结果是**1s**之后直接就会输出三次`first------ + new Date()`，因为所有的回调函数都注册在了`PromiseA`中的`onFulfilledCallbacks`队列里，在后面`resolve`后会全部执行，这个并不能满足**异步串行**。
需要将每个回调函数注册在对应`promise`实例的`onFulfilledCallbacks`里面，然后再返回一下新的`promise`以做到异步串行效果。

改写代码如下：

- 改写`prototype`上的`then`方法
- 改写`resolve`、`reject`上的代码

### Promise.prototype.then

```javascript
// 修改原型上的then方法
PromiseA.prototype.then = function(onFulfilled, onRejected) {
  // 返回一个新的PromiseA对象
  return new PromiseA((onFulfilledNext, onRejectedNext) => {
    // 封装一个成功时执行的函数
    let fulfilled = value => {
      if (isFunction(onFulfilled)) {
        callAsync(
          onFulfilled,
          value,
          res => {
            if (res instanceof PromiseA) {
              // 如果当前回调函数返回PromiseA对象，必须等待其状态改变后在执行下一个回调
              res.then(onFulfilledNext, onRejectedNext);
            } else {
              // 否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
              onFulfilledNext(res);
            }
          },
          onRejectedNext
        );
      } else {
        try {
          onFulfilledNext(value);
        } catch (err) {
          // 如果函数执行出错，新的PromiseA对象的状态为失败
          onRejectedNext(err);
        }
      }
    };
    // 封装一个失败时执行的函数
    let rejected = error => {
      if (isFunction(onRejected)) {
        callAsync(
          onRejected,
          error,
          res => {
            if (res instanceof PromiseA) {
              // 如果当前回调函数返回PromiseA对象，必须等待其状态改变后在执行下一个回调
              res.then(onFulfilledNext, onRejectedNext);
            } else {
              // 否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
              onFulfilledNext(res);
            }
          },
          onRejectedNext
        );
      } else {
        try {
          onRejectedNext(error);
        } catch (err) {
          // 如果函数执行出错，新的PromiseA对象的状态为失败
          onRejectedNext(err);
        }
      }
    };

    switch (this.status) {
      // 当状态为pending时，将then方法回调函数加入执行队列等待执行
      case PENDING:
        this.onFulfilledCallbacks.push(fulfilled);
        this.onRejectedCallbacks.push(rejected);
        break;
      // 当状态已经改变时，立即执行对应的回调函数
      case FULFILLED:
        fulfilled(this.value);
        break;
      case REJECTED:
        rejected(this.value);
        break;
    }
  });
};
```

接着修改 `resolve` 和 `reject` ：依次执行队列中的函数

当 `resolve` 或 `reject` 方法执行时，依次提取成功或失败**任务队列**当中的函数开始执行，并**清空队列**，从而实现 `then` 方法的多次调用，实现的代码如下：

```javascript
function callAsync(fn, arg, callback, onError) {
  executeAsync(function() {
    try {
      callback ? callback(fn(arg)) : fn(arg);
    } catch (e) {
      onError(e);
    }
  });
}
// resolve方法
function resolve(val) {
  // 状态一旦定型不可改变
  if (self.status !== PENDING) return;
  // 改变当前实例状态
  self.status = FULFILLED;
  // 依次执行成功队列中的函数，并清空队列
  const runFulfilled = value => {
    let cb;
    // 循环执行
    while ((cb = self.onFulfilledCallbacks.shift())) {
      cb(value);
    }
  };
  // 依次执行失败队列中的函数，并清空队列
  const runRejected = error => {
    let cb;
    while ((cb = self.onRejectedCallbacks.shift())) {
      cb(error);
    }
  };
  /* 如果resolve的参数为PromiseA对象，则必须等待（参数PromiseA）对象状态改变后,
    当前PromsieA的状态才会改变，且状态取决于参数PromsieA对象的状态
    */
  if (val instanceof PromiseA) {
    // 如果为参数为PromiseA对象，在参数PromiseA对象的then方法中执行后续runFulfilled or runRejected操作
    val.then(
      value => {
        self.value = value;
        runFulfilled(value);
      },
      err => {
        self.value = err;
        runRejected(err);
      }
    );
  } else {
    // 如果参数是普通类型，直接执行runFulfilled
    self.value = val;
    runFulfilled(val);
  }
}
// 添加reject时执行的函数
function reject(err) {
  if (self.status !== PENDING) return;
  self.status = REJECTED;
  self.value = err;
  // 依次执行失败队列中的函数，并清空队列
  let cb;
  while ((cb = self.onRejectedCallbacks.shift())) {
    cb(err);
  }
}

try {
  handle(resolve.bind(this), reject.bind(this));
} catch (err) {
  reject(err);
}
```

在这里面最关键的就是回调函数返回**异步PromiseA对象**时，要等**异步PromiseA对象**有结果时，当前的实例才能根据前面的异步结果改变自己的状态。

测试一下代码，在第一`PromiseA`实例的`then`的回调函数中，返回一个新的`PromiseA`实例并且在`2s`之后直接执行当前实例的`resolve`方法，在下一个`then`回调函数中再返回一个`PromiseA`实例，在`3s`后执行当前实例的`resolve`，最后一个`then`传入的回调函数中输出`resolve`中储存的值。

```javascript
    // p1
    new PromiseA((resolve, reject) => {
        setTimeout(() => {
            resolve("first------" + new Date());
        }, 1000);
    })
    // t1
    .then(first => {
        console.log(first);
        // 第二个promise实例
        // p2
        return new PromiseA((resolve, reject) => {
            setTimeout(() => {
                resolve("second------" + new Date());
            }, 2000);
        });
    })
    // t2
    .then(second => {
        console.log(second);
        // 第三个promise实例
        // p3
        return new PromiseA((resolve, reject) => {
            setTimeout(() => {
                resolve("third------" + new Date());
            }, 3000);
        });
    })
    // t3
    .then(res => {
        console.log(res);
    });
    // 1s之后执行
    // "first------" + new Date()
    // 在上面的基础上等待2s之后执行
    // "second------" + new Date()
    // 在上面的基础上等待3s之后执行
    // "third------" + new Date()
```

整理一下内部的执行过程：

1. 当实例化`p1`时，在它`1s`后调用`resolve`函数，在它的`t1`中传入一个函数，这个函数返回一个`p2`实例。
2. `t2`的执行，要等到`p2`的状态改变才执行。当执行`t2`时，又会产生一个`p3`实例。
3. 等到`p3`的状态改变时，才会触发后续的`t3`执行。

请看下文[Promise系列（二） 实现一个自己的Promise](/blog/es6/es6-promise.html)