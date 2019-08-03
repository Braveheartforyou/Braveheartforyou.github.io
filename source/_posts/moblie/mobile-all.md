---
title: 移动端注意事项 (未完待续)
date: 2017-07-17 09:33:57
tags: [Css, Html]
categories: [mobile]
description: 一些移动端常见问题 和解决方案(未完成 2017-7-17)
---
## 一些问题阐述 <font color="#ff502c">(未完待续)</font>
 - 设备更新换代快 + 浏览器厂商不统一———— 兼容问题多
 - 网络更复杂——弱网络，低端机性能差————页面打开慢
 - 未知问题——坑多  
<!--more-->
## meta基础知识
- H5页面窗口自动调整到设备宽度，并禁止用户缩放页面
`<meta name="viewport" content="width=device-width,initial-scale=1.0,minimum-scale=1.0,maximum-scale=1.0,user-scalable=no" />`
- 忽略将页面中的数字识别为电话号码
`<meta name="format-detection" content="telephone=no" />`
- 忽略Android平台中对邮箱地址的识别
`<meta name="format-detection" content="email=no" />`
- 当网站添加到主屏幕快速启动方式，可隐藏地址栏，仅针对ios的safari
`<meta name="apple-mobile-web-app-capable" content="yes" />`
`<!-- ios7.0版本以后，safari上已看不到效果 -->`
- viewport模板 ———— 通用
 
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no" name="viewport">
        <meta content="yes" name="apple-mobile-web-app-capable">
        <meta content="black" name="apple-mobile-web-app-status-bar-style">
        <meta content="telephone=no" name="format-detection">
        <meta content="email=no" name="format-detection">
        <title>标题</title>
        <link rel="stylesheet" href="index.css">
    </head>
    <body>
    这里开始内容
    </body>
</html>
```
## 常见问题

### 移动端如何定义字体font-family
中文字体使用系统默认即可，英文用Helvetica
`/* 移动端定义字体的代码 */`
`body{font-family:Helvetica;}`

### 移动端字体单位font-size选择px还是rem
对于只需要适配少部分手机设备，且分辨率对页面影响不大的，使用px即可
对于需要适配各种移动设备，使用rem，例如只需要适配iPhone和iPad等分辨率差别比较挺大的设备
/* 长宽占位 rem算法, 根据root的rem来计算各元素相对rem, 默认html 750/10 = 75px */
可以参考 > <http://www.cnblogs.com/well-nice/p/5509589.html>
```javascript
updataHtml();
function updataHtml() {
    var w = document.documentElement.offsetWidth;
    document.documentElement.style.fontSize = w / 10 + "px";
}
window.onresize = function () {
    updataHtml();
}
```

### 移动端click屏幕产生200-300 ms的延迟响应
在IOS safari下，大概为300毫秒，IOS 自带的双击页面放大，点击完成第一次时，会等待第二次点击，页面需要过一段时间才响应，给用户慢体验感觉，对于web开发者来说是，页面js捕获click事件的回调函数处理，需要300ms后才生效，也就间接导致影响其他业务逻辑的处理
- fastclick可以解决在手机上点击事件的300ms延迟
- zepto的touch模块，tap事件也是为了解决在click的延迟问题  

### 触摸事件的响应顺序
`ontouchstart`
`ontouchmove`
`ontouchend`
`onclick`
解决300ms延迟的问题，也可以通过绑定ontouchstart事件，加快对事件的响应

### 什么是Retina 显示屏，带来了什么问题
retina：一种具备超高像素密度的液晶屏，同样大小的屏幕上显示的像素点由1个变为多个，如在同样带下的屏幕上，苹果设备的retina显示屏中，像素点1个变为4个
在高清显示屏中的位图被放大，图片会变得模糊，<font color="#ff502c">因此移动端的视觉稿通常会设计为传统PC的2倍</font>
设计稿切出来的图片长宽保证为偶数，并使用backgroud-size把图片缩小为原来的1/2
```css
//例如图片宽高为：200px*200px，那么写法如下
.css{width:100px;height:100px;background-size:100px 100px;}
```
其它元素的取值为原来的1/2，例如视觉稿40px的字体，使用样式的写法为20px
```css
.css{font-size:20px}
```

### ios系统中元素被触摸时产生的半透明灰色遮罩怎么去掉
ios用户点击一个链接，会出现一个半透明灰色遮罩, 如果想要禁用，可设置-webkit-tap-highlight-color的alpha值为0，也就是属性值的最后一位设置为0就可以去除半透明灰色遮罩
```css
a,button,input,textarea{-webkit-tap-highlight-color: rgba(0,0,0,0;)}
```

### 部分android系统中元素被点击时产生的边框怎么去掉
android用户点击一个链接，会出现一个边框或者半透明灰色遮罩, 不同生产商定义出来额效果不一样，可设置-webkit-tap-highlight-color的alpha值为0去除部分机器自带的效果
```css
a,button,input,textarea{
    -webkit-tap-highlight-color: rgba(0,0,0,0;)
    -webkit-user-modify:read-write-plaintext-only; 
}
```
兼容性不是很好

### winphone系统a、input标签被点击时产生的半透明灰色背景怎么去掉、
```html
<meta name="msapplication-tap-highlight" content="no">
```
### webkit表单元素的默认外观怎么重置
```css
.css{-webkit-apperarance:none}
```
<font color="#ff502c">把各个浏览器表单的默认样式重置</font>
### 伪元素改变number类型input框的默认样式
```css
input[type=number]::-webkit-textfield-decoration-container {
    background-color: transparent;    
}
input[type=number]::-webkit-inner-spin-button {
    -webkit-appearance: none;
}
input[type=number]::-webkit-outer-spin-button {
    -webkit-appearance: none;
}
```