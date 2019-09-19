---
title: cookie_function
date: 2017-08-31 11:33:31
tags: [JavaScript]
categories: [JavaScript]
description: cookie常用方法封装
---

## 概述

### Cookie 在客户端 JavaScript 中

用 JavaScript 操作<font color="#ff502c">Cookie</font>，用原声的接口<font color="#ff502c">document.cookie</font>属性是比较难用的，所以封装一个添加、修改、删除的操作方法还是很有必要的。
同时要注意的是： Cookie 的所有<font color="#ff502c">name</font>和<font color="#ff502c">value</font>都是要经过<font color="#ff502c">URI</font>编码的，必须使用<font color="#ff502c">decodeURICompoent()</font>来编码。

## 创建Cookie

在浏览器端可以通过`document.cookie`来创建`cookie`，但是`cookie`只能是字符串形式并且格式必须是`url`格式，所以就要自行封装方法来使用。

代码封装如下：

```javascript
  /**
  * @params {String} key 键名
  * @params {String} value 键值
  * @params {String} options 配置项 如 readOnly expires path secure
  */
  function setCookie (key, value, options) {
    // 判断是否有document
    if (typeof document === 'undefined') {
      return
    }
    // 默认配置
    var defalutConfig = {};
    // 合并配置
    var options = extend(defalutConfig, {}, options);
    // 判断expires是否为数字
    if (typeof options.expires === 'number') {
      var days = options.expires, t = options.expires = new Date();
      t.setMilliseconds(t.getMilliseconds() + days * 864e+5);
    }
    // 判断value是否为object，是的话通过JSON.stringify序列化
    var value = Object.prototype.toString.call(value) !== '[object Object]' ? String(value) : JSON.stringify(value);
    document.cookie = [
      encodeURIComponent(key), '=', value,
      options.expires ? '; expires=' + options.expires.toUTCString() : '', // use expires attribute, max-age is not supported by IE
      options.path    ? '; path=' + options.path : '',
      options.domain  ? '; domain=' + options.domain : '',
      options.secure  ? '; secure' : ''
    ].join('');
  }

  /**
  * @descriptor 合并options
  *
  */
  function extend () {
    var result = {},
      args = Array.prototype.slice.call(arguments),
      len = args.length;
    for (let i = 0; i < len; i++) {
      var options = args[i];
      for ( let key in options) {
        result[key] = options[key];
      }
    }
    return result;
  }
```

测试代码：

```javascript
  setCookie('token', 'asd1231asdas123sfdsdf3453asas121asd', {expires: 1, path: '/', domain: 'localhost'})
  // "token=asd1231asdas123sfdsdf3453asas121asd; expires=Fri, 20 Sep 2019 06:46:09 GMT; path=/; domain=localhost"
```

## 读取Cookie

可以通过`document.cookie`获取`cookie`，但是要转换为对象，实现代码如下：

```javascript
  function decode (s) {
    return s.replace(/(%[0-9A-Z]{2})+/g, decodeURIComponent)
  }
  /**
  * @params {Stirng} key 要获取的键值
  * @params {Boolean} json 是否返回json格式
  * @return {Object} 返回cookie
  */
  function getCookie (key, json) {
    // 判断是否存在document
    if (typeof document === 'undefined') {
      return
    }
    // 声明变量
    var result = {},
        cookies = document.cookie ? document.cookie.split('; ') : [],
        len = cookies.length;
    // 循环获取cookie，拼接or匹配 key
    for (var i = 0; i < len; i++) {
      // 以‘=’分割数组
      var parts = cookies[i].split('='),
        cookie = parts.slice(1).join('=');
      if (!json && cookie.charAt(0) === '') {
        cookie = cookie.slice(1, -1);
      }
      try {
        // 通过decodeURIComponent 解码
        var name = decode(parts[0]);
        cookie = decode(cookie);
        // 根据实参json返回不同的数据类型
        if (json) {
          try {
            cookie = JSON.parse(cookie);
          } catch (e) {}
        }
        result[name] = cookie;
        if (key === name) {
          break;
        }
      } catch (e) {}
    }
    // 返回数据
    return key ? result[key]: result;
  }
  // 测试代码
  getCookie('user', true)
  // {name: "admin", age: 18}
```

## 删除cookie

同样通过`setCookie`来实现删除**cookie**，只是传入特定参数`expires: -1`，实现代码如下：

```javascript
  function removeCookie (key, options) {
    setCookie(key, '', extend(options, { expires: -1 }));
    return !getCookie(key);
  }
  // 测试代码
  removeCookie('token');
  // true
```

到此`cookie`的操作到此结束。

## 总结

这篇文章比较简单，记录了对`cookie`的**增删改查**。
