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

首先它支持一下