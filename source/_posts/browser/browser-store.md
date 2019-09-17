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

```javascript
  documnet.cookie
  // 这里就不多做赘述，有一篇文章专门讲解了
```

### cookie的特点

**优点**

- 储存用户信息（用户token）
- 标记用户行为（uuid、埋点）

**弊端**

- `Cookie`会被附加在每个`HTTP`请求中，所以无形中增加了流量
- `Cookie`可能被禁用。当用户非常注重个人隐私保护时，他很可能禁用浏览器的`Cookie`功能；
- 由于在`HTTP`请求中的`Cookie`是明文传递的，潜在的安全风险，`Cookie` 可能会被篡改
- `Cookie`数量和长度的限制。每个域名(Domain)下
  **IE6或IE6-(IE6以下版本)：最多20个cookie**
  **IE7或IE7+(IE7以上版本)：最多50个cookie**
  **FF:最多50个cookie**
  **Opera:最多30个cookie**
  **Chrome和safari没有硬性限制**
当超过单个域名限制之后，再设置cookie，浏览器就会清除以前设置的cookie。IE和Opera会清理近期最少使用的cookie，FF会随机清理cookie；

- 每个`Cookie`长度不能超过**4KB**

### cookie安全问题

`Cookie`面临什么样的安全问题，常见的**xss**、**csrf**等等下面开始。

**xss 和 防御xss**

如果在服务器端`Set-Cookie`时没有设置`HttpOnly=true`时，在浏览器端就可以通过`document.cookie`来读取和修改`Cookie`中的值，这是十分安全的会造成`xss`。当`Cookie`中有关键性信息是要设置`HttpOnly=true`。

**防止中间人劫持 和 中间人劫持**

大致的分布图是：

```javascript
         DNS
     <----->
用户          中间人       外网
     <----->
       HTTP
```

当使用`HTTPS`协议和购买正规的`CA证书`时，即使中间人劫持也无法解密。并且在`Set-Cookie`设置`Secure=true`时`Cookie`只应通过被`HTTPS`协议加密过的请求发送给服务端。

**csrf 和 csrf防御**

CSRF: 跨站请求伪造`（CSRF）`是一种冒充受信任用户，向服务器发送非预期请求的攻击方式。

例如，这些非预期请求可能是通过在跳转链接后的 URL 中加入恶意参数来完成:
```html
<img src="https://www.example.com/index.php?action=delete&id=123">
```
对于在 `https://www.example.com` 有权限的用户，这个 `<img>` 标签会在他们根本注意不到的情况下对 `https://www.example.com` 执行这个操作，即使这个标签根本不在 `https://www.example.com` 内亦可。

`SameSite Cookie`允许服务器要求某个`cookie`在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击`（CSRF）`。但目前`SameSite Cookie`还处于实验阶段，并不是所有浏览器都支持。

- `strict`：浏览器在任何跨域请求中都不会携带`Cookie`，这样可以有效的防御`CSRF`攻击，但是对于有多个子域名的网站采用主域名存储用户登录信息的场景，每个子域名都需要用户重新登录，造成用户体验非常的差。
- `lax`：相比较`strict`，它允许从三方网站跳转过来的时候使用`Cookie`。

**其他防御**

- 设置`cookie`有效期时间
- 防止`cookie`是明文，服务器端生成密钥验证
- 生成随机数和`cookie`发送给服务器端
- `flash编程安全`，审核`flash代码`，尽量不要用`flash`用最新的视频`vedio` + `https` + `socket`或者动画

到此`cookie`的**产生原因**、**作用**、**特点/缺点**、**安全问题**。

## Session

**session是什么？**
`Session`是一种记录客户状态的机制，不同于`Cookie`的是`Cookie`保存在客户端浏览器中，而Session保存在服务器上。避免了在客户端`Cookie`中存储敏感数据。

### Session机制

`Session`从字面意思上可以理解为**会话**，谁与谁的会话呢？其实是**客户端浏览器与服务器之间一系列交互的动作称为一个 Session**。

**创建Session（java）**

1. `Session`在服务器端程序运行的过程中创建的，不同语言实现的应用程序有不同创建`Session`的方法， 在`Java`中是通过调用`HttpServletRequest`的`getSession`方法(使用true作为参数)创建的。 创建`Session`的同时，服务器会为该`Session`生成唯一的`session id`， 这个`session id`在随后的请求中会被用来重新获得已经创建的`Session`。
2. `Session`被创建之后，就可以调用`Session`相关的方法往`Session`中增加内容了， 而这些内容只会保存在服务器中，发到客户端的只有`session id`
3. 当客户端再次发送请求的时候，会将这个`session id`带上， 服务器接受到请求之后就会依据`session id`找到相应的`Session`，从而再次使用`Session`。

### Session的生命周期

`Session`保存在服务器端。为了获得更高的存取速度，服务器一般把`Session`放在内存中。 每个用户都会有一个独立的`Session`。 如果`Session`内容过于复杂，当大量客户访问服务器时可能会导致内存溢出。 因此，`Session`里的信息应该尽量精简。

`Session`在用户第一次访问服务器的时候自动创建。 需要注意只有访问`JSP、Servlet`等程序时才会创建`Session`， 只访问`HTML、IMAGE`等静态资源并不会创建`Session`。 如果尚未生成`Session`，也可以使用`request.getSession(true)`强制生成`Session`。

