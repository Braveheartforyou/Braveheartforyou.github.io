---
title: Vue中的依赖收集
date: 2019-06-08 17:19:46
tags: [Vue]
categories: [Vue]
description: Vue首先会会通过Obsever,创建响应式数据,并且在getter中做依赖收集setter中派发。
---
## 简介
依赖收集是