---
title: javascript中new相关的面试题
date: 2019-08-03 23:13:45
tags: [InterviewQuestion, JavaScript]
categories: [InterviewQuestion]
description: 这一篇记录二个new相关的面试题，会详细分析答案
---
## 第一题
```javascript
    function Foo () {
        getName = function () {
            console.log(1);
        }
        return this;
    }
    Foo.getName = function () { console.log(2); };
    Foo.prototype.getName = function () { console.log(3); };
```