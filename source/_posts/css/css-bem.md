---
title: css中的BEM简介
date: 2018-09-20 16:34:25
tags: [Css]
categories: [Css]
description: BEM到底是什么
---
## 简介
**BEM其实是一种CSS书写规范中的一种**，使用BEM规范来命名CSS，组织HTML中选择器的结构，利于CSS代码的维护，使得代码结构更清晰（弊端主要是名字会稍长）。
BEM代表的是**块（block）**、**元素（element）**、**修饰符（modifier）**，是由Yandex团队提出的一种前端命名方法论。

在选择其中，由一下三种符号来表示扩展的关系：
```
-   中划线 ：仅作为连字符使用，表示某个块或者某个子元素的多单词之间的连接记号。
__  双下划线：双下划线用来连接块和块的子元素
_   单下划线：单下划线用来描述一个块或者块的子元素的一种状态

type-block__element_modifier
```

## 块（block）
一个快是设计或者布局的一部分，它有具体且唯一的意义，语义上的或视觉上的。
在大多数情况下，任何**独立的页面元素**（或复杂或简单）都可以被视作一个块。它的HTML容器会有一个**唯一的CSS类名**，也就是这个块的名字。
针对块的CSS类名会加一些前缀（ ui-），这些前缀在CSS中有类似 **命名空间** 的作用。
一个块的正式（实际上是半正式的）定义有下面三个基本原则：
1. CSS中只能使用类名（不能是ID）。
2. 每一个块名应该有一个命名空间（前缀）
3. 每一条CSS规则必须属于一个块。

一个自定义列表样例如下：
```html
  // html
  <ul class="list">
    <li>列表项一</li>
    <li>列表项二</li>
  </ul>
  // css
  <style>
    .list {}
  </style>
```
通常会把`ul`看做一个完整得块，所以在`ul`上面定义一个块的`class`为`list`。

## 元素（element）
块中的子元素是块的子元素，并且子元素的子元素在 **bem** 里也被认为是块的直接子元素。**一个块中元素的类名必须用父级块的名称作为前缀**。
在上面的例子上扩展如下：
```html
  // html
  <ul class="list">
    <li class="list__item">列表项一</li>
    <li class="list__item">列表项二</li>
  </ul>
  // css
  <style>
    .list {}
    .list__item {}
  </style>
```
因为每一个`li`都是`ul`的子元素，所以在`li`上定义一个为`list__item`的`class`。

## 修饰符（modifier）
一个“修饰符”可以理解为一个块的**特定状态**，标识着它持有一个**特定的属性**。
在上面的例子上扩展如下：
```html
  // html
  <ul class="list">
    <li class="list__item list__item_active">列表项一</li>
    <li class="list__item">列表项二</li>
  </ul>
  // css
  <style>
    .list {}
    .list__item {}
    .list__item_active {}
  </style>
```
假如`ul`是一个`menu`，它的第一项默认是**选中状态**给第一项添加一个选中的`class`为`list__item_active`。

## 书写原则
1. **原则上不会出现*2层以上*选择器嵌套**
使用`BEM`原则，用命名来解耦，所有类名都为一层，增加效率和复用性

2. **两层选择器嵌套出现在`.mod-xxx__item_current`子元素的情况**

请看下面一个样例：
```html
// html
<ul class="xxx">
    <li class="xxx__item">第一项
        <div class="xxx__product-name">我是名称</div>
        <span class="xxx__ming-zi-ke-yi-hen-chang">看类名</span>
        <a href="#" class="xxx__link">我是link</a>
    <li>
    <li class="xxx__item xxx__item_current">第二项 且 当前选择项
        <div class="xxx__product-name">我是名称</div>
        <a href="#" class="xxx__item-link">我是link</a>
    <li>
    <li class="xxx__item xxx__item_hightlight">第三项 且 特殊高亮
         <div class="xxx__product-name">我是名称</div>
        <a href="#" class="xxx__item-link">我是link</a>
    <li>
</ul>

// 常规写法：
<style>
  .xxx{}
  .xxx__item{}
  .xxx__item_current{}
  // 嵌套写法
  .xxx__item_current .mod-xxx__link{}
</style>

// 推荐：
<style>
.xxx{}
.xxx__item{}
.xxx__item_hightlight{}
.xxx__product-name{}
.xxx__link{}
.xxx__ming-zi-ke-yi-hen-chang{}

// 嵌套写法
.xxx__item_current{
  .xxx__link{}
}
</style>
```

## BEM 解决问题