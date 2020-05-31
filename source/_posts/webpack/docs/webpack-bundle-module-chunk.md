---
title: webpack中的bundle、module、chunk分别是什么
date: 2019-06-26 21:43:12
tags: [WebPack]
categories: [WebPack]
description: webpack中的bundle、module、chunk分别是什么
---

**_祸兮福之所倚，福兮祸之所伏。——老子_**

## 简介

---

`bundle`、`module`、`chunk` 都是 webpack 中的术语，下面就一个一个介绍他们的定义是什么，怎么产生的。

### Bundle

**Bundle**是由多个不同的模块生成，bundles 包含了早已经过加载和编译的最终源文件版本。
**Bundle 分离（Bundle Splitting）:**这个流程提供了一个优化 build 的方法，允许 webpack 为应用程序生成多个 bundle。最终效果是，当其他某些 bundle 的改动时，彼此独立的另一些 bundle 都可以不受到影响，减少需要重新发布的代码量，因此由客户端重新下载并利用浏览器缓存。

### Module

**模块（Module）**提供比较完整程序接触面（surface area）更小的离散功能块。精心编写的模块提供了可靠的抽象和封装界限，使得应用程序中每个模块都具有条理清楚的设计和明确的目的。
**模块解析（Module Resolution）**一个模块可以作为另一个模块的依赖模块，resolver 是一个库（libary）用于帮助找不到模块的绝对路径，模块将在**resolve.modules**中指定的所有目录内搜索。

### Chunk

**Chunk**这是 webpack 特定的术语被用在内部来管理 building 过程。bundle 是由 chunk 组成，其中有几种类型（例如，入口 chunk(entry chunk)和子 chunk(child chunk)）。通常 chunk 会直接对应所输出的 bundle，但是有一些配置并不会产生一对一的关系。
**代码分离（Code Splitting）**指将代码分离到每个 bundles/chunks 里面，你可以按需加载，而不是加载一个包含全部的 bundle。
**配置（Configuration）**webpack 的配置文件是一个普通的 JavaScript 文件，它导出为一个对象。然后由 webpack 根据这个对象定义的属性进行处理。

## Bundle VS Chunk VS Module

我们从定义和时期来说：

- “模块”`(module)`的概念大家都比较熟悉，如 `CommonJS 模块`、`AMD`、`ES6 Modules` 模块
- `chunk` 表示打包的时候产生得模块，由他来组成 `bundle`
- 打包完成的源代码

我们现在就只创建一个能编译`js`的`webpack`配置，步骤如下：

1. 创建一个空文件加，并且在当文件夹中打开 `bash or cmd`。
2. `npm init -y` 生成`package.json`。
3. 如果你安装了`cnpm or yarn` 就执行 `cnpm i webpack webpack-cli -D`, 安装`webpack`的包。
4. 创建`src`，在`src`内部创建`chunk0.js`、`chunk1.js`、`common.js`、`index.js`、`style.css`，并且编写内部代码
5. 在项目根目录创建 `webpack.config.js`
6. 直接在`cmd`中运行 `webpack`

下面是代码
**chunk0.js**

```javascript
export default function add(a, b) {
  return a + b;
}
```

**chunk1.js**

```javascript
export default function flow() {
  return 'flow';
}
```

**common.js**

```javascript
export default function commonJs() {
  return 'commonJs';
}
```

**index.js**

```javascript
import add from './chunk0.js';
import commonJs from './common';
console.log(add(1, 2));
console.log(commonJs());
```

**webpack.config.js**

```javascript
module.exports = {
  mode: 'production', // 如果不添加就会警告
  entry: {
    index: './src/index.js', // 一个入口文件
    chunk1: './src/chunk1.js' // 两一个入口文件
  },
  output: {
    filename: '[name].bundle.js' // 出口文件
  }
};
```

运行的效果如下
![webpack bundle module chunk](../../images/webpack/webpack1-1.png)

通过上面的代码知道，`module` 就是没有被编译之前的代码，通过 `webpack` 的根据文件引用关系生成 `chunk` 文件，webpack 处理好 `chunk` 文件后，生成运行在浏览器中的代码 `bundle`。

## 参考

[面试必备！webpack 中那些最易混淆的 5 个知识点](https://juejin.im/post/5cede821f265da1bbd4b5630)[深入理解 Webpack 打包分块（上）](http://qingbob.com/webpack-chunks-split-01/)
