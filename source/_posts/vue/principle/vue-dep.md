---
title: 深入Vue系列 Vue中的依赖收集
date: 2019-06-08 17:19:46
tags: [Vue]
categories: [Vue]
description: Vue首先会会通过Obsever,创建响应式数据,并且在getter中做依赖收集setter中派发。
---

[深入 Vue 系列 Vue 中的响应式对象](/blog/vue/vue-definedProperty.html)
[深入 Vue 系列 Vue 中的依赖收集](/blog/vue/vue-dep.html)
[深入 Vue 系列 Vue 中的派发更新](/blog/vue/vue-notify.html)

## 简介

通过响应式对象知道，每一个 `data` 的属相都会实例化一个 `Dep`，并且它的 `get` 函数中通过 `dep.depend`做依赖收集。通过下面这张图比较直观的看出依赖收集的过程：
![vue-Dep](../../images/vue/vue-Dep-1-1.jpg)
`defineReactive` 的功能就是定义一个响应式对象，给对象动态添加 `getter` 和 `setter`，它的定义在 `src/core/observer/index.js`，在 `getter` 中会做依赖收集，代码如下：

```javascript
/**
 * Define a reactive property on an Object.
 */
export function defineReactive(
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  /*在闭包中定义一个dep对象*/
  const dep = new Dep();

  const property = Object.getOwnPropertyDescriptor(obj, key);
  if (property && property.configurable === false) {
    return;
  }
  // 如果之前该对象已经预设了getter/setter则将其缓存，新定义的getter/setter中会将其执行
  // cater for pre-defined getter/setters
  const getter = property && property.get;
  const setter = property && property.set;
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key];
  }
  // 子对象递归调用 observe
  let childOb = !shallow && observe(val);
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {
      // 如果原本对象拥有getter方法则执行
      const value = getter ? getter.call(obj) : val;
      // 如果当前有watcher在读取当前值
      if (Dep.target) {
        // 那么进行依赖收集，dep.addSub
        dep.depend();
        if (childOb) {
          /*子对象进行依赖收集，其实就是将同一个watcher观察者实例放进了两个depend中，一个是正在本身闭包中的depend，另一个是子元素的depend*/
          childOb.dep.depend();
          // 这里是对数组进行劫持
          if (Array.isArray(value)) {
            /*是数组则需要对每一个成员都进行依赖收集，如果数组的成员还是数组，则递归。*/
            dependArray(value);
          }
        }
      }
      return value;
    },
    set: function reactiveSetter(newVal) {
      // 先getter
      const value = getter ? getter.call(obj) : val;
      /* eslint-disable no-self-compare */
      // 如果跟原来值一样则不管
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return;
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter();
      }
      // 如果原本对象拥有setter方法则执行
      if (setter) {
        setter.call(obj, newVal);
      } else {
        val = newVal;
      }
      /*新的值需要重新进行observe，保证数据响应式*/
      childOb = !shallow && observe(newVal);
      /*dep对象通知所有的观察者*/
      dep.notify();
    }
  });
}
```

`getter` 的时候进行依赖的收集，注意这里，只有在 `Dep.target` 中有值的时候才会进行依赖收集，这个 `Dep.target` 是在`Watcher`实例的 `get` 方法调用的时候 `pushTarget` 会把当前取值的`watcher`推入 `Dep.target`，原先的`watcher`压栈到 `targetStack` 栈中，当前取值的`watcher`取值结束后出栈并把原先的`watcher`值赋给 `Dep.target`，`cleanupDeps` 最后把新的 `newDeps` 里已经没有的`watcher`清空，以防止视图上已经不需要的无用`watcher`触发`setter` 的时候首先 `getter`，并且比对旧值没有变化则`return`，如果发生变更，则`dep`通知所有`subs`中存放的依赖本数据的`Watcher`实例 `update` 进行更新，这里 `update` 中会 `queueWatcher( )` 异步推送到调度者观察者队列 `queue` 中，在`nextTick`时 `flushSchedulerQueue`( ) 把队列中的`watcher`取出来执行 `watcher.run` 且执行相关钩子函数。

## Dep

`Dep` 是整个 `getter` 依赖收集的核心，它的定义在 `src/core/observer/dep.js` 中：

