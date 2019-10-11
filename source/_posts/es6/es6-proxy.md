---
title: ES系列 Proxy
date: 2019-09-20 18:33:44
tags: [ECMAScript6]
categories: [ECMAScript6]
description: Object.defineProperty和Proxy分别是什么，他们之间的区别和优缺点，vue源码中的为什么把Object.defineProperty用proxy重写。
---

> [ES系列 Object.defineProperty](/blog/es6/es6-definedproperty.html)
> [ES系列 Proxy](/blog/es6/es6-proxy.html)
> [ES系列 Object.defineProperty和Proxy的对比](/blog/es6/es6-definedproperty-proxy.html)

## 简介

在上一篇[ES系列 Object.defineProperty](/blog/es6/es6-definedproperty.html)已经介绍过了**Object.defineProperty**相关内容，这篇文章中会介绍在**Vue 3.x**中代替`Object.defineProperty`的`Proxy`。
最后会介绍它们之间的**优缺点**和实现**双向绑定简单实例**。

## Proxy

从字面上可以把`Proxy`理解为**代理**，但是感觉解释为类似于**代理模式**会更贴合一点。**"阮大佬：Proxy 用于修改某些操作的默认行为，等同于在语言层面做出修改，所以属于一种“元编程”（meta programming），即对编程语言进行编程。"**

首先要了解其中的**术语**。

- `handler`: 包含陷阱（traps）的占位符对象。
- `traps`: 提供属性访问的方法。这类似于操作系统中捕获器的概念。
- `target`: 代理虚拟化的对象。它通常用作代理的存储后端。根据目标验证关于对象不可扩展性或不可配置属性的不变量（保持不变的语义）。

`new Proxy(target, handler)`方式创建一下新的对象，参数如下：

- `target (Object)`: 用`Proxy`包装的目标对象（可以是**任何类型的对象，包括原生数组，函数，甚至另一个代理**）。
- `handler(Object)`: 一个对象，其属性是当执行一个操作时定义代理的行为的函数。

## handler

`handler` 对象是一个占位符对象，它包含**Proxy**的捕获器。同时**handler**对象包含了用于拦截的**13**种操作。如下：
大致可以分为一类**代理对象`自身属性`操作拦截**：

|  拦截方法名  |   详情   |
|:----------:|:----------:|
| `handler.get(target, property, receiver)` | 在**读取**代理对象的某个**属性时**触发该操作，比如在执行 `proxy.foo` 时。 |
| `handler.set(target, property, value, receiver)` | 在给代理对象的某个**属性赋值时**触发该操作，比如在执行 `proxy.foo = 1` 时。 |
| `handler.has(target, prop)` | 在判断代理对象**是否拥有某个属性时**触发该操作，比如在执行 `"foo" in proxy` 时。 |
| `handler.defineProperty(target, property, descriptor)` | 在**定义代理对象某个属性时的属性描述时**触发该操作，比如在执行 `Object.defineProperty(proxy, "foo", {})` 时。 |
| `handler.deleteProperty(target, property)` | 在**删除代理对象的某个属性时**触发该操作，即使用`delete`运算符，比如在执行 `delete proxy.foo` 时。 |
| `handler.getOwnPropertyDescriptor(target, prop)` | 在**获取代理对象某个属性的属性描述时**触发该操作，比如在执行 `Object.getOwnPropertyDescriptor(proxy, "foo")` 时。 |

