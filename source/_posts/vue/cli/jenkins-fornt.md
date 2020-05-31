---
title: jenkins配置前端不同环境打包不同的文件
date: 2018-11-13 18:09:31
tags: [WebPack, Deploy]
categories: [VueCli]
description: '使用webpack打包 前端应用，并且配置jenkins脚本打包不同环境路径，配置文件如vue-cli、create-react-app都可以配置，后续文章记录常用的webpack优化，从文件大小，传输类型，缓存优化'
---

## vue-cli 配置

vue-cli `2.x`版本，因为自己不太习惯 3.x 的这种配置，很多配置不容易写进去，同时 3.x 也在不断的修 bug,所以没有升级.
最新代码在[github](https://github.com/Braveheartforyou/vue-cli-jenkins.git)，这个项目基本上是一些打包优化，缓存，等等并不是后台模版，你可以在这个项目上再搭建自己的东西，直接只是提供了一个项目比较基础的一些东西。

## 配置脚本

在`package.json`文件中的`scripts`添加三个字段:

- "build:sit": "cross-env API_ROOT=sit node build/build.js",
- "build:uat": "cross-env API_ROOT=uat node build/build.js",
- "build:prod": "cross-env API_ROOT=prod node build/build.js"

其中`cross-env`包是为了兼容 liunx 和 window 不同系统都可以把 <font color="green">API_ROOT`参数传进进程中. ![jenkins_vue_cli](../../images/vue_build/jenkins_vue_cli.png) 然后配置`config`文件夹下的<font color="green">dev.env.js、prod.env.js`

### dev.env.js

```javascript
'use strict';
const merge = require('webpack-merge');
const prodEnv = require('./prod.env');

/**
 * 判断环境 运行不同的api
 * 登录地址切换
 */
switch (process.env.API_ROOT) {
  case 'sit':
    module.exports = merge(prodEnv, {
      NODE_ENV: '"development"',
      API_URL: '"http://localhost:3800/api"',
      API_PUBLIC: '"http://localhost:3800/public_api"'
    });
    break;
}
```

### prod.env.js

```javascript
'use strict';
/**
 * 判断当前环境 打包不同的 api地址  login地址
 */
switch (process.env.API_ROOT) {
  case 'sit':
    module.exports = {
      NODE_ENV: '"production"',
      API_URL: '"http://localhost:3800/api"',
      API_PUBLIC: '"http://youpublicapi-sit.com/api"',
      version: 'v1.0.0'
    };
    break;
  case 'uat':
    module.exports = {
      NODE_ENV: '"production"',
      API_URL: '"http://youapi-uat.com/api"',
      API_PUBLIC: '"http://youpublicapi-uat.com/api"',
      version: 'v1.0.0'
    };
    break;
  case 'prod':
    module.exports = {
      NODE_ENV: '"production"',
      API_URL: '"http://youapi-prod.com/api"',
      API_PUBLIC: '"http://youpublicapi-prod.com/api"',
      version: 'v1.0.0'
    };
    break;
}
```

最后也是最重要的也就是我们要有一个统一的调用地址，如果不是统一的一个，那就多声明几个模块如`PAYMENT、PRODUCT`等来区分不同环境的不同后台接口
![jenkins_vue_cli2](../../images/vue_build/jenkins_vue_cli2.png)

## jenkins 配置

- 新建一个 构建一个自由风格的软件项目
- 配置》源码管理》Git(git 地址和 ssh 帐号密码、拉去代码的分支)
  ![jenkins_vue_cli3](../../images/vue_build/jenkins_vue_cli3.png)
- 锁定编译环境 node 版本为 8.9.3 或者别的
  ![jenkins_vue_cli4](../../images/vue_build/jenkins_vue_cli4.png)
- jenkins 前端的构建脚本
  ![jenkins_vue_cli5](../../images/vue_build/jenkins_vue_cli5.png)
  这个只是最简单的打包发送到对应的服务器，其实你在这个时候还可以做很多其他的事，如运行单元测试、sonar 平台质量检测、备份等等
