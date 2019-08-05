---
title: webSocket(二) 短轮询、长轮询、Websocket、sse
date: 2019-03-17 13:18:43
tags: [Http, Socket]
categories: [Http]
description: webSocket(二) Http中的轮询、长轮询对比
---
> [webSocket(一) 浅析](/blog/http/http-web-socket1.html)
> [webSocket(二) 短轮询、长轮询、Websocket、sse](/blog/http/http-web-socket2.html)

## 简介
Web Sockets定义了一种在通过一个单一的 socket 在网络上进行**全双工通讯的通道**。仅仅是传统的 HTTP 通讯的一个增量的提高，尤其对于**实时、事件驱动**的应用来说是一个飞跃。
通过<font color="#ff502c">Polling(轮询)</font>、<font color="#ff502c">Long-Polling(长轮询)</font>、<font color="#ff502c">Websocket</font>、<font color="#ff502c">sse</font>的对比。其实它们都是服务器推送中的一种，这里就对比它们之间优缺点。
对比优缺点如下：

| # |      短轮询(Polling)      |  长轮询(Long-Polling) | Websocket | sse |
|:----------:|:-------------:|:------:|:------:|:------:|
| 通信协议 |  http | http | tcp | http |
| 触发方式 |  http | http | tcp | http |
