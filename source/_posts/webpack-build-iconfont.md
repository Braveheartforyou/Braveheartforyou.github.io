---
title: webpack-build-iconfont
date: 2017-10-26 14:49:40
tags: [WebPack]
categories: [WebPack]
description: 用vue-cli打包iconfont
---
## 简述
在使用webpack打包自己在icomoon中的制作的iconfont,即使放到最外层的static文件家中，打包后它的路径也是不对的，这个在vue的github的issue中有人是这么解决的
在build文件夹中的utils.js中用vue-style-loader来编译css,修改如下
原
```javascript
    if (options.extract) {
        return ExtractTextPlugin.extract({
            use: loaders,
            fallback: 'vue-style-loader'
        })
    } else {
        return ['vue-style-loader'].concat(loaders)
    }
```
添加路径后
```javascript
    if (options.extract) {
        return ExtractTextPlugin.extract({
            use: loaders,
            fallback: 'vue-style-loader',
            // 添加路径
            publicPath: '../../'
        })
    } else {
        return ['vue-style-loader'].concat(loaders)
    }
```
这样编译完成以后的iconfont路径是正确的