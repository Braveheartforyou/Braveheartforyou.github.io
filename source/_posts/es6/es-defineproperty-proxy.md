---
title: ES系列 Object.defineProperty和Proxy的对比
date: 2019-09-23 09:12:32
tags: [ECMAScript6]
categories: [ECMAScript6]
description: Object.defineProperty和Proxy分别是什么，它们之间的优缺点，实现简单的双向绑定。
---

## 简介

在前两篇文章中分别介绍了`Object.defineProperty`和`Proxy`两个新的特性，其实看起来`Proxy`更像是对`Object.defineProperty`的一种补充和完善(个人见解)。当然不是说`Object.defineProperty`很差，感觉更像是一开始的定位是为了处理对象的特定属性，但是在`Vue`等等框架的中被用来劫持整个对象属性，所以后面就出来了`Proxy`，更强大的劫持功能。

## 优缺点

根据上面两篇文章的介绍，大致优缺点如下：

***Proxy相对于defineProperty的优点***

- 对于对象已有属性：`Object.defineProperty`只能劫持对象的单个属性，如果想劫持整个对象就要**循环递归**调用`Object.defineProperty`。而`Proxy`拦截整个对象，并且返回一下新的对象。
- 对于对象新增属性：`Proxy`劫持整个对象，对于新增的属性自动拦截。而`Object.defineProperty`需要**重新劫持**新增的属性
- 对于数组操作： `Object.defineProperty`无法监控到数组下标的变化。而`Proxy`可以监听数组变化。
- 拦截或劫持方法： `Object.defineProperty`描述符基本上分为两类**数据描述符**、**存取描述符**、**通用描述符**三种。而`Proxy`中有**13**种**traps**方法供你选择。
- 是否支持取消劫持： `Object.defineProperty`如果想取消劫持，只能**重写**描述符，但是**configurable: false**时就**不能重写**描述符了。而`Proxy`可以通过`Proxy.revocable`返回一个可取消的 `Proxy` 实例。
- 浏览器对劫持或拦截的支持： `Proxy`在后续应该会有更好的支持，不然`Vue`也不会修改核心代码。
- 性能： `Proxy`性能是比`Object.defineProperty`高的，在多个对象属性中。

**Proxy相对于defineProperty的缺点**

- `this指向`： `defineProperty`因为只绑定对象的属性，一般不会涉及到`this问题`。而`Proxy`返回的对象的`this`和`target`的`this`不相同。
- `使用难度`: 相对于`Proxy`的**api**，反而`defineProperty`上手更容易。

**Proxy和defineProperty的一些注意事项**

- `对象冻结`：无论是`defineProperty`、`Object.freeze`、`Object.seal（密封）`都不是深度冻结，如果想深度冻结只能递归实现。
- `this问题`： `Proxy`在使用是要注意`this`指向问题。

## 各自实现双向绑定

现在的三大框架非常的流行，在数据流中分为两派`React`的单项数据流，`Angluar/Vue`的双向数据流。其实`React`也是实现了的**双向数据绑定**的，只不过要通过`setState`来触发。

在不同框架中实现**双向数据绑定**也是不相同的，大致如下图所示：

![双向绑定](../../images/es/es-defineProperty.png)

`Object.defineProperty`和`proxy`都是`Vue`不同版本的重要组成部分，它们都是可以实现双向绑定中的**数据劫持**，其实也就是响应式对象，在以前的文章有[深入Vue系列 Vue中的响应式对象](/blog/vue/vue-definedProperty.html)、
[深入Vue系列 Vue中的依赖收集](/blog/vue/vue-dep.html)、[深入Vue系列 Vue中的派发更新](/blog/vue/vue-notify.html)，如果感兴趣的可以去看看。

依照`Vue`代码中的双向绑定思路，大致分为以下三步：

- 把普通对象通过`Object.defineProperty`变为响应式对象
- 同时`getter`中收集依赖，也就是渲染`wather`
- 在`setter`中派发更行

下面写的实例不会这么复杂，当然也会仿照`Vue`源码中的`mvvm`去写。

### Object.defineProperty实现双向绑定

### Proxy实现双向绑定

## 总结

## 参考
