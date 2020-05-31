---
title: 深入理解css系列 css中float
date: 2018-06-27 21:32:12
tags: [Css]
categories: [Css]
description: 在平常工作中经常用到float布局，并且用多种方法清除浮动，但是对float整体都是没有太深入的理解，这篇文章中就是整体记录记录一下float。
---

## 简介

`float CSS`属性指定一个元素应沿其容器的左侧或右侧放置，允许文本和内联元素**环绕**它。该元素从网页的正常流动(文档流)中**移除**，尽管仍然保持部分的**流动性**（与绝对定位**相反**）。

**浮动元素是 float 的计算值非 none 的元素。**
基本上可以认为：**“让block元素无视float元素，让inline元素像流水一样围绕着float元素实现浮动布局。”**

## float特性

`float`有四种特性：

- 包裹性
- 高度破坏性
- 没有任何margin合并

### 包裹性

**包裹性： 包裹性指的是元素尺寸刚好容纳内容，并且不会超越父级元素的宽度**。

具有包裹性的其他属性：

- display:inline-block/table-cell/...
- position:absolute/fixed/sticky
- overflow:hidden/scroll

通过一个实例来看一下什么是包裹性：

**html**

```html
  <div class="container">
    内容
  </div>
  <div class="container fl">
    内容
  </div>
```

**css**

```css
  * {
    margin: 0;
    padding: 0;
  }
  body {
    width: 300px;
  }
  .container{
    border: 1px solid green;
    padding: 30px;
    background-color: green;
    background-clip: content-box;/*将背景裁剪到内容框，方便看浮动元素效果*/
  }
  .fl{
      float: left;
  }
```

效果图如下：

![css margin](../../images/css/css-float-1-1.png)

可以看到下面的元素的宽度就是它内部文本信息的宽度，而上面的元素是占了整个一行的元素。

### 包裹性的原理

浮动之所以会产生包裹性这样的效果是**因为float属性会改变元素display属性最终的计算值**。

设置float前的display属性——》设置浮动后的display属性计算值:

- **inline——》block**
- **inline-block——》block**
- **inline-table——》table**
- **table-row——》block**
- **table-row-group——》block**
- **table-column——》block**
- **table-column-group——》block**
- **table-caption——》block**
- **table-header-group——》block**
- **table-footer-group——》blcok**
- **flex——》flex**
- **inline-flex——》inline-flex【inline-flex在chrome下测试，float后display:flex】**
- **other——》unchanged**

可以自己自行测试在不同浏览其中的表现，**chrome**测试方法如下：

**修改css如下**

```css
  .fl {
    display: inline-block;
    float: left;
  }
```

然后通过**chrome**的控制台首先查看**Styles**面板，可以看到`display: inline-block;float: left;`属性生效，如下：
![css margin](../../images/css/css-float-1-2.png)

然后再查看**Computed**面板查看真正生效到元素上的所有属性，如下：
![css margin](../../images/css/css-float-1-3.png)

可以看到`display: flex;`属性，得以验证上面的结论。

### 高度破坏性

**破坏性是指元素浮动后可能导致父元素高度塌陷**。

修改**html**代码如下：

```html
  <div class="container">
    内容
    <div class="nofl">
      测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1
    </div>
    <div class="fl">
      测试内容测试内容测试内容测试内容测试内容测试内容测试内容测试内容测试内容测试内容
    </div>
  </div>
```

效果图如下：
![css margin](../../images/css/css-float-1-4.png)

在`div.container`元素没有设置高度时，`div.nofl`不设置`float: left;`时元素的高度会撑开父元素的高度。而`div.fl`元素设置了`float: left;`元素的高度不会包含在父元素的高度内。

**其他破坏性的属性**：

- display:none
- position:absolute/fixed/sticky

**浮动破坏性原理：**
因为浮动元素被从文档**正常流中移除**了，**父元素**当然还处在**正常流**中，所以父元素**不能**被浮动元素**撑大**。

### 没有任何margin合并

修改**html**代码如下：

```html
  <div class="container">
    内容
    <div class="fl one">
      测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1
    </div>
    <div class="fl two">
      测试内容测试内容测试内容测试内容测试内容测试内容测试内容测试内容测试内容测试内容
    </div>
  </div>
```

修改**css**如下：

```css
  .container {
    border: 1px solid green;
    padding: 10px;
    background-color: green;
    background-clip: content-box; /*将背景裁剪到内容框，方便看浮动元素效果*/
  }
  .container::after {
    content: "";
    display: block;
    clear: both;
  }
  .fl {
    float: left;
    margin: 10px;
    border: 1px solid #000;
  }
```

![css margin](../../images/css/css-float-1-5.png)

根据效果图可以看到`div.one`和`div.two`都是有`margin: 10px;`的属性，但是它们之间的`margin`并没有重合。如果没有`float: left`属性的话，`div.one`的下边距和`div.two`的上边距会发生合并，也就是说它们合并后的外间距就会是`10px`。

