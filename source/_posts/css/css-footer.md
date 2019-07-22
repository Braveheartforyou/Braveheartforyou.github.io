---
title: css实现Sticky footer效果
date: 2019-07-20 16:12:43
tags: [css]
categories: [css]
description: css实现Sticky footer效果
---
## 简介
在网页设计中，Sticky footers设计是最古老和最常见的效果之一，大多数人都曾经经历过。所谓 “Sticky Footer”，并不是什么新的前端概念和技术，它指的就是一种网页效果： **如果页面内容不足够长时，页脚固定在浏览器窗口的底部；如果内容足够长时，页脚固定在页面的最底部。但如果网页内容不够长，置底的页脚就会保持在浏览器窗口底部**。

大致分为两种，一种是footer为固定高度、一种为不固定的高度，当然不固定高度是包含了固定高度的，他们实现方式是不同的。
## 固定footer高度
- 负margin-bottom实现
- calc()
- position: absolute

### 负margin-bottom实现
把wrapper部分最小高度设为100%，再利用内容部分的负底部外边距值来达到当高度不满时，页脚保持在窗口底部，当高度超出则随之推出的效果。
html代码如下：
```html
    <div class="wrapper">
        <div>100</div>
        <div>100</div>
        <div>100</div>
    </div>
    <footer class="footer"></footer>
    <style>
        html,
        body { height: 100%; margin: 0; }
        .wrapper {
            min-height: 100%;
            /* 等于footer的高度 */
            margin-bottom: -50px;
            background: #0ff00f;
        }
        .footer{
            height: 50px;
            background: #ff0ff0;
        }
    </style>
```
需要注意的是`.wrapper`的 `margin-bottom`值需要和`.footer`的负的`height`值保持一致，这一点不太友好。

### calc()
使用CSS3新增的计算函数calc()
```html
    <div class="wrapper">
        <div class="content">
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
        </div>
        <footer class="footer"></footer>
    </div>
    <style>
        .content {
            /* 等于footer的高度 */
            min-height: calc(100vh - 50px);
            background: #0ff00f;
        }
        .footer {
            height: 50px;
            background: #ff0ff0;
        }
    </style>
```
如果不需考虑 calc() 以及 vh 单位的兼容情况，这是个很理想的实现方案。同样的问题是 footer 的高度值需要与 content 其中的计算值一致。
一般都是不建议使用css计算属性，会造成一定的性能损失。
### position: absolute
通过绝对定位处理应该是常见的方案，只要使得页脚一直定位在主容器预留占位位置。
```html
    <div class="wrapper">
        <div class="content">
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
        </div>
        <footer class="footer"></footer>
    </div>
    <style>
        .wrapper {
            position: relative;
            min-height: 100%;
            padding-bottom: 50px;
            box-sizing: border-box;
            background: #0ff00f;
        }
        .footer {
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 50px;
            background: #ff0ff0;
        }
    </style>
```
wrapper 的 padding-bottom 需要与 footer 的 height 一致。

## 不固定footer高度
- 使用flexbox弹性盒布局
- grid布局
- table布局

不固定footer高度是包含固定高度的Sticky footer的。

### flexbox
弹性布局，把主轴设置为column，再通过吧.content的felx设置为1（1, 1, auto）.
```html
    <div class="wrapper">
        <div class="content">
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
        </div>
        <footer class="footer"></footer>
    </div>
    <style>
        .wrapper {
            display: flex;
            min-height: 100vh;
            flex-direction: column;
            background: #0ff00f;
        }
        .content {
            flex: 1;
        }
        .footer {
            min-height: 50px;
            background: #ff0ff0;
        }
    </style>
```
如果不知道flex布局的一些属性或者用法，可以看我另一篇博客。
### grid布局
grid是比较新的css3的属性，没有看过可以看我另外一篇博客。
```html
    <div class="wrapper">
        <div class="content">
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
        </div>
        <footer class="footer"></footer>
    </div>
    <style>
        .wrapper {
            display: grid;
            min-height: 100vh;
            grid-template-rows: 1fr auto;
            background: #0ff00f;
        }
        .footer {
            grid-row-start: 2;
            grid-row-end: 3;
            min-height: 50px;
            background: #ff0ff0;
        }
    </style>
```
网格布局（Grid layout）现在的支持还不太好，如果想查询浏览器或者在webview上的属性兼容性。可以去[caniuse](https://caniuse.com)查询，但是以后肯定是grid的天下。

### table布局
display: table的兼容性应该是最好的，直接上代码。
```html
    <div class="wrapper">
        <div class="content">
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
            <div>100</div>
        </div>
        <footer class="footer"></footer>
    </div>
    <style>
        .wrapper {
            display: table;
            width: 100%;
            min-height: 100vh;
            background: #0ff00f;
        }
        .content {
            display: table-row;
            height: 100%;
        }
        .footer {
            min-height: 50px;
            background: #ff0ff0;
        }
    </style>
```
需要注意的是，使用 table 方案存在一个比较常见的样式限制，通常 margin、padding、border 等属性会不符合预期。 笔者不建议使用这个方案。当然，问题也是可以解决的：别把其他样式写在 table 上。

## 总结
每个方案都有自己的优势和劣势，根据页面具体需求，选择最适合的方案。

## 参考
> [各种CSS实现Sticky Footer](https://mp.weixin.qq.com/s?__biz=MzU0OTE3MjE1Mw%3D%3D&mid=2247483693&idx=1&sn=ea846c8a1b404a8a0aa5a5175059e0f4&chksm=fbb2a7fbccc52eed1b62f21503d93449c8425c464d5b4ac576facadf560f95ab9ea8aca5484b&mpshare=1&scene=23&srcid=1120MlKsKxWYxEsbttZ5V0CO)
> [CSS五种方式实现Footer置底](https://www.jianshu.com/p/8fdb81e074e2)