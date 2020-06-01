---
title: ES6 Array系列(一) 一些常用array的扩展方法
date: 2017-07-28 16:45:36
tags: [ECMAScript6]
categories: [ECMAScript6]
description: JavaScript 数组对象是用于构造数组的全局对象; 它是高阶，类似列表的对象。
---

<!--## Array中常用的属性，或者方法-->

## 属性

---

### Array.length

length 属性表示一个无符号 32-bit 整数，返回一个数组中的元素个数。

## 方法

## Array.from

Array.from() 方法从一个类似数组或可迭代的对象中创建一个新的数组实例。

### 语法

```javascript
    Array.from(arrayLike[, mapFn[, thisArg]])
    // 参数
    // arrayLike 想要转换成真实数组的类数组对象或可遍历对象。
    // mapFn(可选) 可选参数，如果指定了该参数，则最后生成的数组会经过该函数的加工处理后再返回。
    // thisArg(可选) 可选参数，执行 mapFn 函数时 this 的值。
    // 返回值 返回一个新的Array类型的实例
```

### 描述

Array.from() 允许你从下面两者来创建数组：

- 类数组对象（拥有一个 length 属性和若干索引属性的任意对象）
- 可遍历对象（你可以从它身上迭代出若干个元素的对象，比如有 Map 和 Set 等）
  Array.from() 方法有一个可选参数 mapFn，让你可以在最后生成的数组上再执行一次 map 方法后再返回。也就是说 Array.from(obj, mapFn, thisArg) 就相当于 Array.from(obj).map(mapFn, thisArg), 除非创建的不是可用的中间数组。 这对一些数组的子类,如 typed arrays 来说很重要, 因为中间数组的值在调用 map() 时需要是适当的类型。

### 示例

```javascript
// Array from a String
Array.from('foo');
// ["f", "o", "o"]

// Array from a Set
let s = new Set(['foo', window]);
Array.from(s);
// ["foo", window]

// Array from a Map
let m = new Map([
  [1, 2],
  [2, 4],
  [4, 8]
]);
Array.from(m);
// [[1, 2], [2, 4], [4, 8]]

// Array from an Array-like object (arguments)
function f() {
  return Array.from(arguments);
}
f(1, 2, 3);
// [1, 2, 3]

//Using arrow functions and Array.from
// Using an arrow function as the map function to
// manipulate the elements
Array.from([1, 2, 3], (x) => x + x);
// [2, 4, 6]

// Generate a sequence of numbers
// Since the array is initialized with `undefined` on each position,
// the value of `v` below will be `undefined`
Array.from({ length: 5 }, (v, i) => i);
// [0, 1, 2, 3, 4]
```

## Array.isArray()

Array.isArray() 用于确定传递的值是否是一个 Array。

### 语法

```javascript
Array.isArray(obj);
// 参数
// obj
// 需要检测的值。
// 返回值
// 如果对象是 Array，则为true; 否则为false。
```

### 描述

如果对象是 Array ，则返回 true，否则为 false。
有关更多详细信息，请参阅文章以绝对精确度确定 JavaScript 对象是否为数组。

### 示例

```javascript
// 下面的函数调用都返回 true
Array.isArray([]);
Array.isArray([1]);
Array.isArray(new Array());
// 鲜为人知的事实：其实 Array.prototype 也是一个数组。
Array.isArray(Array.prototype);

// 下面的函数调用都返回 false
Array.isArray();
Array.isArray({});
Array.isArray(null);
Array.isArray(undefined);
Array.isArray(17);
Array.isArray('Array');
Array.isArray(true);
Array.isArray(false);
Array.isArray({ __proto__: Array.prototype });
```

### 兼容性代码

假如不存在 Array.isArray()，则在其他代码之前运行下面的代码将创建该方法。

```javascript
if (!Array.isArray) {
  Array.isArray = function (arg) {
    return Object.prototype.toString.call(arg) === '[object Array]';
  };
}
```

## 扩展运算符（spread）

扩展运算符（spread）是三个点（...）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列。

```javascript
function push(array, ...items) {
  array.push(...items);
}

function add(x, y) {
  return x + y;
}

var numbers = [4, 38];
add(...numbers); // 42
```

