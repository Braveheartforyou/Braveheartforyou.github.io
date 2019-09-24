---
title: Http中的缓存（二） ETag和If-None-Match字段验证
date: 2019-01-15 22:29:16
tags: [Http]
categories: [Http]
description: 利用http协议的 首部字段 Etag 和 If-None-Match字段来做接口缓存
---

> [Http 中的缓存（一） 强缓存、协商缓存](/blog/http/http-cache.html) > [Http 中的缓存（二） ETag 和 If-None-Match 字段验证](/blog/http/http-etag-cache.html) > [Http 中的缓存（三） PWA 中的 serviceworker](/blog/http/http-cache-serviceworker.html)

## 简述

首部字段 ETag 能告知客户实体标识，它是一种可将资源以字符串形式做唯一性标识的方式。服务器会为每份资源分配对应的 ETag 值。

## 强 ETag 值和弱 ETag 值

Etag 中有强 ETag 值和弱 ETag 值之分。

### 强 ETag 值

强 ETag 值，不论实体发生多么细微的变化都会改变其值。

```http
ETag: "usagi-1234"
```

### 弱 ETag 值

弱 ETag 值只用于提示资源是否相同。只有资源发生了根本改变，产生差异时才会改变 ETag 值。这时，会在字段值最开始处附加 W/

```http
ETag: "W/usagi-1234"
```

## 实现

1. <font color="#ff502c">服务器端</font>会给每份资源分配对应的<font color="#ff502c">ETag 值</font>。在<font color="#ff502c">response</font>中的头部返回给<font color="#ff502c">客户端</font>.

```http
etag: "123456"
```

2. <font color="#ff502c">第二次客户端</font>请求的时候在<font color="#ff502c">request 首部字段</font><font color="#ff502c">If-None-Match</font>中传递刚才服务端发送到客户端的<font color="#ff502c">ETag 值</font>。

```http
if-none-match: "123456"
```

3. 服务器端会拿客户端 if-none-match 传递的值和 ETag 的值比对，如果相同，就会把 if-none-match 的值修改为 false,服务器将返回“304 Not Modified”响应。