## 清除浮动

在上面的实例中就用到了`clear`清除浮动属性，它可以解决`float`带来一些副作用比如说`高度破坏性`。

```css
clear: none | left | right | both
```

- **none：默认值，允许两边都有浮动对象；**
- **left：不允许左侧有浮动对象；**
- **right：不允许右侧有浮动对象；**
- **both：两侧不允许有浮动对象。**

具体原理：在元素上外边距之上增加清除空间，比如清除左浮动会让元素的上外边距刚好在左边浮动元素的下外边距之下。

清除浮动课两大类方法：

- 在兄弟元素最后一个设置 clear:both
- 父元素生成BFC(IE8+)或者hashlayout(IE6/IE7)

### 兄弟元素清除

修改**html**代码如下：

```html
  <div class="container">
    <!-- ....省略代码 -->
    <div class="clearfix"></div>
  </div>
```

**css**

```css
  /* 注释掉 */
  /* .container::after {
    content: "";
    display: block;
    clear: both;
    overflow: hidden;
    zoom: 1;
  } */
  /* ....省略代码  */
  .clearfix {
    clear: both;
  }
```

可以看到在没有添加`.clearfix { clear: both; }`时显示如图一，在添加`.clearfix`后显示为图二。

**图一**
![css margin](../../images/css/css-float-1-6.png)

**图二**
![css margin](../../images/css/css-float-1-5.png)

这种方式也是有很多不好的地方，比如增加了一个**无用标签**、**结构更复杂**、**比较难复用**。

### 父元素添加after伪元素

为了解决上面兄弟元素的问题，引出父元素的after伪元素。

**html**

```html
  <div class="container clearfix">
    内容
    <div class="fl">
      测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1
    </div>
    <div class="fl">
      测试内容测试内容测试内容测试内容测试内容测试内容测试内容测试内容测试内容测试内容
    </div>
  </div>
```

**css**

```css
  .clearfix::after {
    content: ".";
    display: block;
    clear: both;
    zoom: 1;
  }
  .clearfix {
    /* IE < 8 */
    /**
    * For IE 6/7 only
    * Include this rule to trigger hasLayout and contain floats.
    */
    *zoom: 1;
  }
```

**这样可以实现与上面想同的效果并且不会产生兄弟元素产生的问题**。

### 父元素生成BFC（IE8+） 或haslayout(IE6/IE7)


**html**

```html
  <div class="container clearfix">
    内容
    <div class="fl">
      测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1
    </div>
    <div class="fl">
      测试内容测试内容测试内容测试内容测试内容测试内容测试内容测试内容测试内容测试内容
    </div>
  </div>
```

**css**

```css
  .clearfix::after {
    content: ".";
    display: block;
    clear: both;
    height: 0;
    overflow: hidden;
    zoom: 1;
  }
  /*IE6和IE7*/
  .clearfix {
    *zoom:1;
  }
```

### 父元素设置为float

## 常用布局

### 一列自适应一列固定布局

**html**

```html
  <div class="container clearfix">
    <div class="left"></div>
    <div class="right">测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1测试内容1</div>
  </div>
```

**css**

```css
  .container {
    background: orange;
    border: 1px solid #000;
  }
  .clearfix::after {
    content: ".";
    display: block;
    clear: both;
    height: 0;
    overflow: hidden;
    zoom: 1;
  }
  .left {
    width: 100px;
    height: 100px;
    float: left;
    border: 1px solid #fff;
    background: purple;
  }
  .right {
    margin-left: 100px;
  }
```

显示效果如下：

![css margin](../../images/css/css-float-1-7.png)

还有**圣杯和双飞翼布局**，这里就不一一列举，可在本在中查找。

## float布局和inline-block布局对比

`float`和`inline-block`都能让元素排成一排，那么应该如何抉择？下面对比一下。

- 文档流：**浮动元素脱离正常流，让文字环绕。inline-block仍然在正常流中**。
- 水平位置：**不能通过给父元素设置text-align:center让浮动元素无法水平居中【因为脱离文档流】，而inline-block可以**。
- 垂直对齐：**浮动元素紧贴顶部，inline-block默认基线对齐，可通过vertical-align调整**。
- 空白：**浮动忽略空白元素彼此紧靠，inline-block保留空白**。

其实`float`和`inline-block`看个人喜好和具体的场景。

## 总结

`float`即使到现在还没有淘汰，虽然现在有很多更方便的布局如`flex`、`colmun`、`grid`等等，现在还有很多场景在应用，所以还是要仔细学习。

## 参考

> [深入理解css浮动](https://www.cnblogs.com/starof/p/4608962.html)
> [【前端Talkking】CSS系列——CSS深入理解之float浮动](https://segmentfault.com/a/1190000014554601#articleHeader4)
