---
title: retina屏高清屏的具体原理
date: 2017-07-27 09:35:19
tags: [Css, Html]
categories: [mobile]
description: retina屏(高清屏)的原理，对前端的页面制作的影响和解决方案
---
## 概述
随着2012年苹果发布第一款Retina Macbook Pro（以下简称RMBP），Retina屏幕开始进入笔记本行业。两年过去了，RMBP的市场占有率越来越高，且获得了一大批设计师朋友的青睐，网站对于Retina屏幕的适配也变成了迫在眉睫的问题。
- 1、一种具有备超高像素密度的液晶屏
- 2、同样大小的屏幕上显示的像素点由1个变为多个
在不同的屏幕上，CSS像素所呈现的物理尺寸是一致的，而不同的是CSS像素所对应的物理像素具数是不一致的。在普通屏幕下1个CSS像素对应1个物理像素，而在Retina屏幕下，1个CSS像素对应的却是4个物理像素。
![css3 3d transfrom](../../images/ios_bug/retina-web-3.jpg)

### 物理像素（physical pixel）
物理像素又被称为设备像素，他是显示设备中一个最微小的物理部件。每个像素可以根据操作系统设置自己的颜色和亮度。正是这些设备像素的微小距离欺骗了我们肉眼看到的图像效果。
![css3 3d transfrom](../../images/ios_bug/retina-web-1.jpg)

### 设备独立像素(density-independent pixel)
设备独立像素也称为密度无关像素，可以认为是计算机坐标系统中的一个点，这个点代表一个可以由程序使用的虚拟像素(比如说CSS像素)，然后由相关系统转换为物理像素。
### 屏幕密度
屏幕密度是指一个设备表面上存在的像素数量，它通常以每英寸有多少像素来计算(PPI)。
### 设备像素比(device pixel ratio)
设备像素比简称为dpr，其定义了物理像素和设备独立像素的对应关系。它的值可以按下面的公式计算得到：
<font color="#ff502c">设备像素比 ＝ 物理像素 / 设备独立像素</font>
在JavaScript中，可以通过window.devicePixelRatio获取到当前设备的dpr。而在CSS中，可以通过-webkit-device-pixel-ratio，-webkit-min-device-pixel-ratio和 -webkit-max-device-pixel-ratio进行媒体查询，对不同dpr的设备，做一些样式适配(这里只针对webkit内核的浏览器和webview)。
dip或dp,（device independent pixels，设备独立像素）与屏幕密度有关。dip可以用来辅助区分视网膜设备还是非视网膜设备。
- devicePixelRatio在大多数浏览器是值得信赖的。
- 在iOS设备，screen.width乘以devicePixelRatio得到的是物理像素值。
- 在Android以及Windows Phone设备，screen.width除以devicePixelRatio得到的是设备独立像素(dips)值。
## 解决方法
通过判断 devicePixelRatio 的值来加载不同尺寸的图片
- <font color="#ff502c">针对普通显示屏(devicePixelRatio = 1.0、1.3)，加载一张1倍的图片</font>
- <font color="#ff502c">针对高清显示屏(devicePixelRatio >= 1.5、2.0、3.0)，加载一张2倍大的图片</font>
dpr为3的手机比较小，建议用两倍的图片
### Media Queries判断当前的dpr
通过媒体查询结合devicePixelRatio可以区分普通的显示屏和高清显示屏，<font color="#ff502c">兼容行比较好</font>
```css
.css{/* 普通显示屏(设备像素比例小于等于1.3)使用1倍的图 */ 
    background-image: url(img_1x.png);
}
@media only screen and (-webkit-min-device-pixel-ratio:1.5){
.css{/* 高清显示屏(设备像素比例大于等于1.5)使用2倍图  */
    background-image: url(img_2x.png);
  }
}
@media only screen and (-webkit-min-device-pixel-ratio:3){
.css{/* 高清显示屏(设备像素比例大于等于1.5)使用2倍图  */
    background-image: url(img_2x.png);
  }
}
```
### image-set 设计retina背景图
image-set，它是Webkit的私有属性，也是Css4的一个属性。<font color="#ff502c">目前支持苹果的 retina 显示屏和部分android 显示屏</font>
```css
.css{
    background: url(../img/bank_ico.png) no-repeat;/* 不支持image-set的显示屏 */ 
    background: -webkit-image-set(
                url(../img/bank_ico.png) 1x,/* 支持image-set的浏览器的[普通屏幕]下 */
                url(../img/bank_ico_retina.png) 2x);/* 支持image-set的浏览器的[Retina屏幕] */
                
}
```
本文
参考: http://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html © w3cplus.com
参考：http://www.zhangxinxu.com/wordpress/2012/08/window-devicepixelratio/
参考：http://www.cnblogs.com/PeunZhang/p/3441110.html