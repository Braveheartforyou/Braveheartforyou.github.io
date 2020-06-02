---
title: 深入Vue系列 Vue中的响应式对象
date: 2019-06-04 15:32:43
tags: [Vue]
categories: [Vue]
description: Vue.js 实现响应式的核心是利用了 ES5 的 Object.defineProperty，这也是为什么 Vue.js 不能兼容 IE8 及以下浏览器的原因。Object.defineProperty是不能观测数据的一些方法的，为什么Vue中却可以实现。
---

## 简介

Vue 的**核心响应式**是通过`Obeject.defineProperty`方法来实现的。 而[Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)是 ES5 中无法**shim**的特性，这也就是为什么 Vue 不支持 IE8 以及更低版本浏览器的原因。

**Object.defineProperty**
Object.defineProperty 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象，先来看一下它的语法：

```javascript
Object.defineProperty(obj, prop, descriptor);
```

obj 是要在其上定义属性的对象；prop 是要定义或修改的属性的名称；descriptor 是将被定义或修改的属性描述符。

由于 Vue 会在初始化实例时对属性执行 **getter/setter** 转化过程，所以属性必须在 `data` 对象上存在才能让 Vue 转换它，这样才能让它是响应的。
响应式原理大致流程如下图所示：
![vue-defineProperty](../../../images/vue/vue-defineProperty-1-1.png)
Vue 数据响应式变化主要涉及**Observer、Watcher、Dep**这三个主要的类。这里主要是响应式对象，后面分别会记录它的依赖收集、派发更新、三种 Watcher。
把普通对象改造为**响应式对象**在 Vue 中的大致流程为：

- `initState`(初始化数据)
- `Observer(`劫持数据)
- `defineReactive`(依赖收集、派发更新)

在 getter 对象中又会依赖收集，在 setter 中派发更新。

> Vue-version(2.6.10)

## initState(初始化数据)

那么我们从一个简单的 Vue 实例的代码来分析 Vue 的响应式原理:

```javascript
  <template>
    <div id="app">
      {{msg}}
    </div>
  </template>

  <script>
    export default {
      name: 'App',
      data () {
        return {
          msg: 'initState'
        }
      }
    }
  </script>
```

在**Vue**的初始化阶段，`_init` 方法执行的时候，会执行 `initState(vm)` 方法，它的定义在 **src/core/instance/state.js**中。

```javascript
export function initState(vm: Component) {
  vm._watchers = [];
  const opts = vm.$options;
  // 初始化props
  if (opts.props) initProps(vm, opts.props);
  // 初始化methods
  if (opts.methods) initMethods(vm, opts.methods);
  // 初始化data
  if (opts.data) {
    initData(vm);
  } else {
    /*该组件没有data的时候绑定一个空对象*/
    observe((vm._data = {}), true /* asRootData */);
  }
  // 初始化computed
  if (opts.computed) initComputed(vm, opts.computed);
  // 初始化手写的watcher
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch);
  }
}
```

`initState` 方法主要是对 `props`、`methods`、`data`、`computed` 和 `wathcer` 等属性做了初始化操作。主要看`initData`。

### initData

`initData`方法如下：

```javascript
function initData(vm: Component) {
  let data = vm.$options.data;
  // 判断data是否为function 如果是直接执行，如果不是获取data
  data = vm._data = typeof data === 'function' ? getData(data, vm) : data || {};
  if (!isPlainObject(data)) {
    data = {};
    process.env.NODE_ENV !== 'production' &&
      warn(
        'data functions should return an object:\n' +
          'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
        vm
      );
  }
  // proxy data on instance
  const keys = Object.keys(data);
  const props = vm.$options.props;
  const methods = vm.$options.methods;
  let i = keys.length;
  while (i--) {
    const key = keys[i];
    if (process.env.NODE_ENV !== 'production') {
      if (methods && hasOwn(methods, key)) {
        warn(
          `Method "${key}" has already been defined as a data property.`,
          vm
        );
      }
    }
    if (props && hasOwn(props, key)) {
      process.env.NODE_ENV !== 'production' &&
        warn(
          `The data property "${key}" is already declared as a prop. ` +
            `Use prop default value instead.`,
          vm
        );
    } else if (!isReserved(key)) {
      // 通过 proxy 把每一个值 vm._data.xxx 都代理到 vm.xxx 上
      proxy(vm, `_data`, key);
    }
  }
  // observe data
  // 另一个是调用 observe 方法观测整个 data 的变化，把 data 也变成响应式
  observe(data, true /* asRootData */);
}
```

