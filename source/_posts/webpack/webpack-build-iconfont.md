---
title: vue-cli中引入static文件中的iconfont无法显示解决方法
date: 2017-10-26 14:49:40
tags: [WebPack]
categories: [WebPack]
description: 用vue-cli打包iconfont
---

## 简述

在使用 webpack 打包自己在 icomoon 中的制作的 iconfont,即使放到最外层的 static 文件家中，打包后它的路径也是不对的，这个在 vue 的 github 的 issue 中有人是这么解决的
在 build 文件夹中的 utils.js 中用 vue-style-loader 来编译 css,修改如下
原

```javascript
if (options.extract) {
  return ExtractTextPlugin.extract({
    use: loaders,
    fallback: "vue-style-loader"
  });
} else {
  return ["vue-style-loader"].concat(loaders);
}
```

添加路径后

```javascript
if (options.extract) {
  return ExtractTextPlugin.extract({
    use: loaders,
    fallback: "vue-style-loader",
    // 添加路径
    publicPath: "../../"
  });
} else {
  return ["vue-style-loader"].concat(loaders);
}
```

这样编译完成以后的 iconfont 路径是正确的
