---
title: Vue中的派发更新
date: 2019-06-11 23:43:54
tags: [Vue]
categories: [Vue]
description: 响应式数据依赖收集过程，收集的目的就是为了当我们修改数据的时候，可以对相关的依赖派发更新。
---
## 简介
通过在`defineReactive`观测的`data`子项中的`getter`函数中完成[依赖收集](/blog/vue/vue-dep.html)，在`defineReactive`观测的`data`子项中的`setter`函数中完成依赖派发。
首先看一下defineReactive中的setter：
```javascript
  /**
 * Define a reactive property on an Object.
 */
export function defineReactive () {
   // 子对象递归调用 observe
  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () { ... }, // 收集依赖
    set: function reactiveSetter (newVal) {
      // 先getter
      const value = getter ? getter.call(obj) : val 
      /* eslint-disable no-self-compare */
      // 如果跟原来值一样则不管
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      // 如果原本对象拥有setter方法则执行
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
       /*新的值需要重新进行observe，保证数据响应式*/
      childOb = !shallow && observe(newVal)
       /*dep对象通知所有的订阅者*/
      dep.notify()
    }
  })
}
```
主要做了两个步：
- 一个是 childOb = !shallow && observe(newVal)，如果 shallow 为 false 的情况，会对新设置的值变成一个响应式对象；
- dep.notify() dep对象通知所有的订阅者

## 执行过程分析
- 执行dep.notify()
- 执行watcher.update() 
- 执行queueWatcher(this)
- 执行nextTick(flushSchedulerQueue)
- 执行watcher.run()
- 执行watcher.get()

### 1. **执行dep.notify()**
  当我们对响应式数据做了修改，就会触发setter的逻辑，最后调用`dep.notify()`方法，它是`Dep`的一个实例方法。代码在`scr/core/observer.js`中:
  ```javascript
    class Dep {
      notify () {
        // stabilize the subscriber list first
        const subs = this.subs.slice()
        for (let i = 0, l = subs.length; i < l; i++) {
          // 循环调用watcher.update()方法
          subs[i].update()
        }
      }
    }
  ```
  主要做的事情是：
  - 遍历所有的 `subs`，也就是 `Watcher` 的实例数组，然后调用每一个 `watcher `的 `update` 方法。

### 2. **执行watcher.update()**
  watcher类定义在`src/core/observer/watcher.js`中：
```javascript
  class Watcher {
    // ...
    update () {
      /* istanbul ignore else */
      // 判断是否为computed watcher
      if (this.computed) {
        // A computed property watcher has two modes: lazy and activated.
        // It initializes as lazy by default, and only becomes activated when
        // it is depended on by at least one subscriber, which is typically
        // another computed property or a component's render function.
        if (this.dep.subs.length === 0) {
          // In lazy mode, we don't want to perform computations until necessary,
          // so we simply mark the watcher as dirty. The actual computation is
          // performed just-in-time in this.evaluate() when the computed property
          // is accessed.
          this.dirty = true
        } else {
          // In activated mode, we want to proactively perform the computation
          // but only notify our subscribers when the value has indeed changed.
          this.getAndInvoke(() => {
            this.dep.notify()
          })
        }
        // 判断是否有sync 修饰符
      } else if (this.sync) {
        this.run()
      } else {
        // 执行queueWatcher
        queueWatcher(this)
      }
    }
  }  
```
  这里的创建时渲染`watcher`所以会走 `queueWatcher(this)`的逻辑。这里主要做了：
- 判断是否为`computed Watcher`、是否有`sync` 修饰符，如果都不满足执行`queueWatcher`

