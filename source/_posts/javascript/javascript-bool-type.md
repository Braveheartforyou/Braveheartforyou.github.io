---
title: JavaScirpt数据类型判断和数据结构判断
date: 2019-05-11 22:24:23
tags: [JavaScript]
categories: [JavaScript]
description: javascript中的类型判断又分为基础类型判断和引用类型多种判断方法
---
## 类型简介
  * 在ECMAScript规范中，共定义了7种数据类型，分为基本数据类型和引用类型两大类，如下所示：

  > 基本类型： Null、Undefined、Symbol（ES6）、Number、Boolean、String
  > 引用类型：Obeject、Array、Date

  - 基本类型也称为简单类型，由于其占据空间固定，是简单的数据段，为了便于提升变量查询速度，将其存储在<font color="blue">栈</font>中，即按值访问。

  - 引用类型也称为复杂类型，由于其值的大小会改变，所以不能将其存放在栈中，否则会降低变量查询速度，因此，其值存储在<font color="blue">堆(heap)</font>中，而存储在变量处的值，是一个指针，指向存储对象的内存处，即按址访问。

### 原始值( primitive values )
  除 Object 以外的所有类型都是不可变的（值本身无法被改变）。例如，与 C 语言不同，JavaScript 中字符串是不可变的（译注：如，JavaScript 中对字符串的操作一定返回了一个新字符串，原始字符串并没有被改变）。我们称这些类型的值为“原始值”。

  注: **想看Java​Script 数据类型和数据结构 可以在： [mdn](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures)中查看**

  有四种方法可以判断Java​Script中的类型如：
  1、typeof运算符
  2、instanceof运算符
  3、constructor(原型对象的构造函数)
  4、toString(内置[[Class]]属性)、Array.isArray()
  下面就分别来讲一下他们能判断什么类型，判断不了什么类型，因为什么还有一些注意事项。