<!-- - `handler.get(target, property, receiver)`: 在**读取**代理对象的某个**属性时**触发该操作，比如在执行 `proxy.foo` 时。
- `handler.set(target, property, value, receiver)`: 在给代理对象的某个**属性赋值时**触发该操作，比如在执行 `proxy.foo = 1` 时。 -->
<!-- - `handler.has(target, prop)`: 在判断代理对象**是否拥有某个属性时**触发该操作，比如在执行 `"foo" in proxy` 时。 -->
<!-- - `handler.defineProperty(target, property, descriptor)`: 在**定义代理对象某个属性时的属性描述时**触发该操作，比如在执行 `Object.defineProperty(proxy, "foo", {})` 时。 -->
<!-- - `handler.deleteProperty(target, property)`: 在**删除代理对象的某个属性时**触发该操作，即使用`delete`运算符，比如在执行 `delete proxy.foo` 时。
- `handler.getOwnPropertyDescriptor(target, prop)`:在**获取代理对象某个属性的属性描述时**触发该操作，比如在执行 `Object.getOwnPropertyDescriptor(proxy, "foo")` 时。 -->

另一类**代理对象`自身`操作拦截**:

|  拦截方法名  |   详情   |
|:----------:|:----------:|
| `handler.getPrototypeOf(target)` | 在**读取代理对象的原型时**触发该操作，比如在执行 `Object.getPrototypeOf(proxy)` 时。 |
| `handler.setPrototypeOf(target, prototype)` | 在**设置代理对象的原型时**触发该操作，比如在执行 `Object.setPrototypeOf(proxy, null)` 时。 |
| `handler.isExtensible(target)` | 在判断一个**代理对象是否是可扩展时**触发该操作，比如在执行 `Object.isExtensible(proxy)` 时。 |
| `handler.preventExtensions(target)` | 在让一个**代理对象不可扩展时**触发该操作，比如在执行 `Object.preventExtensions(proxy)` 时。 |
| `handler.apply(target, thisArg, argumentsList)` | 拦截 **Proxy 实例作为函数调用**的操作，比如`proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)`。 |
| `handler.ownKeys(target)` | 拦截`Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in`循环，**返回一个数组**。该方法返回目标对象**所有自身的属性的属性名**，而Object.keys()的返回结果仅包括**目标对象自身的可遍历属性**。 |
| `handler.construct(target, argumentsList, newTarget)` | 拦截 **Proxy 实例作为构造函数调用**的操作，比如`new proxy(...args)`。 |

<!-- - `handler.getPrototypeOf(target)`: 在**读取代理对象的原型时**触发该操作，比如在执行 `Object.getPrototypeOf(proxy)` 时。 -->
<!-- - `handler.setPrototypeOf(target, prototype)`: 在**设置代理对象的原型时**触发该操作，比如在执行 `Object.setPrototypeOf(proxy, null)` 时。 -->
<!-- - `handler.isExtensible(target)`: 在判断一个**代理对象是否是可扩展时**触发该操作，比如在执行 `Object.isExtensible(proxy)` 时。 -->
<!-- - `handler.preventExtensions(target)`: 在让一个**代理对象不可扩展时**触发该操作，比如在执行 `Object.preventExtensions(proxy)` 时。 -->
<!-- - `handler.apply(target, thisArg, argumentsList)`: 拦截 **Proxy 实例作为函数调用**的操作，比如`proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)`。 -->
<!-- - `handler.ownKeys(target)`: 拦截`Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in`循环，**返回一个数组**。该方法返回目标对象**所有自身的属性的属性名**，而Object.keys()的返回结果仅包括**目标对象自身的可遍历属性**。
- `handler.construct(target, argumentsList, newTarget)`: 拦截 **Proxy 实例作为构造函数调用**的操作，比如`new proxy(...args)`。 -->

可以看到`Proxy`的拦截方法上就比`Object.defineProperty`的配置多很多，并且在最近的浏览器支持中也是各大浏览器上对`Proxy`大理支持，优化性能等等。

## 代理对象自身属性操作拦截

### get()

`get(target, property, receiver)`方法用于拦截某个属性的读取操作，可以接受三个参数，依次为**目标对象**、**属性名**和 **proxy 实例本身**（*严格地说，是操作行为所针对的对象*），其中最后一个参数可选。

#### 实例