```javascript
import type Watcher from './watcher';
import { remove } from '../util/index';
let uid = 0;
/**
 * A dep is an observable that can have multiple
 * directives subscribing to it.
 */
export default class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;
  constructor() {
    this.id = uid++;
    // 订阅者的列表
    this.subs = [];
  }
  /*添加一个观察者对象*/
  addSub(sub: Watcher) {
    this.subs.push(sub);
  }
  /*移除一个观察者对象*/
  removeSub(sub: Watcher) {
    remove(this.subs, sub);
  }
  //给watcher收集依赖
  //这里是一个关键步骤，Dep.target是一个watcher实例
  //先将这个Dep实例添加到Watcher的依赖中
  //然后在watcher中调用dep.addSub将watcher添加到dep的订阅者中
  depend() {
    if (Dep.target) {
      Dep.target.addDep(this);
    }
  }
  /*通知所有订阅者*/
  notify() {
    // stabilize the subscriber list first
    const subs = this.subs.slice();
    //遍历这个依赖的所有订阅者watcher
    for (let i = 0, l = subs.length; i < l; i++) {
      //update()的最终目的就是要执行Watcher的getter
      //执行这个Watcher的getter的时候就会触发这个Watcher的依赖们的get()
      //然后重新收集依赖
      subs[i].update();
    }
  }
}
// the current target watcher being evaluated.
// this is globally unique because there could be only one
// watcher being evaluated at any time.
Dep.target = null;
// watcher栈
const targetStack = [];
/* 将watcher观察者实例设置给Dep.target，用以依赖收集。同时将该实例存入target栈中 */
export function pushTarget(_target: ?Watcher) {
  if (Dep.target) targetStack.push(Dep.target);
  // 改变目标指向
  Dep.target = _target;
}
/* 将观察者实例从target栈中取出并设置给Dep.target */
export function popTarget() {
  Dep.target = targetStack.pop();
}
```

- `Dep.target` 是一个静态属性，这是一个全局唯一 `Watcher`，这是一个非常巧妙的设计，因为在同一时间只能有一个全局的 `Watcher` 被计算。
- 定义一些 `Dep` 上得方法，添加依赖方法、移除方法、调用 `watcher.update()`的方法
- 实例属性 `subs` 保存 `watcher` 订阅者的列表

## watcher

`src/core/observer/watcher.js` 代码如下:

```javascript
    let uid = 0

/**
 * A watcher parses an expression, collects dependencies,
 * and fires callback when the expression value changes.
 * This is used for both the $watch() api and directives.
 */
export default class Watcher {

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    this.vm = vm
    if (isRenderWatcher) {
      vm._watcher = this
    }
    vm._watchers.push(this)
    // options
    this.cb = cb
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn) //  // 在get方法中执行
    }
    if (this.computed) {
      this.value = undefined
      this.dep = new Dep()
    } else {
      this.value = this.get() // 调用get方法
    }
  }

  /**
   * Evaluate the getter, and re-collect dependencies.
   */
  get () {
    /*将自身watcher观察者实例设置给Dep.target，用以依赖收集。*/
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      /*如果存在deep，则触发每个深层对象的依赖，追踪其变化*/
      if (this.deep) {
        /*递归每一个对象或者数组，触发它们的getter，使得对象或数组的每一个成员都被依赖收集，形成一个“深（deep）”依赖关系*/
        traverse(value)
      }
      /*将观察者实例从target栈中取出并设置给Dep.target*/
      popTarget()
      this.cleanupDeps()
    }
    return value
  }

  /**
   * Add a dependency to this directive.
   */
  addDep (dep: Dep) {...  }/* 添加一个依赖关系到Deps集合中 */

  /**
   * Clean up for dependency collection.
   */
  cleanupDeps () {  ...}/* 清理newDeps里没有的无用watcher依赖 */
  // ...
}

```

`Watcher`是一个观察者对象。依赖收集以后`Watcher`对象会被保存在`Dep`的`subs`中，数据变动的时候`Dep`会通知`Watcher`实例，然后由`Watcher`实例回调`cb`进行视图的更新。

## 触发流程

大致流程如下：

1. Vue 的 `mount` 过程是通过 `mountComponent` 函数

```javascript
// 初始化渲染 watcher
new Watcher(
  vm,
  updateComponent,
  noop,
  {
    before() {
      if (vm._isMounted) {
        callHook(vm, 'beforeUpdate');
      }
    }
  },
  true /* isRenderWatcher */
);
```

2. 初始化渲染`watcher`的时候，会执行`watcher`的构造函数，再会执行`this.get()`方法，进入 get 函数，首先执行：

```javascript
pushTarget(this);
// pushTarget方法实现
export function pushTarget(_target: Watcher) {
  // 如果存在Dep.target 就把 Dep.target 压入targetStack 栈，为了后面恢复使用
  if (Dep.target) targetStack.push(Dep.target);
  // 把 Dep.target 赋值为当前的渲染 watcher
  Dep.target = _target;
}
```

实际上就是把 `Dep.target` 赋值为当前的渲染 `watcher` 并压栈（为了恢复用）。

3. 接着会执行：

```javascript
// this.getter 对应就是 updateComponent 函数
value = this.getter.call(vm, vm);
// 所以就会执行
updateComponent = () => {
  // 其实执行的就是个
  vm._update(vm._render(), hydrating);
};
// vm._update(vm._render(), hydrating)
```

