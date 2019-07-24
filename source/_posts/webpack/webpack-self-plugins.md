---
title: 编写一个自己的webpack插件plugin
date: 2019-05-22 11:38:39
tags: [WebPack]
categories: [WebPack]
description: 编写一个自己的webpack插件plugin
---
## 简介
插件是 webpack 的支柱功能。`webpack` 自身也是构建于，你在 `webpack` 配置中用到的相同的插件系统之上！插件目的在于解决 `loader` 无法实现的其他事。
要想写好插件就要知道`Webpack`中的两个比较核心的概念`compiler`、`compilation`、`tapable`。在[webpack编译流程](/blog/webpack/webpack-process.html)已经都要记录。