在 initData 过程中主要做了两件事：

- 通过 `proxy` 把每一个值 vm.\_data.[key] 都代理到 vm.[key] 上；
- 调用 `observe` 方法观测整个 `data` 的变化，把 `data` 也变成响应式（可观察），可以通过 `vm._data.[key]` 访问到定义 `data` 返回函数中对应的属性。

## Observer(劫持数据)

observe 的功能就是用来监测数据的变化，它的定义在 src/core/observer/index.js 中：

```javascript
/**
 * Attempt to create an observer instance for a value,
 * returns the new observer if successfully observed,
 * or the existing observer if the value already has one.
 */
export function observe(value: any, asRootData: ?boolean): Observer | void {
  // 判断是否为VNode 如果是直接返回
  if (!isObject(value) || value instanceof VNode) {
    return;
  }
  let ob: Observer | void;
  // 如果存在__ob__ 直接返回
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__;
  } else if (
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    // 如果不存在 实例化一个 Observer
    ob = new Observer(value);
  }
  if (asRootData && ob) {
    ob.vmCount++;
  }
  // 返回 Observer实例
  return ob;
}
```

`observe` 方法的作用就是给非 VNode 的对象类型数据添加一个 `Observer`，如果已经添加过则直接返回，否则在满足一定条件下去实例化一个 `Observer` 对象实例。接下来我们来看一下 `Observer` 的作用。

**Observer**
observe 的功能就是用来监测数据的变化，它的定义在 src/core/observer/index.js 中：
`Observer` 是一个类，它的作用是给对象的属性添加 `getter` 和 `setter`，用于**依赖收集**和**派发更新**：

```javascript
/**
 * Observer class that is attached to each observed
 * object. Once attached, the observer converts the target
 * object's property keys into getter/setters that
 * collect dependencies and dispatch updates.
 */
// 给对象的属性添加 getter 和 setter， 用于依赖收集、派发更新
class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that have this object as root $data

  constructor(value: any) {
    this.value = value;
    // 实例化 Dep 对象
    this.dep = new Dep();
    this.vmCount = 0;
    // 把自身实例添加到数据对象 value 的 __ob__ 属性上
    def(value, '__ob__', this);
    // 判断value 是否为Array 做不同的调用
    if (Array.isArray(value)) {
      if (hasProto) {
        protoAugment(value, arrayMethods);
      } else {
        copyAugment(value, arrayMethods, arrayKeys);
      }
      this.observeArray(value);
    } else {
      this.walk(value);
    }
  }

  /**
   * Walk through all properties and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */
  //  遍历对象属性 并且每个属性添加 getter、setter
  walk(obj: Object) {
    const keys = Object.keys(obj);
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i]);
    }
  }

  /**
   * Observe a list of Array items.
   */
  observeArray(items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i]);
    }
  }
}
```

上面这个方法做的事情如下：

- 实例化 `Dep` 对象
- 通过执行 `def` 函数把自身实例添加到数据对象 `value`的 **ob** 属性上
- 对 `value` 做判断，对于数组会调用 `observeArray` 方法，否则对纯对象调用 `walk`方法。

> 可以看到 `observeArray` 是遍历数组再次调用 `observe` 方法，而 `walk` 方法是遍历对象的 `key` 调用 `defineReactive` 方法，那么我们来看一下这个方法是做什么的。

**def 方法**
def 的定义在 src/core/util/lang.js 中：

