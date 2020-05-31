---
title: 什么是尾递归？ 尾递归和普通的递归的区别
date: 2018-08-13 14:32:09
tags: [JavaScript, Algorithm]
categories: [Algorithm]
description: 什么是尾递归？什么是尾调用？尾递归解决的什么问题，它和斐波拉契数列有什么关联，你都可以通过本篇文章学习到。
---

## 简介

首先解释什么是`尾递归`和`尾调用`，后面再来解释什么是`斐波拉契数列`，怎么用`JavaScript`实现`斐波拉契数列`，尾递归和它有什么关联。
本文文章大致章节如下：

- 理解调用栈
- 什么是尾调用和尾调用优化
- 什么是尾递归
- 什么是斐波拉契数列

通过上面的几个章节一步一步加深理解。

## 调用栈

首先要了解什么是`调用栈`，后面才能更好的了解`尾递归`和`尾调用`。

**调用栈**是**解释器（就像浏览器中的 JavaScript 解释器）追踪函数执行流的一种机制**。简单来说就是能够通过`调用栈`追踪到那个函数正在执行，执行的函数中又调用了那个函数。可能这样说还是不太具体，我们可以把它更具体一下。

首先引入一个概念：`栈帧`，**栈帧**是指一个函数调用单独分配的那部**分栈空间**。`栈帧`中有两种比较重要的帧`当前帧`、`调用帧`，先看一张图：

![algorithm-tailCall](../../images/algorithm/algorithm-1-1.png)

当运行中的程序`调用另一个函数时`，就要进入一个`新的栈帧`，`原来函数`的栈帧称为`调用者的帧`，`新的栈帧`称为`当前帧`。
那么`调用栈`大致是怎么执行的呢？大致步骤如下：

- 每调用一个函数，解释器就会把该函数添加进`调用栈`并开始执行。
- 正在`调用栈`中**执行的函数(调用者帧)**还`调用了其它函数`，那么`新函数(当前帧)`也将会被`添加进调用栈`，一旦这个函数被调用，便会立即执行。
- `新函数(当前帧)`执行完毕后，解释器将其`清出调用栈`，继续执行**执行的函数(调用者帧)**环境下的剩余的代码。
- 当分配的`调用栈空间`被占满时，会引发`“堆栈溢出”(递归爆栈)`。

