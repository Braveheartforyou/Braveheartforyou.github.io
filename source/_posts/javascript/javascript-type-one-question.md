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

我们先来解释显示转换再来隐式转换。
如果想了解[javaScript中的类型](/blog/javascript/javascript-Type-conversion.html)，或者[if运算符转换规则](/blog/javascript/javascript-IF-False-options.html)的知识可以看看我其他的博客。

### 显示转换
> 1. ToString
> 2. ToNumber
> 3. ToBoolean

首先介绍显示转换中的基本类型互转，字符串、数字、布尔值、null、undefined之间的转换。

### 隐式转换
> 1. ToPrimitive(转换为原始值)
> 2. valueOf(返回指定对象的原始值)
> 3. 运算符中的转换（+、-、*、/）
> 4. ==

再来看一下ToPrimitive把对象转为原始值，当然也有Object.prototype.toString()的介绍，再试valueOf返回对象的原始值，再最后就是运算符在基本类型和object之间的运算，其中最坑的应该是==的隐士转换，因为他的左右规则不同，左右类型不同执行的又不同。

### ToPrimitive的规则
我们要用到的ToPrimitive的规则，因为显示转换也会用到ToPrimitive的规则，如果还想所有的转换规则请看.[ecma规则](http://www.ecma-international.org/)
**ToPrimitive**(转换为原始值)
ToPrimitive 运算符接受一个值，和一个可选的 期望类型作参数。
```javascript
/**
* @params obj 要转换的对象 ，required
* @params type 期望转换为的原始数据类型， 
*/
ToPrimitive(obj, type);
```
根据<font color="#ff502c">type</font><font color="#ff502c"></font>的不同会他后面的步骤也不相同，如下所示

**type为string**
> 1. 先调用obj的<font color="#ff502c">toString</font>方法，如果返回<font color="#ff502c">原始值</font>，则不往下执行，如果返回不是原始值，则执行<font color="#ff502c">第二步</font>。
> 2. 调用obj的<font color="#ff502c">valueOf</font>方法，如果为<font color="#ff502c">原始值</font>，则不往下执行，如果返回不是原值值，则执行<font color="#ff502c">第三步</font>。
> 3. 抛出<font color="#ff502c">TypeError</font>异常

**type为Number**
> 1. 先调用obj的<font color="#ff502c">valueOf</font>方法，如果返回<font color="#ff502c">原始值</font>，则不往下执行，如果返回不是原始值，则执行<font color="#ff502c">第二步</font>。
> 2. 调用obj的<font color="#ff502c">toString</font>方法，如果为<font color="#ff502c">原始值</font>，则不往下执行，如果返回不是原值值，则执行<font color="#ff502c">第三步</font>。
> 3. 抛出<font color="#ff502c">TypeError</font>异常

**type参数为空**
> 1. 该对象为<font color="#ff502c">Date，则type被设置为<font color="#ff502c">String</font>
> 2. 否则，type被设置为<font color="#ff502c">Number</font>

这基本上就是object常用的转换的一些规则，在下面会验证ToPrimitive规则是否正确。

## 显示转换
我们先看一下基本类型之间的转换的图标如下：

|    	| Null | Undefined | Boolean(true) | Boolean(false) | Number |  String |
|:----------:|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|
| Boolean | false | false | - | - | (0/NAN)=>false OR true | ('')=>false OR true |
| Number | 0 | NaN | 1 | 0 | - | ('')=>false OR true |
| String | 'null' | 'undefined' | 'true' | 'false' | 'Number' | - |

我们下面一个一个说明ToString、ToNumber、ToBoolean的大致规则。
### ToString
抽象操作ToString负责处理非字符串到字符串的强制类型转换。
验证一下上面表格中的转换规则，验证的时候加上object的转换验证，代码如下：
```javascript
console.log(String(null)); // 'null'
console.log(String(undefined)); // 'undefined'
console.log(String(true)); // 'true'
console.log(String(false)); // 'false'
console.log(String(1)); // '1'
console.log(String(0)); // '0'
console.log(String(NaN)); // 'NaN'
console.log(String([1, 2, 3])); // '1,2,3' or [].toString()
console.log(String({a: 123})); // [object Object]
console.log(String(new Date())); // 'Sat Apr 20 2019 00:00:00 GMT+0800 (中国标准时间)'
```
上面的代码证明了我们上面table中的基本类型的String转换规则。
> 注意：对象这里要先转换为原始值，调用ToPrimitive转换，type就指定为string了，继续回到ToPrimitive进行转换（看ToPrimitive）。

### ToNumber
抽象操作ToNumber负责处理非Number到Number的强制类型转换。
验证一下上面表格中的转换规则，验证的时候加上object的转换验证，代码如下：
```javascript
console.log(Number(null)); // 0
console.log(Number(undefined)); // NaN
console.log(Number(true)); // 1
console.log(Number(false)); // 0
console.log(Number('1')); // 1
console.log(Number('0')); // 0
console.log(Number('abcd')); // 'NaN'
console.log(Number([1, 2, 3])); // NaN
console.log(Number({a: 123})); // NaN
console.log(Number(new Date())); // 1555689600000
```
上面的代码证明了我们上面table中的基本类型的Number转换规则。
> 注意：对象这里要先转换为原始值，调用ToPrimitive转换，type就指定为number了，继续回到ToPrimitive进行转换（看ToPrimitive）。

### ToBoolean
抽象操作ToNumber负责处理非Number到Number的强制类型转换。
验证一下上面表格中的转换规则，验证的时候加上object的转换验证，代码如下：
```javascript
console.log(Boolean(null)); // false
console.log(Boolean(undefined)); // false
console.log(Boolean('true')); // true
console.log(Boolean('false')); // true
console.log(Boolean('')); // false
console.log(Boolean(NaN)); // false
console.log(Boolean(1)); // true
console.log(Boolean(0)); // false
console.log(Boolean('abcd')); // true
console.log(Boolean([1, 2, 3])); // true
console.log(Boolean({a: 123})); // true
console.log(Boolean([])); // true
console.log(Boolean({})); // true
console.log(Boolean(new Date())); // 1555689600000
```
在表格上只有基本类型的转换为Boolean的规则，具体的转换规则为：
- <font color="#ff502c">false</font>
- <font color="#ff502c">null</font>
- <font color="#ff502c">undefined</font>
- <font color="#ff502c">空字符串' '</font>
- <font color="#ff502c">数字零 0</font>
- <font color="#ff502c">NaN</font>
只有上面的会转为false,其他的都会转为true.
他和if()条件运算符转换的规则基本上一至，请看[if运算符转换](/blog/javascript/javascript-IF-False-options.html)。

## 隐士转换
在上面已经列出了ToPrimitive的常用的转换规则，在下面会验证上面的规则是否正确。最坑的也是规则对多的就是运算符的转换，可能现在只知道一部分，后面会慢慢补全。

### ToPrimitive验证
**Object转String规则验证**
首先验证Obejct转换为String的规则，直接上代码。
```javascript
var obj = {
  toString: function () { console.log('toString'); return {}; },
  valueOf: function () { console.log('valueOf'); return {};}
};
String(obj);
// 1. toString 
// 2. valueOf
// Uncaught TypeError: Cannot convert object to primitive value 
```
根据上面的输出结果，证明上面的String()，走了<font color="#ff502c">ToPrimitive(obj, string)</font>的<font color="#ff502c">type</font>为<font color="#ff502c">string</font>的规则。详情见上面。
**Object转Number规则验证**
再验证Obejct转换为Number的规则，直接上代码。
```javascript
var obj = {
  toString: function () { console.log('toString'); return {}; },
  valueOf: function () { console.log('valueOf'); return {};}
};
Number(obj);
// 1. valueOf
// 2. toString
// Uncaught TypeError: Cannot convert object to primitive value 
```
根据上面的输出结果，证明上面的Number()，走了<font color="#ff502c">ToPrimitive(obj, number)</font>的<font color="#ff502c">type</font>为<font color="#ff502c">number</font>的规则。详情见上面。

**其他转变如数组转String或者Number**
再验证Obejct转换为Number的规则，直接上代码。
```javascript
var obj = {
  toString: function () { console.log('toString'); return {}; },
  valueOf: function () { console.log('valueOf'); return {};}
};
Number(obj);
// 1. valueOf
// 2. toString
// Uncaught TypeError: Cannot convert object to primitive value 
```
根据上面的输出结果，证明上面的Number()，走了<font color="#ff502c">ToPrimitive(obj, number)</font>的<font color="#ff502c">type</font>为<font color="#ff502c">number</font>的规则。详情见上面。

### valueOf
JavaScript 调用 valueOf() 方法用来把对象转换成原始类型的值（数值、字符串和布尔值）。但是我们很少需要自己调用此函数，valueOf 方法一般都会被 JavaScript 自动调用。
- String => 返回字符串值
- Number => 返回数字值
- Boolean => 返回Boolean的this值
- Object => 返回this
- Date => 返回一个数字，即时间值,字符串中内容是依赖于具体实现的

代码如下：
```javascript
console.log('abc'.valueOf()); // 'abc'
console.log(Number(123).valueOf()); // 123
console.log(Boolean(true).valueOf()); // true
console.log({a: 123}.valueOf()) // {a: 123}
console.log(new Date()); // Sat Apr 20 2019 00:00:00 GMT+0800 (中国标准时间)
```
### 运算符中的转换（+、-、*、/)
在运算符中的转换又分为两种，自动转换为string类型、自动转换为number类型。

#### 自动转换为string类型
在基础类型中，当一个值为字符串，另一个值非字符串，则后者转为字符串，当有一个是对象时，会走ToPrimitive(obj, number)，ToPrimitive转换请看上面，下面看代码。
```javascript
console.log('a' + 1); // 'a1'
console.log('a' + true); // 'atrue'
console.log('a' + false); // 'afalse'
console.log('a' + undefined); // 'aundefined'
console.log('a' + null); // 'anull'
console.log('a' + []); // 'a'
console.log('a' + {toString: function () { return '1' }}); // 'a1'
console.log('a' + {valueOf: function () { return 1 }}); // 'a1'
```
#### 自动转换为number类型
加法运算符，如果没有一个为string类型的时候，都会优先转换为Number类型，如果有一个为object时，会走ToPrimitive(obj, number)，ToPrimitive转换请看上面。一元运算符，也是需要注意。下面看代码。
```javascript
console.log(true + false); // 1
console.log(true + null); // 1
console.log(true + undefined); // NaN
console.log(true + []); // ‘true’
console.log(true + null); // 1
console.log(true - false); // 1
console.log(true - true); // 0
console.log('1' - '0'); // 1
console.log('1' * 0); // 0
console.log('0' * 1); // 0
console.log('1' - '0'); // 1
console.log('abc' - 0); // NaN
console.log('abc' - '0'); // NaN
console.log(null - 0); // 0
console.log('1' - 0); //NaN
console.log(1 - {valueOf: function () { return 1 }}); // 0
console.log(1 - {valueOf: function () { return {} }, toString: function () { return 1 }}); // 0
// 一元运算符
console.log(+'abc'); // NaN
console.log(-'abc'); // NaN
console.log(+'1'); // 1
console.log(-'1'); // -1
console.log(+true); // 1
console.log(-false); // -0
console.log(+true); // 1
```
> 注意：null转为数值时为0，而undefined转为数值时为NaN

### == 
== 抽象相等比较与+运算符不同，不再是String优先，而是Nuber优先。假设左面为x、y为右面，大概的规则如下。
> 1. 如果x,y都为number,直接比较
> 2. 如果x为string，y为number，x转换为number比较，反则相反。
> 3. 如果存在对象，通过ToPrimitive(obj, number)type为number进行转换，再进行比较。
> 4. 如果x，y有一方存在boolean,按照ToNumber将boolean转换为1或0，再进行比较。
验证代码如下：
```javascript
// x,y 都number 比较
console.log(1 == 2) // false
console.log(1 == 1) // true

// 一方为string一方为number
console.log('1' == 2) // false
console.log(1 == '1') // true

// 如果一方为 对象
console.log(null == 2) // false
console.log(1 == undefined) // false
// 因为对象的规则比较繁杂，如果普通对象[] 和 ![] 规则不同请看下面文章

// 存在 booelan
console.log(true == 0); // false
console.log(true == '0'); // false
console.log(false == '0'); // true
```
在这里就不多赘述了，看我另一篇文章 [![] == []](/blog/javascript/javascript-false-true.html)，通过一道面试题。来讲解基本的转换规则，因为这个规则其实挺复杂的，一两句话讲不清楚。

## 参考
> [https://mp.weixin.qq.com/s/_THIZY4KTa1IlVb4k9qbJg](https://mp.weixin.qq.com/s/_THIZY4KTa1IlVb4k9qbJg)
> [https://javascript.ruanyifeng.com/grammar/conversion.html](https://javascript.ruanyifeng.com/grammar/conversion.html)
> [https://github.com/amandakelake/blog/issues/34](https://github.com/amandakelake/blog/issues/34)