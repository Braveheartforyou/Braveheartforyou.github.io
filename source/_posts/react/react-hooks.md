---
title: react v16.8.0中的Hook
date: 2019-07-04 10:23:43
tags: [React]
categories: [React]
description: Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。
---
## 简介
> Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。
我们知道，functional component在使用的时候有一些限制，比如需要生命周期、state的时候就不能用functional component。而有了Hooks，你就可以在funtional component里，使用class component的功能:props，state，context，refs，和生命周期函数等等。
**虽然Hooks已经有要取代正宫class的趋势了，但是react目前没有计划抛弃class，所以不要慌，你还是可以跟往常一样使用class。**
## 为什么引入hook
<hr/>

### 组件难以理解
<font color="#ff502c"></font>
在使用class组件构建我们的程序时，他们各自拥有自己的状态，业务逻辑的复杂使这些组件变得越来越庞大，各个<font color="#ff502c">生命周期</font>中会调用越来越多的逻辑，越来越难以维护。<font color="#ff502c">Hook</font> **将组件中相互关联的部分拆分成更小的函数（比如设置订阅或请求数据）**。而并非强制按照生命周期划分。你还可以使用 <font color="#ff502c">reducer</font> 来管理组件的内部状态，使其更加可预测。使用<font color="#ff502c">Hook</font>，可以让你更大限度的将<font color="#ff502c">公用逻辑</font>抽离，将一个组件分割成<font color="#ff502c">更小</font>的函数，而不是<font color="#ff502c">强制</font>基于生命周期方法进行分割。

### 组件嵌套问题
如果我们需要抽离一些重复的逻辑，就会选择 <font color="#ff502c">HOC</font> 或者 <font color="#ff502c">render props</font> 的方式。这种方式首先提高了 debug 的难度，并且也很难实现共享状态。
但是通过 <font color="#ff502c">Hooks</font> 的方式去抽离重复逻辑的话，一是不会增加组件的嵌套，二是可以实现状态的共享。

### 使用函数代替class
相比函数，编写一个<font color="#ff502c">class</font>可能需要掌握更多的知识，需要注意的点也越多，比如<font color="#ff502c">this</font>指向、绑定事件等等。另外，计算机理解一个函数比理解一个<font color="#ff502c">class</font>更快。<font color="#ff502c">Hooks</font>让你可以在<font color="#ff502c">class</font>之外使用更多React的新特性。

### 减少状态逻辑复用的风险
<font color="#ff502c">Hook</font>和<font color="#ff502c">Mixin</font>在用法上有一定的相似之处，但是<font color="#ff502c">Mixin</font>引入的<font color="#ff502c">逻辑和<font color="#ff502c">状态</font>是可以<font color="#ff502c">相互覆盖</font>的，而多个<font color="#ff502c">Hook</font>之间互不影响，这让我们不需要在把一部分精力放在防止避免逻辑复用的冲突上。
在不遵守约定的情况下使用<font color="#ff502c">HOC</font>也有可能带来一定冲突，比如<font color="#ff502c">props覆盖</font>等等，使用Hook则可以避免这些问题。