在线代码体验[调用栈执行过程](https://codepen.io/njmcode/pen/dMPmGq)，如果访问比较慢可以看[调用栈执行过程 demo](https://github.com/Braveheartforyou/Blog-Static/tree/master/callStack)把代码下载到本地运行查看。

执行效果大致如下：
![algorithm-tailCall](../../images/algorithm/algorithm-1-3.png)
因为 gif 文件过大，就放一张图片好了。

### 调试

我们也可以通过`chrome`中的控制台，通过`console.trace()`来追踪当前的调用栈，如下图所示：
![algorithm-tailCall](../../images/algorithm/algorithm-1-2.png)

或者通过在代码中`打断点调试`看到当前调用栈：
![algorithm-tailCall](../../images/algorithm/algorithm-1-4.png)

其实`调用栈`和`事件轮询(event loop)`有很大的关联，如果对`事件轮询(event loop)`有兴趣的话看我另一篇文章[事件轮询/事件模型](/blog/javascript/evenloop.html)。

## 尾调用和尾调用优化

### 尾调用

`尾调用`是函数式编程中一个很重要的概念，当一个`函数执行时`的`最后一个步骤`是返回`另一个函数`的`调用`，这就叫做`尾调用`。
什么样算尾调用，什么不算尾调用呢？

注意这里函数的调用方式是无所谓的，以下方式均可：

```js
    函数调用:     func(···)
    方法调用:     obj.method(···)
    call调用:     func.call(···)
    apply调用:    func.apply(···)
```

并且只有下列表达式会包含尾调用：

```js
    条件操作符:      ? :
    逻辑或:         ||
    逻辑与:         &&
    逗号:           ,
```

**不是尾调用**的实例

```js
// 不是尾调用 调用函数后还有复制操作
function notCallStack(name) {
  let name = otherFunc(name);
  return name;
}
// 不是尾调用 调用函数后还有拼接操作
function notCallStack(name) {
  return otherFunc(name) + 1;
}

// g()有可能是尾调用，f()不是
const a = (x) => (x ? f() : g());
```

**是尾调用**的实例

```js
// 尾调用正确示范1.0
function f(x) {
  return g(x);
}

// 尾调用正确示范2.0
function f(x) {
  if (x > 0) {
    return m(x);
  }
  return n(x);
}
```

这个就是`尾调用`，下面我们就可以通过`尾调用`去优化执行栈的调用过程。

### 尾调用优化

函数在调用的时候会在`调用栈（call stack）`中存有记录，每一条记录叫做一个`调用帧（call frame）`，每调用一个函数，就向栈中`push`一条记录，函数执行结束后`依次向外弹出`，直到`清空调用栈`，参考下图：

```js
function one() {
  two();
}
function two() {
  three();
}
function three() {
  console.trace();
}
one();
```

执行过程如下下图所示：

![algorithm-tailCall](../../images/algorithm/algorithm-1-5.png)

我们在一个函数中调用另一个函数，但是并没有通过`return`来结束当前函数的执行，`JS引擎`会认为当前的函数并没有执行完成，会在执行当前函数调用的函数，等他执行完成才会释放当前函数。

- `one函数`执行时，会把`one函数`添加进`调用栈`中，`one函数`现在为`当前帧`。
- 在`one函数`中又调用了`two函数`，当时在调用`two函数`时没有`return`，所以会把`two函数`添加进`调用栈`。现在`one函数`为`调用者帧`，而`two函数`为`当前帧`
- 在`two函数`中又调用`three函数`，执行过程与`two函数`执行相同。
- 当`three函数`执行完成时（默认返回 undefined），`three函数`就会被`调用栈`弹出并且被销毁。再在后面逐步销毁`two函数`、`one函数`，到此`调用栈为空`。

下面通过尾调用优化，修改代码如下：

```js
    “use strict”;
    function one () {
        return two();
    }
    function two () {
        return three();
    }
    function three () {
        console.trace();
        return false;
    }
    one();
```

执行效果如下图所示：

![algorithm-tailCall](../../images/algorithm/algorithm-1-6.png)

- `one函数`执行时，会把`one函数`添加进`调用栈`中，`one函数`现在为`当前帧`。
- 在`one函数`中又调用了`two函数`，当时在调用`two函数`添加了`return`，`调用栈`会把`one函数`弹出，当前`调用栈`中只有一个`two函数`。
- 在`two函数`中又调用`three函数`，因为有`return`当前`调用栈`中只有`three函数`。
- 当`three函数`执行完成后，`调用栈`弹出`three函数`，此时`调用栈`当前为空。

> 注意： 无论是通过`console.trace()`,还是通过`chrome`断点查看`call stack`都并没有改变`调用栈`，意思就是和上面的一样，应该是`chrome`禁止了尾调用优化。(暂无找到原因--我佛了)
> safari 中是好的

## 尾递归

**递归**

递归是指在`函数`的定义中使用`函数自身`的一种方法。`函数调用自身`即称为递归。

```js
function foo() {
  foo();
}
```

这是一个死循环，会造成页面或者进程假死，也就是堆栈溢出。

**尾递归**

当一个函数在最后调用自身就叫做**尾递归**。

```js
function foo() {
  return foo();
}
```

> 尾调用优化只在严格模式下有效。
> 尾调用优化后，每次 return 的内层函数的调用记录会取代外层函数的调用记录，调用栈中始终只保持了一条调用帧。

### 尾递归作用

比如我们要实现一个阶加，可以用尾递归实现，下面直接上代码。

环境`chrome 78.0.3904.70`，硬件`mac pro 16G i5`。

```js
function factorial(num) {
  if (num === 1) {
    return 1;
  }
  console.trace();
  return num + factorial(num - 1);
}
// 测试代码
factorial(4); // 24
factorial(20); // 2432902008176640000
factorial(100000); //  Maximum call stack size exceeded
```

根据我们上面知道调用栈的知识，如果我们传入一下`100000`，它在执行过程中它会把`每次执行`的函数添加进`调用栈`，只有在`最后被调用的函数执行完成`，才会把调用栈中的函数一个个`弹出和销毁`。`100000`个函数已经超出了浏览器最大的内存范围了，所以会造成栈溢出错误。

即使加上`"use strict";`也还是会报错。

> `尾调用优化`和`尾递归`在`firfox`和`chrome`中会报错，`safari`在`尾优化不会报错`但是`尾递归还是会报错`。
> 如果真的想感受尾优化的威力可以去`node v.6.x`版本中通过`--harmony_tailcalls参数`，`node`新的版本并已经移除了这个参数

## 斐波拉契数列

> `斐波那契数列（Fibonacci sequence）`，又称`黄金分割数列`、因数学家列昂纳多·斐波那契（Leonardoda Fibonacci）以兔子繁殖为例子而引入，故又称为“兔子数列”，指的是这样一个数列：1、1、2、3、5、8、13、21、34、……

简单的说，斐波那契数列中的`每一项都是前两项的和`。
即`F(1)=1，F(2)=1, F(n)=F(n-1)+F(n-2)（n>2，n∈N*）`

### 基础版本

我们通过递归实现

```js
function factorial(num) {
  if (num === 0 || num === 1) {
    return num;
  }
  console.trace();
  // console.log(factorial(num - 1) + factorial(num - 2));
  return factorial(num - 1) + factorial(num - 2);
}
```

我们通过`console.trace()`可以看到在执行过程中，`调用栈`中最多会存在`4`个函数信息，这十个信息是每一层次调用的详细信息（如参数、局部变量、返回地址等等），以确保该层次的操作完成，这也是造成`栈溢出`的原因。

测试代码：

```js
factorial(4);
// 展开来看如下
factorial(3) +
  factorial(2)(factorial(2) + factorial(1)) +
  (factorial(1) + factorial(0))(factorial(1)) +
  factorial(0) +
  factorial(1);
```

### 尾递归版本

通过`尾递归来优化`上面的问题，其实在现在`浏览器`或者`node`中都没有作用。

```js
'use strict';
function factorial(num, num1 = 0, num2 = 1) {
  if (num === 0) {
    return num1;
  }
  console.trace();
  return factorial(num - 1, num2, num1 + num2);
}
```

测试代码：

```js
// 测试代码
factorial(5); // 5
// 展开显示
factorial(5, 0, 1);
factorial(4, 1, 1);
factorial(3, 1, 2);
factorial(2, 2, 3);
factorial(1, 3, 5);
factorial(0, 5, 8);

// 当num-1为0时，直接返回num1
```

可以发现当使用尾递归优化时，展开看到的`调用栈`中只会有当前执行的函数，不会储存上层的数据，这样就会减少内存的使用。
`尾递归`的本质实际上就是将方法需要的上下文通过方法的参数传递进下一次调用之中，以达到去除上层依赖。

> chrome/firefox 测试无效，node 新版本测试无效
> Proper tail calls have been implemented but not yet shipped given that a change to the feature is currently under discussion at TC39.意思就是人家已经做好了，但是就是还不能。

### 尾递归的问题

- 首先，由于引擎`消除尾递归是隐式的`，函数是否符合尾调用而被消除了尾递归很难被程序员自己辨别。
- 其次，尾调用优化`要求除掉尾调用执行时的调用堆栈`，这将导致执行流中的`堆栈信息丢失`。

### 多种实现方式

**循环实现**

可以通过循环实现，代码如下：

```js
function factorial(n) {
  if (n === 0 || n === 1) {
    return n;
  }
  let x = 0;
  let y = 1;
  let sum = 0;
  for (let i = 2; i <= n; i++) {
    sum = x + y;
    x = y;
    y = sum;
  }
  return sum;
}

// 测试代码
factorial(5); // 5
```

**公式实现**

通过`Math`来实现，代码如下：

```js
// 公式法
function factorial(n) {
  if (n <= 0) return 0;
  else {
    const sqrtFive = Math.sqrt(5);
    const res =
      (Math.pow(0.5 + sqrtFive / 2, n) - Math.pow(0.5 - sqrtFive / 2, n)) /
      sqrtFive;
    return Math.round(res);
  }
}
// 测试代码
factorial(5);
```

**函数柯里化**

通过柯里化实现，代码如下

```js
function tailCalls(num, num1, num2) {
  if (num === 0) {
    return num1;
  }
  console.trace();
  return tailCalls(num - 1, num2, num1 + num2);
}

function factorial(num) {
  return tailCalls(num, 0, 1);
}
```

其实上面的柯里化只能称为模仿柯里化实现。

**性能对比**

我们把`普通递归`、`尾递归`、`普通循环`、`公式法`、`柯里化`它们的性能对比。

```js
// 普通递归
function factorial(num) {
  if (num === 0 || num === 1) {
    return num;
  }
  return factorial(num - 1) + factorial(num - 2);
}

// 尾递归
('use strict');
function tailfactorial(num, num1 = 0, num2 = 1) {
  if (num === 0) {
    return num1;
  }
  return factorial(num - 1, num2, num1 + num2);
}

// 循环实现
function loopfactorial(n) {
  if (n === 0 || n === 1) {
    return n;
  }
  let x = 0;
  let y = 1;
  let sum = 0;
  for (let i = 2; i <= n; i++) {
    sum = x + y;
    x = y;
    y = sum;
  }
  return sum;
}

// 公式法
function mathfactorial(n) {
  if (n <= 0) return 0;
  else {
    const sqrtFive = Math.sqrt(5);
    const res =
      (Math.pow(0.5 + sqrtFive / 2, n) - Math.pow(0.5 - sqrtFive / 2, n)) /
      sqrtFive;
    return Math.round(res);
  }
}

// 柯里化
function tailCalls(num, num1, num2) {
  if (num === 0) {
    return num1;
  }
  return tailCalls(num - 1, num2, num1 + num2);
}

function curryfactorial(num) {
  return tailCalls(num, 0, 1);
}

// 测试代码
console.time();
factorial(20);
console.timeEnd();

console.time();
tailfactorial(20);
console.timeEnd();

console.time();
loopfactorial(20);
console.timeEnd();

console.time();
mathfactorial(20);
console.timeEnd();

console.time();
curryfactorial(20);
console.timeEnd();

// default: 0.10205078125ms 普通递归
// default: 0.112060546875ms 尾递归
// default: 0.052978515625ms 循环
// default: 0.06591796875ms 公式
// default: 0.052978515625ms 柯里化
```

我们可以通过上面的测试，可以看到`递归`速度都是比较慢的，`循环`的速度是比较快的。

- **慎用直接递归的方式，不仅会带来极差的运行效率，同时会导致浏览器直接无响应。**
- **尾递归**有着与循环同样优秀的计算性能，使用尾递归可以同时拥有着**循环的性能**以及递归的数学表达能力。

## PTC 与 STC

ES6 标准规定了 [尾调用不会创建额外的调用帧](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-preparefortailcall)。
在严格模式下 [尾调用不会造成调用栈溢出](https://v8.dev/blog/modern-javascript)。
`Proper Tail Calls(PTC)`已经实现了，但是还未部署，该功能仍然在[TC39](https://github.com/tc39/ecma262)标准委员会中讨论。

### PTC

什么是`Proper Tail Calls(PTC)`?如果有兴趣可以去看原文的解释[原文](https://webkit.org/blog/6240/ecmascript-6-proper-tail-calls-in-webkit/)。

> Typically when calling a function, stack space is allocated for the data associated with making a function call. This data includes the return address, prior stack pointer, arguments to the function, and space for the function’s local values. This space is called a stack frame. A call made to a function in tail position will reuse the stack space of the calling function.

简单来说就是**比如说一个递归程序，我们调用函数时，内存会帮函数分配返回地址、先前堆栈指针、内部变量参数称为 stack frame。在尾部位置对函数的调用将重用调用函数的堆栈空间。**

要触发`PTC`就要满足一下条件：

- `strict mode`严格模式下
- 普通函数或者箭头函数
- 不能是生成器(generator)函数
- 被调用函数的返回值由调用函数返回。

`PTC`是能提升性能的一种策略，但是他也存在很多的限制。

**PTC**存在的限制

**兼容性**

因为在推行一些新的策略或者方案是，就是标准是否支持它，也就是兼容性。标准的兼容性、浏览器的兼容性是一段很长的路。

**调试难度**

在`PTC`的实现中，许多调用帧都`被抛弃`了，导致很难再调用栈中调试他们的代码。

```js
// 举个例子
function foo(n) {
  return bar(n * 2); // 尾调用
}

function bar() {
  throw new Error();
}

foo(1);
// 由于尾调用优化
// 在Error.stack或者开发者工具中，foo的调用帧被丢掉了。
```

**Error.stack**

启用`PTC`导致`Javascript异常`有了不一致的`error.stack`信息。

```js
/*
    output without PTC
    Error
        at bar
        at foo
        at Global Code

    output with PTC (note how it appears that bar is called from Global Code)
    Error
        at bar
        at Global Code
    */
```

### STC

语义上的尾调用（`Syntactic Tail Call`）是针对上述`PTC`的问题而提出的建议。

`STC`采用类似于 `return continue` 的语法来明确标识出要进行`尾调用优化`，而在`非尾调用`的场景下使用该语法会`抛出语法错误异常`。
该语法有三种实现形式：

**语法级**

```js
function factorial(n, acc = 1) {
    if (n === 1) {
        return acc;
    }
    return continue factorial(n - 1, acc * n)
}
```

**函数级**

```js
#function() { /* all calls in tail position are tail calls */ }
```

**表达式/调用点**

```js
    function () {
        !return expr
    }
```

## 总结

通过本篇文章了解了什么是`调用栈`、`尾调用`、`尾调用`、`斐波拉切数列`，怎么实现`斐波拉切数列`，多种方法对比。
尽量少使用`递归`因为递归的比较消耗性能，虽然有`尾递归优化`但是各大浏览器都并没有部署，所以尽量使用循环来实现。

## 参考

[JavaScript 调用栈、尾递归和手动优化](https://juejin.im/entry/592e8a2d0ce463006b510b34)
[尾调用优化——记一道面试题的思考](https://segmentfault.com/a/1190000014747296)
[朋友你听说过尾递归吗](https://imweb.io/topic/584d33049be501ba17b10aaf#5-1-ptc-)
[尾递归的后续探究](https://imweb.io/topic/5a244260a192c3b460fce275)
[尾调用和尾递归](https://juejin.im/post/5acdd7486fb9a028ca53547c)