### 3. **执行queueWatcher(this)**
`queueWatcher` 的定义在 `src/core/observer/scheduler.js` 中：
```javascript
  // watcher队列
  const queue: Array<Watcher> = []
  // has 对象保证同一个 Watcher 只添加一次
  let has: { [key: number]: ?true } = {}
  let waiting = false
  let flushing = false
  /**
  * Push a watcher into the watcher queue.
  * Jobs with duplicate IDs will be skipped unless it's
  * pushed when the queue is being flushed.
  */
  export function queueWatcher (watcher: Watcher) {
    const id = watcher.id
    // 保证watcher只添加一次
    if (has[id] == null) {
      has[id] = true
      if (!flushing) {
        queue.push(watcher)
      } else {
        // if already flushing, splice the watcher based on its id
        // if already past its id, it will be run next immediately.
        let i = queue.length - 1
        while (i > index && queue[i].id > watcher.id) {
          i--
        }
        queue.splice(i + 1, 0, watcher)
      }
      // queue the flush
      if (!waiting) {
        waiting = true
        //  保证对 nextTick(flushSchedulerQueue) 的调用逻辑只有一次
        nextTick(flushSchedulerQueue)
      }
    }
  }
```
    上面代码主要做了如下：
  - 把`watcher`添加到一个队列里
  - 在 nextTick 后执行 flushSchedulerQueue

  > has 对象保证同一个 Watcher 只添加一次
  > 判断是否为渲染`watcher`
  > 通过 `waiting` 保证对 `nextTick(flushSchedulerQueue)` 的调用逻辑只有一次

### 4. **执行nextTick(flushSchedulerQueue)**
  [nextTick](/blog/vue/vue-next-tick.html)这里单独记录，主要看`flushSchedulerQueue`，代码在`src/core/observer/scheduler.js`中：
  ```javascript
    let flushing = false
    let index = 0
    /**
    * Flush both queues and run the watchers.
    */
    function flushSchedulerQueue () {
      flushing = true
      let watcher, id
      // Sort queue before flush.
      // This ensures that:
      // 1. Components are updated from parent to child. (because parent is always
      //    created before the child)
      // 2. A component's user watchers are run before its render watcher (because
      //    user watchers are created before the render watcher)
      // 3. If a component is destroyed during a parent component's watcher run,
      //    its watchers can be skipped.
      // 对watcher 根据id进行排序，保证从父到子执行
      queue.sort((a, b) => a.id - b.id)

      // do not cache length because more watchers might be pushed
      // as we run existing watchers
      // 遍历queue队列 并且执行 watcher的 run方法
      for (index = 0; index < queue.length; index++) {
        watcher = queue[index]
        if (watcher.before) {
          watcher.before()
        }
        id = watcher.id
        has[id] = null
        watcher.run()
        // in dev build, check and stop circular updates.
        // 防止死循环
        if (process.env.NODE_ENV !== 'production' && has[id] != null) {
          circular[id] = (circular[id] || 0) + 1
          if (circular[id] > MAX_UPDATE_COUNT) {
            warn(
              'You may have an infinite update loop ' + (
                watcher.user
                  ? `in watcher with expression "${watcher.expression}"`
                  : `in a component render function.`
              ),
              watcher.vm
            )
            break
          }
        }
      }

      // keep copies of post queues before resetting state
      const activatedQueue = activatedChildren.slice()
      const updatedQueue = queue.slice()
      // 把这些控制流程状态的一些变量恢复到初始值，把 watcher 队列清空。
      resetSchedulerState()

      // call component updated and activated hooks
      callActivatedHooks(activatedQueue)
      callUpdatedHooks(updatedQueue)

      // devtool hook
      /* istanbul ignore if */
      if (devtools && config.devtools) {
        devtools.emit('flush')
      }
    }
  ```

  上面的代码主要做了以下几步：
  - 队列排序
    1. .组件的更新**由父到子**；因为父组件的创建过程是先于子的，所以 `watcher` 的创建也是**先父后子**，执行顺序也应该保持**先父后子**。
    2. 用户的**自定义** `watcher` 要**优先**于**渲染** `watcher` 执行；因为用户自定义 `watcher` 是在渲染 `watcher` 之前创建的。
    3. 如果一个组件在父组件的 `watcher` 执行期间被销毁，那么它对应的 `watcher` 执行都可以被跳过，所以父组件的 `watcher` 应该先执行。
  - 队列遍历
  在对 `queue` 排序后，接着就是要对它做遍历，拿到对应的 `watcher`，执行 `watcher.run()`。这里需要注意一个细节，在遍历的时候每次都会对 `queue.length` 求值，因为在 `watcher.run()` 的时候，很可能用户会再次添加新的 `watcher`，这样会再次执行到 `queueWatcher`，如下：
  ```javascript
    export function queueWatcher (watcher: Watcher) {
      const id = watcher.id
      if (has[id] == null) {
        has[id] = true
        if (!flushing) {
          queue.push(watcher)
        } else {
          // if already flushing, splice the watcher based on its id
          // if already past its id, it will be run next immediately.
          let i = queue.length - 1
          while (i > index && queue[i].id > watcher.id) {
            i--
          }
          queue.splice(i + 1, 0, watcher)
        }
        // ...
      }
    }
  ```
  可以看到，这时候 `flushing` 为 `true`，就会执行到 `else` 的逻辑，然后就会从后往前找，找到第一个待插入 `watcher` 的 `id` 比当前队列中 `watcher` 的 `id` 大的位置。把 `watcher` 按照 `id`的插入到队列中，因此 `queue` 的长度发生了变化。
  - 状态恢复
  这个过程就是执行 `resetSchedulerState` 函数，它的定义在 `src/core/observer/scheduler.js` 中:
  ```javascript
    const queue: Array<Watcher> = []
    let has: { [key: number]: ?true } = {}
    let circular: { [key: number]: number } = {}
    let waiting = false
    let flushing = false
    let index = 0
    /**
    * Reset the scheduler's state.
    */
    function resetSchedulerState () {
      index = queue.length = activatedChildren.length = 0
      has = {}
      if (process.env.NODE_ENV !== 'production') {
        circular = {}
      }
      waiting = flushing = false
    }
  ```
  逻辑非常简单，就是把这些控制流程状态的一些**变量恢复到初始值**，把 `watcher` 队列清空。

