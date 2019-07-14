---
title: es6中的async方法的使用和实现原理
date: 2017-08-01 15:56:26
tags: [ECMAScript6]
categories: [ECMAScript6]
description: JavaScript中异步函数实际上都是一个 AsyncFunction  对象。
---
## 基本用法
<font color="#ff502c">AsyncFunction</font> 构造函数 创建一个新的  async function 对象。在JavaScript中，每个异步函数实际上都是一个 AsyncFunction  对象。
<font color="#ff502c">async function</font> 关键字可以用来定义一个异步函数表达式。<font color="#ff502c">返回一个 Promise</font>
它就是 <font color="#ff502c">Generator</font> 函数的语法糖。
```javascript
    function resolveAfter2Seconds(x) {
    return new Promise(resolve => {
        setTimeout(() => {
        resolve(x);
        }, 2000);
    });
    };

    var add1 = async function(x) {
    var a = resolveAfter2Seconds(20);
    var b = resolveAfter2Seconds(30);
    return x + await a + await b;
    }

    add1(10).then(v => {
    console.log(v);  // prints 60 after 2 seconds.
    });

    var add2 = async function(x) {
    var a = await resolveAfter2Seconds(20);
    var b = await resolveAfter2Seconds(30);
    return x + a + b;
    };

    add2(10).then(v => {
    console.log(v);  // prints 60 after 4 seconds.
    });
```
### 语法
```javascript
    async function [name]([param1[, param2[, ..., paramN]]]) {
        statements
    }
    // 参数
    // name
        // 函数名称。 可以省略，以此来声明一个匿名的函数。也是用于本地调用函数体的一个名称

    // paramN
        // 传入函数的参数名
    // statements
        // 函数体内的语句声明
```
#### 返回 Promise 对象
async函数返回一个 Promise 对象。
async函数内部return语句返回的值，会成为then方法回调函数的参数。
```javascript
    async function f() {
        return 'hello world';
    }
    f().then(v => console.log(v))
    // "hello world"
```
### 描述
async function 表达式非常类似于 async function 声明语句，并且几乎拥有等同的语法。他们之间主要的区别在于函数名称，async function表达式可以省略函数名称来创建一个匿名的函数。另见 functions 章节获取更多信息。

### async函数对 Generator 函数的改进，体现在以下四点。
#### 内置执行器。
Generator函数的执行必须靠执行器，所以才有了<font color="#ff502c">co</font>模块,而<font color="#ff502c">async</font>函数自带执行器，也就是说，<font color="#ff502c">async</font>函数的执行，与普通函数一模一样，只要一行。
```javascript
    var asyncReadFile = async function () {
        var f1 = await readFile('/etc/fstab');
        var f2 = await readFile('/etc/shells');
        console.log(f1.toString());
        console.log(f2.toString());
    };
    var result = asyncReadFile();
```
#### 更好的语义
<font color="#ff502c">async</font>和<font color="#ff502c">await</font>，比起<font color="#ff502c">*</font>和<font color="#ff502c">yield</font>，语义更清楚了。async表示函数里有异步操作，await表示紧跟在后面的表达式需要等待结果。
#### 返回值是 Promise。
<font color="#ff502c">async</font>函数的返回值是 <font color="#ff502c">Promise</font>对象，这比 <font color="#ff502c">Generator</font>函数的返回值是 <font color="#ff502c">Iterator</font>对象方便多了。你可以用<font color="#ff502c">then</font>方法指定下一步的操作。  
进一步说，<font color="#ff502c">async</font>函数完全可以看作多个异步操作，包装成的一个<font color="#ff502c">Promise</font> 对象，而<font color="#ff502c">await</font>命令就是内部<font color="#ff502c">then</font>命令的语法糖。
```javascript
    function timeout(ms) {
        return new Promise((resolve) => {
            setTimeout(resolve, ms);
        })
    }

    async function asyncPrint(value, ms) {
        await timeout(ms);
        console.log(value);
    }
    asyncPrint('hello wrold', 50);
```
## await
<font color="#ff502c">await</font>  操作符被用于等待由一个async function返回的一个Promise。如果不是，会被转成一个立即resolve的 Promise 对象。
### 描述
await 表达式会造成异步函数停止执行并且等待 promise 的解决，当值被 resolved，异步函数会恢复执行以及返回 resolved 值。如果该值不是一个 promise，它将会被转换成一个 resolved 后的 promise。
```javascript
    async function f() {
        return await 123;
    }
    f().then(v => console.log(v))
    // 123
```
await命令后面的 Promise 对象如果变为reject状态，则reject的参数会被catch方法的回调函数接收到。那么整个async函数都会中断执行。
```javascript
    async function f() {
        // await Promise.reject('出错了');
       return await Promise.resolve('hello world'); // 不会执行
    }
    f().then(v => console.log(v))
    .catch(e => console.log(e))
    // 出错了
```
### 使用注意点
- 第一点，前面已经说过，await命令后面的Promise对象，运行结果可能是rejected，所以最好把await命令放在try...catch代码块中。
```javascript
    async function myFunction() {
        try {
            await somethingThatReturnsAPromise();
        } catch (err) {
            console.log(err);
        }
    }

    // 另一种写法

    async function myFunction() {
        await somethingThatReturnsAPromise()
        .catch(function (err) {
            console.log(err);
        });
    }
```
- 第二点，多个await命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。因为只有getFoo完成以后，才会执行getBar,完全可以让他们同事触发。
```javascript
    // 写法一
    let [foo, bar] = await Promise.all([getFoo(), getBar()]);

    // 写法二
    let fooPromise = getFoo();
    let barPromise = getBar();
    let foo = await fooPromise;
    let bar = await barPromise;
```
- 第三点，await命令只能用在async函数之中，如果用在普通函数，就会报错。 正确的写法是采用for循环。
```javascript
    async function dbFuc(db) {
        let docs = [{}, {}, {}];
        // 报错
        docs.forEach(function (doc) {
            await db.post(doc);
        });
    }

    async function dbFuc(db) {
        let docs = [{}, {}, {}];

        for (let doc of docs) {
            await db.post(doc);
        }
    }
```