---
title: 你不知道的JavaScript--this调用机制
date: 2021-01-16 10:01:52
tags: [javascript, book]
---

### this是什么

当一个函数被调用时，会创建一个活动记录（有时候也称为执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。 this 就是记录的其中一个属性，会在函数执行的过程中用到。

作用：this是JavaScript的一种机制， this 提供了一种更优雅的方式来隐式“传递”一个对象引用，因此可以将 API 设计得更加简洁并且易于复用。

this的上下文取决于函数调用时的各种条件。 this的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。

<!--more-->

### 调用位置

在理解 this 的绑定过程之前，首先要理解调用位置：调用位置就是函数在代码中被调用的位置（而不是声明的位置）。只有仔细分析调用位置才能回答这个问题：这个 this 到底引用的是什么？

例1:
```js
function baz() {
  // 当前调用栈是：baz
  // 因此，当前调用位置是全局作用域
  console.log( "baz" );
  bar(); // <-- bar 的调用位置
}

function bar() {
  // 当前调用栈是 baz -> bar
  // 因此，当前调用位置在 baz 中
  console.log( "bar" );
  foo(); // <-- foo 的调用位置
}

function foo() {
  // 当前调用栈是 baz -> bar -> foo
  // 因此，当前调用位置在 bar 中
  console.log( "foo" );
}

baz(); // <-- baz 的调用位置
```

### this的绑定规则

#### 1. 默认绑定

例2:
```js
function foo() {
  console.log( this.a );
}

var a = 2;
foo(); // 2
```
在例2中可以看到当调用 `foo()` 时, 在`foo()`前面没有任何的对象调用该函数，从而把`this`绑定到全局对象或`undefined`上(取决于是否是严格模式)，这种绑定到全局对象的方式就是`默认绑定`。

> 注意<br>
> 在严格模式下( strict mode )，全局的 `this`会绑定到 undefined

#### 2. 隐式绑定

例3:
```js
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2,
  foo: foo
};
obj.foo(); // 2
```
在例3中，在`foo()`前面有调用对象且`this`默认指向该对象的方式就是`隐式绑定`。

例4:
```js
function foo() {
  console.log( this.a );
}
var obj2 = {
  a: 42,
  foo: foo
};
var obj1 = {
  a: 2,
  obj2: obj2
};
obj1.obj2.foo(); // 42
```
在例4中可以看出对象属性引用链中只有最顶层或者说最后一层会影响调用位置。

隐式丢失

一个最常见的 `this` 绑定问题就是被隐式绑定的函数会丢失绑定对象，也就是说它会应用默认绑定，从而把 `this` 绑定到全局对象或者 `undefined` 上，取决于是否是严格模式。

例5:
```js
function foo() {
  console.log( this.a );
}
var obj = {
  a: 2,
  foo: foo
};
var bar = obj.foo; // 函数别名！
var a = "oops, global"; // a 是全局对象的属性
bar(); // "oops, global"
```
在例5中，虽然 `bar` 是 `obj.foo` 的一个引用，但是实际上，它引用的是 foo 函数本身，因此此时的`bar()` 其实是一个不带任何修饰的函数调用，因此应用了默认绑定。

一种更微妙、更常见并且更出乎意料的情况发生在传入回调函数时：
例6:
```js
function foo() {
  console.log( this.a );
}
function doFoo(fn) {
  // fn 其实引用的是 foo
  fn(); // <-- 调用位置！
}
var obj = {
  a: 2,
  foo: foo
};
var a = "oops, global"; // a 是全局对象的属性
doFoo( obj.foo ); // "oops, global"
```

如果把函数传入语言内置的函数而不是传入你自己声明的函数，会发生什么呢？结果是一样的，没有区别：

例7:
```js
function foo() {
  console.log( this.a );
}
var obj = {
  a: 2,
  foo: foo
};
var a = "oops, global"; // a 是全局对象的属性
setTimeout( obj.foo, 100 ); // "oops, global"
```
JavaScript 环境中内置的 `setTimeout()` 函数实现和下面的伪代码类似：

