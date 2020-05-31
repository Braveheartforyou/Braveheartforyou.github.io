---
title: Css怎么实现一个 高度随宽度变化 1:1 或者别的比例的布局
date: 2019-01-05 09:23:26
tags: [InterviewQuestion, Css]
categories: [InterviewQuestion]
description: Css怎么实现一个 高度随宽度变化
---

## 简介

---

一个网站的`logo`它的宽度是不固定的，并且要求它的高度与它的宽度相等也就是正方向，用`css`实现。
在以前的一篇文章中有记录[深入理解 css 系列 css 中 padding](/blog/css/css-padding.html)，可以通过`padding`的百分比是**规定基于父元素的宽度的百分比的内边距。**
也可以通过`vh、vw`实现

## 实现

### 通过 padding 实现

```html
  <style>
    .header {
      wdith: 100px;
    }
    .header-logo {
      padding: 50%;
      background-color: #000;
    }
  </style>
  <header class="header">
    <div class="hedaer-logo">
    <div>
  </header>
```

这样就可以创建一个宽高`1:1`的正方形，只需要修改`header`元素的宽度就可以动态修改`logo`元素的大小。

### 通过 vh、vw 实现

```html
  <style>
    .header-logo {
      width: 10vh; // 或者vw
      height: 10vh; // 或者vw
      background-color: #000;
    }
  </style>
  <header class="header">
    <div class="hedaer-logo">
    <div>
  </header>
```

把`logo`元素的宽高通过设置为`vh、vw`来实现他们的的宽度和高度值都来自与屏幕的**宽或高**。

### 通过 rem 实现

```html
  <html lang="en" font-size="12px">
  <style>
    .header-logo {
      width: 10rem;
      height: 10rem;
      background-color: #000;
    }
  </style>
  <header class="header">
    <div class="hedaer-logo">
    <div>
  </header>
```

通过设置`html`元素上的`font-size: 12px`，`logo`元素的宽高通过 rem 控制为`10rem`也可以实现宽高`1:1`的元素。

### 通过 js 实现

```html
<style>
  /* header {
      width: 100px;
    } */
  .header-logo {
    width: 20%;
    background-color: #000;
  }
</style>
<header class="header">
  <div class="header-logo"></div>
</header>
<script>
  var dom = document.querySelector('.header-logo'),
    nDomWidth = dom.clientWidth;
  dom.style.height = nDomWidth + 'px';

  window.onresize = function () {
    let nDomWidth = dom.clientWidth;
    dom.style.height = nDomWidth + 'px';
  };
</script>
```

只设置`logo`元素的宽度，默认通过 js 获取`logo`元素的**宽度**赋值给**高度**，后面监听窗口变化适时更新高度。

## 总结

上面一共通过**四种方法**来实现的宽高 1:1，并且高度跟随宽度变化。其实上面四种解决方法比较相近，如果有更好的方法请留言探讨，谢谢。

## 参考

[深入理解 css 系列 css 中 padding](/blog/css/css-padding.html)
