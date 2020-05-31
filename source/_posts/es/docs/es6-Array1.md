---
title: ES6 Array系列(二) 一些常用array的扩展方法（二）
date: 2017-08-01 17:37:05
tags: [JavaScript, JavaScript]
categories: [JavaScript]
description: JavaScript 数组对象是用于构造数组的全局对象; 它是高阶，类似列表的对象。
---

## 迭代方法

---

every(): 对数组中的每一项运行给函数，如果该函数对每一项都返回 true,则返回 true.
filter(): 对数组中的每一项运行给函数，如果该函数会返回 true 的项目成的数组。
forEach(): 对数组中的每一项运行给函数，这个项目没有返回值.
map(): 对数组中的每一项运行给函数，返回每次函数调用的结果组成的数组。
some(): 对数组中的每一项运行给函数，如果该函数对任何项都返回 true,则返回 true.

### every()/some()

```javascript
var numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
var everyResult = numbers.every((item, index, array) => {
  return item > 2;
});
console.log(everyResult); // false

var everyResult1 = numbers.some((item, index, array) => {
  return item > 2;
});
console.log(everyResult1); // true
```

### filter() 过滤

```javascript
var numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
var everyResult = numbers.filter((item, index, array) => {
  return item > 2;
});
console.log(everyResult); // [3, 4, 5, 6, 7, 8, 9, 10, 11]
```

### forEach()

方法对数组的每个元素执行一次提供的函数。

```javascript
    array.forEach(callback(currentValue, index, array){
        //do something
    }, this)

    array.forEach(callback[, thisArg])
   // 参数
    // callback
      //   为数组中每个元素执行的函数，该函数接收三个参数：
    // currentValue(当前值)
       //  数组中正在处理的当前元素。
    // index(索引)
       //  数组中正在处理的当前元素的索引。
    // array
    // forEach()方法正在操作的数组。
    // thisArg可选
       //  可选参数。当执行回调 函数时用作this的值(参考对象)。
    // 返回值
       // undefined.
```

callback 函数会被依次传入三个参数：
<font color="#ff502c">数组当前项的值</font>
<font color="#ff502c">数组当前项的索引</font>
<font color="#ff502c">数组对象本身</font>
<font color="yellow">注意： 没有办法中止或者跳出 forEach 循环，除了抛出一个异常。如果你需要这样，使用 forEach()方法是错误的，你可以用一个简单的循环作为替代。如果您正在测试一个数组里的元素是否符合某条件，且需要返回一个布尔值，那么可使用 Array.every 或 Array.some。如果可用，新方法 find() 或者 findIndex() 也可被用于真值测试的提早终止。</font>

### map()

map() 方法创建一个新数组，其结果是该数组中的每个元素都调用一个提供的函数后返回的结果。
参数与 forEach 相同

```javascript
let numbers = [1, 5, 10, 15];
let doubles = numbers.map((x) => {
  return x * 2;
});
// 返回值
// 一个新数组，每个元素都是回调函数的结果。
```

返回值<font color="#ff502c">一个新数组，每个元素都是回调函数的结果。</font>

## 栈方法/队列方法

栈是一种 LIFO(Last-In-First-Out,后进先出)的数据结构，也就是最新添加的项被移除。
push() 添加到数组末尾
pop() 删除到数组末尾
shift() 添加到数组头部
unshift() 删除到数组头部

```javascript
var arr = [4, 5, 6, 7];
arr.push(8);
console.log(arr); // [4, 5, 6, 7, 8]
arr.pop();
console.log(arr); // [4, 5, 6, 7]
arr.unshift(1);
console.log(arr); // [1, 4, 5, 6, 7]
arr.shift();
console.log(arr); // [4, 5, 6, 7]
```

## 位置方法 indexOf()、lastIndexOf()、find()、findIndex()

接受两个参数：要查找的项和（可选的）表示查找起点位置的索引。
indexOf()方法从数组开头
lastIndexOf()方法则从数组的末尾

```javascript
var arr = [1, 2, 3, 4, 5, 6, 4, 8];
console.log(arr.indexOf(4)); // 3
console.log(arr.lastIndexOf(4)); // 6
console.log(arr.lastIndexOf(4)); // 3
console.log(arr.lastIndexOf(4)); // 3
```

find() 方法返回数组中满足提供的测试函数的第一个元素的值。否则返回 undefined。

**未完待续。。。。。**