例8:
```js
function setTimeout(fn,delay) {
  // 等待 delay 毫秒
  fn(); // <-- 调用位置！
}
```

#### 3. 显式绑定

例9:
```js
function foo() {
  console.log( this.a );
}
var obj = {
  a:2
};
foo.call( obj ); // 2
```
在例9中，通过 `foo.call(..)` ，我们可以在调用 `foo` 时强制把它的 `this` 绑定到 `obj` 上, 这种可以直接指定 `this` 的绑定对象的方式就是`显式绑定`。

> 注意<br>
> 1.如果你传入了一个原始值（字符串类型、布尔类型或者数字类型）来当作 this 的绑定对象，这个原始值会被转换成它的对象形式（也就是 new String(..) 、 new Boolean(..) 或者new Number(..) ）。这通常被称为“装箱”。<br>
> 2.显式绑定仍然无法解决我们之前提出的丢失绑定问题

可以改变this的内置函数有:
- call
- apply
- bind

硬绑定(显式绑定变种)

作用: 防止丢失绑定问题
缺点：绑定之后不能通过显式绑定或隐式绑定改变this

硬绑定的典型应用场景就是创建一个包裹函数，传入所有的参数并返回接收到的所有值：
```js
function foo(something) {
  console.log( this.a, something );
  return this.a + something;
}
var obj = {
  a:2
};
var bar = function() {
  return foo.apply( obj, arguments );
};
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

另一种使用方法是创建一个 i 可以重复使用的辅助函数：
```js
function foo(something) {
  console.log( this.a, something );
  return this.a + something;
}
// 简单的辅助绑定函数
function bind(fn, obj) {
  return function() {
    return fn.apply( obj, arguments );
  };
}
var obj = {
  a:2
};
var bar = bind( foo, obj );
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

由于硬绑定是一种非常常用的模式，所以在 ES5 中提供了内置的方法 Function.prototype.
bind
- bind()函数

有两种实现bind的方法，下面第一种不支持使用new调用新创建的构造函数，而第二种支持:

第一种：
```js
// Does not work with `new (funcA.bind(thisArg, args))`
if (!Function.prototype.bind) (function(){
  Function.prototype.bind = function() {
    var thatFunc = this;
    var thatArg = [].slice.call(arguments, 1);
    if (typeof thatFunc !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - ' +
             'what is trying to be bound is not callable');
    }
    return function(){
      return thatFunc.apply(thatArg, thatArg.concat.call(thatArg, arguments));
    };
  };
})();
```
第二种：
```js
//  Yes, it does work with `new (funcA.bind(thisArg, args))`
if (!Function.prototype.bind) (function(){
  Function.prototype.bind = function(otherThis) {
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var baseArgs = [].slice.call(arguments, 1);
    var fToBind = this;
    var fNOP = function() {};
    var bound = function() {
      return fToBind.apply(
          fNOP.prototype.isPrototypeOf(this) ? this : otherThis, baseArgs.concat.call(baseArgs, arguments);
      );
    };

    if (this.prototype) {
      // Function.prototype doesn't have a prototype property
      fNOP.prototype = this.prototype;
    }
    fBound.prototype = new fNOP();

    return bound;
  };
})();
```

软绑定

作用： 软绑定就是在具有硬绑定相同的效果下，同时可以灵活的用显式绑定或隐式绑定修改this指向

实现：
```js
Function.prototype.softBind = function(obj) {
  var fn = this;
  // 捕获所有 curried 参数
  var args = [].slice.call(arguments, 1);
  var bound = function () {
    // 改变this指向
    return fn.apply(
      !this || this === (window || global)? obj : this, 
      args.concat.apply(args, arguments)
    );
  }
  bound.prototype = Object.create(fn.prototype);
  return bound;
}

function show(){
  console.log(this.name);
}

var obj1 = {
  name: 'Jhon'
}

var obj2 = {
  name: 'Kim'
}

var show1 = show.softBind(obj1);
show1(); // => Jhon
show1.apply(obj2); // => Kim
show1(); // => Jhon
```

