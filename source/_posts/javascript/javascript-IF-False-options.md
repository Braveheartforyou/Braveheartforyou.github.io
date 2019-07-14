---
title: 在JavaScript中if运算符只有固定的值会转为false
date: 2017-11-03 13:13:39
tags: [JavaScript]
categories: [JavaScript]
description: 在用javaScript的if的时候他会把固定的值转换为false，其他的统一认为为true
---
## 简介
<!-- <font color="#ff502c"></font> -->
在JavaScript中使用if的时候，自己如果不注意的话很可能出现判断进错，其实在JavaScript中只有<font color="#ff502c">固定的几个值会转为false，其它的统一认为为true。
- <font color="#ff502c">false</font>
- <font color="#ff502c">null</font>
- <font color="#ff502c">undefined</font>
- <font color="#ff502c">空字符串' '</font>
- <font color="#ff502c">数字零 0</font>
- <font color="#ff502c">NaN</font>
其他的全部都算为true,<font color="#ff502c">'false'</font>、<font color="#ff502c">'0'</font>也是为true,其实这也是一种隐性的类型转换。和 == 又有不同。

## 逻辑运算符
由于逻辑表达式是<font color="#ff502c">从左往右</font>计算的，由于运算符优先级的存在，下面的表达式的结果却不相同。如下例所示
```javascript
    false && true  || true      // 结果为 true
    false && (true || true)     // 结果为 false
```
右侧被小括号括起来的操作变成了独立的表达式。
<font color="#ff502c">转换规则</font>:
- 将 AND  转换为 OR
- 将 OR 转换为 AND
- 删除嵌套的 AND
- 删除嵌套的 OR
可参考>https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Logical_Operators
### 逻辑与(&&)
<font color="#ff502c"></font>
尽管 && 和 || 运算符能够使用<font color="#ff502c">非布尔值</font>的操作数, 但它们依然被看作是<font color="#ff502c">布尔操</font>作符，因为它们的返回值总是能够被转换为<font color="#ff502c">布尔值</font>。
expr1 && expr2
如果<font color="#ff502c">expr1</font>能转换为<font color="#ff502c">false</font>则返回<font color="#ff502c">expr1</font>,否则返回<font color="#ff502c">expr2</font>。因此，与布尔值一起使用时，如果<font color="#ff502c">两个</font>操作数都为<font color="#ff502c">true</font>时<font color="#ff502c">&&</font>返回<font color="#ff502c">true</font>,否则返回<font color="#ff502c">false</font>.
```javascript
    a1=true && true       // t && t 结果为 true
    a2=true && false      // t && f 结果为 false
    a3=false && true      // f && t 结果为 false
    a4=false && (3 == 4)  // f && f 结果为 false
    a5="Cat" && "Dog"     // t && t 结果为 Dog
    a6=false && "Cat"     // f && t 结果为 false
    a7="Cat" && false     // t && f 结果为 false
```
### 逻辑与(||)
expr1 && expr2
如果<font color="#ff502c">expr1</font>能转换为<font color="#ff502c">true</font>则返回<font color="#ff502c">expr1</font>,否则返回<font color="#ff502c">expr2</font>。因此，与布尔值一起使用时，如果<font color="#ff502c">任意一个</font>操作数为<font color="#ff502c">true</font>时<font color="#ff502c">||</font>返回<font color="#ff502c">true</font>.
```javascript
    o1=true || true       // t || t 结果为 true
    o2=false || true      // f || t 结果为 true
    o3=true || false      // t || f 结果为 true
    o4=false || (3 == 4)  // f || f 结果为 false
    o5="Cat" || "Dog"     // t || t 结果为 Cat
    o6=false || "Cat"     // f || t 结果为 Cat
    o7="Cat" || false     // t || f 结果为 Cat
```
### 逻辑非(!)
!expr	如果单个表达式能转换为<font color="#ff502c">true</font>的话返回<font color="#ff502c">false</font>，否则返回<font color="#ff502c">true</font>。
```javascript
    n1=!true              // !t 结果为 false
    n2=!false             // !f 结果为 true
    n3=!"Cat"             // !t 结果为 false
```