```javascript
  var proxy = new Proxy({}, {
    get: function (target, property) {
      if (property === 'name') {
        console.log('get: name');
        return 'dangdang';
      }
    }
  });
  proxy.name;
  // get: name
  // dangdang
```

只要通过`proxy.xxx`访问了`proxy`上面的属性，就会触发`proxy`上配置的`get`拦截方法。
上面这个实例是最简单的实例，其实它可以做很多的事情，举几个例子：

- 实现数组读取负数的索引。
- 链式操作。
- 一个生成各种 DOM 节点的通用函数dom。

#### 拦截

该方法会拦截目标对象的以下操作:

- **访问属性: proxy[foo]和 proxy.bar**
- **访问原型链上的属性: Object.create(proxy)[foo]**
- **Reflect.get()**

#### 其他特性

- **get方法可以继承**。
- **get第三个参数，它总是指向原始的读操作所在的那个对象，一般情况下就是 Proxy 实例。**
- **一个属性不可配置（configurable）且不可写（writable），则 Proxy 不能修改该属性，否则通过 Proxy 对象访问该属性会报错。**

### set()

`set(target, property, value, receiver)`方法用来拦截某个属性的**赋值**操作，可以接受四个参数，依次为**目标对象**、**属性名**、**属性值**和 **Proxy 实例本身**，其中**最后一个参数可选**。

#### 实例

```javascript
  var proxy = new Proxy({}, {
    set: function (target, property, value, receiver) {
      if (property === 'name') {
        console.log('set: dangdang');
        target[property] = value;
      }
    }
  });

  proxy.name = 'dangdang';
  // set: dangdang
  proxy.name; // dangdang
```

只要给`proxy`的任意属性赋值，就会触发`proxy`上配置的`set`拦截方法。
当然这个实例是最简单的使用方式，还有其他的用法，举例如下：

- 校验属性是否符合格式（表单验证validation）
- 统计函数调用次数
- 配合get设置内部私有属性

#### 拦截

该方法会拦截目标对象的以下操作:

- **指定属性值: proxy[foo] = bar 和 proxy.foo = bar**
- **指定继承者的属性值: Object.create(proxy)[foo] = bar**
- **Reflect.set()**

#### 其他特性

- **get第三个参数，它总是指向原始的读操作所在的那个对象，一般情况下就是 Proxy 实例。**
- **若目标属性是不可写及不可配置的，则不能改变它的值。**
- **在严格模式下，若set方法返回false，则会抛出一个 TypeError 异常。**

### has()

`has(target, prop)`方法用来拦截`HasProperty`操作，即判断对象是否具有某个属性时，这个方法会生效。典型的操作就是`in`运算符。
`has(target, prop)`方法可以接受两个参数，分别是**目标对象**、**需查询的属性名**。

**返回值**

`has()` 方法返回一个 `boolean` 属性的值.

#### 实例

```javascript
  var proxy = new Proxy({
      name: 'dangdang',
      age: 25
    }, {
    has: function (target, property) {
      if (property === 'name') {
        return true;
      } else {
        return false;
      }
    }
  });
  'name' in proxy; // true
  'age' in proxy; // false
```

设置`has('name')`返回`true`再通过`in`检测时返回`true`，设置`has('age')`返回`false`再通过`in`检测时返回`false`。
当然这个实例是最简单的使用方式，还有其他的用法，举例如下：

- 可以做私有属性

#### 拦截

这个钩子可以拦截下面这些操作:

- **属性查询: foo in proxy**
- **继承属性查询: foo in Object.create(proxy)**
- **with 检查: with(proxy) { (foo); }**
- **Reflect.has()**

#### 其他特性

- **如果目标对象的某一属性本身不可被配置，则该属性不能够被代理隐藏。会抛出TypeError。**
- **如果目标对象为不可扩展对象，则该对象的属性不能够被代理隐藏。会抛出TypeError。**
- **has（）方法不判断一个属性是对象自身的属性，还是继承的属性。**
- **has（）拦截对for...in循环不生效。**