上面代码中，array.push(...items)和 add(...numbers)这两行，都是函数的调用，它们的都使用了扩展运算符。该运算符将一个数组，变为参数序列。

## Array.of()

Array.of() 方法创建一个具有可变数量参数的新数组实例，而不考虑参数的数量或类型。

### 语法

```javascript
    // 参数
    // elementN
    // 任意个参数，将按顺序成为返回数组中的元素。
    // 返回值
    // 新的 Array 实例。
    Array.of(element0[, element1[, ...[, elementN]]])
```

### 示例

```javascript
Array.of(1); // [1]
Array.of(1, 2, 3); // [1, 2, 3]
Array.of(undefined); // [undefined]
Array.of(); // []
```

### 兼容旧环境

```javascript
if (!Array.of) {
  Array.of = function () {
    return Array.prototype.slice.call(arguments);
  };
}
```

## Array.prototype.concat()

concat() 方法用于合并两个或多个数组。此方法不会更改现有数组，而是返回一个新数组。

### 语法

```javascript
    var new_array = old_array.concat(value1[, value2[, ...[, valueN]]])
    // 参数
    // valueN
    // 需要与原数组合并的数组或非数组值。详见下文。
    // 返回值
    // 新的 Array 实例。
```

### 描述

concat 方法并不修改调用它的对象(this 指向的对象) 和参数中的各个数组本身的值,而是将他们的每个元素拷贝一份放在组合成的新数组中.原数组中的元素有两种被拷贝的方式:

- 对象引用(非对象直接量):concat 方法会复制对象引用放到组合的新数组里,原数组和新数组中的对象引用都指向同一个实际的对象,所以,当实际的对象被修改时,两个数组也同时会被修改.
- 字符串和数字(是原始值,而不是包装原始值的 String 和 Number 对象): concat 方法会复制字符串和数字的值放到新数组里.

### 示例

```javascript
// 两个数组合并为一个新数组
var alpha = ['a', 'b', 'c'];
var numeric = [1, 2, 3];
var three = [4, 5, 6];
// 组成新数组 ["a", "b", "c", 1, 2, 3]; 原数组 alpha 和 numeric 未被修改
var alphaNumeric = alpha.concat(numeric);

// 组成新数组["a", "b", "c", 1, 2, 3, 4, 5, 6]; 原数组 num1, num2, num3 未被修改
var nums = num1.concat(num2, num3);

// 多个数组和多个非数组值合并为一个新数组
// 组成新数组 ["a", "b", "c", 1, 2, 3], 原alpha数组未被修改
var alphaNumeric1 = alpha.concat(1, [2, 3]);
```

## Array.prototype.copyWithin()

copyWithin() 方法浅复制数组的一部分到同一数组中的另一个位置，并返回它，而不修改其大小。

### 语法

```javascript
arr.copyWithin(target);
arr.copyWithin(target, start);
arr.copyWithin(target, start, end);
arr.copyWithin(目标索引, [源开始索引], [结束源索引]);
// 参数
// target
// 0 为基底的索引，复制序列到该位置。如果是负数，target 将从末尾开始计算。如果 target 大于等于 arr.length，将会不发生拷贝。如果 target 在 start 之后，复制的序列将被修改以符合 arr.length。

// start
// 0 为基底的索引，开始复制元素的起始位置。如果是负数，start 将从末尾开始计算。    如果 start 被忽略，copyWithin 将会从0开始复制。

// end
// 0 为基底的索引，开始复制元素的结束位置。copyWithin 将会拷贝到该位置，但不包括 end 这个位置的元素。如果是负数， end 将从末尾开始计算。 如果 end 被忽略，copyWithin 将会复制到 arr.length。

// 返回值
// 改变了的数组。
```

### 描述

