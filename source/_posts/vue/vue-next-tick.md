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

Vue的响应式原理：在data选项里所有属性都会被`watcher`监控，当修改了`data`的某一个值，并不会**立即**反映到视图中。Vue会将我们对`data`的更改放到`watcher`的一个队列中（异步），只有在当前任务空闲时才会去执行`watcher`队列任务。这就有一个延迟时间，所以对dom的操作要放在`$nextTick`中来操作，才能获取到最新的`dom`。
> [响应式对象 Observer](/blog/vue/vue-definedProperty.html)
> [依赖收集 Dep](/blog/vue/vue-dep.html)
> [派发更新 Watcher](/blog/vue/vue-notify.html)

`nextTick` 是 Vue 的一个**核心**实现，如果还不了解js运行机制，可以看一下另一篇文章[js运行机制](/blog/javascript/evenloop.html)，这里就不多赘述了。

在浏览器环境中常见的macro task 和 micro task如下：
**macro task**： 
- setTimeout、setTimeInterval
- MessageChannel
- postMessage
- setImmediate
- requestAnimationFrame
- I/O
- UI渲染

**micro task**：
- MutationObsever
- Promise.then
- process.nextTick

## vue源码解析
在[派发更新 Watcher](/blog/vue/vue-notify.html)里面有用到`nextTick(flushScheduerQueue)`，其实就是`vue`对派发更新的一个优化。下面直接看源码，在 src/core/util/next-tick.js 中：
```javascript
  // nextTick 中执行回调函数的原因是保证在同一个 tick 内多次执行 nextTick，不会开启多个异步任务，而把这些异步任务都压成一个同步任务，在下一个 tick 执行完毕。
  import { noop } from 'shared/util'
  import { handleError } from './error'
  import { isIOS, isNative } from './env'
  // flushScheduerQueue
  /*存放异步执行的回调*/
  const callbacks = []
  //一个标记位，如果已经有timerFunc被推送到任务队列中去则不需要重复推送
  let pending = false
  /*下一个tick时的回调*/
  function flushCallbacks () {
    pending = false
    //复制callback
    const copies = callbacks.slice(0)
    //清除callbacks
    callbacks.length = 0
    for (let i = 0; i < copies.length; i++) {
      //触发callback的回调函数
      copies[i]()
    }
  }

  // Here we have async deferring wrappers using both microtasks and (macro) tasks.
  // In < 2.4 we used microtasks everywhere, but there are some scenarios where
  // microtasks have too high a priority and fire in between supposedly
  // sequential events (e.g. #4521, #6690) or even between bubbling of the same
  // event (#6566). However, using (macro) tasks everywhere also has subtle problems
  // when state is changed right before repaint (e.g. #6813, out-in transitions).
  // Here we use microtask by default, but expose a way to force (macro) task when
  // needed (e.g. in event handlers attached by v-on).
  /**
  其大概的意思就是：在Vue2.4之前的版本中，nextTick几乎都是基于microTask实现的，
  但是由于microTask的执行优先级非常高，在某些场景之下它甚至要比事件冒泡还要快，
  就会导致一些诡异的问题；但是如果全部都改成macroTask，对一些有重绘和动画的场
  景也会有性能的影响。所以最终nextTick采取的策略是默认走microTask，对于一些DOM
  的交互事件，如v-on绑定的事件回调处理函数的处理，会强制走macroTask。
  **/
  let microTimerFunc
  let macroTimerFunc
  let useMacroTask = false

  // Determine (macro) task defer implementation.
  // Technically setImmediate should be the ideal choice, but it's only available
  // in IE. The only polyfill that consistently queues the callback after all DOM
  // events triggered in the same loop is by using MessageChannel.
  /* istanbul ignore if */


  // 而对于macroTask的执行，Vue优先检测是否支持原生setImmediate（高版本IE和Edge支持），
  // 不支持的话再去检测是否支持原生MessageChannel，如果还不支持的话为setTimeout(fn, 0)。
  if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
    macroTimerFunc = () => {
      setImmediate(flushCallbacks)
    }
  } else if (typeof MessageChannel !== 'undefined' && (
    /**
    在Vue 2.4版本以前使用的MutationObserver来模拟异步任务。
    而Vue 2.5版本以后，由于兼容性弃用了MutationObserver。
    Vue 2.5+版本使用了MessageChannel来模拟macroTask。
    除了IE以外，messageChannel的兼容性还是比较可观的。
    **/
    isNative(MessageChannel) ||
    // PhantomJS
    MessageChannel.toString() === '[object MessageChannelConstructor]'
  )) {
    /**
    可见，新建一个MessageChannel对象，该对象通过port1来检测信息，port2发送信息。
    通过port2的主动postMessage来触发port1的onmessage事件，
    进而把回调函数flushCallbacks作为macroTask参与事件循环。
    **/
    const channel = new MessageChannel()
    const port = channel.port2
    channel.port1.onmessage = flushCallbacks
    macroTimerFunc = () => {
      port.postMessage(1)
    }
  } else {
    /* istanbul ignore next */
    macroTimerFunc = () => {
      setTimeout(flushCallbacks, 0)
    }
  }

  // Determine microtask defer implementation.
  /* istanbul ignore next, $flow-disable-line */
  if (typeof Promise !== 'undefined' && isNative(Promise)) {
    const p = Promise.resolve()
    microTimerFunc = () => {
      p.then(flushCallbacks)
      // in problematic UIWebViews, Promise.then doesn't completely break, but
      // it can get stuck in a weird state where callbacks are pushed into the
      // microtask queue but the queue isn't being flushed, until the browser
      // needs to do some other work, e.g. handle a timer. Therefore we can
      // "force" the microtask queue to be flushed by adding an empty timer.
      if (isIOS) setTimeout(noop)
    }
  } else {
    // fallback to macro
    microTimerFunc = macroTimerFunc
  }

  /**
  * Wrap a function so that if any code inside triggers state change,
  * the changes are queued using a (macro) task instead of a microtask.
  */
  /*
    推送到队列中下一个tick时执行
    cb 回调函数
    ctx 上下文
  */
  export function withMacroTask (fn: Function): Function {
    return fn._withTask || (fn._withTask = function () {
      useMacroTask = true
      const res = fn.apply(null, arguments)
      useMacroTask = false
      return res
    })
  }
  export function nextTick (cb?: Function, ctx?: Object) {
    let _resolve
    callbacks.push(() => {
      if (cb) {
        try {
          cb.call(ctx)
        } catch (e) {
          handleError(e, ctx, 'nextTick')
        }
      } else if (_resolve) {
        _resolve(ctx)
      }
    })
    if (!pending) {
      pending = true
      if (useMacroTask) {
        macroTimerFunc()
      } else {
        microTimerFunc()
      }
    }
    // $flow-disable-line
    if (!cb && typeof Promise !== 'undefined') {
      return new Promise(resolve => {
        _resolve = resolve
      })
    }
  }
```
`nextTick`这就是我们在上一节执行 `nextTick(flushSchedulerQueue)` 所用到的函数。它的逻辑也很简单，把传入的回调函数 `cb` 压入 `callbacks` 数组，最后一次性地根据 `useMacroTask` 条件执行 `macroTimerFunc` 或者是 `microTimerFunc`，而它们都会在下一个 `tick` 执行 `flushCallbacks`，`flushCallbacks` 的逻辑非常简单，对 `callbacks` 遍历，然后执行相应的回调函数。

**macroTimerFunc、microTimerFunc**
`next-tick.js` 申明了 `microTimerFunc` 和 `macroTimerFunc` 2 个变量，它们分别对应的是 `micro task` 的函数和 `macro task` 的函数。对于 `macro task` 的实现，优先检测是否支持原生 `setImmediate`，这是一个高版本 `IE` 和 `Edge `才支持的特性，不支持的话再去检测是否支持原生的 `MessageChannel`，如果也不支持的话就会降级为 `setTimeout` 0；而对于 `micro task` 的实现，则检测浏览器是否原生支持 `Promise，`不支持的话直接指向 `macro task` 的实现。

这里使用 `callbacks` 而不是直接在 `nextTick` 中执行回调函数的原因是保证在同一个 `tick` 内多次执行 `nextTick`，不会开启多个异步任务，而把这些异步任务都压成一个同步任务，在下一个 `tick` 执行完毕。

