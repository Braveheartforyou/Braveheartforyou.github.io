---
title: react中的mixins、hoc、hooks三者对比
date: 2019-07-06 10:23:43
tags: [React, Hooks]
categories: [Hooks]
description: react中的mixins、hoc、hooks三者对比
---

[react 的高阶组件浅析](/blog/react/react-hoc.html)
[react 中 Hooks 浅析](/blog/react/react-hooks.html)
[react 的 mixins、hoc、hooks 对比](/blog/react/react-mixins-hoc-hooks.html)

## 简介

`Mixin（混入）`是一种通过扩展收集功能的方式，它本质上是将一个对象的属性拷贝到另一个对象上面去，不过你可以`拷贝任意多个对象的任意个方法`到一个新对象上去，这是继承所不能实现的。它的出现主要就是为了`解决代码复用问题`。
如下图所示：
![reac-mixins](../../images/react/react-mixins-1-1.png)
Mixin 是用来提取可复用的逻辑或者方法，并且它比继承要更令活，它的应用范围也是比较广的比如在`JQuery`的`extend`方法。

Mixin 大致实现如下：

```javascript
function setMixin(target, mixin) {
    if (arguments[2]) {
        for (var i = 2, len = arguments.length; i < len; i++) {
        t   arget.prototype[arguments[i]] = mixin.prototype[arguments[i]];
        }
    }
    else {
        for (var methodName in mixin.prototype) {
            if (!Object.hasOwnProperty(target.prototype, methodName)) {
                target.prototype[methodName] = mixin.prototype[methodName];
            }
        }
    }
}
setMixin(User,LogMixin,'actionLog');
setMixin(Goods,LogMixin,'requestLog');
```

您可以使用`setMixin 方法`将任意对象的任意方法扩展到目标对象上。

## React 中的 Mixin

`React`也提供了`Mixin`的实现，如果完全不同的组件有相似的功能，我们可以引入来实现代码复用，当然只有在使用`createClass`来创建`React 组件`时才可以使用，因为在`React 组件`的 es6 写法中它已经`被废弃掉`了。

例如下面的例子，很多组件或页面都需要记录用户行为，性能指标等。如果我们在每个组件都引入写日志的逻辑，会产生大量重复代码，通过`Mixin`我们可以解决这一问题：

```javascript
var LogMixin = {
  log: function () {
    console.log('log');
  },
  componentDidMount() {
    console.log('in');
  },
  componentWillUnmount() {
    console.log('out');
  }
};

var User = React.createClass({
  mixins: [LogMixin],
  render() {
    return <div></div>;
  }
});

var Goods = React.createClass({
  mixins: [LogMixin],
  render() {
    return <div></div>;
  }
});
```

## Mixin 的缺陷

`React`官方文档在[Mixins Considered Harmful](https://react.docschina.org/blog/2016/07/13/mixins-considered-harmful.html)一文中提到了 Mixin 带来了危害：

- **Mixin 可能会相互依赖，相互耦合，不利于代码维护**
- **不同的 Mixin 中的方法可能会相互冲突**
- **Mixin 非常多时，组件是可以感知到的，甚至还要为其做相关处理，这样会给代码造成滚雪球式的复杂性**

`React`现在已经`不再推荐使用 Mixin`来解决代码复用问题，因为`Mixin`带来的`危害`比他产生的`价值`还要巨大，并且 React 全面推荐使用`高阶组件`来替代它。

## 高阶组件

React 推荐高阶组件和代替 Mixin 来做逻辑、代码复用，这里就不多赘述 Hoc 的使用和作用，如果不太清楚的可以看另一篇博客来了解 React 中的 HOC,[react 的高阶组件浅析](/blog/react/react-hoc.html)。

### HOC 的作用

HOC 可以比 Mixin 做更多的事情，也没有 Mixin 产生那么大副作用，具体实现什么样的功能如下：

- **操作 props(属性)(通过属性代理实现、通过反向继承实现)**
- **通过 Refs 访问到组件实例 （通过属性代理实现）**
- **组件状态提升（通过属性代理实现）**
- **操作 state （通过反向继承实现）**
- **渲染劫持（通过属性代理实现、通过反向继承实现）**
- **用其他元素包裹 WrappedComponent （通过属性代理实现）**

## HOC 的缺陷

- `HOC`需要在原组件上进行包裹或者嵌套，如果大量使用`HOC`，将会产生非常多的嵌套，这让调试变得非常困难。
- `HOC`可以劫持`props`，在不遵守约定的情况下也可能造成冲突。

**在新版的 React 中推行 Hooks 来替代 HOC。它可以同时解决 Mixin 和 HOC 带来的问题**

## Hooks

在`React v16.7.0-alpha`中加入了新的特性`Hooks`，他可以让你在`class`以外使用`state`和其他`React`特性。

使用`Hooks`，你可以在将含有`state 的逻辑从组件中抽象出来`，这将可以让这些逻辑容易被测试。同时，`Hooks`可以帮助你在不重写组件结构的情况下复用这些逻辑。所以，它也可以作为一种实现`状态逻辑复用的方案`。

可以看另一篇博客[react 中 Hooks 浅析](/blog/react/react-hooks.html)

## hook 使用事项

**Hook 本质就是 JavaScript 函数，但是在使用它时需要遵循两条规则。**

### 只在最顶层使用 Hook

`不要`在`循环，条件或嵌套函数`中`调用 Hook`， 确保总是在你的 `React 函数的最顶层调用`他们。

### 使用范围

`不要`在`普通的 JavaScript 函数中`调用 Hook。只能在`React 的函数组件中调用 Hook`。不要在其他 JavaScript 函数中调用。
**Hook 的提出主要就是为了解决 class 组件的一系列问题，所以我们能在 class 组件中使用它**。

## 总结

程序的发展提出一个可以解决当前问题的方案，但随着时间的变化发现以前的方案有很大的副作用或者有更好的方案，就会更新一套更好的方案来解决问题，就像`React 中复用逻辑的变化从 Mixin(已废弃)到 HOC(用的比较多)再到现在的 Hooks(新贵)`，可能以后 Hooks 也会落伍但是一定要保持学习。

## 参考

> [【React 深入】从 Mixin 到 HOC 再到 Hook](https://juejin.im/post/5cad39b3f265da03502b1c0a#heading-36) > [react 的高阶组件浅析](/blog/react/react-hoc.html) > [react 中 Hooks 浅析](/blog/react/react-hooks.html)
