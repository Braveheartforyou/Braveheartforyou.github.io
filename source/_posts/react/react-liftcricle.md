---
title: react的生命周期
date: 2019-07-14 23:14:43
tags: [React]
categories: [React]
description: 在react的v16.x之前和后面是有比较大的变化，先从之前的生命周期说起，并且大致说一下在老的生命周期做不同的操作，新的生命周期怎么做比较好。
---
## 简介
在<font color="#ff502c">react</font> v16.x对生命周期有比较大的变化，可以通过下面的两张比较经典的图片来一览变化。
**react v15.x**
![reac-lifecricle](../../images/react/react-lifecricle-1-2.png)
**react v16.x**
![reac-lifecricle](../../images/react/react-lifecricle-1-3.jpg)

<font color="#ff502c">react</font> v16.x的生命周期是在 react v15.x的生命周期基础上删减了一些生命周期，同时也新增了一些生命周期，删除的生命周期也可以通过hook来模拟实现。

**React 16.3 新增的生命周期方法**
- **getDerivedStateFromProps()**
- **getSnapshotBeforeUpdate()**

**逐渐废弃的生命周期方法**
- **componentWillMount()**
- **componentWillReceiveProps()**
- **componentWillUpdate()**

> 虽然废弃了这三个生命周期方法，但是为了向下兼容，将会做渐进式调整。（详情见#12028）
V16.3 并未删除这三个生命周期，同时还为它们新增以 UNSAFE_ 前缀为别名的三个函数 <font color="#ff502c">UNSAFE_componentWillMount()</font>、<font color="#ff502c">UNSAFE_componentWillReceiveProps()</font>、<font color="#ff502c">UNSAFE_componentWillUpdate()</font>。
在 16.4 版本给出警告将会弃用 <font color="#ff502c">componentWillMount()</font>、<font color="#ff502c">componentWillReceiveProps()</font>、<font color="#ff502c">componentWillUpdate()</font> 三个函数
然后在 17 版本将会删除 <font color="#ff502c">componentWillMount()</font>、<font color="#ff502c">componentWillReceiveProps()</font>、<font color="#ff502c">componentWillUpdate()</font> 这三个函数，会保留使用 <font color="#ff502c">UNSAFE_componentWillMount()</font>、<font color="#ff502c">UNSAFE_componentWillReceiveProps()</font>、<font color="#ff502c">UNSAFE_componentWillUpdate()</font>

一般生命周期分为三个阶段：
1. 创建阶段（Mounting）
2. 更新阶段（Updating）
3. 卸载阶段（UnMounting）

从 React v16.x 开始，还对生命周期加入了错误处理（Error Handling）。

## 生命周期
其实用下面的这个图更能展现v15.x的流程图三个阶段的生命周期，图如下：
![reac-lifecricle](../../images/react/react-lifecricle-1-1.png)

## 挂载(Mounting)阶段
- constructor
- static getDerivedStateFromProps()(**新增**)
- componentWillMount()/UNSAFE_componentWillMount()(**废弃**)
- render()
- componentDidMount()

### constructor()
对于每个组件实例这个方法只会调用**一次**。
<font color="#ff502c">constructor参数接受两个参数**props**，**context**。
```javascript
  constructor(props, context) {
    super(props, context);
    console.log(this.props,this.context);
  };
```
可以获取到父组件传下来的的**props**,**context**,如果你想在<font color="#ff502c">constructor</font>构造函数内部(注意是内部哦，在组件其他地方是可以直接接收的)使用**props**或**context**,则需要传入，并传入<font color="#ff502c">super</font>对象。


构造函数通常用于：
  - 使用 **this.state** 来初始化 **state**
  - 给事件处理函数绑定 **this**

> ES6 子类的构造函数必须执行一次 <font color="#ff502c">super</font>。React 如果构造函数中要使用 **this.props**，必须先执行 super(props)。

### static getDerivedStateFromProps()
`
static getDerivedStateFromProps(nextProps, prevState)
`
首先，这是一个静态方法生命周期钩子。也就是说，定义的时候得在方法前加一个`static`关键字，或者直接挂载到`class`类上。

简要区分一下实例方法和静态方法：
- 实例方法，挂载在this上或者挂载在prototype上，class类不能直接访问该方法，使用new关键字实例化之后，实例可以访问该方法。
- 静态方法，直接挂载在class类上，或者使用新的关键字static，实例无法直接访问该方法。