4. 执行`vm._render()` 这个方法会生成 渲染 `VNode`，并且在这个过程中会对 `vm` 上的**数据访问**，这个时候就触发了数据对象的 `getter`。每个对象属性的`getter`都持有一个`Dep`实例，在触发 getter 的时候就会调用`dep.depend()`方法，也就会执行`Dep.target.addDep(this)`。

5. 执行`Dep.target.addDep(this)` 这个时候`Dep.target`已经被赋值为渲染`watcher`，因为在上面执行了`pushTarget(this)`。执行 addDep 方法代码如下：

```javascript
    addDep (dep: Dep) {
    const id = dep.id
        if (!this.newDepIds.has(id)) {
            this.newDepIds.add(id)
            this.newDeps.push(dep)
            if (!this.depIds.has(id)) {
                dep.addSub(this)
            }
        }
    }
```

这时候会做一些逻辑判断（保证同一数据不会被添加多次）后执行 `dep.addSub(this)`，那么就会执行 `this.subs.push(sub)`，也就是说把当前的 `watcher` 订阅到这个数据持有的 `dep` 的 `subs` 中，这个目的是为后续数据变化时候能通知到哪些 `subs` 做准备。

6. 接着执行 `watcher` 中 `get()`方法中的 `traverse(value)`、`popTarget()`

```javascript
if (this.deep) {
  /*递归每一个对象或者数组，触发它们的getter，使得对象或数组的每一个成员都被依赖收集，形成一个“深（deep）”依赖关系*/
  traverse(value);
}
/*将观察者实例从target栈中取出并设置给Dep.target*/
popTarget();

// popTarget 实现 在Dep类中
// src/core/observer/dep.js
/* 将观察者实例从target栈中取出并设置给Dep.target */
export function popTarget() {
  Dep.target = targetStack.pop();
}
```

执行`traverse(value)`递归触发子项的`getter`完成依赖收集。再执行`popTarget()`实际上就是把 `Dep.target` 恢复成上一个状态，因为当前 `vm` 的数据依赖收集已经完成，那么对应的渲染`Dep.target` 也需要改变。

7. 接着执行`watcher` 中 `get()`方法中的 `this.cleanupDeps()`, `cleanupDeps()`函数定义在`watcher`类中。

```javascript
    // cleanupDeps 函数
    cleanupDeps () {
        let i = this.deps.length
        while (i--) {
            const dep = this.deps[i]
            if (!this.newDepIds.has(dep.id)) {
                dep.removeSub(this)
            }
        }
        let tmp = this.depIds
        this.depIds = this.newDepIds
        this.newDepIds = tmp
        this.newDepIds.clear()
        tmp = this.deps
        this.deps = this.newDeps
        this.newDeps = tmp
        this.newDeps.length = 0
    }
```

首先理解四个变量`depIds`、`newDepIds` 、`deps`、 `newDeps`。

- `depIds Hash`表，用于快速查找（`dep`）
- `newDepIds Hash`表，用于快速查找（`newDeps`）
- `deps` 缓存上一轮执行观察者函数用到的`dep`实例
- `newDeps` 存储本轮执行观察者函数用到的`dep`实例

在执行 `cleanupDeps` 函数的时候，会首先遍历 `deps`，移除对 `dep.subs` 数组中 `Wathcer` 的订阅，然后把 `newDepIds` 和 `depIds` 交换，`newDeps` 和 `deps` 交换，并把 `newDepIds` 和 `newDeps` 清空。

**为什么清除 Deps**
因此`Vue`设计了在每次**添加完新的**订阅，会**移除掉旧的**订阅，这样就保证了在我们刚才的场景中，如果渲染 b 模板的时候去修改 a 模板的数据，a 数据订阅回调已经被移除了，所以不会有任何浪费。

## 总结

其实在 `Vue` 中初始化渲染时，视图上绑定的数据就会实例化一个 `Watcher`，**依赖收集**就是是通过属性的 `getter` 函数完成的，`Observer` 、`Watcher` 、`Dep` 都与依赖收集相关。其中 `Observer` 与 `Dep` 是**一对一**的关系， `Dep` 与 `Watcher` 是**多对多**的关系，Dep 则是 `Observer` 和 `Watcher` 之间的**纽带**。

## 参考

[依赖收集](https://ustbhuangyi.github.io/vue-analysis/reactive/getters.html#dep)
[深入理解 Vue 响应式原理](http://jungahuang.com/2018/02/07/About-responsive-of-Vue/)
[Vue2.0 源码阅读：响应式原理](https://zhouweicsu.github.io/blog/2017/03/07/vue-2-0-reactivity/#more)
[Vue 源码阅读-依赖收集原理](https://juejin.im/post/5b40c8495188251af3632dfa)
