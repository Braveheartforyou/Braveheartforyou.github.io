---
title: 深入理解 css中margin
date: 2019-01-03 14:23:34
tags: [Css]
categories: [Css]
description: margin属性有很多的奇特的操作，比如说负margin、margin重叠、margin占据的区域等等，这边文章希望能记录清楚margin的一些特性和特殊属性。
---

## 简介

在平常开发过程中经常用到**margin**属性，但是也会遇到很多的问题，比如说**margin重叠（BFC）**、**负margin**等等问题，下面就一个一个来记录问什么会这样。

**标准盒模型**

![css margin](../../images/css/css-margin-1-1.png)

要了解**margin**就要先了解**css**中的**盒子模型（Box Model）**，**标准盒模型**可以分为：外边距(margin)、边框（border）、内边距（padding）、内容区域（content）。在**标准盒模型**中平常所说道的高度只是**content**的高度，不包含**border**的高度，而在**ie盒模型**中是把**border**算在内的。

**margin属性**

margin有四个属性**margin-top**、**margin-left**、**margin-bottom**、**margin-right**，它们的值可以为**百分比**、**数值（左右可为负数）**、**em、rem、vh、vw**、**auto**等等。

## margin与容器尺寸

元素尺寸：①可视尺寸 clientWidth（标准）；②占据尺寸

![css margin](../../images/css/css-margin-1-2.png)

margin与可视尺寸：**①适用于没有设定width/height的普通block元素；②只适用于水平方向尺寸**
margin与占据尺寸：**①block/inline-block水平元素均适用；②与有没有设定width/height无关；**

③适用于水平方向和垂直方向。可用于页面的上下留白（padding兼容性不好）。

### margin与可视尺寸

特性：

- 适用于没有设定`width/height`的普通`block`元素
- **只适用于水平方向尺寸**

应用：一侧定宽自适应

```html
  <img width="150px" style="float:left">
  <p style="margin-left:170px;">图片左浮动....</p>
```

![css margin](../../images/css/css-margin-1-3.png)

### margin与占据尺寸

特性：

- `block、inline-block`水平元素均适应
- 与有没有设定`width、height`值无关
- 适应于水平方向和垂直方向

## margin与百分比单位

- 普通元素的百分比：**相对于容器宽度计算**。
- 绝对定位元素的百分比：**相对于第一个定位的祖先容器的宽度计算的**。

## margin中的重叠

### margin重叠通常特性

1. `block`水平元素（不包括`float`和`absolute`元素）
2. 不考虑`writing-mode`，只发生垂直方向（`margin-top/margin-bottom`）

### margin重叠的3种情景

1. **相邻的兄弟元素**
2. **父级和第一个/最后一个子元素**
3. **空的block元素**

### 父子margin重叠其他条件

**margin-top重叠**

| margin-top重叠 | 解决 |
|:------:|:----------:|
| 父元素**非块状格式化上下文**元素 | 设置父元素`overflow`：`hidden` |
| 父元素没有`border-top`设值 | 设置父元素`border` |
| 父元素没有`padding-top`值 | 设置父元素`padding-top` |
| 父元素和第一个子元素之间没有`inline`元素分隔 | **插入一个内联元素如空格&bsp;** |

**margin-bottom重叠**

1. 父元素**非块状格式化上下文**元素
2. 父元素没有`border-bottom`设置
3. 父元素没有`padding-bottom`值
4. 父元素和最后一个子元素之间没有`inline`元素分隔
5. 父元素没有`height`相关声明

### 空block元素margin重叠

条件限制：

1. 元素没有`border`设置
2. 元素没有`padding`值
3. 里面没有`inline`元素
4. 没有`height、或者min-height`

### 重叠的计算规则

- **正正取大值**
- **正负值相加**
- **负负最负值**

### 理解CSS中的margin:auto

`margin:auto`的作用机制：**自动分配剩余空间**

### 垂直居中方法(margin实现)

**writing-mode**

更改流为垂直方向，但是水平居中失效

```css
  .father { height:200px; width:100%; writing-mode:vertical-lr; }
  .son { height:100px; width:500px; margin:auto; }
```

![css margin](../../images/css/css-margin-1-4.png)

**绝对定位元素的margin:auto居中**

```css
  .father { height:200px; position:relative; }
  .son { position:absolute; top:0; right:0; left:0; bottom:0; height:100px; width:500px; margin:auto; }
```

![css margin](../../images/css/css-margin-1-5.png)

## margin负值

为了方便理解负值`margin`，我们引入参考线的定义，参考线就是就是`margin`移动的基准点，而`margin`的值就是移动的数值。
`margin`的参考线有两类，一类是`top、left`，它们以外元素作为参考线;
另一类是`right、bottom`，它们以自身作为参考线。
简单点说就是：

- `top`负值就是以包含块`(Containing block)` 内容区域的上边或者上方相连元素 `margin` 的下边线为参考线;
- `left`负值就是以包含块`(Containing block)` 内容区域的左边或者左方相连元素 `margin` 的右边为参考线;
- `right`负值就是以元素本身`border`的右边为参考线；
- `bottom`负值就是以元素本身`border`的下边为参考线；

