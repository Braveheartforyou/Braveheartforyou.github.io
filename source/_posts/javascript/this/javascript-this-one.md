---
title: JavaScript中的this（一）this是什么？怎么产生的？什么是执行上下文？
date: 2018-12-06 22:32:43
tags: [JavaScript, This]
categories: [JavaScript]
description: this的产生
---

## 简介

`this`如论是在平常开发中，还是在面试时都是经常会听到，所以有了这篇文章来更系统的记录`this`从产生到使用。到最后会有一整篇的面试题来介绍`this`。

`this`是在什么时候产生的呢？`this`的作用是什么，首先要知道一个概念就是`执行上下文`。要了解清楚`执行上下文`，又要了解`"调用栈"`，要了解调用栈，又要了解`作用域`中的`动态作用域`。其实后面还有`event loop`、`线程执行`等等，所以学无止境，回头是岸。

还是回到`"调用栈"`和`执行上下文`，首先了解这两个概念就能知道`this`是什么时候产生的，`this`是什么。

首先介绍几个概念：

- `ECS`: 执行环境栈，**Execution Context Stack**。
- `EC`: 函数执行环境（或执行上下文），**Execution Context**。
- `VO`: 变量对象，**variable Object**。
- `AO`: 活动对象，**Active Object**。
- `S`: 作用域，**Scope**。
- `SC`: 作用域链，**Scope Chain**。

首先`执行栈`中包含`执行上下文`，`执行上下文`中包含有`活动变量`、`变量对象`、`作用域`，一层一层的作用域又会形成`作用域链`。它们之间都是有关联的。

## “调用栈”

什么是`调用栈`其实也是常说的`执行栈`，它其实应该叫做`执行上下文堆栈`会更准确一些。我们还是把它简称为`执行栈`。
因为 JavaScript 解释器被实现为单线程。这意味着`JavaScript 引擎`只能同时执行一件事，其它需要执行的事情会被放到一个栈里面储存，这个栈就叫做`执行栈`。
`执行栈`是一种拥有`LIFO（后进先出）`数据结构的栈，被用来存储代码运行时创建的所有执行上下文。

### 执行栈执行过程

当 `JavaScript 引擎`第一次遇到你的脚本时，它会创建一个`全局`的`执行上下文`并且压入当前`执行栈`。每当引擎遇到一个函数调用，它会为该函数创建一个`新的执行上下文并`压入栈的`顶部`。

引擎会执行那些`执行上下文位于栈顶的函数`。当该函数执行结束时，执行上下文`从栈中弹出`，控制流程到达当前栈中的下一个上下文。

首先我们看一张比较经典的图。

![this-one](../../images/javascript/javascript-this-1-1.jpg)

一个简单的实例，代码如下：

```js
let name = 'global name';

function first() {
  console.log('inside first function');
  second();
}
function second() {
  console.log('inside second function');
  three();
}
first();
console.log('inside global Execution Context');
```

执行效果如下图所示：

![this-one](../../images/javascript/javascript-this-1-2.png)

当上述代码在浏览器加载时，`JavaScript 引擎`创建了一个`全局执行上下文`并把它压入当前`执行栈`。当遇到 `first()` 函数调用时，`JavaScript 引擎`为该函数`创建`一个新的`执行上下文`并把它压入`当前执行栈`的顶部。

当从 `first()` 函数内部调用 `second()` 函数时，`JavaScript 引擎`为 `second()` 函数创建了一个`新的执行上下文`并把它压入当前`执行栈的顶部`。当 `second()` 函数执行完毕，它的`执行上下文`会从当前栈`弹出`，并且控制流程到达下一个`执行上下文`，即 `first()` 函数的执行上下文。
当 `first()` 执行完毕，它的执行上下文从栈`弹出`，控制流程到达全局执行上下文。一旦所有代码执行完毕，JavaScript 引擎从当前栈中`移除`全局执行上下文。

有 5 个需要记住的关键点，关于**执行栈（调用栈）**：