参数 target,start 和 end 必须为整数。
如果 start 为负，则其指定的索引位置等同于 length+start，length 为数组的长度。end 也是如此。
copyWithin 方法不要求其 this 值必须是一个数组对象；除此之外，copyWithin 是一个可变方法，它可以改变 this 对象本身，并且返回它，而不仅仅是它的拷贝。
copyWithin 就像 C 和 C++ 的 memcpy 函数一样，且它是用来移动 Array 或者 TypedArray 数据的一个高性能的方法。复制以及粘贴序列这两者是为一体的操作;即使复制和粘贴区域重叠，粘贴的序列也会有拷贝来的值。
copyWithin 函数是设计为通用的，其不要求其 this 值必须是一个数组对象。
The copyWithin 是一个可变方法，它不会改变 this 的 length，但是会改变 this 本身的内容，且需要时会创建新的属性。

### 例子

```javascript
[1, 2, 3, 4, 5].copyWithin(-2);
// [1, 2, 3, 1, 2]

[1, 2, 3, 4, 5].copyWithin(0, 3);
// [4, 5, 3, 4, 5]

[1, 2, 3, 4, 5].copyWithin(0, 3, 4);
// [4, 2, 3, 4, 5]

[1, 2, 3, 4, 5].copyWithin(-2, -3, -1);
// [1, 2, 3, 3, 4]

[].copyWithin.call({ length: 5, 3: 1 }, 0, 3);
// {0: 1, 3: 1, length: 5}

// ES2015 Typed Arrays are subclasses of Array
var i32a = new Int32Array([1, 2, 3, 4, 5]);

i32a.copyWithin(0, 2);
// Int32Array [3, 4, 5, 4, 5]

// On platforms that are not yet ES2015 compliant:
[].copyWithin.call(new Int32Array([1, 2, 3, 4, 5]), 0, 3, 4);
// Int32Array [4, 2, 3, 4, 5]
```

## Array.prototype.entries()、Array.prototype.keys()、Array.prototype.values() 遍历数组

ES6 提供三个新的方法——entries()，keys()和 values()——用于遍历数组。它们都返回一个遍历器对象（详见《Iterator》一章），可以用 for...of 循环进行遍历，唯一的区别是 keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历。

- entries() 方法返回一个新的 Array Iterator 对象，该对象包含数组中每个索引的键/值对。
- keys() 方法返回一个新的 Array 迭代器，它包含数组中每个索引的键。
- values() 方法返回一个新的 Array Iterator 对象，该对象包含数组每个索引的值。

```javascript
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"

let arr = ['w', 'y', 'k', 'o', 'p'];
let eArr = arr.values();
// 您的浏览器必须支持 for..of 循环
// 以及 let —— 将变量作用域限定在 for 循环中
for (let letter of eArr) {
  console.log(letter);
}

// 另一种 迭代方式
let arr = ['w', 'y', 'k', 'o', 'p'];
let eArr = arr.values();
console.log(eArr.next().value); // w
console.log(eArr.next().value); // y
console.log(eArr.next().value); // k
console.log(eArr.next().value); // o
console.log(eArr.next().value); // p
```

可以通过 for...of 来循环数组

## 转换方法

在 javascript 中所有的对象都具有 toLocaleString()、toString()、valueOf()方法。

### 语法

```javascript
var colors = ['red', 'blue', 'green'];
console.log(colors.toString()); // red,blue,green
console.log(colors.valueOf()); // [red,blue,green]
console.log(colors.toLocaleString()); // red,blue,green
console.log(colors.join(',')); // red,blue,green
// 在默认情况下都会以逗号分隔的字符串的形式返回数组项
```

### 描述

如果数组中的某一项的值是 null 或者 undefined,那么该只在 join()、toLocaleString()、toString()、valueOf()方法返回的结果中以空字符串表示

## 重排序方法

reverse() 方法会反转数组项的顺序。 sort()方法按升序排列数组项

### 语法

```javascript
// reverse() 数组反转
var values = [1, 3, 4, 5, 6, 7, 8];
values.reverse();
console.log(values); // [8, 7, 6, 5, 4, 3, 1]

//sort() 排序
function compare(value1, value2) {
  if (value1 > value2) {
    return 1;
  } else if (value1 < value2) {
    return -1;
  } else {
    return 0;
  }
}
var values1 = ['2', 4, 1, 3, 5, 2, 4, 65, 2];
values1.sort(compare);
console.log(values1); //  [1, "2", 2, 2, 3, 4, 4, 5, 65]
```

**未完待续。。。。。**
