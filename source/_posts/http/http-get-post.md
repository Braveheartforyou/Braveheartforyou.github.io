---
title: Http系列(四) Http中Get/Post的区别
date: 2019-02-22 22:59:35
tags: [Http]
categories: [Http]
description: 在http协议中get请求和post请求的区别
---

***为无为，事无事，味无味。——老子***

> [Http系列(-) Http发展历史](/blog/http/http-http2.html)
> [Http系列(二) Http2中的多路复用](/blog/http/http-http2-1.html)
> [Http系列(三) Http/Tcp三次握手和四次挥手](/blog/http/http-tcp.html)
> [Http系列(四) Http中Get/Post的区别](/blog/http/http-get-post.html)

## 简介

在客户机和服务器之间进行请求-响应时，两种最常被用到的方法是：`GET` 和 `POST`。

- **GET - 从指定的资源请求数据。**
- **POST - 向指定的资源提交要被处理的数据**

最直观的区别就是`GET`把参数包含在`URL`中，`POST`通过`request body`传递参数。`POST`和`GET`大致区别如下表所示：

| # |      GET      |  POST |
|:----------:|:-------------:|:------:|
| 后退按钮/刷新 |  无害 | 数据会被重新提交（浏览器应该告知用户数据会被重新提交） |
| 书签 |   可收藏为书签   | 不可收藏为书签 |
| 缓存 | 能被缓存 | 不能缓存 |
| 编码类型 | application/x-www-form-urlencoded | application/x-www-form-urlencoded 或 multipart/form-data。为二进制数据使用多重编码 |
| 历史 | 参数保留在浏览器历史中 | 参数不会保存在浏览器历史中 |
| 对数据长度的限制 | 当发送数据时，GET 方法向 URL 添加数据；URL 的长度是受限制的（URL 的最大长度是 2048 个字符） | 无限制 |
| 对数据类型的限制 | 只允许 ASCII 字符 | 没有限制。也允许二进制数据 |
| 安全性 | 与 POST 相比，GET 的安全性较差，因为所发送的数据是 URL 的一部分 在发送密码或其他敏感信息时绝不要使用 GET ！ | POST 比 GET 更安全，因为参数不会被保存在浏览器历史或 web 服务器日志中 |
| 可见性 | 数据在 URL 中对所有人都是可见的 | 数据不会显示在 URL 中 |

## 区别

---

**其实`GET`和`POST`都是`http协议中`的两种发送请求的方法。**

- **GET在浏览器回退时是无害的，而POST会再次提交请求。**
- **GET产生的URL地址可以被Bookmark，而POST不可以。**
- **GET请求会被浏览器主动cache，而POST不会，除非手动设置。**
- **GET请求只能进行url编码，而POST支持多种编码方式。**
- **GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。**
- **GET请求在URL中传送的参数是有长度限制的，而POST么有。**
- **对参数的数据类型，GET只接受ASCII字符，而POST没有限制。**
- **GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。**
- **GET参数通过URL传递，POST放在Request body中。**
- **http协议并未规定get和post的长度限制**
- **get的最大长度限制是因为浏览器和web服务器限制了URL的长度**
- **对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）；而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）**

### 其实很多人说get请求比post请求快，主要是底下两条

1. **get请求比post请求少一步**
2. **get请求可以缓存**

**get请求过程**：

1. **浏览器请求tcp连接（第一次握手）**
2. **服务器答应进行tcp连接（第二次握手）**
3. 浏览器确认，**并发送get请求头和数据**（第三次握手，这个报文比较小，所以http会在此时进行第一次数据发送）
4. 服务器返回200 OK响应

**post请求过程**：

1. **浏览器请求tcp连接（第一次握手）**
2. **服务器答应进行tcp连接（第二次握手）**
3. **浏览器确认，并发送post请求头**（第三次握手，这个报文比较小，所以http会在此时进行第一次数据发送）
4. **服务器返回100 Continue响应**
5. **浏览器发送数据**
6. 服务器返回200 OK响应

## 总结

- `GET`在浏览器回退时是**无害**的，而`POST`会**再次**提交请求。
- `GET`产生的URL地址可以被`Bookmark`，而`POST`不可以。
- `GET`请求会被浏览器主动`cache`，而`POST`不会，除非手动设置。
- `GET`请求只能进行`url`编码，而`POST`支持多种编码方式。
- `GET`请求参数会被完整保留在浏览器历史记录里，而`POST`中的参数不会被保留。
- `GET`请求在`URL`中传送的参数是有长度限制的，而`POST`么有。
- 对参数的数据类型，`GET`只接受`ASCII`字符，而`POST`没有限制。
- `GET`比`POST`更不安全，因为参数直接暴露在`URL`上，所以不能用来传递敏感信息。
- `GE`T参数通过`URL`传递，`POST`放在`Request body`中。
- `http`协议并未规定`get`和`post`的长度限制
- `get`的最大长度限制是因为浏览器和`web`服务器限制了`URL`的长度
- 对于`GET`方式的请求，浏览器会把`http header`和`data`一并发送出去，服务器响应`200`（返回数据）；而对于POST，浏览器先发送`header`，服务器响应`100 continue`，浏览器再发送`data`，服务器响应`200 ok`（返回数据）

## 参考

> [HTTP 方法：GET 对比 POST](https://www.w3school.com.cn/tags/html_ref_httpmethods.asp)
> [GET和POST两种基本请求方法的区别](https://www.cnblogs.com/logsharing/p/8448446.html#!comments)