### 5. **执行watcher.run()**
  接下来我们继续分析 `watcher.run()` 的逻辑，它的定义在 `src/core/observer/watcher.js` 中。
  ```javascript
    class Watcher {
      /**
      * Scheduler job interface.
      * Will be called by the scheduler.
      */
      run () {
        if (this.active) {
          this.getAndInvoke(this.cb)
        }
      }

      getAndInvoke (cb: Function) {
        // 执行watcher.run()函数
        const value = this.get()
        if (
          value !== this.value ||
          // Deep watchers and watchers on Object/Arrays should fire even
          // when the value is the same, because the value may
          // have mutated.
          isObject(value) ||
          this.deep
        ) {
          // set new value
          const oldValue = this.value
          this.value = value
          this.dirty = false
          if (this.user) {
            try {
              cb.call(this.vm, value, oldValue)
            } catch (e) {
              handleError(e, this.vm, `callback for watcher "${this.expression}"`)
            }
          } else {
            // 执行watcher 的回调函数
            cb.call(this.vm, value, oldValue)
          }
        }
      }
    }
  ```
  主要做了如下：
  - 执行 `this.getAndInvoke` 方法，并传入 `watcher` 的回调函数
  - `getAndInvoke` 函数先 通过 `this.get()` 得到它当前的值，然后做判断，如果满足**新旧值不等**、**新值是对象类型**、**deep** 模式任何一个条件，则执行 `watcher` 的回调
  > 注意回调函数执行的时候会把**第一个**和**第二个**参数传入新值 `value` 和旧值 `oldValue`，这就是当我们添加自定义 `watcher` 的时候能在**回调函数**的参数中拿到**新旧值**的原因。

### 6. **执行watcher.get()**
  那么对于渲染 watcher 而言，它在执行 this.get() 方法求值的时候，会执行 getter 方法：
  ```javascript
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  ```
  所以这就是当我们去修改组件相关的**响应式数据**的时候，会触发组件**重新渲染**的原因，接着就会重新执行 `patch` 的过程。

## 总结
派发更新的过程是：修改数据触发观测数据的`setter`方法=>调用`dep.notify()` 通知所有的订阅者=>循环调用**订阅者**`watcher.update()`=>循环调用`watcher.update()`=>调用`queueWatcher()`,添加`watcher`到一个`queue`队列中=>调用`nextTick(flushSchedulerQueue)` ，对`queue`根据`id`排序 => 调用`watcher.run()`方法 => 触发`watcher.get()`方法=>调用`vm._update(vm._render(), hydrating)`。

## 参考
> [派发更新](https://ustbhuangyi.github.io/vue-analysis/reactive/setters.html#%E8%BF%87%E7%A8%8B%E5%88%86%E6%9E%90)
> [深入理解Vue响应式原理](http://jungahuang.com/2018/02/07/About-responsive-of-Vue/)