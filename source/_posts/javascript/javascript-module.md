---
title: require和import的区别
date: 2019-02-27 21:44:57
tags: [JavaScript]
categories: [JavaScript]
description: CommonJS/Require和ES6/import的区别
---
## 简单比较
对比表格如下

| 对比类型 |  import  |  require |
|:----------:|:-------------:|:------:|
| 何时加载 |  编译时加载 | 运行时加载 |
| 静态化 |   ES6 模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量   | 只有运行时才能得到这个对象，不能在编译时做到静态化 |
| 调用时间 | import是编译时调用，所以严格模式必须放在文件开头 | require是运行时调用，所以require理论上可以运用在代码的任何地方 |
| 模块输出 | 模块输出的是值的引用，不会缓存，而是动态地去被加载的模块取值，并且变量总是绑定其所在的模块 | 模块输出的是一个值的拷贝，并且在第一次加载是缓存，后面加载都会读取缓存 |
| 模块加载 | 按引入加载 | 整体加载 |
| 循环加载 | ES6 模块是动态引用，如果使用import从一个模块加载变量，那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值 | CommonJS 模块的重要特性是加载时执行，即脚本代码在require的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出 |

下面我们一项一项具体解释：

### 何时加载/静态化
require / exports ：
遵循 <font color="#ff502c">CommonJS/AMD</font>，只能在<font color="#ff502c">运行时</font>确定模块的依赖关系及输入/输出的变量，<font color="#ff502c">无法</font>进行<font color="#ff502c">静态优化</font>。

import / export：
遵循 <font color="#ff502c">ES6</font> 规范，支持<font color="#ff502c">编译时静态分析</font>，便于JS引入宏和类型检验。<font color="#ff502c">动态绑定</font>。
### 调用时间
import / export：
import命令具有<font color="#ff502c">提升效果</font>，会<font color="#ff502c">提升</font>到整个模块的头部，<font color="#ff502c">首先执行</font>。（是在编译阶段执行的）
因为import是静态执行的，不能使用表达式和变量，即在运行时才能拿到结果的语法结构.

require / exports ：
require是<font color="#ff502c">运行时</font>调用，所以require理论上可以运用在代码的<font color="#ff502c">任何地方</font>

### 模块输出/模块加载
require / exports ：
CommonJS 模块输出的是<font color="#ff502c">值的拷贝</font>，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值，并且会在第一次加载是<font color="#ff502c">缓存</font>这个值的拷贝，他是<font color="#ff502c">完全</font>输出这个值的拷贝。

import / export：
模块输出的是值的<font color="#ff502c">引用</font>，<font color="#ff502c">不会</font>缓存，而是<font color="#ff502c">动态</font>地去被加载的模块取值，并且变量总是绑定其所在的模块，并且只会<font color="#ff502c">加载</font>引入的值，不去全部加载。