当<font color="#ff502c">创建时</font>、<font color="#ff502c">接收新的 props 时</font>、<font color="#ff502c">setState 时</font>、<font color="#ff502c">forceUpdate 时</font>会执行这个方法。
> 注意：v16.3 setState 时、forceUpdate 时不会执行这个方法，v16.4 修复了这个问题。

这个生命周期钩子也经历了一些波折，原本它是被设计成<font color="#ff502c">初始化r</font>、<font color="#ff502c">父组件更新r</font>和<font color="#ff502c">接收到propsr</font>才会触发，现在只要渲染就会触发，也就是<font color="#ff502c">初始化r</font>和<font color="#ff502c">更新阶段r</font>都会触发。

```javascript
class ExampleComponent extends React.Component {
  // Initialize state in constructor,
  // Or with a property initializer.
  state = {
    isScrollingDown: false,
    lastRow: null,
  };

  static getDerivedStateFromProps(props, state) {
    if (props.currentRow !== state.lastRow) {
      return {
        isScrollingDown: props.currentRow > state.lastRow,
        lastRow: props.currentRow,
      };
    }

    // Return null to indicate no change to state.
    return null;
  }
}
```
这个方法在建议尽量少用，只在必要的场景中使用，一般使用场景如下：
- 无条件的根据 `props` 更新 `state`
- 当 `props` 和 `state` 的不匹配情况更新 `state`

### componentWillMount
- 组件刚经历<font color="#ff502c">constructor</font>,初始完数据
- 组件还未进入<font color="#ff502c">render</font>，组件还未渲染完成，dom还**未渲染**

在组件**挂载**到DOM前调用，且只会被调用一次，在这边调用this.setState不会引起组件重新渲染（**如果是异步的话，会触发重新渲染**）。
> 这是React不再推荐使用的API。

### render（会多次执行）
该方法会创建一个**vnode**，用来表示组件的输出。对于一个组件来讲，<font color="#ff502c">render</font>方法是唯一一个必需的方法。<font color="#ff502c">render</font>方法需要满足下面几点：
- 1. 只能通过 **this.props** 和 **this.state** 访问数据（不能修改）
- 2. 可以返回 null,false 或者任何<font color="#ff502c">React</font>组件
- 3. 只能出现一个顶级组件，不能返回一组元素
- 4. 不能改变组件的状态(**state**)
- 5. 不能修改DOM的输出

可以返回下面几种类型：

### componentDidMount
组件挂载到DOM后调用，且只会被调用一次。
一般用于下面的场景：
- 异步请求 ajax
- 添加事件绑定（注意在 <font color="#ff502c">componentWillUnmount</font> 中取消，以免造成内存泄漏）

## 更新(update)阶段
- componentWillReceiveProps()/UNSAFE_componentWillReceiveProps()(**废弃**)
- static getDerivedStateFromProps()(**新增**)
- shouldComponentUpdate(nextProps,nextState)
- componentWillUpdate()/UNSAFE_componentWillUpdate()(**废弃**)
- render()
- static getSnapshotBeforeUpdate()(**新增**)
- componentDidUpdate()

### componentWillReceiveProps (nextProps)
componentWillReceiveProps</font>在接受父组件改变后的**props**需要重新渲染组件。
它接受一个参数：
- nextProps
**通过对比nextProps和this.props，将nextProps setState为当前组件的state，从而重新渲染组件**

### shouldComponentUpdate(nextProps,nextState)
在接收新的 props 或新的 state 时，在渲染前会触发该方法。该方法通过返回 true 或者 false 来确定是否需要触发新的渲染。返回 false， 则不会触发后续的 componentWillUpdate()、render() 和 componentDidUpdate()（但是 state 变化还是可能引起子组件重新渲染）。

> PureComponent 的原理就是对 props 和 state 进行浅对比（shallow comparison），来判断是否触发渲染。

### static getDerivedStateFromProps()
见上文`static getDerivedStateFromProps()`
### shouldComponentUpdate(nextProps,nextState)

### componentWillUpdate()/UNSAFE_componentWillUpdate()

### static getSnapshotBeforeUpdate()

### componentDidUpdate()
