---
title: css中的多种垂直水平居中
date: 2019-08-10 20:13:12
tags: [Css]
categories: [Css]
description: css中的多种垂直水平居中
---
## 简介
在面试的时候**css面试题**里面基本上都会问一个元素**垂直水平居中**，其实这个有多种方式实现，同时元素可以是**固定宽高、不固定宽高**的。

**固定宽高**
- position absolute + 负margin
- position absolute + margin auto
- position absolute + calc

**不固定宽高**
- position absolute + transform
- css-table
- flex
- grid

下面就直接上代码，公用的`html代码`和`css代码`就写在这里后面都会在这个基础上增加代码。
**html公用代码**：
```html
    <div class="container">
        <div class="box-center">
            box-center
        </div>
    </div> 
```
**css公用代码**：
```css
    .container {
        width: 500px;
        height: 300px;
        border: 1px solid red;
    }
    .box-center {
        width: 100px;
        height: 100px;
        background-color: red;
        color: #fff;
    }
```
**有两个元素它们是父子级的关系，要达到的效果是子元素要在父元素中垂直水平居中**。
## 固定宽高
固定宽高的意思就是要居中的这个元素它的**宽高都是固定的值**，下面一个一个用代码实现。

### position absolute + 负margin
css代码如下：
```css
    /* 此处引用上面的公共代码 */
    .container {
        position: relative;
    }
    .box-center {
        position: absolute;
        top: 50%;
        left: 50%;
        margin-top: -50px;
        margin-left: -50px;
    }
```
利用绝对定位让子元素基于父元素的左上角偏移50%，但是这样不是真正的居中因为它多移动了本身元素的**宽度的一半**和**高度的一半**，这个时候我们可以通过`负margin`来修正这个问题，所以就有了`-50px`这两个属性。

### position absolute + margin auto
css代码如下：
```css
    /* 此处引用上面的公共代码 */
    .container {
        position: relative;
    }
    .box-center {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        margin: auto;
    }
```
这种方式通过设置各个方向的距离都是0，此时再讲margin设为auto，就可以在各个方向上居中了。

### position absolute + calc
css代码如下：
```css
    /* 此处引用上面的公共代码 */
    .container {
        position: relative;
    }
    .box-center {
        position: absolute;
        top: calc(50% - 50px);
        left: calc(50% - 50px);
    }
```
通过`calc`计算属性减去元素本身高度和宽度的一半。

## 不固定宽高
固定宽高的意思就是要居中的这个元素它的**宽高都是不固定的值**，不固定宽高的方法是可以覆盖上面固定宽高的方法，下面一个一个用代码实现。

### position absolute + transform
css代码如下：
```css
    /* 此处引用上面的公共代码 */
    .container {
        position: relative;
    }
    .box-center {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
    }
```
修复绝对定位的问题，还可以使用css3新增的`transform`，`transform`的`translate`属性也可以设置百分比，其是相对于自身的宽和高，所以可以讲`translate`设置为`-50%`，就可以做到居中了。

### css-table
css代码如下：
```css
    /* 此处引用上面的公共代码 */
    .container {
        display: table-cell;
        text-align: center;
        vertical-align: middle;
    }
    .box-center {
        display: inline-block;
    }
```
通过`display: table-cell`把`div`元素变为`table`元素的实现效果，通过这个特性也可以实现垂直水平居中。

### flex
css代码如下：
```css
    /* 此处引用上面的公共代码 */
    .container {
        display: flex;
        justify-content: center;
        align-items: center;
    }
    .box-center {
        text-align: center;
    }
```
通过`flex`的两个属性实现垂直水平居中。


### grid
css代码如下：
```css
    /* 此处引用上面的公共代码 */
    .container {
        display: grid;
        justify-items: center;
        align-items: center;
    }
    .box-center {
        text-align: center;
    }
```
通过`grid`布局实现居中，如果`grid`不太了解可以看[grid布局](/blog/css/css-grid.html)

## 其他

有两种比较特殊的垂直水平居中的方式，应用场景比较少或者代价比较大，所以在这几记录一下如下：
- 行内元素居中
- table布局

### 行内元素居中
css代码如下：
```css
    /* 此处引用上面的公共代码 */
    .container {
        text-align: center;
        line-height: 300px;
        font-size: 0; // 兼容代码
    }
    .box-center {
        display: inline-block;
        vertical-align: middle;
        line-height: initial;
        font-size: 14px;
    }
```
把`container`设置为行内元素，通过`text-align`就可以做到水平居中，通过`vertical-align`也可以在垂直方向做到居中。

### table布局
改变`html`结构如下：
```html
    <table>
        <tbody>
            <tr>
                <td class="container">
                    <div class="box-center">box-center</div>
                </td>
            </tr>
        </tbody>
    </table>
```
`css`代码如下：
```css
    /* 此处引用上面的公共代码 */
    .container {
        text-align: center;
    }
    .box-center {
        display: inline-block;
    }
```
利用`table`属性实现。


## 总结
上面实现总结如下面表格所示：

| 方法 | 居中元素定宽高固定 | PC兼容性 | 移动端兼容性 |
|:---------:|:---------:|:---------:|:---------:|
| position absolute + 负margin | 固定宽高 | ie6+, chrome4+, firefox2+ | 安卓2.3+, iOS6+ |
| position absolute + margin auto | 固定宽高 | ie6+, chrome4+, firefox2+ | 安卓2.3+, iOS6+ |
| position absolute + calc | 固定宽高 | ie9+, chrome19+, firefox4+ | 安卓4.4+, iOS6+ |
| position absolute + transform | 不固定宽高 | ie9+, chrome4+, firefox3.5+ | 安卓3+, iOS6+ |
| css-table | 不固定宽高 | ie8+, chrome4+, firefox2+ | 安卓2.3+, iOS6+ |
| flex | 不固定宽高 | ie10+, chrome4+, firefox2+ | 安卓2.3+, iOS6+ |
| grid | 不固定宽高 | ie10+, chrome57+, firefox52+ | 安卓6+, iOS10.3+ |
| table布局 | 不固定宽高 | ie6+, chrome4+, firefox2+ | 安卓2.3+, iOS6+ |
| 行内元素居中 | 不固定宽高 | ie6+, chrome4+, firefox2+ | 安卓2.3+, iOS6+ |

推荐用法：
- PC端有兼容性要求，宽高固定，推荐absolute + 负margin
- PC端有兼容要求，宽高不固定，推荐css-table
- PC端无兼容性要求，推荐flex
- 移动端推荐使用flex

以后肯定`grid`会大方异彩。

## 参考
> [CSS实现水平垂直居中的1010种方式 原创](https://yanhaijing.com/css/2018/01/17/horizontal-vertical-center/)