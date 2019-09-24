---
title: 理解 react的 setState 在不同情况下同步异步之间切换（未完待续）
date: 2019-01-15 16:27:09
tags: [React]
categories: [React]
description: react的setState是异步的，但是他有没有可能在一定情况下是同步的呢,很多人都是一口断定是异步但是确实是不对的，其实react在一定的条件下是同步的
---

## 总结 setState 在不同环境中同步、异步不同

1. <font color="#ff502c">setState</font> 只在<font color="#ff502c">合成事件</font>和<font color="#ff502c">钩子函数</font>中是"异步"的,在<font color="#ff502c">原生事件</font>和<font color="#ff502c">setTimeout</font>中都是<font color="#ff502c">同步</font>的。
2. <font color="#ff502c">setState</font>的“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形式了所谓的“异步”，当然可以通过第二个参数 setState(partialState, callback) 中的<font color="#ff502c">callback</font>拿到更新后的结果。
3. <font color="#ff502c">setState</font> 的批量更新优化也是建立在<font color="#ff502c">“异步”（合成事件、钩子函数）</font>之上的，在原生事件和 setTimeout 中不会批量更新，在“异步”中如果对同一个值进行<font color="#ff502c">多次 setState </font>， setState 的批量更新策略会对其进行<font color="#ff502c">覆盖</font>，取最后一次的执行，如果是同时 setState 多个不同的值，在更新时会对其进行<font color="#ff502c">合并批量更新</font>。
