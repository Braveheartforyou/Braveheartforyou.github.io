---
title: cookie、localStorage、sessionStorage、session深入理解区别
date: 2018-09-13 14:32:09
tags: [JavaScript, Browser]
categories: [Browser]
description: 浏览器中的cookie、localStorage、sessionStorage、session它们有很多的异同点，比如cookie、session是以前的通用的解决方案，有了以前的方案为什么还要引入新的解决方案localStorage、sessionStorage。
---

## 简介

在以前经常用到的`cookie`、`session`，但在**H5**中新引入了新的浏览器本地缓存方案。因为大家使用的不太规范用来作为本地储存工具，在下次请求时会默认带上`cookie`中的数据导致浪费性能和流量。

下面就从开始介绍为什么产生的`cookie`，它的出现是为了解决什么问题，它有什么问题；后面`localStorage`是为什么产生，它又解决了那部分的问题。最后是`cookie`、`session`、`localStorage`、`sessionStorage`之间的对比。

## cookie

首先了解为什么会产生`cookie`?`cookie`是什么?

### cookie是什么、cookie产生原因

在`HTTP`请求建立连接时，有一个**客户端**、**服务端**它们两个之间建立的连接。但是`HTTP`协议每次建立连接都是独立，也可以说是`HTTP`协议是无状态的连接。

- 第一次建立连接，客户在**客户端**中登录，**服务端**验证登录信息，生成**Token**为以后的请求不需要从新登录
- 第二次建立连接，**客户端**携带服务端在登录成功时返回的**Token**，但是这个**Token**要储存在哪里？一般会存在**cookie**里。

简单总结一下就是，因为`HTTP`协议是无状态的所以客户端需要一个`cookie`来储存起来。
<!-- **`Cookie`实际上是一小段的文本信息**，**用来服务端和客户端之间传递信息**。 -->
**`HTTP Cookie`（也叫Web Cookie或浏览器Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。**

`Cookie`主要用于以下三个方面：

- 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
- 个性化设置（如用户自定义设置、主题等）
- 浏览器行为跟踪（如跟踪分析用户行为等）

### 创建Cookie

当服务器收到`HTTP`请求时，服务器可以在响应头里面添加一个`Set-Cookie`选项。浏览器收到响应后通常会保存下`Cookie`，之后对该服务器每一次请求中都通过`Cookie`请求头部将`Cookie`信息发送给服务器。

在创建`Cookie`是可以设置很多属性，如`Expires`、`Max-Age`、`Domain `、`Path`、`Secure `、`HttpOnly`，因为它会自动携带到服务器端，同时又支持服务器端设置。所以有很多的方面要注意，比如**时效性**、**作用域**、**安全性**。下面就从这三个方面来解释他属性的作用。

### 时效性

如果在`Set-Cookie`时不通过`Expries`、`Max-Age`两个字段设置`Cookie`的时效性，那么这个`Cookie`是一个简单的**会话期Cookie**。它在关闭浏览器是会被自动删除。

如果设置了`Expries`、`Max-Age`那么这个`Cookie`在指定时间内都是有效的。

> 提示：当Cookie的过期时间被设定时，设定的日期和时间只与客户端相关，而不是服务端。

### 作用域

`Domain` 和 `Path` 标识定义了`Cookie`的作用域：即`Cookie`应该发送给哪些`URL`。

`Domain` 标识指定了哪些主机可以接受`Cookie`。如果不指定，默认为**当前文档的主机（不包含子域名）**。如果指定了`Domain`，则一般包含子域名。

`Path` 标识指定了主机下的哪些路径可以接受`Cookie`**（该URL路径必须存在于请求URL中）**。以字符 %x2F ("/") 作为路径分隔符，子路径也会被匹配。

### 安全性

标记为 `Secure` 的`Cookie`只应通过被`HTTPS`协议加密过的请求发送给服务端。但即便设置了 `Secure` 标记，敏感信息也不应该通过`Cookie`传输，因为`Cookie`有其固有的不安全性，`Secure` 标记也无法提供确实的安全保障。

> 从 Chrome 52 和 Firefox 52 开始，不安全的站点（http:）无法使用Cookie的 Secure 标记。

为避免跨域脚本 **(XSS)** 攻击，通过**JavaScript**的 `Document.cookie` **API**无法访问带有 `HttpOnly` 标记的`Cookie`，它们只应该发送给服务端。

### 客户端操作Cookie

通过`Document.cookie`属性可创建新的`Cookie`，也可通过该属性访问非`HttpOnly`标记的`Cookie`。

