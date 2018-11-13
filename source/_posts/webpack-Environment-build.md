---
title: webpack_Environment_build
date: 2017-10-25 14:20:59
tags: [WebPack, vue-cli]
categories: [vue-cli]
description: 在用webpack打包的时候，要区分正式环境、测试环境、预发布环境，要调用不同的接口和一些路径不同，我用vue官方脚手架vue-cli上做的
---
## 简述
在公司整体的框架就是要分为测试环境、预发布环境、正式环境，要切换不同的后台api地址，登录地址也是跳转别的子站点登录，所以就要通过编译不同环境的前端代码。
用的vue默认的脚手架<font color="red">vue-cli</font>
第一种就是通过<font color="red">process.env</font>来传递不同的值，在<font color="red">config</font>文件夹中通过 <font color="red">dev.env.js、pro.env.js</font>区分是什么环境要调用的接口地址和登录地址
## 思路1
可以通过最外层的<font color="red">package.json</font>中的<font color="red">scripts</font>对象中的<font color="red">dev添加 <font color="red">set API_ROOT=localhost&& node build/dev-server.js</font>并且在<font color="red">config文件夹中的dev.env.js</font>加一个判断,分别判断 sit、uat、pro正式环境还有本地loaclhost，打包不同的文件和接口
```javascript
    switch (process.env.API_ROOT) {
    case 'sit':
        module.exports = merge(prodEnv, {
            NODE_ENV: '"development"',
            API_URL: '',
            LOGIN_URL: '',
            LOGOUT_URL: ''
        })
        break;
    case 'uat':
        module.exports = merge(prodEnv, {
            NODE_ENV: '"development"',
            API_URL: '',
            LOGIN_URL: '',
            LOGOUT_URL: ''
        })
        break;
    case 'pro':
        module.exports = merge(prodEnv, {
            NODE_ENV: '"development"',
            API_URL: '',
            LOGIN_URL: '',
            LOGOUT_URL: ''
        })
        break;
    default:
        module.exports = merge(prodEnv, {
            NODE_ENV: '"development"',
            API_URL: '',
            LOGIN_URL: '',
            LOGOUT_URL: ''
        })
        break;
}
```
但是有一个不好的地方，就是要写多个dev-sit、dev-uat、dev-pro和build-sit、build-uat、build-pro要运行不同的环境和打包不同的环境
同时在<font color="red">api.js</font>中读取<font color="red">process.env.API_URL</font>来调用不同的接口位置

## 思路2
可以在 cmd 运行npm run dev时添加进程的信息，如在window端添加进程信息set API_ROOT=localhost&& npm run dev这样式和上面有同样的结果，这样更简洁一些，这只是个人的一些简单的见解。