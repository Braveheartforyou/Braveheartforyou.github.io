---
title: Http中的缓存（三） PWA中的serviceworker
date: 2019-02-07 18:43:53
tags: [Http]
categories: [Http]
description: ServiceWorker其实很早就已经出来了，但是因为浏览器的差别
---

> [Http中的缓存（一） 多级缓存结构](/blog/http-cache-multiple.html)
> [Http中的缓存（二） HTTP中的缓存](/blog/http/http-cache-http.html)
> [Http中的缓存（三） PWA中的ServiceWorker](/blog/http/http-cache-serviceworker.html)

## 简介

首先了解一下`PWA（Progressive web apps，渐进式 Web 应用）`运用现代的 `Web API` 以及传统的渐进式增强策略来创建跨平台 `Web` 应用程序。
**PWA的优点**
`PWA` 是**可被发现**、**易安装**、**可链接**、**独立于网络**、**渐进式**、**可重用**、**响应性**和**安全**的。
`PWA`中可以通过`Service Worker`来实现离线的应用，这个也是PWA中一个比较重要的环节，它们主要应用到**Web App**中，已获得更好的体验，并且在现在也在大规模的应用。
`Service Worker`是一个事件驱动`worker`，运行在一个单独的后台进程，是`PWA（ProgressiveWeb App）`运行的基础。主要用于代理网页请求，可缓存请求结果；可实现离线缓存功能，也拥有单独的作用域范围和运行环境。我们以后把`Service Worker`简称为`SW`。

本文大致分为：

- SW特性
- SW生命周期和使用
- SW中的缓存策略
- SW中的消息推送
- SW一些注意事项

## SW的特性

它们的运行在一个与我们页面的 `JavaScript 主线程独立的线程上`，并且`没有对 DOM 结构`的任何访问权限。
这引入了与传统 Web 编程不同的方法 - `API 是非阻塞的`，并且可以在不同的`上下文之间发送和接收信息`。

### SW使用限制

`SW`除了`work`线程的限制外，由于可拦截页面请求，为了保证页面安全，浏览器端对`sw`的使用限制也不少。

- **无法直接操作DOM对象**，也无法访问`window`、`document`、`parent`对象。可以访问`navigator`、`location`； `SW` 通过响应 `postMessage` 接口发送的消息来与其控制的页面通信，页面可在必要时对 `DOM` 执行操作。
- **可代理的页面作用域限制**。默认是`sw.js`所在**文件目录及子目录的请求可代理**，可在注册时手动设置作用域范围；
- **必须**在 `https` 中使用，允许在开发调试的`localhost`使用。

### SW主要作用

- **可以用来做缓存，以达到提升体验、节省浏览等等**
- **SW 是一种可编程网络代理，让您能够控制页面所发送网络请求的处理方式。**
- **离线缓存接口请求及文件，更新、清除缓存内容；**
- **可分配给 Service Worker 一些任务，并在使用基于 Promise 的方法当任务完成时收到结果。**
- **Service Worker处于空闲状态会被终止，在下一次需要时重启。**

### SW兼容性