## typeof运算符

  * ### typeof语法
  typeof运算符后跟操作数：
  > typeof operand
  > or
  > typeof (operand)

  * ### 参数
  operand 是一个表达式，表示对象或[原始值](https://developer.mozilla.org/zh-CN/docs/Glossary/Primitive)，其类型将被返回。

  下面表格总结了typeof可能返回的值。

  | 类型 | 结果 |
  |:--------:|:-------:|
  | Undefined | "undefined" |
  | Null | "object" |
  | Boolean | "boolean" |
  | Number | "number" |
  | String | "string" |
  | Symbol （ECMAScript 6 新增） | "symbol" |
  | 宿主对象（由JS环境提供） | Implementation-dependent |
  | 函数对象（[[Call]] 在ECMA-262条款中实现了） | "function" |
  | 任何其他对象 | "object" |

  * ### 实例

  ```javascript
  typeof undefined; // undefined 有效
  typeof null; // object 无效
  typeof true; // boolean 有效
  typeof 1; // number 有效
  typeof '123'; // string 有效
  typeof Symbol('foo'); // symbol 有效
  typeof window; // object 
  typeof function(){}; // function 有效
  typeof {a:1}; // object 
  typeof []; // object 
  ```
  在这其中要注意的是：
  - 其中null返回了object是因为JavaScript语言设计遗留的问题。
  - 对于引用类型，除 function 以外，一律返回 object 类型
  - 对于 function 返回 function

* ### typeof null

  在 JavaScript 最初的实现中，JavaScript 中的值是由一个表示类型的标签和实际数据值表示的。由于 null 代表的是机器代码的空指针，一个对象类型的引用，值是零（大多数平台下值为 0x00）。自然前三位也是0，所以执行typeof时会返回"object".

  这个bug是第一版Javascript留下来的。在这个版本，数值是以32字节存储的，由标志位（1~3个字节）和数值组成。标志位存储的是低位的数据。这里有五种标志位：
  - 000：对象，数据是对象的应用。
  - 1：整型，数据是31位带符号整数。
  - 010：双精度类型，数据是双精度数字。
  - 100：字符串，数据是字符串。
  - 110：布尔类型，数据是布尔值。
  最低位有一位，那么标志位只有一个1字节长度；或者是零位，标志位有3个字节长度，多出两个了字节，一共多出四种类型。

  注：**原文地址[英文](http://2ality.com/2013/10/typeof-null.html)**
      **翻译地址[中文](http://www.cnblogs.com/xiaoheimiaoer/p/4572558.html)**

    **判断null类型也很简单，就用null === null来判断**
    **看typeof所有的类型细节请看 [mdn-typeof](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof#%E8%AF%AD%E6%B3%95)**

## instanceof运算符

  instanceof运算符用于测试构造函数的prototype属性是否出现在对象的原型链中的任何位置.
  原始值使用instanceof都会返回false，如果使用new 声明 是可以检测出来。对于是使用new声明的类型，它还可以检测出多层继承关系。

  * ### 语法
    > - object instanceof constructor

  * ### 参数
    object 要检测的对象
    constructor 某个构造函数

  * ### 实例
    instanceof 运算符用来检测 constructor.prototype 是否存在于参数 object 的原型链上。

  ```javascript
  var str = '123';
  str instanceof String; // false

  function C () {}
  var c1 = new C(); 
  c1 instanceof C; // true，因为 Object.getPrototypeOf(c1) === C.prototype

  C.prototype = {};

  c1 instanceof C; // false, C.prototype指向了一个空对象,这个空对象不在o的原型链上.

  var c2 = new C();

  c2 instanceof C; // true，因为 Object.getPrototypeOf(c2) === C.prototype

  c2.__proto__ = {}; // 借助于非标准的__proto__伪属性

  c2 instanceof C; // false 

  ```

  注：**需要注意的是，如果表达式 obj instanceof Foo 返回true，则并不意味着该表达式会永远返回true，因为Foo.prototype属性的值有可能会改变，改变之后的值很有可能不存在于obj的原型链上，这时原表达式的值就会成为false。另外一种情况下，原表达式的值也会改变，就是改变对象obj的原型链的情况，虽然在目前的ES规范中，我们只能读取对象的原型而不能改变它，但借助于非标准的__proto__伪属性，是可以实现的。比如执行obj.__proto__ = {}之后，obj instanceof Foo就会返回false了。**

 * ### []、Array、Object 三者之间的关系：
  从 instanceof 能够判断出 [ ].__proto__  指向 Array.prototype，而 Array.prototype.__proto__ 又指向了Object.prototype，最终 Object.prototype.__proto__ 指向了null，标志着原型链的结束。因此，[]、Array、Object 就在内部形成了一条原型链：
  <img src="../images/javascript/javascript1.png" alt="[]-Array-Object" width="60%" style="margin: 0 auto;"/>
  从原型链可以看出，[] 的 __proto__  直接指向Array.prototype，间接指向 Object.prototype，所以按照 instanceof 的判断规则，[] 就是Object的实例。依次类推，类似的 new Date()、new Person() 也会形成一条对应的原型链 。**因此，instanceof 只能用来判断两个对象是否属于实例关系， 而不能判断一个对象实例具体属于哪种类型。**

  * ### instanceof和多全局对象(多个frame或多个window之间的交互)
  在浏览器中，我们的脚本可能需要在多个窗口之间进行交互。多个窗口意味着多个全局环境，不同的全局环境拥有不同的全局对象，从而拥有不同的内置类型构造函数。这可能会引发一些问题。比如，表达式 [] instanceof window.frames[0].Array 会返回false，因为 Array.prototype !== window.frames[0].Array.prototype，并且数组从前者继承。

  实际上你可以通过使用 Array.isArray(myObj) 或者Object.prototype.toString.call(myObj) === "[object Array]"来安全的检测传过来的对象是否是一个数组。

  * ### 实现一个简单的instanceof
  ```javascript
    // L instanceof R
    function instance_of(L, R) { //L 表示左表达式，R 表示右表达式
      var O = R.prototype; // 取的R的显式原型
      L = L.__proto__; // 取的L的隐式原型
      while (true) {
        if (L === null) { // 已找到顶层
          return false;
        }
        if (O === L) { // 当 O严格等于L时，返回true
          return true;
        }
        L = L.__proto__; // 继续向上一层原型链查找
      }
    }
    // 测试代码
    instance_of([], Array); // true
    instance_of([], Object); // true
  ```
## constructor(原型对象的构造函数)
  当一个函数 F被定义时，JS引擎会为F添加 prototype 原型，然后再在 prototype上添加一个 constructor 属性，并让其指向 F 的引用。如下所示：
  <img src="../images/javascript/javascript2.png" alt="constructor"  style="margin: 0 auto;"/>
  当执行 var f = new F() 时，F 被当成了构造函数，f 是F的实例对象，此时 F 原型上的 constructor 传递到了 f 上，因此 f.constructor === F.

  可以看出，F 利用原型对象上的 constructor 引用了自身，当 F 作为构造函数来创建对象时，原型上的 constructor 就被遗传到了新创建的对象上， 从原型链角度讲，构造函数 F 就是新对象的类型。这样做的意义是，让新对象在诞生以后，就具有可追溯的数据类型。

  实例
  ```javascript
  ''.constructor === String; // true;
  new Number(1).constructor === Number; // true;
  true.constructor === Boolean; // true;
  new Data().constructor === Date; // true
  ```

  > - null 和 undefined 是无效的对象，因此是不会有 constructor 存在的，这两种类型的数据需要通过其他方式来判断
  > - 函数的 constructor 是不稳定的，这个主要体现在自定义对象上，当开发者重写 prototype 后，原有的 constructor 引用会丢失，constructor 会默认为 Object

  总结： **手动设置或更新构造函数可能会导致不同且有时令人困惑的后果。为了防止它，只需在每个特定情况下定义构造函数的角色。在大多数情况下，不使用构造函数，并且不需要重新分配构造函数。**

## toString(内置[[Class]]属性)
  原型上的toString() 方法返回一个表示该对象的字符串。
  调用该方法，默认返回当前对象的 [[Class]] 。这是一个内部属性，其格式为 [object Xxx] ，其中 Xxx 就是对象的类型。
  对于 Object 对象，直接调用 toString()  就能返回 [object Object] 。而对于其他对象，则需要通过 call / apply 来调用才能返回正确的类型信息。
  ```javascript
    Object.prototype.toString.call('') ;   // [object String]
    Object.prototype.toString.call(1) ;    // [object Number]
    Object.prototype.toString.call(true) ; // [object Boolean]
    Object.prototype.toString.call(Symbol()); //[object Symbol]
    Object.prototype.toString.call(undefined) ; // [object Undefined]
    Object.prototype.toString.call(null) ; // [object Null]
    Object.prototype.toString.call(new Function()) ; // [object Function]
    Object.prototype.toString.call(new Date()) ; // [object Date]
    Object.prototype.toString.call([]) ; // [object Array]
    Object.prototype.toString.call(new RegExp()) ; // [object RegExp]
    Object.prototype.toString.call(new Error()) ; // [object Error]
    Object.prototype.toString.call(document) ; // [object HTMLDocument]
    Object.prototype.toString.call(window) ; //[object global] window 是全局对象 global 的引用
  ```
  注：**但是它不能检测非原生构造函数的构造函数名。**
    jquery中的$.type原理就是通过Object.prototype.toString.call();
  Array.isArray其实也是通过[[Class]]来判定当前是否维数组.

## 总结
  **以上就是已知的4中检测类型的方法，那个方法都不识最完美的，就看你要检测的是那个对应的类型，就用对应的检测方法。**

  我们可以通过四种方式获取数据类型：
  - <font color="blue">typeof运算符，用来区分对象和原始值</font>
  - <font color="blue">instanceof运算符，用来分类对象</font>
  - <font color="blue">constructor，用来创建实例对象的 Object 构造函数的引用</font>
  - <font color="blue">[[Class]]是一个内部属性字符串，用来给对象分类</font>

  > 参考：<https://www.cnblogs.com/onepixel/p/5126046.html>
  > 参考：<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/constructor>
  > 参考：<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/toString>
  > 参考：<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof>
  > 参考：<https://segmentfault.com/a/1190000015264821>