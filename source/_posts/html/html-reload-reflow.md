---
title: 浏览器渲染原理 （三）repaint(重绘)和reflow(回流)详解
date: 2019-06-23 08:00:46
tags: [Html]
categories: [Html]
description: 在浏览器渲染原理（一）中，我们知道了大致的渲染步骤，但是经常说的repaint(重绘)和reflow(回流)我们还不清楚它发生在那个阶段，和为什么会发生reload和reflow，怎么优化他们。
---
> [浏览器渲染原理 （一）在网址中输入一个网站后面都做了什么](/blog/html/html-style-javascript.html)
> [浏览器渲染原理 （二）css、javascript、dom阻塞关系](/blog/html/html-browser-render.html)
> [浏览器渲染原理 （三） repaint(重绘)和reflow(回流)详解](/blog/html/html-reload-reflow.html)

## 简介
repaint(重绘)和reflow(回流)发生在什么渲染的那个阶段，我们要了解什么叫做repaint(重绘)和reflow(回流).
**repaint(重绘)**
<font color="blue"></font>
repaint就是在<font color="blue">不影响排版的情况下</font>对这个元素重新绘制的过程。例如改变一个元素的背景颜色、字体颜色等。
**reflow(回流、重排)**
reflow就是对<font color="blue">当浏览器发现某个部分发生了点变化影响了布局，需要倒回去重新渲染。reflow 会从<html>这个 root frame 开始递归往下，依次计算所有的结点几何尺寸和位置。reflow 几乎是无法避免的。现在界面上流行的一些效果，比如树状目录的折叠、展开（实质上是元素的显示与隐藏）等，都将引起浏览器的 reflow。鼠标滑过、点击……只要这些行为引起了页面上某些元素的占位面积、定位方式、边距等属性的变化，都会引起它内部、周围甚至整个页面的重新渲染。通常我们都无法预估浏览器到底会 reflow 哪一部分的代码，它们都彼此相互影响着。
## repaint(重绘)和reflow(回流)说的时那一阶段
我们知道repaint(重绘)和reflow(回流)的大致的触发条件和所做的事情。再根据[浏览器渲染原理 （一）在网址中输入一个网站后面都做了什么](/blog/html/html-style-javascript.html)这篇文章中已经大致了解浏览器的渲染过程。



