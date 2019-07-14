---
title: cookie_function
date: 2017-08-31 11:33:31
tags: [JavaScript]
categories: [JavaScript]
---
<font color="#ff502c"></font>
## 概述

### Cookie在客户端JavaScript中
用JavaScript操作<font color="#ff502c">Cookie</font>，用原声的接口<font color="#ff502c">document.cookie</font>属性是比较难用的，所以封装一个添加、修改、删除的操作方法还是很有必要的。
同时要注意的是： Cookie的所有<font color="#ff502c">name</font>和<font color="#ff502c">value</font>都是要经过<font color="#ff502c">URI</font>编码的，必须使用<font color="#ff502c">decodeURICompoent()</font>来编码。
### 实例
```javascript
    
```