### defineProperty（）

`defineProperty(target, property, descriptor)`方法拦截了`Object.defineProperty`操作。
参数这里就不多做介绍了和**Object.defineProperty**相同，主要关注一下返回值。

**返回值**

`defineProperty` 方法必须以一个 `Boolean` 返回，表示定义该属性的操作成功与否。

**注意**

- 如果目标对象**不可扩展**， 将**不能添加**属性。
- 不能添加或者修改一个属性为不可配置的，如果它不作为一个目标对象的不可配置的属性存在的话。
- 如果目标对象存在一个对应的可配置属性，这个属性可能不会是不可配置的。
- 如果一个属性在目标对象中存在对应的属性，那么 `Object.defineProperty(target, prop, descriptor)` 将不会抛出异常。
- 在严格模式下， `false` 作为 `handler.defineProperty` 方法的返回值的话将会抛出 `TypeError` 异常.

### deleteProperty()

`deleteProperty(target, property)`方法用于拦截`delete`操作，如果这个方法抛出错误或者返回`false`，当前属性就无法被`delete`命令删除。

**返回值**

`deleteProperty` 必须返回一个 `Boolean` 类型的值，表示了该属性是否被成功删除。

```javascript
  var proxy = new Proxy({
      name: 'dangdang',
      age: 25
    }, {
    deleteProperty: function (target, property) {
      if (property === 'name') {
        return true;
      } else {
        return false;
      }
    }
  });

  delete proxy.name; // true
  delete proxy.age; // false
```

**注意**

- 目标对象自身的**不可配置（configurable）的属性**，**不能**被`deleteProperty`方法删除，否则报错。

### getOwnPropertyDescriptor()

`getOwnPropertyDescriptor`方法拦截`Object.getOwnPropertyDescriptor()`，返回一个**属性描述对象**或者`undefined`。

**返回值**

`getOwnPropertyDescriptor` 方法必须返回一个 `object` 或 `undefined`。

```javascript
  var proxy = new Proxy({
      name: 'dangdang',
      age: 25
    }, {
    getOwnPropertyDescriptor: function (target, property) {
      if (property === 'name') {
        return Object.getOwnPropertyDescriptor(target, property);
      } else {
        return undefined;
      }
    }
  });

  Object.getOwnPropertyDescriptor(proxy, 'name'); // {value: "dangdang", writable: true, enumerable: true, configurable: true}
  Object.getOwnPropertyDescriptor(proxy, 'age'); // undefined
```

## 代理对象自身操作拦截

### getPrototypeOf()/setPrototypeOf()

**原型相关的操作拦截**

#### getPrototypeOf()

`getPrototypeOf(target)`方法主要用来拦截**获取对象原型**。具体来说，拦截下面这些操作。

- `Object.prototype.__proto__`
- `Object.prototype.isPrototypeOf()`
- `Object.getPrototypeOf()`
- `Reflect.getPrototypeOf()`
- `instanceof`

**返回值**

`getPrototypeOf` 方法的返回值必须是**一个对象**或者 `null`。

#### setPrototypeOf()

`setPrototypeOf(target, obj)` 方法主要用来拦截 `Object.setPrototypeOf(target, obj)`。

**返回值**

如果成功修改了`[[Prototype]]`, `setPrototypeOf` 方法返回 `true`,否则返回 `false`。

```javascript
  var handler = {
    setPrototypeOf (target, proto) {
      throw new Error('Changing the prototype is forbidden');
    }
  };
  var proto = {};
  var target = function () {};
  var proxy = new Proxy(target, handler);
  Object.setPrototypeOf(proxy, proto);
  // Error: Changing the prototype is forbidden
```

**注意**

如果目标对象不可扩展`（non-extensible）`，`setPrototypeOf`方法不得改变目标对象的原型。

