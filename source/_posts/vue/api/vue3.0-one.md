---
title: Vue 3.0 API 学习
date: 2021-01-01 12:32:33
tags: [Docs, Vue]
categories: [Vue]
description: Vue 3.0 已经是公测版很久了，大家逐渐都在升级Vue 3.0版本，提前学习一下为未来升级Vue 3.0 作准备。
---

## 简介

首先通篇阅读一下[Vue 3.0](https://v3.cn.vuejs.org/api/application-api.html#provide)的文档，后面也不会有很大的修改，本篇文章主要是用于增加**Vue 3.0**变化Api的理解和应用。

初始化一个**Vue实例**在Vue 2.x中通过`new Vue({ el: '#app' })`实现或者通过`new Vue({}).$mount('#app')`实现；
**Vue 3.0**在中要通过`createApp`来创建一个实例，代码如下所示:
```javascript
  
  import { createApp } from 'vue';

  const app = createApp({})
```

## 应用配置

`errorHandler/warnHandler`用于监听Vue内部运行出现的报错和警告，报错可以结合`sertry 和 bugsnag`来记录报错信息。使用代码：

``