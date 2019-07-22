---
title: Vue中的依赖收集
date: 2019-06-08 17:19:46
tags: [Vue]
categories: [Vue]
description: Vue首先会会通过Obsever,创建响应式数据,并且在getter中做依赖收集setter中派发。
---
## 简介
通过响应式对象知道，每一个data的属相都会实例化一个Dep，并且它的get函数中通过dep.depend做依赖收集。通过下面这张图比较直观的看出依赖收集的过程：
![vue-defineProperty](../../images/vue/vue-defineProperty-1-1.png)
`defineReactive` 的功能就是定义一个响应式对象，给对象动态添加 `getter` 和 `setter`，它的定义在 src/core/observer/index.js，在getter中会做依赖收集，代码如下：
```javascript
  /**
 * Define a reactive property on an Object.
 */
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  /*在闭包中定义一个dep对象*/
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }
  // 如果之前该对象已经预设了getter/setter则将其缓存，新定义的getter/setter中会将其执行
  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }
   // 子对象递归调用 observe
  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      // 如果原本对象拥有getter方法则执行
      const value = getter ? getter.call(obj) : val 
      // 如果当前有watcher在读取当前值
      if (Dep.target) { 
        // 那么进行依赖收集，dep.addSub
        dep.depend() 
        if (childOb) {
          /*子对象进行依赖收集，其实就是将同一个watcher观察者实例放进了两个depend中，一个是正在本身闭包中的depend，另一个是子元素的depend*/
          childOb.dep.depend()
          // 这里是对数组进行劫持
          if (Array.isArray(value)) {
            /*是数组则需要对每一个成员都进行依赖收集，如果数组的成员还是数组，则递归。*/
            dependArray(value)
          }
        }
      }
      return value
    },
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
       /*dep对象通知所有的观察者*/
      dep.notify()
    }
  })
}
```
`getter` 的时候进行依赖的收集，注意这里，只有在 `Dep.target` 中有值的时候才会进行依赖收集，这个 `Dep.target` 是在`Watcher`实例的 `get` 方法调用的时候 `pushTarget` 会把当前取值的`watcher`推入 `Dep.target`，原先的`watcher`压栈到 `targetStack` 栈中，当前取值的`watcher`取值结束后出栈并把原先的`watcher`值赋给 `Dep.target`，`cleanupDeps` 最后把新的 `newDeps` 里已经没有的`watcher`清空，以防止视图上已经不需要的无用`watcher`触发`setter` 的时候首先 `getter`，并且比对旧值没有变化则`return`，如果发生变更，则`dep`通知所有`subs`中存放的依赖本数据的`Watcher`实例 `update` 进行更新，这里 `update` 中会 `queueWatcher( )` 异步推送到调度者观察者队列 `queue` 中，在`nextTick`时 `flushSchedulerQueue`( ) 把队列中的`watcher`取出来执行 `watcher.run` 且执行相关钩子函数。