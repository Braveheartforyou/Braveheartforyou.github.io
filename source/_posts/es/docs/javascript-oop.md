---
title: es6中 class（类）的用法
date: 2017-07-26 18:11:13
tags: [ECMAScript6, Extends]
categories: [JavaScript]
description: javascript中实现面向对象编程简单实现和理解
---

## 简介

javascript 是一种基于对象的语言，你遇到的所有东西几乎都是对象。但是,它又不是一种真正的面向对象编程（OOP）语言，因为它的语法中没有 class(类)。
es6 中新提出了类（Class）概念，JavaScript 中的类只是 JavaScript 中的类只是 JavaScript 现有基于原型的继承的一种语法包装（语法糖）,他能让我们用建明的语法实现继承。

## 基本语法

声明一个 Person "类"，它内部有一个 constructor 方法，这个就是构造方法，而 this 关键字则代表实例对象。
使用 new 关键子创建，是跟构造函数的用法完全一致。构造函数和普通的 JavaScript 中的构造函数是一个样的。

```javascript
// 声明类
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  getPerson() {
    return '名字：' + this.name + '年龄：' + this.age;
  }
}
var oObj = new Person('小明', 20);
oObj.getPerson(); // "名字：小名年龄：20"
```

### 实例

#### 使用构造函数

```javascript
//父类
class Polygon {
  // ..and an (optional) custom class constructor. If one is
  // not supplied, a default constructor is used instead:
  // constructor() { }
  // 只能有一个 constructor 不能多次定义、如果没有声明 默认定义 constructor() { }
  constructor(height, width) {
    this.name = 'Polygon';
    this.height = height;
    this.width = width;
  }

  // Simple class instance methods using short-hand method
  // declaration
  sayName() {
    ChromeSamples.log('Hi, I am a ', this.name + '.');
  }

  sayHistory() {
    ChromeSamples.log(
      '"Polygon" is derived from the Greek polus (many) ' + 'and gonia (angle).'
    );
  }

  // We will look at static and subclassed methods shortly
}
class Square extends Polygon {
  constructor(length) {
    // 这里把length传参给父类的构造方法
    // 作为父类Polygon的宽和高
    super(length, length);
    // 备注：在衍生类中使用this前必须先调用super()方法
    // 忽视这一点将会导致一个引用错误
    this.name = 'Square';
  }

  get area() {
    return this.height * this.width;
  }

  set area(value) {
    this.area = value;
  }
}
let s = new Square(5);
s.sayName();
ChromeSamples.log('The area of this square is ' + s.area);
```

构造函数的 prototype(原型)属性，类的所有方法都定义在类的 prototype 属性上面。

```javascript
class B {}
let b = new B();
b.constructor === B.prototype.constructor; // true
```

### 子类的使用

可以通过 extends 关键字来实现继承

### constructor 方法

一个类必须有 constructor 方法,默认添加。默认返回 this（实例对象），也可指定别的对象。
类必须使用 new 调用，否则会报错。

```javascript
class Foo {
  constructor() {
    return Object.create(null);
  }
}

Foo();
// TypeError: Class constructor Foo cannot be invoked without 'new'
```

### 类的实例对象

类的所有实例共享一个原型对象。通过实例的**proto**属性为“类”添加方法。

```javascript
var oObj = new Person('小明', 20);
var oObj1 = new Person('小明', 20);
oObj.__proto__ === oObj1.__proto__; // true
oObj.__proto__.des = function () {
  return '学习';
};
oObj.des(); // 学习
oObj1.des(); // 学习
var oObj2 = new Preson('小红', 18);
oObj2.des(); // 学习
```

## extends

extends 关键词被用在类声明或者类表达式上，以创建一个类是另一个类的子类。

### 描述

extends 关键词用来创建一个普通类或者内建对象的子类。
扩展的.prototype 必须是一个 Object 或者 null。

### 使用 extends

```javascript
//父类
class Polygon {
  // ..and an (optional) custom class constructor. If one is
  // not supplied, a default constructor is used instead:
  // constructor() { }
  // 只能有一个 constructor 不能多次定义、如果没有声明 默认定义 constructor() { }
  constructor(height, width) {
    this.name = 'Polygon';
    this.height = height;
    this.width = width;
  }

  // Simple class instance methods using short-hand method
  // declaration
  sayName() {
    ChromeSamples.log('Hi, I am a ', this.name + '.');
  }

  sayHistory() {
    ChromeSamples.log(
      '"Polygon" is derived from the Greek polus (many) ' + 'and gonia (angle).'
    );
  }

  // We will look at static and subclassed methods shortly
}
class Square extends Polygon {
  constructor(length) {
    // 这里把length传参给父类的构造方法
    // 作为父类Polygon的宽和高
    super(length, length);
    // 备注：在衍生类中使用this前必须先调用super()方法
    // 忽视这一点将会导致一个引用错误
    this.name = 'Square';
  }

  get area() {
    return this.height * this.width;
  }

  set area(value) {
    this.area = value;
  }
}
let s = new Square(5);
s.sayName();
ChromeSamples.log('The area of this square is ' + s.area);
```

### 使用 extends 扩展内建对象

这个示例继承了 Date 对象。 你可以从实战演示看到这个例子。

```javascript
class myDate extends Date {
  constructor() {
    super();
  }

  getFormattedDate() {
    var months = [
      'Jan',
      'Feb',
      'Mar',
      'Apr',
      'May',
      'Jun',
      'Jul',
      'Aug',
      'Sep',
      'Oct',
      'Nov',
      'Dec'
    ];
    return (
      this.getDate() + '-' + months[this.getMonth()] + '-' + this.getFullYear()
    );
  }
}
```

### 扩展 null

可以像扩展普通类一样扩展 null，但是新对象的原型将不会继承 Object.prototype.

```javascript
class nullExtends extends null {
  constructor() {}
}
Object.getPrototypeOf(nullExtends); // Function.prototype
Object.getPrototypeOf(nullExtends.prototype); // null
```

## 注意事项

### class 表达式

```javascript
let person = new (class {
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(this.name);
  }
})('张三');

person.sayName(); // "张三"
```

### 不存在变量提升

```javascript
new Foo(); // ReferenceError
class Foo {}
```

### class 的取值函数（getter）和存值函数（setter）

与 ES5 一样，在“类”的内部可以使用 get 和 set 关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。

```javascript
class MyClass {
  constructor() {
    // ...
  }
  get prop() {
    return 'getter';
  }
  set prop(value) {
    console.log('setter: ' + value);
  }
}

let inst = new MyClass();

inst.prop = 123;
// setter: 123

inst.prop;
// 'getter'
```

### Class 的静态方法

类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上 static 关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod(); // 'hello'

var foo = new Foo();
foo.classMethod();
// TypeError: foo.classMethod is not a function
class Bar extends Foo {}
Bar.classMethod(); // 'hello'
```

如果在实例上调用静态方法，会抛出一个错误，表示不存在该方法。
父类的静态方法，可以被子类继承。
