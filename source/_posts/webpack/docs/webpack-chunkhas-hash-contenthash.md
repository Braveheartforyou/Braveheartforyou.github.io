---
title: webpack中的hash、chunkhash、contenthash分别是什么
date: 2019-06-30 20:23:35
tags: [WebPack]
categories: [WebPack]
description: webpack中的hash、chunkhash、contenthash分别是什么,在什么场景下用什么样的hash
---

**_知人者智，自知者明，胜人者有力，自胜者强。——老子_**

## 简介

在`webpack`中有三种`hash`可以配置，分别是`hash`、`chunkhash`、`contenthash`他们是不对的可以针对不同的配置，首相要搞清楚这三种的`hash`的区别，什么场景下，适合用哪种。

**hash**
所有文件哈希值相同，只要改变内容跟之前的不一致，所有哈希值都改变，没有做到缓存意义

**chunkhash**
当有多个`chunk`，形成多个`bundle`时，如果只有一个`chunk`和一个`bundle`内容变了，其他的`bundle`的`hash`都会发生变化，因为大家都是公用的一个`hash`，这个时候`chunkhash`的作用就出来了。它根据不同的入口文件`(Entry)`进行依赖文件解析、构建对应的 `chunk`，生成对应的**哈希值**。

**contenthash**
在打包的时候我们会在`js`中导入`css`文件，因为他们是同一个入口文件，如果我只改了`js`得代码，但是他的`css`抽取生成`css`文件时候`hash`也会跟着变换。这个时候`contenthash`的作用就出来了。

下面直接用代码验证上面的猜想。

## 一个简单的 webpack 配置

我们现在就只创建一个能编译`js`的`webpack`配置，步骤如下：

1. 创建一个空文件加，并且在当文件夹中打开 `bash or cmd`。
2. `npm init -y` 生成`package.json`。
3. 如果你安装了`cnpm or yarn` 就执行 `cnpm i webpack webpack-cli -D`, 安装`webpack`的包。
4. 创建`src`，在`src`内部创建`chunk0.js`、`chunk1.js`、`common.js`、`index.js`、`style.css`，并且编写内部代码
5. 在项目根目录创建 `webpack.config.js`
6. 直接在`cmd`中运行 `webpack`

文件目录如下：
![webpack contenthash hash chunkhash](../../images/webpack/webpack-1-3.png)

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

**style.css**

```css
body {
  background: #000;
}
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
    filename: '[name].[hash].js' // 出口文件
  }
};
```

## hash

我们直接运行`webpack`，运行结果如下图所示：

只有一个 hash，所有文件的 hash 都是相同：

![webpack contenthash hash chunkhash](../../images/webpack/webpack-1-4.png)
如果我们改变修改**chunk1.js**中的代码：

```javascript
export default function flow() {
  return 'flow1'; // flow => folw1
}
```

再运行 webpack 发现所有的 hash 都<font color="#ff502c">变化</font>了，如下图所示：

![webpack contenthash hash chunkhash](../../images/webpack/webpack-1-4.png)
![webpack contenthash hash chunkhash](../../images/webpack/webpack-1-5.png)

对比发现他们的 hash 并不相同了，这个时候如果想修改了**chunk1.js**,index.js 不产生变化，就要用到 chunkhash。

## chunkhash

- 第一步 我们先把**webpack.config.js**做一下修改

```javascript
module.exports = {
  mode: 'production',
  entry: {
    index: './src/index.js',
    chunk1: './src/chunk1.js'
  },
  output: {
    filename: '[name].[chunkhash].js' // hash => chunkhash
  }
};
```

- 第二步 我们运行`webpack`

![webpack contenthash hash chunkhash](../../images/webpack/webpack-1-6.png)
根据上面图片发下，两个`chunk`的`hash`并不相同了。

- 第三部 我们修改 `chunk1.js`

```javascript
export default function flow() {
  return 'flow11111'; // flow1 => flow11111
}
```

