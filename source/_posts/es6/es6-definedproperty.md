---
title: ES系列 Object.defineProperty
date: 2019-09-16 14:23:54
tags: [ECMAScript6]
categories: [ECMAScript6]
description: Object.defineProperty和Proxy分别是什么，它们之间的优缺点，实现简单的双向绑定。
---

> [ES系列 Object.defineProperty](/blog/es6/es6-definedproperty.html)
> [ES系列 Proxy](/blog/es6/es6-proxy.html)
> [ES系列 Object.defineProperty和Proxy的对比](/blog/es6/es6-definedproperty-proxy.html)

## 简介

如果用过`VUE`框架的话都听说过他的数据观测在**2.x**是是通过`Object.defineProperty`实现的，其实就是把普通的对象变为**响应式对象**，但是在最近**3.x**中作者要通过**Proxy**重写`Vue`中的双向绑定核心的**响应式对象**实现，在本篇文章中逐渐了解`Object.defineProperty`和`Proxy`是什么，它们之间的区别是什么，和它们之间的有缺点。

如果想了解`Vue`中的双向对象的实现，请看本站中的**Vue响应式对象**、**依赖收集**、**派发更新**等等**Vue源码文章**。

本文章目录：

- `Object.defineProperty`使用简介
- `Proxy`使用简介
- `Object.defineProperty`和`Proxy`之间的区别和优缺点
- 为什么`Vue`要重写核心的数据观测实现
- `Object.defineProperty`和`Proxy`实现简单的双向绑定

## Object.defineProperty