### preventExtensions/isExtensible

**扩展配置拦截操作**

#### preventExtensions

`preventExtensions(target)` 方法用于设置对`Object.preventExtensions()`的拦截。

**返回**

`preventExtensions` 必须返回一个布尔值，否则会被自动转为布尔值。

```javascript
  var proxy = new Proxy({}, {
    preventExtensions: function (target) {
      return true;
    }
  })
  Object.preventExtensions(proxy); // Uncaught TypeError: 'preventExtensions' on proxy: trap returned truish but the proxy target is extensible

  var proxy1 = new Proxy({}, {
    preventExtensions: function (target) {
      Object.preventExtensions(target);
      return true;
    }
  });
  Object.preventExtensions(proxy1); // Proxy {}

```

**注意**

如果`Object.isExtensible(proxy)`是`false`，`Object.preventExtensions(proxy)`只能返回`true`。

#### isExtensible

`isExtensible(target)` 方法用于拦截对对象的`Object.isExtensible()`。

**返回值**

`isExtensible`方法必须返回一个 `Boolean`值或可转换成`Boolean`的值。

```javascript
var p = new Proxy({}, {
  isExtensible: function(target) {
    console.log('called');
    return true;//也可以return 1;等表示为true的值
  }
});

console.log(Object.isExtensible(p)); // "called"
                                     // true

var p = new Proxy({}, {
  isExtensible: function(target) {
    return false;//return 0;return NaN等都会报错
  }
});

Object.isExtensible(p); // TypeError is thrown

```

**注意**

`Object.isExtensible(proxy)` 必须同`Object.isExtensible(target)`返回相同值。也就是必须返回`true`或者为`true`的值,返回`false`和为`false`的值都会报错。

### apply/construct

**改变this方式**

#### apply

`apply(target, thisArg, argumentsList)` 方法用于拦截**函数的调用**、`Reflect.apply`、`call`和`apply`操作。
`apply`方法可以接受三个参数，分别是**目标对象**、**被调用时的上下文对象**和**被调用时的参数数组**数组。

```javascript

  var proxy = new Proxy(function () {}, {
    apply: function (target, thisArg, argumentsList) {
      console.log('called:' + argumentsList.join(', '));
      return argumentsList[0] + argumentsList[1] + argumentsList[2];
    }
  });

  proxy(1, 2, 3); // called:1, 2, 3
  // 6

  proxy.call(null, 1, 2, 3) // called:1, 2, 3
  // 6

  proxy.apply(null, [1, 2, 3]) // called:1, 2, 3
  // 6
```

**注意**

`target`必须是可被调用的。也就是说，它**必须是一个函数对象**。

#### construct

`construct(target, argumentsList, newTarget)` 方法用于拦截`new` 操作符. 为了使`new`操作符在生成的`Proxy`对象上生效，用于初始化代理的目标对象自身必须具有`[[Construct]]`内部方法（即 `new targe`t 必须是有效的）。

**参数**

- `target`：目标对象
- `args`：构造函数的参数对象
- `newTarget`：创造实例对象时，`new`命令作用的构造函数（下面例子的p）

**返回值**

`construct` 方法**必须返回一个对象**。

```javascript
  var p = new Proxy(function() {}, {
    construct: function(target, argumentsList, newTarget) {
      console.log('called: ' + argumentsList.join(', '));
      return { value: argumentsList[0] * 10 };
    }
  });

  console.log(new p(1).value); // "called: 1"
                              // 10
```

### ownKeys

`ownKeys(target)` 方法用于拦截 `Reflect.ownKeys()`。

**返回值**

`ownKeys` 方法必须返回**一个可枚举对象**。

#### 拦截

该拦截器可以拦截以下操作:

- `Object.getOwnPropertyNames()`
- `Object.getOwnPropertySymbols()`
- `Object.keys()`
- `Reflect.ownKeys()`