- 再运行`webpack`

![webpack contenthash hash chunkhash](../../images/webpack/webpack-1-6.png)
![webpack contenthash hash chunkhash](../../images/webpack/webpack-1-7.png)

根据图片我们看到了`chunk1.js`的`hash`变化，而`index.js`的`hash`并没有变化，达到了我们预期的效果，对我们线上的缓存也是比较好的。

## contenthash

但是当我们一个`js`文件里面引用了一个`css`文件，如果我么修改了`css`文件内的内容，我们`css`中的内容，会发发现这整个`bundle`的`hash`也会发生更新。
我们要引入`css`，并且要把`css`提出、压缩生成一个`css`文件，就要借助一个`webpack`的插件，叫做`MiniCssExtractPlugin`,他可以帮我提取 css 到 css 文件，并且压缩 css。

- 第一步先安装`css-loader`、`mini-css-extract-plugin`包

```bash
cnpm install css-loader mini-css-extract-plugin -D
```

- 第二步修改`webpack.config.js` 如下

```javascript
const MiniCssExtractPlugin = require('mini-css-extract-plugin'); // 新增
module.exports = {
  mode: 'production',
  entry: {
    index: './src/index.js',
    chunk1: './src/chunk1.js'
  },
  output: {
    filename: '[name].[chunkhash].js'
  },
  module: {
    // 新增
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    // 新增
    // 提取css插件
    new MiniCssExtractPlugin({
      // Options similar to the same options in webpackOptions.output
      // both options are optional
      filename: '[name].[chunkhash].css'
    })
  ]
};
```

- 第三步运行 webpack

![webpack contenthash hash chunkhash](../../images/webpack/webpack-1-8.png)

看代码可以看到`index.css`和`index.js`的`hash`是一样的。

- 第四步修改 style.css

```javascript
html {
  font-size: 13px;
}
```

- 第五步运行 webpack

![webpack contenthash hash chunkhash](../../images/webpack/webpack-1-8.png)
![webpack contenthash hash chunkhash](../../images/webpack/webpack-1-9.png)

对比两次构建的`hash`，发现只修改了`style.css`的文件，引入他的`index.js`确也更新了`hash`，这个时候就需要`contenthash`来发挥作用了。

- 第六步修改`webpack.config.js` 并且运行 webpack

```javascript
const MiniCssExtractPlugin = require('mini-css-extract-plugin'); // 新增
module.exports = {
  mode: 'production',
  entry: {
    index: './src/index.js',
    chunk1: './src/chunk1.js'
  },
  output: {
    filename: '[name].[chunkhash].js'
  },
  module: {
    // 新增
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    // 新增
    // 提取css插件
    new MiniCssExtractPlugin({
      // Options similar to the same options in webpackOptions.output
      // both options are optional
      filename: '[name].[contenthash].css'
    })
  ]
};
```

![webpack contenthash hash chunkhash](../../images/webpack/webpack-1-10.png)

看到他们直接 hash 就是不同的。

- 修改`common.js`，直接运行 webpack

```javascript
export default function commonJs() {
  return 'commonJs1';
}
```

![webpack contenthash hash chunkhash](../../images/webpack/webpack-1-10.png)
![webpack contenthash hash chunkhash](../../images/webpack/webpack-1-11.png)

看到修改 js 时我们的 css 文件的 hash 并没有变更。

> 注意，当使用`contenthash`时，如果修改 js 文件，css 文件的 hash 不会变化，但是修改 js 的文件，css 文件的 hash 也会变化。

## 总结

**hash 所有文件哈希值相同；**
**chunkhash 根据不同的入口文件(Entry)进行依赖文件解析、构建对应的 chunk，生成对应的哈希值；**
**contenthash 计算与文件内容本身相关，主要用在 css 抽取 css 文件时。**

## 参考

[面试必备！webpack 中那些最易混淆的 5 个知识点](https://juejin.im/post/5cede821f265da1bbd4b5630)