另外关于包含块的定义具体[请参考KB008包含块(Containing block)](http://w3help.org/zh-cn/kb/008/)。

![css margin](../../images/css/css-margin-1-6.png)

**公用代码**

```html
  <style>
    .box {
      width:200px;
      height: 200px;
      margin: auto;
      border: 1px black solid;
    }
    .box div {
      width:100px;
      height: 100px;
    }
    .one {
      background:orange;
    }
    .two {
      background:green;
    }
  </style>
  <div class="box">
     <div class="one">one</div>
    <div class="two">two</div>
  </div>
```

### margin-top和margin-left负值

#### margin-top

修改css代码如下：

```css
  .two {
    margin-top: -50px;
  }
```

效果图如下：

![css margin](../../images/css/css-margin-1-7.png)

当设置`.two`的`div`的`margin-top: -50px`的时候，它的参考线是`div.one`的下边，整个`div.two`向上移动`-50px`使得`div.two`覆盖`div.one`。

#### margin-left

修改css代码如下：

```css
  .box div {
    float: left;
  }
  .two {
    margin-left: -50px;
  }
```

效果图如下：

![css margin](../../images/css/css-margin-1-8.png)

当设置`.two`的`div`的`margin-left: -50px`的时候，它的参考线是`div.one`的右边线，整个`div.two`向左移动`-50px`使的`div.two`覆盖`div.one`。

### margin-right和margin-bottom负值

#### margin-right

修改css代码如下：

```css
  .one {
    margin-right: -50px;
  }
```

效果图如下：

![css margin](../../images/css/css-margin-1-8.png)

当设置`.one`的`div`的`margin-right: -50px`的时候，它的参考线是`div.one`的右边线，整个`div.one`向左收缩`-50px`使的`div.two`覆盖`div.one`。

#### margin-bottom

修改css代码如下：

```css
  .box div {
    float: static;
  }
  .one {
    margin-bottom: -50px;
  }
```

效果图如下：

![css margin](../../images/css/css-margin-1-7.png)

当设置`.one`的`div`的`margin-bottom: -50px`的时候，它的参考线是`div.one`的下边线，整个`div.one`向上收缩`-50px`使的`div.two`覆盖`div.one`。

### 实际应用

**margin负值下的两端对齐：**

```html
  <style>
    * {
      margin: 0;
      padding: 0;
    }
    .box {
      width: 1200px;
      margin: auto;
      background: orange;
    }
    .ul {
      list-style: none;
      overflow: hidden;
      margin-right: -20px;
    }
    .li {
      width: 386.66px;
      height: 300px;
      margin-right: 20px;
      background: green;
      float: left;
    }
  </style>
  <div class="box">
    <ul class="ul">
      <li class="li">列表1</li>
      <li class="li">列表2</li>
      <li class="li">列表3</li>
    </ul>
  </div>
```

![css margin](../../images/css/css-margin-1-9.png)

**margin负值下的等高布局：**

![css margin](../../images/css/css-margin-1-10.png)

**margin负值下的两栏自适应布局：**

![css margin](../../images/css/css-margin-1-11.png)

## margin无效情形解析

1. `inline`水平元素的垂直`margin`无效前提：

  - 非替换元素，例如不是`<img>`元素；
  - 正常书写模式。

2. `margin`重叠
3. `display:table-cell`与`margin：display:table-cell/display:table-row`等声明的`margin`无效。
4. `position:absolute`与`margin`：绝对定位元素未设置定位方向的`margin`值”无效“。例如，`img{top:10%}`的`margin-top`有效其他均无效。
5. 内联特性导致的`margin`无效：

## 了解margin-start/margin-end属

### -webkit-margin-start、-webkit-margin-end

- 正常的流向，`margin-start`等同于`margin-left`，两者重叠不累加
- 如果水平流是从右往左，`margin-start`等同于`margin-right`
- 在垂直流下（`writing-mode:vertical-*;`）,`margin-start`等同于`margin-top`

### margin-collaps

```css
  -webkit-margin-collaps: <collaps> | <discard> | <separate>
```

- `collaps`,默认，重叠
- `discard`，取消重叠，使`margin`无效
- `separate`，取消重叠，不合并

## 总结

在本篇文章中介绍主要的`margin百分比`、`margin重叠条件`、`margin在盒模型中的区域`，但是本文总结的`margin负值`并不是全部情况，比如说`div.two`设置为`margin-right: -50px`为什么不会收缩自己的宽度等等。希望大家多多做补充。

## 参考

> [CSS深入理解之relative](https://www.imooc.com/learn/565)
> [CSS深入理解之margin](https://www.kancloud.cn/dunizb/web-dev-marrow/647633#11_margin_2)
> [CSS深入理解学习笔记之margin](https://cloud.tencent.com/developer/article/1053594)
> [浅谈margin负值](https://zhuanlan.zhihu.com/p/25892372)
