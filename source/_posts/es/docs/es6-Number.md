---
title: ES6中Number一些新增的方法和属性
date: 2017-08-08 14:45:32
tags: [ECMAScript6]
categories: [ECMAScript6]
description: ES6中Number新增了一些方法如Number.isFinite()、Number.isInteger()、Number.isNaN()等等
---

## 概念

JavaScript 的 Number 对象是经过封装的能让你处理数字值的对象。Number 对象由 Number() 构造器创建。

## 描述

Number 对象主要用于：

- `如果参数无法被转为数字，则返回 NaN.`
- `早非构造器上下文中（如：没有 new 操作符），Number 能被用来执行类型转换`

## 属性

### `Number.EPSTION`

两个可表示(representable)数之间的最小间隔。
`EPSILON 属性的值接近于 2.2204460492503130808472633361816E-16，或者 2-52。`

```javascript
x = 0.2;
y = 0.3;
z = 0.1;
equal = Math.abs(x - y + z) < Number.EPSILON;
```

### `Number.MAX_SAFE_INTEGER Number.MIN_SAFE_INTEGER`

Number.MAX_SAFE_INTEGER 常量表示在 JavaScript 中最大的安全整数（maxinum safe integer)（253 - 1）。
Number.MIN_SAFE_INTEGER 代表在 JavaScript 中最小的安全的 integer 型数字 (-(253 - 1)).
`MAX_SAFE_INTEGER 常量值为 9007199254740991。`
`MIN_SAFE_INTEGER 的值是-9007199254740991.`

```javascript
Number.MAX_SAFE_INTEGER; // 9007199254740991
Math.pow(2, 53) - 1; // 9007199254740991
Number.MIN_SAFE_INTEGER - // -9007199254740991
  (Math.pow(2, 53) - 1); // -9007199254740991
```

### `Number.MAX_VALUE Number.MIN_VALUE`

Number.MAX_VALUE 属性表示在 JavaScript 里所能表示的最大数值
Number.MIN_VALUE 属性表示在 JavaScript 中所能表示的最小的正值
`MAX_VALUE 属性值接近于 1.79E+308。大于 MAX_VALUE 的值代表 "Infinity"。`
`MIN_VALUE 的值约为 5e-324。小于 MIN_VALUE ("underflow values") 的值将会转换为 0。`
因为 MAX_VALUE 是 Number 对象的一个静态属性，所有应该直接使用，Number.MAX_VALUE ，而不是作为一个创建的 Number 实例的属性。

```javascript
if (num1 * num2 <= Number.MAX_VALUE) {
  func1();
} else {
  func2();
}

if (num1 / num2 >= Number.MIN_VALUE) {
  func1();
} else {
  func2();
}
```

### `Number.NaN Number.prototype`

Number.NaN 表示“非数字”（Not-A-Number）。和 NaN 相同。
Number.prototype 属性表示 Number 构造函数的原型。
`所有 Number 实例都继承自 Number.prototype。修改 Number 构造函数的原型对象会影响到所有 Number 实例。.`
`不必创建一个 Number 实例来访问该属性，使用 Number.NaN 来访问该静态属性。`

#### 属性

`constructor`
返回创建该实例对象的构造函数。默认为 Number 对象。

```javascript
    Properties inherited from Object:
    __parent__, __proto__
```

## 方法

实例本身的方法 Number.isFinite()、Number.isInteger()、Number.isNaN()、Number.isSafeInteger()、Number.parseFloat()、Number.parseInt()
prototype 原型上的方法 Number.prototype.toExponential()、Number.prototype.toFixed()、Number.prototype.toLocaleString()、Number.prototype.toPrecision()、Number.prototype.toSource()、Number.prototype.toString()、Number.prototype.valueOf()

### Number.isFinite() Number.isSafeInteger()

Number.isFinite() 方法用来检测传入的参数是否是一个有穷数（finite number）。
Number.isSafeInteger() 方法用来判断传入的参数值是否是一个“安全整数”（safe integer）。一个安全整数是一个符合下面条件的整数

#### 语法

```javascript
Number.isFinite(value);
// 参数
// value
// 要被检测有穷性的值
Number.isFinite(Infinity); // false
Number.isFinite(NaN); // false
Number.isFinite(-Infinity); // false

Number.isFinite(0); // true
Number.isFinite(2e64); // true

Number.isFinite('0'); // false, 全局函数 isFinite('0') 会返回 true

Number.isSafeInteger(testValue);
// 参数
// testValue
// 需要检测的参数。
// 返回值
// 一个布尔值 表示给定的值是否是一个安全整数（safe integer）。
Number.isSafeInteger(3); // true
Number.isSafeInteger(Math.pow(2, 53)); // false
Number.isSafeInteger(Math.pow(2, 53) - 1); // true
Number.isSafeInteger(NaN); // false
Number.isSafeInteger(Infinity); // false
Number.isSafeInteger('3'); // false
Number.isSafeInteger(3.1); // false
Number.isSafeInteger(3.0); // true
```

