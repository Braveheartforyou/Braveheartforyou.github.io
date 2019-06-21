---
title: 利用ETag和If-None-Match字段验证缓存的响应
date: 2019-01-15 22:29:16
tags: [Http]
categories: [Http]
description: 利用http协议的 首部字段 Etag 和 If-None-Match字段来做接口缓存
---
## 简述
首部字段ETag能告知客户实体标识，它是一种可将资源以字符串形式做唯一性标识的方式。服务器会为每份资源分配对应的ETag值。
## 强ETag值和弱ETag值
Etag中有强ETag值和弱ETag值之分。
### 强ETag值
强ETag值，不论实体发生多么细微的变化都会改变其值。
``` http
ETag: "usagi-1234"
```
### 弱ETag值
弱ETag值只用于提示资源是否相同。只有资源发生了根本改变，产生差异时才会改变ETag值。这时，会在字段值最开始处附加 W/
``` http
ETag: "W/usagi-1234"
```
## 实现
1. <font color="red">服务器端</font>会给每份资源分配对应的<font color="red">ETag值</font>。在<font color="red">response</font>中的头部返回给<font color="red">客户端</font>.
``` http
etag: "123456"
```
2. <font color="red">第二次客户端</font>请求的时候在<font color="red">request首部字段</font><font color="red">If-None-Match</font>中传递刚才服务端发送到客户端的<font color="red">ETag值</font>。  
``` http
if-none-match: "123456"
```
3. 服务器端会拿客户端if-none-match传递的值和ETag的值比对，如果相同，就会把if-none-match的值修改为false,服务器将返回“304 Not Modified”响应。