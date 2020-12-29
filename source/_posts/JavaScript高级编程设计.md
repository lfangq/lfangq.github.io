---
title: JavaScript高级编程设计
date: 2020-09-21 10:01:52
tags: [javascript, book]
---
## 1. ECMAScript与JavaScript区别

ECMAScript是一种脚本语言规范，JavaScript则是ECMAScript的实例。
*****
<!-- more -->
## 2. 变量和类型

### 基本数据类型

* null
* undefined
* number
* string
* boolean
* symbol(ES6)

### 引用类型

* object
* function
* array

### null和undefined的区别

* null表示"没有对象"，即该处不应该有值
* undefined表示"缺少值"，就是此处应该有一个值，但是还没有定义

### JavaScript对象的底层数据结构是什么

### Symbol类型在实际开发中的应用、可手动实现一个简单的Symbol

* 应用: 用来定义对象的唯一属性名
* 实现:
```
  function Symbol(value){
    var obj = new Map();
  }
```

### JavaScript中的变量在内存中的具体存储形式

* 基本类型
基本类型是保存在栈内存中的简单数据段，它们的值都有固定的大小，保存在栈空间，通过按值访问
* 引用类型
引用类型是保存在堆内存中的对象，值大小不固定，栈内存中存放的该对象的访问地址指向堆内存中的对象，JavaScript不允许直接访问堆内存中的位置，因此操作对象时，实际操作对象的引用

### 基本类型对应的内置对象，以及他们之间的装箱拆箱操作

* 内置对象：
Object是JavaScript 中所有对象的父对象<br>
数据封装类对象：Object、Array、Boolean、Number 和 String<br>
其他对象：Function、Math、Date、RegExp、Error。<br>
特殊的基本包装类型(String、Number、Boolean)<br>
arguments: 只存在于函数内部的一个类数组对象<br>

* 装箱和拆箱
引用类型有个特殊的基本包装类型，它包括String、Number和Boolean。<br>
作为字符串的a可以调用方法

* 装箱
基本数据类型转化为对应的引用数据类型的操作**，装箱分为隐式装箱和显示装箱
> 在《javascript高级程序设计》中有这样一句话：
每当读取一个基本类型的时候，后台就会创建一个对应的基本包装类型对象，从而让我们能够调用一些方法来操作这些数据。(隐式装箱)
1. 隐式装箱
```javascript
let a = 'sum';
let b = a.indexof('s');
// 上面代码在后台实际的步骤为：
let a = new String('sum');
let b = a.indexof('s');
a = null;
```
在上面的代码中，a是基本类型，它不是对象，不应该具有方法，js内部进行了一些列处理（装箱)， 使得它能够调用方法。在这个基本类型上调用方法，其实是在这个基本类型对象上调用方法。这个基本类型的对象是临时的，它只存在于方法调用那一行代码执行的瞬间，执行方法后立刻被销毁。实现机制：
* 创建String类型的一个实例
* 在实例上调用指定的方法
* 销毁这个实例
2. 显示装箱
通过内置对象可以对Boolean、Object、String等可以对基本类型显示装箱
```javascript
let a = new String('sum')
```
* 拆箱
拆箱和装箱相反，就是把引用类型转化为基本类型的数据，通常通过引用类型的`valueof()`和`toString()`方法实现
```javascript
let name = new String('sun')
let age = new Number(24)
console.log(typeof name) // object
console.log(typeof age) //  object
// 拆箱操作
console.log(typeof age.valueOf()); // number // 24  基本的数字类型
console.log(typeof name.valueOf()); // string  // 'sun' 基本的字符类型
console.log(typeof age.toString()); // string  // '24' 基本的字符类型
console.log(typeof name.toString()); // string  // 'sun' 基本的字符类型
```

### 理解值类型和引用类型

### 至少可以说出三种判断JavaScript数据类型的方式，以及他们的优缺点，如何准确的判断数组类型

* typeof:
```
缺点： 无法区别对对象和数字、null返回的都是object
```
* instanceof: 
```

```
* constructor:

### 可能发生隐式类型转换的场景以及转换原则，应如何避免或巧妙应用

* string转number
```
var string = "string";
var number = string.length;
```

### 出现小数精度丢失的原因，JavaScript可以存储的最大数字、最大安全数字，JavaScript处理大数字的方法、避免精度丢失的方法

*****
## 3. 原型和原型链
* 1.理解原型设计模式以及JavaScript中的原型规则

* 2.instanceof的底层实现原理，手动实现一个instanceof
instanceof用来判断一个构造函数的prototype属性所指向的对象是否存在另外一个要检测对象的原型链上 

* 4.实现继承的几种方式以及他们的优缺点

* 5.至少说出一种开源项目(如Node)中应用原型继承的案例

* 6.可以描述new一个对象的详细过程，手动实现一个new操作符

* 7.理解es6 class构造以及继承的底层实现原理

*****
## 4. 作用域和闭包
* 1.理解词法作用域和动态作用域

* 2.理解JavaScript的作用域和作用域链

* 3.理解JavaScript的执行上下文栈，可以应用堆栈信息快速定位问题

* 4.this的原理以及几种不同使用场景的取值

* 5.闭包的实现原理和作用，可以列举几个开发中闭包的实际应用

* 6.理解堆栈溢出和内存泄漏的原理，如何防止

* 7.如何处理循环的异步操作

* 8.理解模块化解决的实际问题，可列举几个模块化方案并理解其中原理

*****
## 5. 执行机制
* 1.为何try里面放return，finally还会执行，理解其内部机制

* 2.JavaScript如何实现异步编程，可以详细描述EventLoop机制

* 3.宏任务和微任务分别有哪些

* 4.可以快速分析一个复杂的异步嵌套逻辑，并掌握分析方法

* 5.使用Promise实现串行

* 6.Node与浏览器EventLoop的差异

* 7.如何在保证页面运行流畅的情况下处理海量数据

*****
## 6. 语法和API
* 1.理解ECMAScript和JavaScript的关系

* 2.熟练运用es5、es6提供的语法规范，

* 3.熟练掌握JavaScript提供的全局对象（例如Date、Math）、全局函数（例如decodeURI、isNaN）、全局属性（例如Infinity、undefined）

* 4.熟练应用map、reduce、filter 等高阶函数解决问题

* 5.setInterval需要注意的点，使用settimeout实现setInterval

* 6.JavaScript提供的正则表达式API、可以使用正则表达式（邮箱校验、URL解析、去重等）解决常见问题

* 7.JavaScript异常处理的方式，统一的异常处理方案
