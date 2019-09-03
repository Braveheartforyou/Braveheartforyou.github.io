---
title: Vue中v-model解析、sync修饰符解析、$attr、$listeners
date: 2019-08-29 20:22:18
tags: [Vue]
categories: [Vue]
description:  Vue中v-model解析、sync修饰符解析、$attr、$listeners
---
## 简介

---

在平时开发是经常用到一些父子组件通信，经常用到`props`、`vuex`等等，这里面记录另外的三种方式`v-model`、`sync`和`$attr`、`$listeners`是怎么使用，再说是怎么实现，其实`v-model`、`sync`都是语法糖。

## 使用方式

---

### v-model

`v-mode1`其实就是一个语法糖，默认会利用名为`value`的`props`和名为`input`