**[ES5](https://www.ecma-international.org/ecma-262/5.1/#sec-15.2.3.6)** 提供了 `Object.defineProperty` 方法，该方法可以在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回这个对象。

### 参数

可以通过`Object.defineProperty(obj, prop, descriptor)`使用，三个参数分别代表：

- `obj(Object)`: 要在其上定义属性的对象。
- `prop(String)`: 要定义或修改的属性的名称。
- `descriptor(Object)`: 将被定义或修改的属性描述符。

**示例**

```javascript
  // 声明example并且赋值一个对象字面量
  var example = {};
  // 通过Object.defineProperty 定义一个新的属性count，并且给他赋值为一个value
  Object.defineProperty(example, 'count', {
    value: 1
  });
  // 输出example.count的值
  example.count; // 1
```

### 属性描述符

对象里目前存在的属性描述符有两种主要形式：**数据描述符和存取描述符**。

**数据描述符和存取描述符均具有**以下可选键值(默认值是在使用Object.defineProperty()定义属性的情况下)：

- `configurable(Boolean)`: 当且仅当该属性的 `configurable` 为 `true` 时，该属性**描述符**才能够被改变，同时该属性也能从对应的对象上被删除。**默认为 false**。
- `enumerable(Boolean)`: 当且仅当该属性的`enumerable`为`true`时，该属性才能够出现在对象的枚举属性中。**默认为 false**。

**数据描述符同时具有以下可选键值：**

- `value(任意有效的Javascript值)`： 该属性对应的值。**默认为 undefined**。
- `writable(Boolean)`: 当且仅当该属性的`writable`为`true`时，`value`才能被赋值运算符改变。**默认为 false**。

**存取描述符同时具有以下可选键值：**

- `get(Function 匿名函数)`：一个给属性提供 `getter` 的方法，如果没有 `getter` 则为 `undefined`。当访问该属性时，该方法会被执行，方法执行时没有参数传入，但是会传入**this**对象（由于继承关系，这里的**this**并不一定是定义该属性的对象）。**默认为 `undefined`**。
- `set(Function 匿名函数)`: 一个给属性提供 `setter` 的方法，如果没有 `setter` 则为 `undefined`。当属性值修改时，触发执行该方法。该方法将接受唯一参数，即该属性新的参数值。**默认为 `undefined`**。

**描述符可同时具有的键值**

|    | **configurable** | **enumerable** | **value** | **writable** | **get** | **set** |
|:------:|:-------:|:-------:|:-------:|:-------:|:-------:|
|  **数据描述符**  | **Yes** | **Yes** | **Yes** | **Yes** | **No** | **No** |
|  **存取描述符**  | **Yes** | **Yes** | **No** | **No** | **Yes** | **Yes** |

> 注意事项： 如果一个描述符同时有(`value`或`writable`)和(`get`或`set`)关键字，将会产生一个异常。

下面一个一个描述符来通过实例来看一下它真正的效果。

### configurable

当设置了`configurable`为`false`时，就不能在通过`defineProperty`设置属性描述了。当前的属性不能**删除**、**修改**、**枚举**等。下面请看实例：

```javascript
  var example = {};
  Object.defineProperty(example, 'count', {
    value: 1,
    configurable: false
  });
  // 修改
  example.count = 2;
  example.count; // 1

  // 删除
  delete example.count;
  example.count; // 1

  // 枚举
  for (key in example) {
    console.log(key); // undefined
  }

  Object.defineProperty(example, 'count', {
    wirtable: true
  });
  // Uncaught TypeError: Cannot redefine property: count
```

### Enumerable

当设置`Enumerable`为`false`时，当前这个属性不能被`for...in`和`Object.keys()`枚举。

### writable

当设置`writable`为`false`时，写入非可写属性**不会改变它**，也**不会**引发错误。

### setter和getter

`get` 和 `set`两个方法在上面是存取属性描述，这两个方法又被称为`getter`和`setter`。可以简称为**存取器属性**。

当配置了`get`和`set`，就不能配置`value`和`writable`因为它们是互斥的，只能设置其中的一组。

当想获取一个属性的值时就会触发设置的`get`方法，当给一个属性赋值时就会触发`set`属性。

**实例**

```javascript
  var example = {}, value = null;
  
  Object.defineProterty(example, 'count', {
    get: function () {
      console.log('执行了 get 操作');
      return value;
    },
    set: function (newValue) {
      console.log('执行了 set 操作')
      value = newValue;
    }
  });
  example.count = 1; // 执行了 set 操作
  example.count; // 1 执行了 get 操作
```

## 其它Object.defineProperty相关

### Object.defineProperties

`Object.defineProperties()`可以同时设置多个`Object.defineProperty`。

`Object.defineProperties(obj, props)` 方法直接在一个对象上定义新的属性或修改现有属性，并**返回**该对象。参数如下：

- `obj(Object)`: 在其上定义或修改属性的对象。
- `props(Object)`: 一个对象包含多个**属性名:descriptor**

**实例**

```javascript
  var example = {};
  Object.defineProperties(example, {
    'count1': {
      value: 1
    },
    'count2': {
      value: 2
    }
  });
```

### Object.getOwnPropertyNames

`Object.getOwnPropertyNames(Object)`方法返回一个由指定对象的**所有自身属性的属性名（包括不可枚举属性但不包括Symbol值作为名称的属性）组成的数组**。

**实例**

```javascript
  var example = { num: 3 };
  Object.defineProperties(example, {
    'count1': {
      value: 1
    },
    'count2': {
      value: 2
    }
  });
  Object.getOwnPropertyNames(example); //  ["num", "count1", "count2"]
```

### Object.getOwnPropertyDescriptor

`Object.getOwnPropertyDescriptor(obj, prop)` 方法返回指定对象上一个**自有属性对应的属性描述符**。**（自有属性指的是直接赋予该对象的属性，不需要从原型链上进行查找的属性）**。参数如下：

- `obj(Object)`: 需要查找的目标对象
- `prop(String)`： 目标对象内属性名称

**返回值**
如果指定的属性存在于对象上，则返回其属性描述符对象`（property descriptor）`，否则返回 `undefined`。

> 在 **ES5** 中，如果该方法的第一个参数不是对象（而是原始类型），那么就会产生出现 `TypeError`。而在 **ES2015**，第一个的参数不是对象的话就会**被强制转换为对象**。

## 冻结相关的

### Object.defineProperty冻结

当`Object.defineProperty(obj, name, { value: 1 })`这样设置实，当前属性的**writable**、**configurable**、**enumerable**、都默认为`false`，可以把当前这个**对象属性**看做是一个冻结对象，那这个对象属性是**深冻结还是浅冻结**呢？

**实例**

```javascript
  var example = {};
  Object.defineProperty(example, 'count', {
    value: {
      num: 1
    }
  });
  // 修改属性
  example.count = 2;
  example.count; // {num: 1}
  // 修改属性下的属性
  example.count.num = 3;
  example.count; // {num: 3}
```

通过上面的实例可以判定`defineProperty`冻结的对象属性，是**浅冻结**对象，如果是**引用类型**是可以修改的。如果想实现**深层冻结**，就需要递归对象所有的属性设置`Object.defineProperty()`。

### Object.freeze冻结

`Object.freeze(obj)` 方法可以冻结一个对象。参数如下：

- `obj(Object)`: 要被冻结的对象。

一个被冻结的对象再也**不能被修改**；
冻结了一个对象则**不能**向这个对象**添加新的属性**，**不能删除已有属性**，不能修改该对象已有属性的**可枚举性**、**可配置性**、**可写性**，以及不能修改已有属性的值。此外，冻结一个对象后该对象的**原型也不能被修改**。**freeze() 返回和传入的参数相同的对象**。

**返回值**

被冻结的对象。

**实例**

```javascript
  var example = {
    count: [0, 1],
    num: 10
  }
  Object.freeze(example);
  // 修改值类型无效
  example.num = 11;
  example.num; // 10

  // 修改引用类型生效
  example.count[0] = 1;
  example.count; // [1, 1]

  // 试图通过 Object.defineProperty 更改属性 抛出 TypeError.
  Object.defineProperty(example, 'num', { value: 11 });

  // 也不能更改原型 会抛出 TypeError.
  example.__proto__ = { x: 20 };
  Object.setPrototypeOf(example, { x: 20 })

```

通过上面代码可以看出`Object.freeze()`也是**浅冻结**，如果冻结的对象有多层引用类型嵌套，子属性是可以修改的。如果想实现**深层冻结**，就需要递归对象所有的属性设置`Object.freeze()`。

#### 冻结数组

```javascript
  let a = [0];
  Object.freeze(a); // 现在数组不能被修改了.

  a[0]=1; // fails silently
  a.push(2); // fails silently

  // In strict mode such attempts will throw TypeErrors
  function fail() {
    "use strict"
    a[0] = 1;
    a.push(2);
  }

  fail();
```

#### Object.isFrozen

`Object.isFrozen(obj)`方法判断一个对象是否被冻结。参数如下：

- `obj(Object)`: 被检测的对象。

**返回值**

表示给定对象是否被冻结的`Boolean`。

> **在 ES5 中，如果参数不是一个对象类型，将抛出一个TypeError异常**。
**在 ES2015 中，非对象参数将被视为一个冻结的普通对象，因此会返回true**。

```javascript
  Object.isFrozen(1);
  // TypeError: 1 is not an object (ES5 code)

  Object.isFrozen(1);
  // true                          (ES2015 code)
```

#### Object.seal

`Object.seal(obj)`方法封闭一个对象，**阻止添加新属性并将所有现有属性标记为不可配置**。**当前属性的值只要可写就可以改变**。

- `obj(Object)`: 将要被密封的对象。

**返回值**

被密封的对象。

> **在ES5中，如果这个方法的参数不是一个（原始）对象，那么它将导致TypeError**。
**在ES2015中，非对象参数将被视为已被密封的普通对象，会直接返回它**。

```javascript
  Object.seal(1);
  // TypeError: 1 is not an object (ES5 code)

  Object.seal(1);
  // 1                             (ES2015 code)
```

它其他的表现适合`Object.freeze`是一致的。

### 实现一个深度冻结

这里只实现一个简单的**深度冻结**方法，一些**循环引用**、**特殊类型**没有考虑在内，实现如下：

- 递归对象属性，调用冻结方法
- 返回冻结完成的对象

```javascript
  function deepFreeze (obj) {
    if (typeof obj !== 'object') {
      console.log('arguments is not object');
      return false;
    }
    let propertyName = Object.getOwnPropertyNames(obj);
    propertyName.forEach(item => {
      if (typeof obj[item] === 'object' && typeof obj[item] !== 'null') {
        deepFreeze(obj[item]);
      }
    });
    return Object.freeze(obj);
  }
  // 测试代码
  let example = deepFreeze({example: { name: 'admin' }});
  example.example.name = 1;
  example.example; // {name: "admin"}
```

## 总结

**因为当前文章写得太长了，所以拆分为三篇博客。**

## 参考

> [mdn Object.defineProperty()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
> [mdn Object.defineProperties()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties)
> [mdn Object.freeze()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)
> [mdn Object.seal()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/seal)
