---
title: 函数节流（throttle）与函数去抖（debounce）
date: 2019-03-04 14:03:51
tags: [JavaScript, JSEffect]
categories: [JavaScript]
description: 防止频繁操作dom带来重绘、回流（重排）带来的性能损耗
---

## 简介

以下场景往往由于事件频繁被触发，因而频繁执行 DOM 操作、资源加载等重行为，导致 UI 停顿甚至浏览器崩溃。

1. window 对象的 resize、scroll 事件

2. 拖拽时的 mousemove 事件

3. 射击游戏中的 mousedown、keydown 事件

4. 文字输入、自动完成的 keyup 事件

实际上对于 window 的 resize 事件，实际需求大多为停止改变大小 n 毫秒后执行后续处理；而其他事件大多的需求是以一定的频率执行后续处理。针对这两种需求就出现了 debounce 和 throttle 两种解决办法。

## 与函数去抖（debounce）

当`调用动作 n 毫秒`后，才会执行该动作，若在这`n 毫秒`内又调用此动作则将`重新`计算执行时间。

```javascript
/**
 * timer为第二次要调用间隔的时间，只有大于间隔时间的调用会立即执func，不然要等到timer之后再执行
 * @param timer {Number} 间隔时间，单位毫秒
 * @param func {Function} 要执行的函数
 * @param * {Function} 返回匿名函数
 */

var debounce = function (timer, func) {
  var nextAction;
  return function () {
    var content = this,
      args = arguments;
    clearTimeout(nextAction);
    nextAction = setTimeout(function () {
      func.apply(content, args);
    }, timer);
  };
};
```

### 实例

调用 onresize 函数，我们不需要触发频率这么快，要让函数执行延迟 500 毫秒再执行。

```javascript
var count = 0;
window.onresize = debounce(500, function () {
  console.log(new Date().getTime());
  console.log(count++);
});
```

### 参考 v1.9.1 的 Underscore.js debounce

```javascript
// Returns a function, that, as long as it continues to be invoked, will not
// be triggered. The function will be called after it stops being called for
// N milliseconds. If `immediate` is passed, trigger the function on the
// leading edge, instead of the trailing.
_.debounce = function (func, wait, immediate) {
  //immediate 不传为 undefind, 默认不立即执行，传true 立即执行一次
  var timeout, result;

  var later = function (context, args) {
    timeout = null;
    if (args) result = func.apply(context, args);
  };
  // restArguments Underscore.js 的restArguments方法 https://underscorejs.org/#restArguments
  // 格式化 参数为数组
  var debounced = restArguments(function (args) {
    if (timeout) clearTimeout(timeout);
    if (immediate) {
      var callNow = !timeout;
      timeout = setTimeout(later, wait);
      if (callNow) result = func.apply(this, args);
    } else {
      // _.delay Underscore.js _.delay https://underscorejs.org/#delay
      timeout = _.delay(later, wait, this, args);
    }

    return result;
  });
  debounced.cancel = function () {
    clearTimeout(timeout);
    timeout = null;
  };

  return debounced;
};
```

## 函数节流（throttle）

预先设定一个`执行周期`，当再次调用动作的`时刻`大于等于`执行周期`则执行该动作，然后进入下一个新周期。

```javascript
/**
 * timer为第二次要调用间隔的时间，只有大于间隔时间的调用会立即执func，不然要等到timer之后再执行
 * @param delay {Number} 间隔时间，单位毫秒
 * @param func {Function} 要执行的函数
 * @param * {Function} 返回匿名函数
 */

var throttle = function (delay, func) {
  var nextAction = 0;
  return function () {
    var currTime = +new Date();
    if (currTime - nextAction > delay) {
      func.apply(this, arguments);
      nextAction = currTime;
    }
  };
};
```

### 实例

和防抖一样调用 onresize 函数

```javascript
var count = 0;
window.onresize = throttle(500, function () {
  console.log(new Date().getTime());
  console.log(count++);
});
```

### 参考 v1.9.1 的 Underscore.js throttle

```javascript
// Returns a function, that, when invoked, will only be triggered at most once
// during a given window of time. Normally, the throttled function will run
// as much as it can, without ever going more than once per `wait` duration;
// but if you'd like to disable the execution on the leading edge, pass
// `{leading: false}`. To disable execution on the trailing edge, ditto.
_.throttle = function (func, wait, options) {
  var timeout, context, args, result;
  var previous = 0;
  if (!options) options = {};

  var later = function () {
    previous = options.leading === false ? 0 : _.now();
    timeout = null;
    result = func.apply(context, args);
    if (!timeout) context = args = null;
  };

  var throttled = function () {
    var now = _.now();
    if (!previous && options.leading === false) previous = now;
    var remaining = wait - (now - previous);
    context = this;
    args = arguments;
    if (remaining <= 0 || remaining > wait) {
      if (timeout) {
        clearTimeout(timeout);
        timeout = null;
      }
      previous = now;
      result = func.apply(context, args);
      if (!timeout) context = args = null;
    } else if (!timeout && options.trailing !== false) {
      timeout = setTimeout(later, remaining);
    }
    return result;
  };

  throttled.cancel = function () {
    clearTimeout(timeout);
    previous = 0;
    timeout = context = args = null;
  };

  return throttled;
};
```
