---
title: vue中的next-tick原理和源码解析
date: 2019-07-23 12:32:54
tags: [Vue]
categories: [Vue]
description: Vue的next-tick其实也是在eventloop的原理实现的
---
## 简介
在vue的官方文档中有一个API叫做nextTick，将回调延迟到下次 DOM 更新循环之后执行。在修改数据之后立即使用这个方法，获取更新后的 DOM。
**语法**
```javascript
  vm.$nextTick([callback])
```
- 参数：
```javascript
  {Function} [callback]
```

**用法**
放在`Vue.nextTick()`回调函数中的执行的应该是**涉及DOM**操作的JavaScript代码。

Vue的响应式原理：在data选项里所有属性都会被`watcher`监控，当修改了`data`的某一个值，并不会**立即**反映到视图中。Vue会将我们对`data`的更改放到`watcher`的一个队列中（异步），只有在当前任务空闲时才会去执行`watcher`队列任务。这就有一个延迟时间。

`nextTick` 是 Vue 的一个**核心**实现，如果还不了解js运行机制，可以看一下另一篇文章[js运行机制](/blog/javascript/evenloop.html)，这里就不多赘述了。

在浏览器环境中常见的macro task 和 micro task如下：
**macro task**： 
- setTimeout、setTimeInterval
- MessageChannel
- postMessage
- setImmediate

**micro task**：
- MutationObsever
- Promise.then


## vue源码解析