#### 注意事项

如果违反了下面的约束，`proxy`将抛出错误 `TypeError`:

- `ownKeys` 的结果必须是一个数组.
- 数组的元素类型要么是一个 `String` ，要么是一个 `Symbol`.
- 结果列表必须包含目标对象的所有不可配置`（non-configurable ）`、自有（`own`）属性的key.
- 如果目标对象不可扩展，那么结果列表必须包含目标对象的所有自有（`own`）属性的`key`，不能有其它值。

## Proxy.revocable

`Proxy.revocable`方法返回一个可取消的 `Proxy` 实例。

```javascript
  let target = {};
  let handler = {};
  let {proxy, revoke} = Proxy.revocable(target, handler);
  proxy.foo = 123;
  proxy.foo // 123

  revoke();
  proxy.foo // TypeError: Revoked
```

`Proxy.revocable`方法返回一个对象，该对象的`proxy`属性是`Proxy`实例，`revoke`属性是一个函数，可以取消`Proxy`实例。上面代码中，当执行`revoke`函数之后，再访问`Proxy`实例，就会抛出一个错误。

**注意**

`Proxy.revocable`的一个使用场景是，目标对象不允许直接访问，必须通过代理访问，一旦访问结束，就收回代理权，不允许再次访问。

### this 问题

虽然 `Proxy` 可以代理针对目标对象的访问，但它不是目标对象的透明代理，即不做任何拦截的情况下，也无法保证与目标对象的行为一致。主要原因就是在 `Proxy` 代理的情况下，目标对象内部的`this`关键字会指向 `Proxy` 代理。

```javascript
  const target = {
    m: function () {
      console.log(this === proxy);
    }
  };
  const handler = {};

  const proxy = new Proxy(target, handler);

  target.m() // false
  proxy.m()  // true
```

上面代码中，一旦`proxy`代理`target.m`，后者内部的`this`就是指向`proxy`，而不是`target`。

下面是一个例子，由于`this`指向的变化，导致 `Proxy` 无法代理目标对象。

```javascript
  const _name = new WeakMap();
  class Person {
    constructor(name) {
      _name.set(this, name);
    }
    get name() {
      return _name.get(this);
    }
  }

  const jane = new Person('Jane');
  jane.name // 'Jane'

  const proxy = new Proxy(jane, {});
  proxy.name // undefined
```

上面代码中，目标对象`jane`的`name`属性，实际保存在外部`WeakMap`对象`_name`上面，通过`this`键区分。由于通过`proxy.name`访问时，`this`指向`proxy`，导致无法取到值，所以返回`undefined`。

此外，有些原生对象的内部属性，只有通过正确的`this`才能拿到，所以 `Proxy` 也无法代理这些原生对象的属性。

```javascript
  const target = new Date();
  const handler = {};
  const proxy = new Proxy(target, handler);

  proxy.getDate();
  // TypeError: this is not a Date object.
```

上面代码中，`getDate`方法只能在`Date`对象实例上面拿到，如果`this`不是`Date`对象实例就会报错。这时，`this`绑定原始对象，就可以解决这个问题。

```javascript
  const target = new Date('2015-01-01');
  const handler = {
    get(target, prop) {
      if (prop === 'getDate') {
        return target.getDate.bind(target);
      }
      return Reflect.get(target, prop);
    }
  };
  const proxy = new Proxy(target, handler);

  proxy.getDate() // 1
```

## 总结

本篇文章记录了`Proxy`相关的的一些`属性访问的方法`，它有**13**种方法，大致分为两类：一类**代理对象`自身属性`操作拦截**，另一类**代理对象`自身`操作拦截**。

下一篇文章对比`defineProterty`和`Proxy`之间的优缺点，用它们实现简单的双线绑定。

## 参考

> [mdn Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
> [阮一峰 Proxy](http://es6.ruanyifeng.com/#docs/proxy)
