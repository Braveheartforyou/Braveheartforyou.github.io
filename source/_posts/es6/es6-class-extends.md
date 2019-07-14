---
title: es6中实现类(class)和类(class)的继承(extends)
date: 2017-07-31 11:51:20
tags: [ECMAScript6]
categories: [ECMAScript6]
description: es6中Classes和extends结合使用，这里主要说的是extends
---
## 简介
extends关键词被用在类声明或者类表达式上，以创建一个类是另一个类的子类。Class 可以通过extends关键字实现继承。
### 语法
```javascript
    class ChildClass extends ParentClass { ... }
```
### 示例
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
            ChromeSamples.log('"Polygon" is derived from the Greek polus (many) ' +
            'and gonia (angle).');
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
声明一个父类Polygon，constructor(构造函数中)创建了三个属性 name,height,width,
<font color="#ff502c">Square</font>通过<font color="#ff502c">extends</font>关键字,继承了<font color="#ff502c">Polygon</font>类中的所有属性和方法。
子类必须在<font color="#ff502c">constructor</font>方法中调用<font color="#ff502c">super</font>方法，否则新建实例时会报错。这是因为子类没有自己的<font color="#ff502c">this</font>对象，而是继承父类的<font color="#ff502c">this</font>对象，然后对其进行加工。如果不调用<font color="#ff502c">super</font>方法，子类就得不到<font color="#ff502c">this</font>对象。
检验<font color="#ff502c">Square</font>是否<font color="#ff502c">继承</font>自<font color="#ff502c">Polygon</font>可以通过Object.getPrototypeOf()
```javascript
    Object.getPrototypeOf(Square) === Polygon; // true
```
### super关键字
super这个关键字，既可以当作<font color="#ff502c">函数</font>使用，也可以当作<font color="#ff502c">对象</font>使用。
<font color="#ff502c">子类的构造函数必须执行一次super函数，代表调用父类的构造函数，不然会报错。</font>
<font color="#ff502c">super内部的this指的是Square,相当于Polygon.prototype.constructor.call(this);</font>
作为函数时，super()只能用在子类的构造函数之中，用在其他地方就会报错
```javascript
    class Polygon {
        constructor () {
            console.log(new.target.name);
        }
    }
    class Square extends Polygon {
        //super(); // 报错
        constructor () {
            super();
        }
    }
    new Polygon(); // Polygon
    new Square(); // Square
```
super可以作为对象在普通函数中使用，指向父类的原型对象，在静态方法中，指向父类
### 类的prototype属性和__proto__属性
在JavaScript中，每一个对象都有__proto__属性，指向对应的构造函数的prototype属性。