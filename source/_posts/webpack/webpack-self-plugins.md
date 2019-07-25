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
`Webpack` 通过 `Plugin` 机制让其更加灵活，以适应各种应用场景。 在 `Webpack` 运行的生命周期中会广播出许多事件，`Plugin` 可以监听这些事件，在合适的时机通过 `Webpack` 提供的 `API` 改变输出结果。

## 实现一个plugin
一个webpack plugin基本包含以下几步：
1. 一个JavaScript函数或者类
2. 在函数原型（prototype）中定义一个注入`compiler`对象的`apply`方法。
3. `apply`函数中通过`compiler`插入指定的事件钩子，在钩子回调中拿到`compilation`对象
4. 使用`compilation`操纵修改`webapack`内部实例数据。
5. 异步插件，数据处理完后使用`callback`回调

