---
title: ES6 Array系列(三) Array中的forEach方法可以用break、continue跳出循环？
date: 2019-08-17 09:24:31
tags: [ECMAScript6]
categories: [ECMAScript6]
description: Array中的forEach方法可以用break、continue跳出循环？
---

**_飄風不終朝，驟雨不終日。——《道德經》_**

## 简介

---

在`Array.prototype`上有很多方法，比较常用的就是`every`、`filter`、`forEach`、`map`、`some`这些循环方法，可以通过`break`、`comtinue`跳出循环？
现在基本上都是通过`forEach`、`every`来代替`for`循环，`for`循环可以通过`break`、`continue`跳出循环。而 forEach 可以不可以呢，下面一步一步的验证一下。

## for

---

在`for`中遇到`break`就会退出当前循环，后面的循环不会再执行。代码如下

```javascript
let arr = [1, 2, 3, 4, 5];
let str = "";
for (let item of arr) {
  if (item === 2) {
    break;
  }
  str += item + "-";
}
console.log(str); // 1-
```

在`for`中遇到`continue`就会退出当前本次循环，后面的循环会执行。代码如下

```javascript
let arr = [1, 2, 3, 4, 5];
let str = "";
for (let item of arr) {
  if (item === 2) {
    continue;
  }
  str += item + "-";
}
console.log(str); // 1-3-4-5-
```

> 在`for`中不能使用 return，不然会报错

## forEach、every、some、filter、map

---

如果想退出`forEach`，就只能通过`return`, 它会退出当前本次循环，后面的循环会执行。代码如下

```javascript
var arr = [1, 2, 3, 4, 5];
var str = "";
arr.forEach(item => {
  if (item === 2) {
    return;
  }
  str += item + "-";
});
console.log(str); // 1-3-4-5-
```

如果想退出`every`，就只能通过`return`, 它会退出当前循环，后面的循环不会再执行。代码如下

```javascript
var arr = [1, 2, 3, 4, 5];
var str = "";
arr.every(item => {
  if (item === 2) {
    return;
  }
  str += item + "-";
});
console.log(str); // 1-
```

如果想退出`some`，就只能通过`return`, `return false`和`return true`它的表现是不一致的。`return false`它的表现和`forEach`中的表现一致。 代码如下

```javascript
var arr = [1, 2, 3, 4, 5];
var str = "";
arr.forEach(item => {
  if (item === 2) {
    return;
    // return false;
  }
  str += item + "-";
});
console.log(str); // 1-3-4-5-
```

`return true`和`every`表现一致，代码如下：

```javascript
var arr = [1, 2, 3, 4, 5];
var str = "";
arr.forEach(item => {
  if (item === 2) {
    return true;
  }
  str += item + "-";
});
console.log(str); // 1-
```

如果想退出`filter`，就只能通过`return`, 它会退出当前本次循环，后面的循环会执行。代码如下

```javascript
var arr = [1, 2, 3, 4, 5];
var str = "";
arr.filter(item => {
  if (item === 2) {
    return;
  }
  str += item + "-";
});
console.log(str); // 1-3-4-5-
```

如果想退出`map`，就只能通过`return`, 它会退出当前本次循环，后面的循环会执行。代码如下

```javascript
var arr = [1, 2, 3, 4, 5];
var str = "";
arr.map(item => {
  if (item === 2) {
    return;
  }
  str += item + "-";
});
console.log(str); // 1-3-4-5-
```

> 在`forEach`、`every`、`some`中不能用`break`、`continue`跳出循环，不然会报错。

都知道`filter`、`map`会返回一个新的数组，而`every`、`some`会返回一个`Boolean`类型的。

## some 和 every 需要注意的地方

---

`some()` 方法测试是否至少有一个元素可以通过被提供的函数方法。该方法返回一个 Boolean 类型的值。`polyfill`实现如下：

```javascript
Array.prototype.some = function(callbackfn, thisArg) {
  let len = Number(this.length);
  let k = 0;
  while (k < len) {
    let Pk = String(k);
    if (Pk in this) {
      let kValue = this[Pk];
      if (callbackfn.call(thisArg, kValue, k, this)) {
        return true;
      }
    }
    k++;
  }
  return false;
};
```

可以看出，遇到回调返回值是 `true` 的话，函数就直接返回、结束了。这是种短路算法，并不是所有回调都执行一遍，然后再最后求所有与值。`every` 也类似，不过与之相反，遇到回调返回值是 `false` 时，整体就直接返回 `false` 了。
从实现上表达出的语义来讲，`some` 是在说：**有一个成功，我就成功，而 `every` 是在说：有一个失败，我就失败**
**另外要强调一点，对于稀疏数组，不存在的索引值时，回调函数是不执行的**。

## 参考

> [Array.prototype.some()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/some)
> [不再写 break 和 continue 了](https://juejin.im/post/5d08a565e51d45773d468614#heading-2)