#### 注意

和`全局的 isFinite()`函数相比，`这个方法不会强制将一个非数值的参数转换成数值`，这就意味着，只有数值类型的值，且是有穷的（finite），才返回 true。
Number.isSafeInteger() 安全整数范围为 -(253 - 1)到 253 - 1 之间的整数，包含 -(253 - 1)和 253 - 1。

### Number.isInteger()

Number.isInteger() 方法用来判断给定的参数是否为整数。
如果被检测的值是整数，则返回 true，否则返回 false。注意 NaN 和正负 Infinity 不是整数。

#### 语法

```javascript
Number.isInteger(value);
// 参数
// value
// 要判断此参数是否为整数
// 返回值
// 判断给定值是否是整数的 Boolean 值。
Number.isInteger(0); // true
Number.isInteger(1); // true
Number.isInteger(-100000); // true
Number.isInteger(0.1); // false
Number.isInteger(Math.PI); // false

Number.isInteger(Infinity); // false
Number.isInteger(-Infinity); // false
Number.isInteger('10'); // false
Number.isInteger(true); // false
Number.isInteger(false); // false
Number.isInteger([1]); // false
```

### Number.isNaN()

Number.isNaN() 方法确定传递的值是否为 NaN 和其类型是 Number。它是原始的全局 isNaN()的更强大的版本。

#### 语法

```javascript
Number.isNaN(value);
// 参数
// value
// 要被检测是否是 NaN 的值。
// 返回值
// 一个布尔值，表示给定的值是否是 NaN。
Number.isNaN(NaN); // true
Number.isNaN(Number.NaN); // true
Number.isNaN(0 / 0); // true

// 下面这几个如果使用全局的 isNaN() 时，会返回 true。
Number.isNaN('NaN'); // false，字符串 "NaN" 不会被隐式转换成数字 NaN。
Number.isNaN(undefined); // false
Number.isNaN({}); // false
Number.isNaN('blabla'); // false

// 下面的都返回 false
Number.isNaN(true);
Number.isNaN(null);
Number.isNaN(37);
Number.isNaN('37');
Number.isNaN('37.37');
Number.isNaN('');
Number.isNaN(' ');
```

#### 描述

在 JavaScript 中，NaN 最特殊的地方就是，`我们不能使用相等运算符（== 和 ===）来判断一个值是否是 NaN`，因为`NaN == NaN` 和`NaN === NaN`都会返回`false`。因此，必须要有一个判断值是否是 NaN 的方法。
`同样：`
和全局函数 isNaN() 相比，该方法不会强制将参数转换成数字，只有在参数是真正的数字类型，且值为 NaN 的时候才会返回 true。

### Number.parseInt() Number.parseFloat()

Number.parseFloat() 方法可以把一个字符串解析成浮点数。该方法与全局的 parseFloat() 函数相同，并且处于 ECMAScript 6 规范中（用于全局变量的模块化）。
Number.parseInt() 方法可以根据给定的进制数把一个字符串解析成整数。

#### 语法

```javascript
Number.parseInt(string, radix);
// 参数
// string
// 被解析的值。如果不是一个字符串，则将其转换为字符串。字符串开头的空白符将会被忽略。
// radix
// 一个整数值，指定转换中采用的基数。总是指定该参数可以保证结果可预测。当忽略该参数时，不同的实现环境可能产生不同的结果。
// 返回值
// 返回解析后的整数值。 如果被解析参数的第一个字符无法被转化成数值类型，则返回 NaN。

Number.parseFloat(string);
// 参数
// string
// 被解析的字符串。
Number.parseFloat('3.14'); // 3.14
Number.parseFloat('314e-2'); // 3.14
Number.parseFloat('FF2'); // NaN
```

#### 描述

`该方法和全局的 parseInt() 函数是同一个函数：`
`该方法和全局的 parseFloat() 函数是同一个函数：`

```javascript
    验证Number.parseInt、Number.parseFloat与全局的 parseInt、parseFloat是相同的
    Number.parseInt === parseInt; // true
    Number.parseFloat === parseFloat; // true

    parseInt("F", 16); // 15
    parseInt("17", 8); // 15
    parseInt("15", 10); // 15
    parseInt("Hello", 8); // NaN
    parseInt("546", 2); // NaN
```