```javascript
/**
 * Define a property.
 */
export function def(obj: Object, key: string, val: any, enumerable?: boolean) {
  // 自身实例添加到数据对象 value 的 __ob__ 属性上
  Object.defineProperty(obj, key, {
    value: val,
    enumerable: !!enumerable,
    writable: true,
    configurable: true
  });
}
```

开发中输出 `data` 上对象类型的数据，会发现该对象多了一个 `__ob__` 的属性。

## defineReactive(依赖收集、派发更新)

`defineReactive` 的功能就是定义一个响应式对象，给对象动态添加 `getter` 和 `setter`，它的定义在 `src/core/observer/index.js` 中：

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
      /*dep对象通知所有的订阅者*/
      dep.notify();
    }
  });
}
```

这里暂时不对**依赖收集**、**派发更新**、**Dep**讲述只记录**数据劫持**的过程记录，后面文章记录具体的**依赖收集**、**派发更新**、**watcher**、**Dep**、记录。
`defineReactive` 函数最开始初始化 `Dep` 对象的实例，接着拿到 `obj` 的属性描述符，然后对子对象递归调用 `observe` 方法，这样就保证了无论 `obj` 的结构多复杂，它的所有**子属性**也能变成响应式的对象，这样我们访问或修改 `obj` 中一个嵌套较深的属性，也能触发 `getter` 和 `setter`。最后利用 `Object.defineProperty` 去给 obj 的属性 key 添加 getter 和 setter。

## 数据观测的特殊处理

访问对象属性，其取值与赋值操作，都能被`Object.defineProperty()`成功拦截，但是`Object.defineProperty()`在处理数组上却存在一些问题。通过调用数据原型上的`push`、`pop`、`shift`、`unshift`、`splice`、`sort`、 `reverse`等方法不能被观测到[兼容性问题](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty#Cross-browser_concerns)。

在**Vue**中是对数组的原型上述方法做了一些增强操作。即**保留**原来操作的基础上，植入**Vue**的特定的操作代码。
代码在 src/core/observer/index.js 中定义：

```javascript
const arrayProto = Array.prototype;
export const arrayMethods = Object.create(arrayProto);

const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
];

/**
 * Intercept mutating methods and emit events
 */
methodsToPatch.forEach(function (method) {
  // cache original method
  const original = arrayProto[method];
  def(arrayMethods, method, function mutator(...args) {
    const result = original.apply(this, args);
    const ob = this.__ob__;
    let inserted;
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args;
        break;
      case 'splice':
        inserted = args.slice(2);
        break;
    }
    // observeArray方法进行一遍观测处理
    if (inserted) ob.observeArray(inserted);
    // notify change
    /*dep对象通知所有的订阅者*/
    ob.dep.notify();
    return result;
  });
});
```

保留数组原来的操作 `push`、`unshift`、`splice`这些方法，会带来**新的数据元素**，而新带来的数据元素，我们是有办法得知的（即为传入的参数）那么新增的元素也是需要被配置为**可观测数据**的，这样子后续数据的变更才能得以处理。所以要对新增的元素调用`observer`实例上的`observeArray`方法进行一遍观测处理由于数组变更了，那么就需要通知观察者，所以通过`ob.dep.notify()`对数组的观察者`watchers`进行通知。

## 总结

从初始化`initData`，到核心就是利用 `Object.defineProperty` 给数据添加了 `getter` 和 `setter`，目的就是为了在我们访问数据以及写数据的时候能自动执行一些逻辑：`getter` 做的事情是**依赖收集**，`setter` 做的事情是**派发更新**。

## 参考

[Vue 源码](https://github.com/vuejs/vue/tree/dev/src)
[响应式对象](https://ustbhuangyi.github.io/vue-analysis/reactive/reactive-object.html#object-defineproperty)
[Vue 响应式原理其实很好懂](https://mp.weixin.qq.com/s/-pZnJWxrRlz-rOGhdshe7g)
[深入理解 Vue 响应式原理](http://jungahuang.com/2018/02/07/About-responsive-of-Vue/)
[深入解析 Vue 依赖收集原理](https://juejin.im/entry/5bdab35d6fb9a0224e0e5794)