可以通过查询[service worker](https://caniuse.com/#search=service%20worker)可以看到他在不同平台或不同浏览器中的兼容性。

## SW生命周期和使用

`SW` 的生命周期完全独立于网页。
`SW` 为网页添加一个类似于 `App` 的生命周期，它只会**响应系统事件**，就算**浏览器关闭**时操作系统也可以唤醒 `SW`，这点非常重要，让`Web App`与 `Native App` 的能力变得类似了。由于是离线缓存，所以在**初始安装时**、**更新**它们的所走的生命周期是不相同。下面我们就根据这**两种场景结合代码来分析**它的执行步骤。
`SW`的生命周期大致分为：**注册**、**更新**、**安装成功**、**安装失败**、**激活**、**销毁**。
<!-- `SW`的事件： `install`、`activate`、`message`、`fetch`、`push`、`async`。 -->

**使用SW前提条件**

- **必须**在 `https` 中使用，允许在开发调试的`localhost`使用。
- 浏览器必须支持SW

## 初始安装时

初始安装时大致流程大致如下图：
![http-cache-serviceworker](../../images/http/http-cache-4-1.png)

大致可以分为`注册SW => 安装SW => 激活 => 空闲 => (缓存和返回请求/终止)`，在初始安装时会大致分为这几个步骤，下面就按照这几个步结合代码实现。

### 注册 Service Worker

用户**首次访问**SW控制的网站或页面时，`sw.js`会立刻被下载和解析。我们要在页面中写入`JavaScript`来注册`SW`。

```javascript
    // 判断浏览器是否支持serviceWorker
    if ('serviceWorker' in navigator) {
        // 在页面加载后
        window.addEventListener('load', function () {
            // 通过navigator.serviceWorker.register 注册'./sw.js
            navigator.serviceWorker.register('./sw.js')
            .then(reg => { //注册成功
                console.log('注册成功', reg)
            }).catch(err => { //注册成功
                console.log('注册失败', err)
            })
        });
    } else {
        console.log('当前浏览器不支持SW')
    }
```

首先检浏览器是否支持SW，如果支持就在浏览器加载后通过`register().then`注册**sw.js**，并且设置注册成功或者失败的回调函数。

> 注意：`register()` 方法的精妙之处在于服务工作线程文件的位置。`SW`降接收此网域上所有的事项的`featch`事件。
如果是Chrome浏览器可以通过`chrome://inspect/#service-workers`或者`console => application => Service Worker`查看是否注册成功

因为现在`sw.js`中我们的代码是空的，所以在浏览器中的`cache stoage`是空的，运行效果如下：

<img src="../../images/http/http-cache-4-4.png" />

### 安装 Service Worker

在受控页面启动注册流程后，下面就是SW获取的第一个事件`install`，并且只发生一次。传递到 `installEvent.waitUntil()` 的一个 `promise` 可表明安装的持续时间以及安装是否成功。

在`install`中要做三件事**打开缓存**、**缓存文件**、**确认所有需要的资产是否已缓存**。

```javascript
    // 在sw.js中监听对应的安装事件，并预处理需要缓存的文件
    // 该部分内容涉及到cacheStorage API

    // 定义缓存空间名称
    const CACHE_NAME = 'sw_cache_v1';
    // 定义需要缓存的文件目录
    let cachelist = ['./app.js', './index.css'];
    // 监听安装事件，返回installEvent对象
    self.addEventListener('install', function (installEvent) {
        // waitUntil方法执行缓存方法
        installEvent.waitUntil(
            // cacheStorage API 可直接用caches来替代
            // open方法创建/打开缓存空间，并会返回promise实例
            // then来接收返回的cache对象索引
            caches.open(CACHE_NAME)
             // cache对象addAll方法解析（同fetch）并缓存所有的文件
            .then(function(cache) {
                console.log('Opened cache');
                return cache.addAll(cachelist);
            })
        );
    });

```

第一个事件为`install`，该事件在`Worker`执行时立即触发。在`install`的回调函数中，我们通过`caches.open(CACHE_NAME)`打开缓存，之后调用`cache.addAll()`并传入路径数组。这是一个`promise`链（`caches.open()`和`chaches.addAll()`。`installEvent.waitUntill()`放大带有`promise`并使用它来判断安装所花时间，以及是否安装成功。

> 注意： 第一个事件`install`，它只能被每个 **SW** 调用一次。如果您更改您的 **SW** 脚本，则浏览器将其视为一个**不同**的 **SW**，并且它将获得自己的 `install` 事件。
如有**任何文件无法下载**，则安装步骤将失败。
当前的状态是在等待状态。
我们可以直接通过`self.skipwaiting()`让当前`sw`立即将状态提升到`active`。

当我们安装成功时，效果如下图所示：

![http-cache-serviceworker](../../images/http/http-cache-4-5.png)

会多了一个`skipWaiting`，还有在`cache stroage`中的当前域名下的`service worker`对应的缓存文件列表。这个时候我们即使刷新也不会走`service worker`的缓存的。

### 激活

`SW` 准备控制客户端并处理 `push` 和 `sync` 等功能事件时，您将获得一个 `activate` 事件。但这不意味着调用 `.register()` 的页面将受控制。如果**第二次加载此演示（换言之，刷新页面）**，该页面将受控制。改写代码`sw.js`如下：

```javascript
    self.addEventListener('install', () => {
        // 一般注册以后，激活需要等到再次刷新页面后再激活
        // 可防止出现等待的情况，这意味着服务工作线程在安装完后立即激活
        self.skipWaiting();
    })
    self.addEventListener('activate', function (event) {
        event.waitUntil(
            // cacheStorage API 可直接用caches来替代
            // open方法创建/打开缓存空间，并会返回promise实例
            // then来接收返回的cache对象索引
            caches.open(CACHE_NAME)
            // cache对象addAll方法解析（同fetch）并缓存所有的文件
            .then(function(cache) {
                console.log('Opened cache');
                return cache.addAll(cachelist);
            })
        );
    })
```

一般在书写的时候，会在`install()`注册之后直接通过`self.skipWaiting();`激活当前的`SW`，在`activate`中书写**打开缓存等等**的逻辑，就不会出现上面还要**刷新或者手动激活**的问题。效果图如下：

![http-cache-serviceworker](../../images/http/http-cache-4-6.png)

但是如果出现**更新SW**，并且**更新了缓存列表**或者**出现异步资源时**，我们可以通过`clients.claim()`更新缓存列表。

### clients.claim

激活 `SW` 后，您可以通过在其中调用 `clients.claim()` 控制未受控制的客户端。[google developer](https://cdn.rawgit.com/jakearchibald/80368b84ac1ae8e229fc90b3fe826301/raw/df4cae41fa658c4ec1fa7b0d2de05f8ba6d43c94/)中的一个异步加载图片的实例。下面修改代码如下：

```javascript
    self.addEventListener('install', (event) => {
        event.waitUntil(
            // cacheStorage API 可直接用caches来替代
            // open方法创建/打开缓存空间，并会返回promise实例
            // then来接收返回的cache对象索引
            caches.open(CACHE_NAME)
            // cache对象addAll方法解析（同fetch）并缓存所有的文件
            .then(function(cache) {
                console.log('Opened cache');
                return cache.addAll(cachelist);
            })
        );
        // 一般注册以后，激活需要等到再次刷新页面后再激活
        // 可防止出现等待的情况，这意味着服务工作线程在安装完后立即激活
        self.skipWaiting();
    })
    self.addEventListener('activate', function (event) {
        // 若缓存数据更改，则在这里更新缓存
        var cacheDeletePromise = caches.keys()
        .then(keyList => {
            Promise.all(keyList.map(key => {
                if (key !== CACHE_NAME) {
                    var deletePromise = caches.delete(key)
                    return deletePromise
                } else {
                    Promise.resolve()
                }
            }));
        });
        event.waitUntil(
            Promise.all([cacheDeletePromise]).then(res => {
                this.clients.claim()
            })
        );
    })
```

用于处理更新缓存，新的文件。到现在我们还是没有用到`serviceWorker`的缓存，下面重头戏来了**缓存和返回请求**。

### 缓存和返回请求

上我们已经安装并且激活了**SW**，现在我们要返回一个缓存的响应。**SW**用户转至其他页面或刷新当前页面后，将开始接受`fetch`事件。

```javascript
    self.addEventListener('fetch', function(event) {
        event.respondWith(
            caches.match(event.request)
            .then(function(response) {
                // Cache hit - return response
                if (response) {
                    return response;
                }
                return fetch(event.request);
            })
        );
    });
```

在定义的`fetch`事件中，我们在`event.respondWith()`中传入来自`caches.match()`的一个`promise`。`caches.match()`这个方法检视该请求，并从服务工作线程所创建的任何缓存中查找缓存的结果。如果命中返回缓存值，否则，将调用`fetch`以发出网络请求。运行效果如下：

![http-cache-serviceworker](../../images/http/http-cache-4-7.png)

如果我们想把新的请求也缓存掉，修改代码如下：

```javascript
    self.addEventListener('fetch', function(event) {
        event.respondWith(
            caches.match(event.request)
            .then(function(response) {
                // Cache hit - return response
                if (response) {
                    return response;
                }
                // return fetch(event.request);
                var requestClone = event.request.clone();
                return fetch(requestClone).then(response => {
                    if(!response || response.status !== 200 || response.type !== 'basic') {
                        return response;
                    }
                    var responseToCache = response.clone();
                    caches.open(CACHE_NAME)
                    .then(function(cache) {
                        cache.put(event.request, responseToCache);
                    });
                    return response;
                });
            })
        );
    });
```

执行操作如下：

1. 在 `fetch` 请求中添加对 `.then()` 的回调。
2. 获得响应后，**确保响应有效。**、**检查并确保响应的状态为 200。**、**确保响应类型为 basic，亦即由自身发起的请求。 这意味着，对第三方资产的请求也不会添加到缓存。**
3. 如果**通过检查**，则克隆响应。


没有缓存新请求时效果如下：
![http-cache-serviceworker](../../images/http/http-cache-4-8.png)

当使用我们下面的代码时，效果图如下：
![http-cache-serviceworker](../../images/http/http-cache-4-9.png)
即使异步请求的`png`图片也被加入了缓存中。

## 更新SW

在以下情况下会触发更新：

- 导航到一个作用域内的页面。
- 更新 `push` 和 `sync` 等功能事件，除非在前 24 小时内已进行更新检查。
- 调用 `.register()`，仅在 `SW` 网址已发生变化时。

当触发更新时，会经过大致如下步骤：

1. 更新您的**服务工作线程** `JavaScript` 文件。 用户导航至您的站点时，浏览器会尝试在**后台重新下载**定义 `SW` 的脚本文件。 如果 `SW` 文件与其当前所用文件存在字节**差异**，则将其视为新 `SW`。
2. **更新**的 `SW` 与**现有** `SW` 一起启动，并获取自己的 `install` 事件。
3. 此时，**旧** `SW` 仍**控制着**当前页面，因此**新** `SW` 将进入 `waiting` 状态。
4. 如果`新 Worker` 出现**不正常状态代码**（例如，404）、解析失败，在执行中**引发错误**或**在安装期间被拒**，则系统`将舍弃新 Worker`，但`当前 Worker 仍处于活动状态`。
5. 安装成功后，`更新的 Worker 将 wait`，直到`现有 Worker` 控制零个客户端。（注意，在刷新期间客户端会重叠。）
6. 
