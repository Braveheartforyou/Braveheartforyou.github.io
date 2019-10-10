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

<!-- 现在的三大框架非常的流行，在数据流中分为两派`React`的单项数据流，`Angluar/Vue`的双向数据流。其实`React`也是实现了的**双向数据绑定**的，只不过要通过`setState`来触发。

在不同框架中实现**双向数据绑定**也是不相同的，大致如下图所示：

![双向绑定](../../images/es/es-defineProperty.png)

`Object.defineProperty`和`proxy`都是`Vue`不同版本的重要组成部分，它们都是可以实现双向绑定， -->

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

- `handler.set(target, property, value, receiver)`: 在**读取**代理对象的某个**属性时**触发该操作，比如在执行 `proxy.foo` 时。
- `handler.get(target, property, receiver)`: 在给代理对象的某个**属性赋值时**触发该操作，比如在执行 `proxy.foo = 1` 时。
- `handler.has(target, prop)`: 在判断代理对象**是否拥有某个属性时**触发该操作，比如在执行 `"foo" in proxy` 时。
- `handler.defineProperty(target, property, descriptor)`: 在**定义代理对象某个属性时的属性描述时**触发该操作，比如在执行 `Object.defineProperty(proxy, "foo", {})` 时。
- `handler.deleteProperty(target, property)`: 在**删除代理对象的某个属性时**触发该操作，即使用`delete`运算符，比如在执行 `delete proxy.foo` 时。
- `handler.getOwnPropertyDescriptor(target, prop)`:在**获取代理对象某个属性的属性描述时**触发该操作，比如在执行 `Object.getOwnPropertyDescriptor(proxy, "foo")` 时。

另一类**代理对象`自身`操作拦截**:

- `handler.getPrototypeOf(target)`: 在**读取代理对象的原型时**触发该操作，比如在执行 `Object.getPrototypeOf(proxy)` 时。
- `handler.setPrototypeOf(target, prototype)`: 在**设置代理对象的原型时**触发该操作，比如在执行 `Object.setPrototypeOf(proxy, null)` 时。
- `handler.isExtensible(target)`: 在判断一个**代理对象是否是可扩展时**触发该操作，比如在执行 `Object.isExtensible(proxy)` 时。
- `handler.preventExtensions(target)`: 在让一个**代理对象不可扩展时**触发该操作，比如在执行 `Object.preventExtensions(proxy)` 时。
- `handler.apply(target, thisArg, argumentsList)`: 拦截 **Proxy 实例作为函数调用**的操作，比如`proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)`。
- `handler.ownKeys(target)`: 拦截`Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in`循环，**返回一个数组**。该方法返回目标对象**所有自身的属性的属性名**，而Object.keys()的返回结果仅包括**目标对象自身的可遍历属性**。
- `handler.construct(target, argumentsList, newTarget)`: 拦截 **Proxy 实例作为构造函数调用**的操作，比如`new proxy(...args)`。

可以看到`Proxy`的拦截方法上就比`Object.defineProperty`的配置多很多，并且在最近的浏览器支持中也是各大浏览器上对`Proxy`大理支持，优化性能等等。

## Proxy实例

在这节中详细记录常用的`Proxy`实例上的拦截方法的参数、注意事项等等。

### 代理对象自身属性操作拦截

