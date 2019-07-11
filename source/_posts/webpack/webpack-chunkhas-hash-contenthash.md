---
title: webpack中的hash、chunkhash、contenthash分别是什么
date: 2019-06-30 20:23:35
tags: [WebPack]
categories: [WebPack]
description: webpack中的hash、chunkhash、contenthash分别是什么,在什么场景下用什么样的hash
---
## 简介
在webpack中有三种hash可以配置，分别是hash、chunkhash、contenthash他们是不对的可以针对不同的配置，首相要搞清楚这三种的hash的区别，什么场景下，适合用哪种。
**hash**
所有文件哈希值相同，只要改变内容跟之前的不一致，所有哈希值都改变，没有做到缓存意义
**chunkhash**
当有多个chunk，形成多个bundle时，如果只有一个chunk和一个bundle内容变了，其他的bundle的hash都会发生变化，因为大家都是公用的一个hash，这个时候chunkhash的作用就出来了。它根据不同的入口文件(Entry)进行依赖文件解析、构建对应的 chunk，生成对应的哈希值。
**contenthash**
在打包的时候我们会在js中导入css文件，因为他们是同一个入口文件，如果我只改了js得代码，但是他的css抽取生成css文件时候hash也会跟着变换。这个时候contenthash的作用就出来了。
下面直接用代码验证上面的猜想。
### hash

### chunkhash

### contenthash

## 总结
hash 所有文件哈希值相同；
chunkhash 根据不同的入口文件(Entry)进行依赖文件解析、构建对应的 chunk，生成对应的哈希值；
contenthash 计算与文件内容本身相关，主要用在css抽取css文件时。
## 引用
> [https://juejin.im/post/5cede821f265da1bbd4b5630](https://juejin.im/post/5cede821f265da1bbd4b5630)