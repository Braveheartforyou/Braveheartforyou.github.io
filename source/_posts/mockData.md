---
title: mockData
date: 2017-09-13 13:30:46
tags: [JavaScript]
categories: [JavaScript]
description: 主要讲的是在后台接口还没有搭建完成，或者说根本还没有api的时候，或者说有api但是没数据的时候，前端怎么模拟数据，我这里只给一个小的demo，真正要构建一套还是要根据业务来
---
## 概述
 在项目开发中，有很多时候是在后台接口还没开发好，有的是api接口都没写，有的是有api接口但是没有数据，不管是那种，都是在大家都对数据格式，接口的方式都是有统一了的，才能做mock数据，不然的话数据格式变化，接口内容变化，基本上前端mock不mocK数据基本上没什么用，还是等于对接两遍。
 - 我自己的建议，当有接口api的时候，但是没有真实数据的时候，我推荐用<font color="red">mock.js</font>来解决问题，不需要自己再用express，或者别的框架自己发布本地接口，直接在ajax返回数据的时候改变response的data数据
 - 另一种就是完全都没有后台接口的时候，要自己本地跑起来接口或者别的地方要有接口api才可以mock数据，建议使用<font color="red">easy-mock</font>

### mock.js
- 首先： 
 可以在自己项目中的package.json中添加mock.js和版本号<font color="red">"mockjs": "^1.0.1-beta3"</font>可以通过在github中看他的tag号，使用他最稳定的版本，通过<font color="red">npm install</font>or<font color="red">cnpm install</font>or<font color="red">yarn</font>都是可以的，这个完全看自己的网络了。
- 然后：  
 在自己的<font color="red">api</font>同级创建一个<font color="red">mock文件夹</font>在里面再创建一个叫做，mock.js的文件。我只是简单的做一个demo,具体的mock怎么构思和构建要看自己的业务，可以见一个总开关来控制是否mock数据，和子开关来控制是否mock子接口，如<font color="red">process.env.NODE_ENV</font> 或者别的全局来判断是否开启mock数据 
 ```javascript
    import Mock from 'mockjs';
    let Random = Mock.Random; // 这个只是 其中的一种形式 还有其他两种
    const oMsgData = Mock.mock({
        'list|10': [{
        name: '@cname',
        id: '@increment',
        content: '@csentence',
        createTime: '@date',
        isRead: Number(Random.boolean()),
        isTop: Number(Random.boolean()),
        state: Number(Random.boolean()),
        type: '@increment'
        }]
    });
    // console.log(JSON.stringify(oMsgData, null, 4));
    // console.log(oMsgData);
    /**
     * @param {String} rurl 要替换的接口路径名
     * @param {String} rtype 要替换的接口请求方式
     * @param {Object} data 要替换的接口的response的data
    /
    // 他会自动帮你填写 域名和端口号 rurl rtype response.data
    Mock.mock('/api/notice/noticeListAdmin', 'post', {oMsgData});
 ```
具体的参数我这个就不讲了，首先引入<font color="red">Mock.js</font>,然后可以通过<font color="red"> '@cname'</font>生成随机的名字，也可以通过<font color="red"> Mock.Random.cname()</font>生成随机名字。
[参考] <http://mockjs.com/> 这个是mock.js的官方文档，里面有mock的具体用法.
### easy-mock
这种是在线方式的，完全可以在他这个里面创建一套符合自己的api接口，具体当可以看下面的连接，这个我感觉没什么好讲的了，因为感觉自己归纳的也没有人家文档好，反正是挺好的。
[easy-mock] <https://easy-mock.com/docs>