parseInt 函数将其第一个参数转换为字符串，解析它，并返回一个整数或 NaN。如果不是 NaN，返回的值将是作为指定`基数（基数）`中的数字的第一个参数的整数。
例如：radix`参数为 10` 将会把第一个参数看作是一个数的`十进制`表示,如果不属于 radix 参数所指定的基数中的字符那么该字符和气候的字符创都将被忽略。
在没有指定基数，或者基数为 0 的情况下，JavaScript 作如下处理：

- 如果字符串 string 以"0x"或者"0X"开头, 则基数是 16 (16 进制).
- 如果字符串 string 以"0"开头, 基数是 8（八进制）或者 10（十进制），那么具体是哪个基数由实现环境决定。ECMAScript 5 规定使用 10，但是并不是所有的浏览器都遵循这个规定。因此，永远都要明确给出 radix 参数的值。
- 如果字符串 string 以其它任何值开头，则基数是 10 (十进制)。
  需要注意的是：
  `如果第一个字符不能被转换成数字，parseInt 返回 NaN。`

### Number.prototype.toFixed()、Number.prototype.toPrecision()

toFixed() 方法使用定点表示法来格式化一个数。
toPrecision() 方法以指定的精度返回该数值对象的字符串表示。

#### 语法

```javascript
numObj.toFixed(digits);
// 参数
// digits
// 小数点后数字的个数；介于 0 到 20 （包括）之间，实现环境可能支持更大范围。如果忽略该参数，则默认为 0。
// 返回值
// 一个数值的字符串表现形式，不使用指数记数法，而是在小数点后有 digits 位数字。

var numObj = 12345.6789;
numObj.toFixed(); // 返回 "12346"：进行四舍五入，不包括小数部分
numObj.toFixed(1); // 返回 "12345.7"：进行四舍五入
numObj.toFixed(6); // 返回 "12345.678900"：用0填充
(1.23e20).toFixed(2); // 返回 "123000000000000000000.00"
(1.23e-10).toFixed(2); // 返回 "0.00"
(2.34).toFixed(1); // 返回 "2.3"
-(2.34).toFixed(1); // 返回 -2.3 （由于操作符优先级，负数不会返回字符串）
(-2.34).toFixed(1); // 返回 "-2.3" （若用括号提高优先级，则返回字符串）

numObj.toPrecision(precision);
// 参数
// precision
// 可选。一个用来指定有效数个数的整数。
// 返回值
// 以定点表示法或指数表示法表示的一个数值对象的字符串表示，四舍五入到 precision 参数指定的显示数字位数。
var numObj = 5.123456;
console.log('numObj.toPrecision()  is ' + numObj.toPrecision()); //输出 5.123456
console.log('numObj.toPrecision(5) is ' + numObj.toPrecision(5)); //输出 5.1235
console.log('numObj.toPrecision(2) is ' + numObj.toPrecision(2)); //输出 5.1
console.log('numObj.toPrecision(1) is ' + numObj.toPrecision(1)); //输出 5

// 注意：在某些情况下会以指数表示法返回
console.log((1234.5).toPrecision(2)); // "1.2e+3"
```

该数值在必要时进行四舍五入，另外在必要时会用 0 来填充小数部分，以便小数部分有指定的位数。

### Number.prototype.toLocaleString()、Number.prototype.toString()、Number.prototype.valueOf()

toString() 方法返回指定 Number 对象的字符串表示形式。
valueOf() 方法返回一个被 Number 对象包装的原始值。

#### 语法

```javascript
numObj.toString([radix]);
// 参数
// radix
// 指定要用于数字到字符串的转换的基数(从2到36)。如果未指定 radix 参数，则默认值为 10。
var count = 10;
console.log(count.toString()); // 输出 '10'
console.log((17).toString()); // 输出 '17'
console.log((17.2).toString()); // 输出 '17.2'
var x = 6;
console.log(x.toString(2)); // 输出 '110'
console.log((254).toString(16)); // 输出 'fe'
console.log((-10).toString(2)); // 输出 '-1010'

numObj.valueOf();
var numObj = new Number(10);
console.log(typeof numObj); // object

var num = numObj.valueOf();
console.log(num); // 10
console.log(typeof num); // number
```

Number 对象覆盖了 Object 对象上的 toString() 方法，它不是继承的 Object.prototype.toString()。对于 Number 对象，toString() 方法以指定的基数返回该对象的字符串表示。
valueOf()该方法通常是由 JavaScript 引擎在内部隐式调用的，而不是由用户在代码中显式调用的
