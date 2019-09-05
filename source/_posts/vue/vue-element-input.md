---
title: element-ui组件解读 （一） input组件
date: 2019-08-21 15:27:13
tags: [Vue]
categories: [Vue]
description:  element-ui中的input怎么实现值的双向绑定、自适应高度的
---

***察见渊鱼者不详，智料隐匿者有殃。——《列子·说符》***

## 简介

---

在日常开发`PC`管理端界面的时候，会用到**element-ui**或者**iview**ui框架，比较常用的`input`组件是怎么封装的呢，以`element-ui`框架的`input`组件为例，分析一下它是怎么封装实现的，也可以为以后自己封装框架提供思路。

因为代码还是挺多的，主要分析几个点：

- 支持**前置内容**、**后置内容**、**后置元素**
- 支持所有原声的`type`，并且可以切换`password`模式
- 支持`readonly`、`disabled`、`autocomplete`、`maxlength`、`minlength`等等
- 支持常规`input`、`focus`、`blur`、`change`，支持新事件`compositionstart`、`compositionupdate`、`compositionend`
- `element-ui`是怎么做到类似与`vue`的双向绑定的
- `type`为`textare`模式时，做到`calcTextareaHeight`

下面就一步一步分下，主要的点会着重分析，比较简单或者比较不重要的点会快速带过。

源码参考[element-ui input](https://github.com/ElemeFE/element/blob/dev/packages/input/src/input.vue)

## 前置、后置内容

