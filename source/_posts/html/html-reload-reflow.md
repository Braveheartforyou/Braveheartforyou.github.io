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
reflow就是对<font color="blue">Dom树排版有影响的样式变化，对所有受影响的dom节点进行重新排版工作</font>。例如，改变一个元素的宽高、改变一个元素的display、float等。
## repaint(重绘)和reflow(回流)说的时那一阶段
我们知道repaint(重绘)和reflow(回流)的大致的触发条件和所做的事情。再根据[浏览器渲染原理 （一）在网址中输入一个网站后面都做了什么](/blog/html/html-style-javascript.html)这篇文章中已经大致了解浏览器的渲染过程。
所以得出结论<font color="blue">repaint(重绘)和reflow(回流)</font>发生在<font color="blue">将DOM树和CSSOM树合并成一个渲染树(rendering tree)</font>；这一阶段之后的<font color="blue">layout布局</font>和下一阶段的<font color="blue">绘制paint</font>.


