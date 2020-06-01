---
title: css中的BEM
date: 2018-09-20 16:34:25
tags: [Css]
categories: [Css]
description: BEM到底是什么
---

## 简介

`**BEM 其实是一种 CSS 书写规范中的一种**`，使用 `BEM 规范`来`命名 CSS`，组织 HTML 中选择器的结构，利于 CSS 代码的维护，使得代码结构更清晰（弊端主要是名字会稍长）。
BEM 代表的是`**块（block）**、**元素（element）**、**修饰符（modifier）**`，是由 Yandex 团队提出的一种前端命名方法论。

在选择其中，由一下三种符号来表示扩展的关系：

```javascript
  -   中划线 ：仅作为连字符使用，表示某个块或者某个子元素的多单词之间的连接记号。
  __  双下划线：双下划线用来连接块和块的子元素
  _   单下划线：单下划线用来描述一个块或者块的子元素的一种状态

  type-block__element_modifier
```

## 块（block）

一个快是设计或者布局的一部分，它有具体且唯一的意义，语义上的或视觉上的。
在大多数情况下，任何`**独立的页面元素**`（或复杂或简单）都可以被视作一个块。它的 HTML 容器会有一个`**唯一的 CSS 类名**`，也就是这个块的名字。
针对块的 CSS 类名会加一些前缀（ ui-），这些前缀在 CSS 中有类似 **命名空间** 的作用。
一个块的正式（实际上是半正式的）定义有下面三个基本原则：

1. CSS 中只能使用类名（不能是 ID）。
2. 每一个块名应该有一个命名空间（前缀）
3. 每一条 CSS 规则必须属于一个块。

一个自定义列表样例如下：

```html
// html
<ul class="list">
  <li>列表项一</li>
  <li>列表项二</li>
</ul>
// css
<style>
  .list {
  }
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
  .list {
  }
  .list__item {
  }
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
  .list {
  }
  .list__item {
  }
  .list__item_active {
  }
</style>
```

假如`ul`是一个`menu`，它的第一项默认是**选中状态**给第一项添加一个选中的`class`为`list__item_active`。

## 书写原则

1. **原则上不会出现*2 层以上*选择器嵌套**

使用`BEM`原则，用命名来解耦，所有类名都为一层，增加效率和复用性

2. **两层选择器嵌套出现在`.mod-xxx__item_current`子元素的情况**

请看下面一个样例：

```html
// html
<ul class="xxx">
  <li class="xxx__item">
    第一项
    <div class="xxx__product-name">我是名称</div>
    <span class="xxx__ming-zi-ke-yi-hen-chang">看类名</span>
    <a href="#" class="xxx__link">我是link</a>
  </li>

  <li></li>
  <li class="xxx__item xxx__item_current">
    第二项 且 当前选择项
    <div class="xxx__product-name">我是名称</div>
    <a href="#" class="xxx__item-link">我是link</a>
  </li>

  <li></li>
  <li class="xxx__item xxx__item_hightlight">
    第三项 且 特殊高亮
    <div class="xxx__product-name">我是名称</div>
    <a href="#" class="xxx__item-link">我是link</a>
  </li>

  <li></li>
</ul>

// 常规写法：
<style>
  .xxx {
  }
  .xxx__item {
  }
  .xxx__item_current {
  }
  // 嵌套写法
  .xxx__item_current .mod-xxx__link {
  }
</style>

// 推荐：
<style>
  .xxx {
  }
  .xxx__item {
  }
  .xxx__item_hightlight {
  }
  .xxx__product-name {
  }
  .xxx__link {
  }
  .xxx__ming-zi-ke-yi-hen-chang {
  }

  // 嵌套写法
  .xxx__item_current {
    .xxx__link {
    }
  }
</style>
```

## BEM 解决问题

组件之间的完全解耦，不会造成命名空间的污染，如：`.mod-xxx ul li` 的写法带来的潜在的嵌套风险。

## 总结

BEM 规则的应用规则如下：

- `一个独立的（语义上或视觉上），可以复用而不依赖其它组件的部分，可作为一个块（Block）`
- `属于块的某部分，可作为一个元素（Element）`
- `用于修饰块或元素，体现出外形行为状态等特征的，可作为一个修饰器（Modifier）`

## 参考

[[规范] CSS BEM 书写规范](https://github.com/Tencent/tmt-workflow/wiki/%E2%92%9B-%5B%E8%A7%84%E8%8C%83%5D--CSS-BEM-%E4%B9%A6%E5%86%99%E8%A7%84%E8%8C%83)
[使用 BEM 命名规范来组织 CSS 代码](https://www.cnblogs.com/imwtr/p/8521031.html)
[BEM 思想之彻底弄清 BEM 语法](https://www.w3cplus.com/css/mindbemding-getting-your-head-round-bem-syntax.html)
