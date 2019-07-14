---
title: ios中1px border解决方法
date: 2017-07-27 10:29:59
tags: [Css]
categories: [mobile]
description: 解决retina屏幕时border大于1时产生的兼容性问题
---
## 问题描述
### 1、引入flexible方案
可通过 手淘的 <font color="#ff502c">flexible方案</font>来解决这个问题
<font color="#ff502c">flexible的原理讲解和使用 <http://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html></font>
<font color="#ff502c">flexible 库地址：<https://github.com/amfe/lib-flexible></font>
#### 基本原理
在devicePixelRatio = 2 时，输出viewport
```html
    <meta name="viewport" content="initial-scale=0.5, maximum-scale=0.5, minimum-scale=0.5, user-scalable=no">
```
在devicePixelRatio = 3 时，输出viewport
```html
    <meta name="viewport" content="initial-scale=0.3333333333333333, maximum-scale=0.3333333333333333, minimum-scale=0.3333333333333333, user-scalable=no">
```
同时通过设置对应viewport的rem基准值，这种方式就可以像以前一样轻松愉快的写1px了。
### 2、 transform: scale(0.5)
使用css3的transfrom的scale属性
```css
    .weui-btn:after {
        content: " ";
        width: 200%;
        height: 200%;
        position: absolute;
        top: 0;
        left: 0;
        border: 1px solid rgba(0, 0, 0, 0.2);
        -webkit-transform: scale(0.5);
        transform: scale(0.5);
        -webkit-transform-origin: 0 0;
        transform-origin: 0 0;
        box-sizing: border-box;
        border-radius: 10px;
    }
```
可参考微信的 web ui中的解决方案
参考地址 <https://weui.io/#button>
### 可用图片实现代替边框
不建议使用