#### 4. new 绑定

例10：
```js
function foo(a) {
  this.a = a;
}
var bar = new foo(2);
console.log( bar.a ); // 2
```
使用 `new` 来调用 `foo(..)` 时，我们会构造一个新对象并把它绑定到 `foo(..)` 调用中的 this
上。 `new` 是最后一种可以影响函数调用时 `this` 绑定行为的方法，我们称之为 `new 绑定`。

使用 `new` 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作
1. 创建（或者说构造）一个全新的对象。
2. 这个新对象会被执行 [[ 原型 ]] 连接。
3. 这个新对象会绑定到函数调用的 this 。
4. 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象。

实现new:
```js
Function.prototype.createObject = function(){
  // 创建（或者说构造）一个全新的对象
  var newObj = {};
  var args = [].shift.call(argments, 0);
  // 这个新对象会被执行 [[ 原型 ]] 连接
  newObj.__proto__ = args.prototype;
  // 这个新对象会绑定到函数调用的 this
  var result = args.apply(newObj, arguments);
  // 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象
  return typeof result === "object" ? result : newObj;
}
```

#### 5. 优先级
1. new 绑定
2. 显式绑定
3. 隐式绑定
4. 默认绑定

例11：
```js
function foo() {
  console.log( this.a );
}
var obj1 = {
  a: 2,
  foo: foo
};
var obj2 = {
  a: 3,
  foo: foo
};
obj1.foo(); // 2
obj2.foo(); // 3
obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```
从例11中可以看出显式绑定比隐式绑定优先级更高

例12：
```js
function foo(something) {
  this.a = something;
}
var obj1 = {
  foo: foo
};
var obj2 = {};
obj1.foo( 2 );
console.log( obj1.a ); // 2
obj1.foo.call( obj2, 3 );
console.log( obj2.a ); // 3
var bar = new obj1.foo( 4 );
console.log( obj1.a ); // 2
console.log( bar.a ); // 4
```
从例12中可以看出new绑定比隐式绑定优先级更高

new 和 call / apply 无法一起使用，因此无法通过 new foo.call(obj1) 来直接
进行测试。但是我们可以使用硬绑定来测试它俩的优先级。

例13：
```js
function foo(something) {
  this.a = something;
}
var obj1 = {};
var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2
var baz = new bar(3);
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```
从例13中可以看出 bar 被硬绑定到 obj1 上，但是 new bar(3) 并没有像我们预计的那样把 obj1.a修改为 3。相反， new 修改了硬绑定（到 obj1 的）调用 bar(..) 中的 this 。因为使用了new 绑定，我们得到了一个名字为 baz 的新对象，并且 baz.a 的值是 3, 所以new绑定比显式绑定优先级更高。

> 总结：可以根据优先级来判断函数在某个调用位置应用的是哪条规则

#### 间接引用
例14：
```js
function foo() {
  console.log( this.a );
}
var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };
o.foo(); // 3
(p.foo = o.foo)(); // 2
```
赋值表达式 p.foo = o.foo 的返回值是目标函数的引用，因此调用位置是 foo() 而不是
p.foo() 或者 o.foo()。根据我们之前说过的，这里会应用默认绑定。

#### 箭头函数

箭头函数不使用 this 的四种标准规则，而是根据外层（函数或者全局）作用域来决
定 this。

```js
function foo() {
  // 返回一个箭头函数
  return (a) => {
    //this 继承自 foo()
    console.log( this.a );
  };
}
var obj1 = {
  a:2
};
var obj2 = {
  a:3
};
var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, 不是 3 ！
```
foo() 内部创建的箭头函数会捕获调用时 foo() 的 this 。由于 foo() 的 this 绑定到 obj1 ，bar （引用箭头函数）的 this 也会绑定到 obj1 ，箭头函数的绑定无法被修改。（ new 也不
行！）








