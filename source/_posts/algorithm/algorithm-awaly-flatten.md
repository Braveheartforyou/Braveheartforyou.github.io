---
title: JavaScript实现flatten多种方法
date: 2019-06-25 21:52:32
tags: [JavaScript, Algorithm]
categories: [JavaScript]
description: JavaScript实现flatten多种方法，最后再来一道经常被问的面试题
---
## 简介
我们在网上看到很多的关于数组的面试题，比如说给如下一个数组，把他拍平、去重、升序：
```javascript
let arr = [8, [5, 9, 4], 1, 3, [7, 5, 10, [3, 4, 6, 2]], 4, 3, 2, 4];
```
其实这个题有很多种解法，比如用Array.prototype.flat,或者自己实现一个flatten函数，我们这里主要关注的时flat方法的实现。
## 第一种解法
用最新的语法
```javascript
let arr = [8, [5, 9, 4], 1, 3, [7, 5, 10, [3, 4, 6, 2]], 4, 3, 2, 4];
let newArr = Array.from(new Set(arr.flat(Infinity))).sort((a, b) => {return a-b});
console.log(newArr);
// [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```
1、就是先拍平一个多维数组 arr.flat(ES6语法)
2、再通过Set集合做去重
3、再通过Array.from把Set集合转为数组
4、再通过sort排序

如果不了解flat的函数的话，可以参考[mdn Array.prototype.flat()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/flat)，或者看[阮一峰老师的flat介绍]（http://es6.ruanyifeng.com/#docs/array#%E6%95%B0%E7%BB%84%E5%AE%9E%E4%BE%8B%E7%9A%84-flat%EF%BC%8CflatMap）。
Set集合的讲解可以参考[mdn Set](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set)，或者看[阮一峰老师的Set介绍]（http://es6.ruanyifeng.com/?search=Set&x=0&y=0#docs/set-map）。
我们这里主要讨论flat的实现。

## 第二种解法
```javascript
let arr = [8, [5, 9, 4], 1, 3, [7, 5, 10, [3, 4, 6, 2]], 4, 3, 2, 4];
let newArr = Array.from(new Set(arr.toString().split(','))).map(item => { return parseInt(item);}).sort((a, b) => {return a -b})
console.log(newArr);
// [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```
1、我们调用Array上的toString方法把他转换为一个字符串 
```javascript
arr.toString()
// "8,5,9,4,1,3,7,5,10,3,4,6,2,4,3,2,4"
```
这列我们就不讨论为什么返回的结果里面不包含<font color="#ff502c">‘[]’</font>这两个字符串，后面我在写一篇博客来说数组，的valueOf、toString方法。
2、再把字符串通过Array.prototype.split方法转换为数组
3、再通过Set集合做去重
4、再通过Array.from把Set集合转为数组
5、再通过sort排序

## 第三种解法
自己通过封装一个flatten，在不基于Array.prototype.flat方法上实现一个拍平函数
### ES6实现
```javascript
let arr = [8, [5, 9, 4], 1, 3, [7, 5, 10, [3, 4, 6, 2]], 4, 3, 2, 4];
const flatten = (arr) => Array.isArray(arr) 
? arr.reduce( (a, b) => [...a, ...flatten(b)] , [])
: [arr]
let newArr = Array.from(new Set(flatten(arr))).sort((a, b) => {return a -b})

// [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```
### ES5

```javascript
let arr = [8, [5, 9, 4], 1, 3, [7, 5, 10, [3, 4, 6, 2]], 4, 3, 2, 4];
function flatten(arr) {
    return Array.isArray(arr) ? 
    arr.reduce(function (prev, current) { return [...prev, ...flatten(current)]}, []) : [arr]
}
let newArr = Array.from(new Set(flatten(arr))).sort((a, b) => {return a -b})
// [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```
### 实现思路
1、检测是否为数组
2、如果是数组，调用reduce函数实现合并函数
3、如果有嵌套数组，就递归调用该方法

## 总结
实现拍平数组大致有三种方法
1、Array.prototype.flat方法
2、Array.prototype.toString方法转为字符串，再split
3、自己实现一个flatten函数