`Session`生成后，只要用户继续访问，服务器就会更新`Session`的最后访问时间，并维护该`Session`。 用户每访问服务器一次，无论是否读写`Session`，服务器都认为该用户的`Session"活跃(active)"了一次`。

### Session的有效期

由于会有越来越多的用户访问服务器，因此`Session`也会越来越多。 为防止内存溢出，服务器会把长时间内没有活跃的`Session`从内存删除。 这个时间就是`Session`的超时时间。如果超过了超时时间没访问过服务器，`Session`就自动失效了。

`Session`的超时时间为`maxInactiveInterval`属性， 可以通过对应的`getMaxInactiveInterval()`获取，通过`setMaxInactiveInterval(longinterval)`修改。

`Session`的超时时间也可以在`web.xml`中修改。 另外，通过调用`Session`的`invalidate()`方法可以使`Session`失效。

三种方法让`Session`失效：

- **服务器意外关闭**。（服务器正常关闭时session是会被服务器保存在服务器的 session.ser 文件中（在work文件夹下））
- `session自杀`： 调用`session.invalidate()`方法可以立即杀死`session`；
- 可以在服务器下的`web.xml`文件中的 `<session-timeout> 30 </session-timeout>` 修改这是默认值(默认30分钟)，是以分为单位。

### 浏览器关闭session会失效?

在几年前看很多网上的资料时有的会说`session`会在浏览器关闭时会失效。为什么会失效？怎么能让它不失效？

**为什么会失效？**

下面梳理一下`session`为什么会在浏览器关闭时失效，其实这样说并不准确：

1. 在服务器端生成`session`，并且把`sessionid`通过`set-cookie`发送给浏览器
2. 以后每次请求除了图片、静态文件请求，其它的请求都会带上**服务端**写入浏览器中`cookie`
3. **服务端**接收到`sessionid`，通过`sessionid`找到对应的`session`信息
4. 当浏览器关闭时，当前域名中设置的`cookie`会被清空
5. 再下次请求使，服务端接收到的`session`为`null`，服务端就会认为当前用户是一个新的用户，重新登录或者直接设置新的`sessionid`

上面也就是为什么会说`session`会在浏览器关闭时会失效。

**怎么能让它不失效？**

在`Set-Cookie`时设置`Expries`或`Max-Age`，其实就是设置`Cookie`的失效时间。
或者直接把`Sessionid`储存在本地。

## web Storage

**Web Storage API**提供机制， 使浏览器能以一种比使用`Cookie`更直观的方式存储键/值对。

Web Storage 包含如下两种机制：

- `sessionStorage` 为每一个给定的源（given origin）维持一个独立的存储区域，该存储区域在页面会话期间可用（即只要浏览器处于打开状态，包括页面重新加载和恢复）。
- `localStorage` 同样的功能，但是在浏览器关闭，然后重新打开后数据仍然存在。

**应注意，无论数据存储在 localStorage 还是 sessionStorage ，它们都特定于页面的协议。**

### localStorage

只读的`localStorage` 属性允许你访问一个`Document` 源（`origin`）的对象 `Storage`；存储的数据将保存在浏览器会话中。存储在 `localStorage` 的数据可以长期保留。

### sessionStorage

`sessionStorage` 属性允许你访问一个 `session Storage` 对象。存储在 `sessionStorage` 里面的数据在页面会话结束时会被清除。页面会话在浏览器打开期间一直保持，并且重新加载或恢复页面仍会保持原来的页面会话。**在新标签或窗口打开一个页面时会在顶级浏览上下文中初始化一个新的会话**，这点和 session cookies 的运行方式不同。

### localStorage和sessionStorage区别

存储在 `localStorage` 的数据可以长期保留，而存储在 `sessionStorage` 里面的数据在页面会话结束时会被清除。

## 总结

`session`和`cookie`的区别：

- `session`储存在服务端，`cookie`储存在客户端
- `session`比`cookie`更安全，因为`session`储存在服务端
- `session`是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中。
- `cookie`是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现`session`的一种方式。

`web storage`和`cookie`的区别：

- `web storages`和`cookie`的作用不同，`web storage`是用于本地大容量存储数据(`web storage`的存储量大到5MB);而`cookie`是用于客户端和服务端间的信息传递；
- `web storage`有`setItem`、`getItem`、`removeItem`、`clear`等方法，`cookie`需要我们自己来封装`setCookie`、`getCookie`、`removeCookie`

## 参考

[HTTP cookies](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)
[Web Storage API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Storage_API)
[Window.localStorage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/localStorage)
[Window.sessionStorage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/sessionStorage)
[这一次把cookie给你说透彻！](https://mp.weixin.qq.com/s/RJuHkTYd7pSS_RhHeuh0iQ)
[深入理解Session和Cookie的区别](https://mp.weixin.qq.com/s/sycJeIBY_2h0h6SwUTWI9A)
[详解cookie和session的运作机制（上篇）](https://mp.weixin.qq.com/s/VeodMTKbwCdodx0fZjTlhw)
[面试稳了！这才是cookie，session与token的真正区别](https://mp.weixin.qq.com/s/Idl0eheciAck5WznXqO0Lw)