- 单线程。
- 同步执行。
- 一个全局上下文。
- 无限制函数上下文。
- 每次函数被调用创建新的执行上下文，包括调用自己。

**一个在线实例**

因为代码太多了这里就不展示了，gif 也比较大，所以只放了一个外国友人的在线实例。有兴趣的可以去看一下。

在线代码体验[执行栈执行过程](https://codepen.io/njmcode/pen/dMPmGq)，如果访问比较慢可以看[执行栈执行过程 demo](https://github.com/Braveheartforyou/Blog-Static/tree/master/callStack)把代码下载到本地运行查看。

## 执行上下文周期

在上面执行栈整个过程中，我们知道每次调用函数时都会创建一个**执行上下文**，现在我们要了解在`JavaScript 引擎`内部是怎么创建**执行上下文**.
在创建完成之后会执行上下文，当执行完成之后会回收当前上下文。

创建执行上下文大致分为三步：

- 创建阶段
- 执行阶段
- 回收阶段

## 创建阶段

**创建阶段**

在每次执行函数时都会走`创建执行上下文`，创建上下文会经历下面这几个事件：

- 创建`作用域`并且形成`作用域链（Scope Chain）`：定义了变量的可访问范围，控制变量的生命周期。
- 创建`变量(variables)`、`函数(functions)`、`参数(arguments)`： 变量和函数在创建时都会存在`Hosting(变量提升)`，就是会提前到最前面声明，但是不赋值。
- 确定`this`的值：`this`的复杂之处就在于它不是声明时就能确定，一般情况来说它是调用时确定的。(箭头函数除外)

我看的有挺多文章的创建的整体顺序是不太一样的，我认为他的顺序是当前我文章中的顺序。会先创建`作用域`再会创建`变量`，最后确定`this`的值。

**伪代码**

```js
    executionContextObj = {
        'scopeChain': {// 作用域链},
        'variables': {// 变量、函数、参数},
        'this': {// 运行时才能确定}
    }
```

到现在我们知道什么是`执行上下文`，我们也知道`this`就是`执行上下文`创建中产生的。但是它的值并不能在创建的时候确定，而是要到调用时才知道。

> 其实作用域的本质是一套规则，它定义了变量的可访问范围，控制变量的可见性和生命周期。

### 创建作用域链

作用域链又可以叫做`词法环境`，[官方的 ES6](http://ecma-international.org/ecma-262/6.0/) 文档把词法环境定义为。

> **词法环境**是一种规范类型，基于 ECMAScript 代码的词法嵌套结构来定义标识符和具体变量和函数的关联。一个词法环境由环境记录器和一个可能的引用外部词法环境的空值组成。

简单来说词法环境是一种持有**标识符—变量映射**的结构。（这里的**标识符**指的是变量/函数的名字，而**变量**是对实际对象[包含函数类型对象]或原始数据的引用）。

词法环境有两种类型：

- `全局环境（在全局执行上下文中）`: 是`没有`外部环境引用的`词法环境`。全局环境的外部环境引用是 `null`。它的`this`的值指向全局对象。
- `函数环境`: 函数内部用户定义的变量存储在`环境记录器`中。并且`引用的外部环境`可能是全局环境，或者任何包含此内部函数的外部函数。

在`函数环境`中，有两个概念来保证它既可以访问管理自己内部的变量等等，又可以访问外部的变量，下面我们就来介绍这两个概念：

- **环境记录器**：是存储变量和函数声明的实际位置。
- **外部环境的引用**：它可以访问其父级词法环境（作用域）。

**环境记录器**又分为两种：

- **声明式环境记录器**：存储变量、函数和参数`（在函数环境中）`。
- **对象环境记录器**：用来定义出现在**全局上下文**中的变量和函数的关系 `(在全局环境中)`。

总结一下`词法环境`分为两种**全局环境**、**函数环境**，`函数环境`又包含两个概念**环境记录器**、**引用的外部环境**，而`环境记录器`又分为两种**声明式环境记录器**、**对象环境记录器**。来张图解释一下吧。

![this-one](../../images/javascript/javascript-this-1-3.png)

**注意**

- 对于`函数环境`，`声明式环境记录器`还包含了一个传递给函数的 `arguments` 对象（此对象存储索引和参数的映射）和传递给函数的参数的 `length`。

```js
    GlobalExectionContext = {
        LexicalEnvironment: {
            EnvironmentRecord: {
            Type: "Object",
            // 在这里绑定标识符
            }
            outer: <null>
        }
    }

    FunctionExectionContext = {
        LexicalEnvironment: {
            EnvironmentRecord: {
            Type: "Declarative",
            // 在这里绑定标识符
            }
            outer: <Global or outer function environment reference>
        }
    }
```

### 创建变量

在`创建变量`之前首先要`创建变量环境`，什么是`变量环境`呢？

`变量环境`也是一个词法环境，它也有`环境记录器`用来记录`变量声明语句`在执行上下文中创建的绑定关系。

`变量环境`和`词法环境`的区别在于`变量环境`被用来`存储函数`声明和`变量（let 和 const）绑定`，而`词法环境`只用来`存储 var 变量绑定`。

```js
let a = 20;
const b = 30;
var c;

function multiply(e, f) {
  var g = 20;
  return e * f * g;
}

c = multiply(20, 30);
```

执行上下文看起来像这样：

```js
    GlobalExectionContext = {

        ThisBinding: <Global Object>,
        LexicalEnvironment: {
            EnvironmentRecord: {
            Type: "Object",
            // 在这里绑定标识符
            a: < uninitialized >,
            b: < uninitialized >,
            multiply: < func >
            }
            outer: <null>
        },
        VariableEnvironment: {
            EnvironmentRecord: {
            Type: "Object",
            // 在这里绑定标识符
            c: undefined,
            }
            outer: <null>
        }
    }

    FunctionExectionContext = {
        ThisBinding: <Global Object>,
        LexicalEnvironment: {
            EnvironmentRecord: {
            Type: "Declarative",
            // 在这里绑定标识符
            Arguments: {0: 20, 1: 30, length: 2},
            },
            outer: <GlobalLexicalEnvironment>
        },
        VariableEnvironment: {
            EnvironmentRecord: {
            Type: "Declarative",
            // 在这里绑定标识符
            g: undefined
            },
            outer: <GlobalLexicalEnvironment>
        }
    }
```

- 只有遇到调用函数 `multiply` 时，`函数执行上下文`才会被创建。

可能你已经注意到 `let` 和 `const` 定义的变量并没有关联任何值，但 `var` 定义的变量被设成了 `undefined`。
这是因为在创建阶段时，引擎检查代码找出`变量和函数声明`，虽然`函数声明完全存储在环境`中，但是变量最初设置为 `undefined`（`var` 情况下），或者未初始化（`let` 和 `const` 情况下）。

这就是为什么你可以在声明之前访问 `var` 定义的变量（虽然是 `undefined`），但是在声明之前访问 `let` 和 `const` 的变量会得到一个引用错误。

### 总结

大致创建过程如下：

- 初始化作用域链：
- 创建变量对象：
  - 创建 arguments 对象，检查上下文，初始化参数名称和值并创建引用的复制。
  - 扫描上下文的函数声明：
    - 为发现的每一个函数，在变量对象上创建一个属性——确切的说是函数的名字——其有一个指向函数在内存中的引用。
    - 如果函数的名字已经存在，引用指针将被重写。
  - 扫面上下文的变量声明：
    - 为发现的每个变量声明，在变量对象上创建一个属性——就是变量的名字，并且将变量的值初始化为 undefined
    - 如果变量的名字已经在变量对象里存在，将不会进行任何操作并继续扫描。
- 求出上下文内部“this”的值。

`词法环境`分为两种**全局环境**、**函数环境**，`函数环境`又包含两个概念**环境记录器**、**引用的外部环境**，而`环境记录器`又分为两种**声明式环境记录器**、**对象环境记录器**。

在上面只是简单的讲解了`变量提升`，如果有兴趣再多了解一下`变量声明提升、函数声明提升`可以去看我的另一篇文章[JavaScript 中的变量提升](/blog/javascript/hoisting.html)，里面又很多比较好的实例。

## 执行阶段

在上面我们介绍了`创建阶段`，现在主要介绍一下`执行阶段`。
**激活/代码执行阶段**

- 在**当前上下文上运行/解释函数代码**，并随着代码一行行执行指派变量的值。

一个**实例**

```js
    let name;
    const age;
    var firstName = 'everybody';
    function test (args) {
        var lastName = 'lastName';
        function testTwo () {
        }
    }
    test('testArgs');
```

`创建阶段`我们用伪代码来表示一下：

```js
    // 全局执行上下文
    GlobalExectionContext = {
        scopeChain: {...}, // 只存在环境记录器（它的记录器叫做对象环境记录器） 不存在外部环境的引用
        variableObject: {
            // name 未初始化
            // age 未初始化
            firstName: undefined
        },
        this: {....}
    }
    // 函数执行上下文
    testExecutionContext = {
        scopeChain: {...}, // 存在环境记录器（它的记录器叫做声明式环境记录器） 不存在外部环境的引用
        variableObject: {
            args: {
                0: 'testArgs',
                length: 1
            },
            args: 'testArgs',
            lastName: undefined,
            testTwo: pointer to function testTwo()
        },
        this: {....}
    }
```

`执行阶段`我们用伪代码描述一下：

```js
    // 全局执行上下文
    GlobalExectionContext = {
        scopeChain: {...}, // 只存在环境记录器（它的记录器叫做对象环境记录器） 不存在外部环境的引用
        variableObject: {
            name: undefined, // 初始化 没有找到变量的值，它被复制为 undefined
            age: undefined, // 初始化 没有找到变量的值，它被复制为 undefined
            firstName: 'everybody' // 赋值为 everybody
        },
        this: {....}
    }
    // 函数执行上下文
    testExecutionContext = {
        scopeChain: {...}, // 存在环境记录器（它的记录器叫做声明式环境记录器） 不存在外部环境的引用
        variableObject: {
            args: {
                0: 'testArgs',
                length: 1
            },
            args: 'testArgs',
            lastName: 'lastName', // 赋值为lastName
            testTwo: pointer to function testTwo()
        },
        this: {....}
    }
```

`let/const`是在`执行阶段`才初始化的，初始化完成就会被赋值。
`arguments`是在`创建阶段`就被创建和赋值。
`functions`也是在`创建阶段`就被创建和赋值。
`variables`是在`创建阶段`被创建，但是没有赋值，在`执行阶段`被赋值。

**functions 赋值式**只会声明不会赋值。

> 如果 `JavaScript 引擎`不能在源码中声明的实际位置找到 `let 变量的值`，它会被赋值为 `undefined`。

## 回收阶段

就是在函数执行完成会把当前的执行上下文回收掉，但是全局执行上下文是不会被回收的，只有关闭当前线程/进程才会把全局执行上下文清楚掉。

## 总结全文

到现在我们知道`this`是`执行上下文`中的一部分，它产生在`创建阶段`(绑定默认值)，`改变this`在发生在`执行阶段`.
我们也知道什么是`执行栈`它是怎么运转的，也知道了`执行上下文`的生命周期的细节，所以本篇文章到此为止。
如果有哪里有问题欢迎留言，谢谢！

## 参考

[What is the Execution Context & Stack in JavaScript?](http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/)
[了解 JavaScript 的执行上下文](https://yanhaijing.com/javascript/2014/04/29/what-is-the-execution-context-in-javascript/)
[[译] 理解 JavaScript 中的执行上下文和执行栈](https://juejin.im/post/5ba32171f265da0ab719a6d7)
