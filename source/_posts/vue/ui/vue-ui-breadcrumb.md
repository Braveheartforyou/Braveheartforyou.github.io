---
title: Vue-Ui 手写实现以下Breadcrumb面包屑组件(初级难度)
date: 2019-11-20 09:42:12
tags: [Ui框架]
categories: [Vue, Ui框架]
description: 在日常开发中会用到很多Ui框架，本系列文章会从简单到复杂实现一套自己Ui。本篇文章中从0开始手写一个面包屑组件Breadcrumb。
---

## 简介

---

我们开始只关注组件的功能实现，不考虑css分装、webpack配置、整体结构设计、单元测试等等，因为在后面会一步一步完善。让大家一起进步，实现一套简单的组件库。

在日常我们开发PC页面时经常会用到一个面包屑导航的功能，其实这个功能算是比较简单的功能，基本上用过`Vue`这个框架的人都能自己写出来。但是既然要写一个通用的可能就不是那么容易实现，我们首先要了解`Breadcrumb`它都有什么功能。下面我们就先分析它都有什么功能，可以参考`element/iview`这种流行的`Ui框架`。

1. 分析`Breadcrumb`组件功能
2. 构思代码、编写代码
3. 测试组件效果，编写单元测试

按照上面的三步一步一步的实现自己一个自己`Breadcrumb`组件，废话不多说直接开干。

## 分析组件功能

我们可以去`element/iview`官方网去阅读一下他们的[文档](https://element.eleme.cn/2.13/#/zh-CN/component/breadcrumb)，在去`github`中看一下他们的源码。首先看一下他们是怎么使用，下面以`element`为例。
**示例**

```html
  <el-breadcrumb separator="/">
    <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item><a href="/">活动管理</a></el-breadcrumb-item>
    <el-breadcrumb-item>活动列表</el-breadcrumb-item>
    <el-breadcrumb-item>活动详情</el-breadcrumb-item>
  </el-breadcrumb>
```

效果图：
![breadcrumb](../../../images/vue-ui/breadcrumb-1.png)

根据上面和代码我们可以看出`Breadcrumb`有两个组件，分别为：

`breadcrumb`组件，并且它接受两个`props`属性：

- `sparator(props)`: 它是用来替换默认`/`分隔符的，并且它的类型为`String`类型。默认`/`
- `sparatorClass(props)`: 它是用来给填充`iconfont`这种的图标分隔符，并且它的类型为`String`类型。没有默认值

`breadcrumb-item`组件，它是被`breadcrumb`包裹的组件，它也接受两个`props`属性：

- `to(props)`: 路由跳转对象，同 `vue-router` 的 `to`, 并且它的类型为`String/Object`类型。没有默认值
- `replace(props)`: 在使用 `to` 进行路由跳转时，启用 `replace` 将不会向 `history` 添加新记录, 类型是`Boolean`。默认`false`

我们大致知道了有两个组件，组件之间有嵌套关系，并且分别都支持两个`props`参数。并且有的`props`还有默认参数。下面我们就来一步一步实现自己已经知道的功能和配置。

## 实现组件

在实现组件之前我们要考虑几个问题点：

- 怎么把多个`breadcrumb-item`放进`breadcrumb`组件里？
- 父组件传入的值怎么传入子组件？

第一个问题我们又很多种实现方式，比如说通过`循环`实现。多传入一下子数组项，
另一个我们可以通过`slot`