---
title: javascript的中的类型转换（二）
date: 2019-04-20 19:31:43
tags: [JavaScript]
categories: [JavaScript]
description: JavaScript中的类型转换，如强制转换（显示转换）、隐式转换
---
## 简述
在JavaScript中关于类型转换的规则是不叫混乱的，有强制转换（显示转换）、隐式转换，并且转换的规则没有完整可参考的文档，只有当时的提议书。并且在隐式转换的时候会出现很多不可思议的bug.

> 类型转换发生在静态类型的语言的编译阶段，而强制类型转换则发生在动态类型语言的运行时（runtime）；
