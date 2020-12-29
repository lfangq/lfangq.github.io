---
title: Airbnb JavaScript 风格指南阅读笔记
date: 2020-10-09 10:01:52
tags: [javascript]
---

### Types
<!-- more -->

#### 1.1 基本类型: 你可以直接获取到基本类型的值

- `string`
- `number`
- `boolean`
- `null`
- `undefined`
- `symbol`

```javascript
const foo = 1
let bar = foo

bar = 9

console.log(foo, bar) // => 1,9
```

#### 1.2 复杂类型: 复杂类型赋值是获取到他的引用的值。 相当于传引用

- `object`
- `array`
- `function`

```javascript
const foo = [1, 2]
const bar = foo

bar[0] = 9

console.log(foo[0], bar[0]) // => 9, 9
```

#### 1.3 javascript 判断数据类型方法

- typeof

```javascript
typeof '' // => string
typeof 1 // => number
typeof true // => boolean
typeof Symbol() // => symbol
typeof undefined // => undefined
typeof null // => object
typeof new Function() // => function
typeof new Date() // => object
typeof [] // => object
typeof new RegExp() // => object
typeof new Error() // => object
typeof document // => object
typeof window // => object
```

引用类型，除了 function 返回 function 类型外，其他都返回 object。但引用类型中的数组、日期、正则都有属于自己的具体类型，而 typeof 对于这些类型的处理，只返回了处于其原型链最顶端的 Object 类型。

- toString

```javascript
Object.prototype.toString.call('') // =>[object String]
Object.prototype.toString.call(1) // => [object Number]
Object.prototype.toString.call(true) // => [object Boolean]
Object.prototype.toString.call(Symbol()) // =>[object Symbol]
Object.prototype.toString.call(undefined) // => [object Undefined]
Object.prototype.toString.call(null) // => [object Null]
Object.prototype.toString.call(new Function()) // => [object Function]
Object.prototype.toString.call(new Date()) // => [object Date]
Object.prototype.toString.call([]) // => [object Array]
Object.prototype.toString.call(new RegExp()) // => [object RegExp]
Object.prototype.toString.call(new Error()) // => [object Error]
Object.prototype.toString.call(document) // => [object HTMLDocument]
Object.prototype.toString.call(window) // =>[object global] window 是全局对象 global 的引用
```

toString() 是 Object 的原型方法，调用该方法，默认返回当前对象的 [[Class]] 。这是一个内部属性，其格式为 [object Xxx] ，其中 Xxx 就是对象的类型。

对于 Object 对象，直接调用 toString() 就能返回 [object Object] 。而对于其他对象，则需要通过 call / apply 来调用才能返回正确的类型信息。

- constructor

```javascript
''.constructor == String // => true
new Number(1).constructor == Number // => true
true.constructor == Boolean // => true
new Function().constructor == Function // => true
new Date().constructor == Date // => true
new Error().constructor == Error // => true
[].constructor == Array // => true
document.constructor == HTMLDocument // => true
window.constructor == Window // => true
```

constructor 是原型 prototype 的一个属性，当函数被定义时候，js 引擎会为函数添加原型 prototype，并且这个 prototype 中 constructor 属性指向函数引用， 因此重写 prototype 会丢失原来的 constructor。
缺点：

1. null 和 undefined 无 constructor，这种方法判断不了

2. 还有，如果自定义对象，开发者重写 prototype 之后，原有的 constructor 会丢失，因此，为了规范开发，在重写对象原型时一般都需要重新给 constructor 赋值，以保证对象实例的类型不被篡改

- instanceof

  instanceof 是用来判断 A 是否为 B 的实例，表达式为：A instanceof B，如果 A 是 B 的实例，则返回 true,否则返回 false。 在这里需要特别注意的是：instanceof 检测的是原型

![](0.png)

由上图可以看出[]的原型指向 Array.prototype，间接指向 Object.prototype, 因此 [] instanceof Array 返回 true， [] instanceof Object 也返回 true。

instanceof 只能用来判断两个对象是否属于实例关系， 而不能判断一个对象实例具体属于哪种类型

### References

#### 2.1 所有的赋值都用 const，避免使用`var`. eslint: `prefer-const`, `no-const-assign`

> Why? 因为这个确保你不会改变你的初始值，重复引用会导致 bug 和代码难以理解

```javascript
// bad
var a = 1
var b = 2

// good
const a = 1
const b = 2
```

#### 2.2 如果你一定要对参数重新赋值，那就用`let`，而不是`var`. eslint: `no-var`

> Why? 因为 let 是块级作用域，而 var 是函数级作用域

```javascript
// bad
var count = 1
if (true) {
  count += 1
}

// good, use the let.
let count = 1
if (true) {
  count += 1
}
```

#### 2.3 注意： `let`、`const` 都是块级作用域

```javascript
// const 和 let 都只存在于它定义的那个块级作用域
{
  let a = 1
  const b = 1
}
console.log(a) // ReferenceError
console.log(b) // ReferenceError
```

### Objects

#### 3.1 使用字面值创建对象. eslint: `no-new-object`

```javascript
// bad
const item = new Object()

// good
const item = {}
```

#### 3.2 当创建一个带有动态属性名的对象时，用计算后属性名

> Why? 这可以使你将定义的所有属性放在对象的一个地方.

```javascript
function getKey(k) {
  return `a key named ${k}`
}

// bad
const obj = {
  id: 5,
  name: 'San Francisco',
}
obj[getKey('enabled')] = true

// good getKey('enabled')是动态属性名
const obj = {
  id: 5,
  name: 'San Francisco',
  [getKey('enabled')]: true,
}
```

#### 3.3 用对象方法简写. eslint: `object-shorthand`

```javascript
// bad
const atom = {
  value: 1,

  addValue: function (value) {
    return atom.value + value
  },
}

// good
const atom = {
  value: 1,

  // 对象的方法
  addValue(value) {
    return atom.value + value
  },
}
```

#### 3.4 用属性值缩写. eslint: `object-shorthand`

```javascript
const lukeSkywalker = 'Luke Skywalker'

// bad
const obj = {
  lukeSkywalker: lukeSkywalker,
}

// good
const obj = {
  lukeSkywalker,
}
```

#### 3.5 将你的所有缩写放在对象声明的开始.

> Why? 这样也是为了更方便的知道有哪些属性用了缩写.

```javascript
const anakinSkywalker = 'Anakin Skywalker'
const lukeSkywalker = 'Luke Skywalker'

// bad
const obj = {
  episodeOne: 1,
  twoJediWalkIntoACantina: 2,
  lukeSkywalker,
  episodeThree: 3,
  mayTheFourth: 4,
  anakinSkywalker,
}

// good
const obj = {
  lukeSkywalker,
  anakinSkywalker,
  episodeOne: 1,
  twoJediWalkIntoACantina: 2,
  episodeThree: 3,
  mayTheFourth: 4,
}
```

#### 3.6 只对那些无效的标示使用引号 ''. eslint: `quote-props`

> Why? 通常我们认为这种方式主观上易读。他优化了代码高亮，并且页更容易被许多 JS 引擎压缩。

```javascript
// bad
const bad = {
  'foo': 3,
  'bar': 4,
  'data-blah': 5,
}

// good
const good = {
  foo: 3,
  bar: 4,
  'data-blah': 5,
}
```

#### 3.7 不要直接调用`Object.prototype`上的方法，如`hasOwnProperty`, `propertyIsEnumerable`, `isPrototypeOf`。
> Why? 在一些有问题的对象上， 这些方法可能会被屏蔽掉 - 如：{ hasOwnProperty: false } - 或这是一个空对象Object.create(null)

```javascript
// bad
console.log(object.hasOwnProperty(key));

// good
console.log(Object.prototype.hasOwnProperty.call(object, key));

// best
const has = Object.prototype.hasOwnProperty; // 在模块作用内做一次缓存
/* or */
import has from 'has'; // https://www.npmjs.com/package/has
// ...
console.log(has.call(object, key));
```

#### 3.8 对象浅拷贝时，更推荐使用扩展运算符`[就是...运算符]`，而不是`Object.assign`。获取对象指定的几个属性时，用对象的rest解构运算符`[也是...运算符]`更好。

```javascript
// very bad
const original = { a: 1, b: 2 };
const copy = Object.assign(original, { c: 3 }); // this mutates `original` ಠ_ಠ
delete copy.a; // so does this

// bad
const original = { a: 1, b: 2 };
const copy = Object.assign({}, original, { c: 3 }); // copy => { a: 1, b: 2, c: 3 }

// good es6扩展运算符 ...
const original = { a: 1, b: 2 };
// 浅拷贝
const copy = { ...original, c: 3 }; // copy => { a: 1, b: 2, c: 3 }

// rest 赋值运算符
const { a, ...noA } = copy; // noA => { b: 2, c: 3 }
```

### Arrays

#### 4.1 用字面量赋值。 eslint: `no-array-constructor`

```javascript
// bad
const items = new Array();

// good
const items = [];
```

#### 4.2 用`Array#push` 代替直接向数组中添加一个值。

```javascript
const someStack = [];

// bad
someStack[someStack.length] = 'abracadabra';

// good
someStack.push('abracadabra');
```

#### 4.3 用扩展运算符做数组浅拷贝，类似上面的对象浅拷贝

```javascript
// bad
const len = items.length;
const itemsCopy = [];
let i;

for (i = 0; i < len; i += 1) {
  itemsCopy[i] = items[i];
}

// good
const itemsCopy = [...items];
```

#### 4.4 用 `...` 运算符而不是Array.from来将一个可迭代的对象转换成数组。

```javascript
const foo = document.querySelectorAll('.foo');

// good
const nodes = Array.from(foo);

// best
const nodes = [...foo];
```

#### 4.5 用 `Array.from` 去将一个类数组对象转成一个数组。

```javascript
const arrLike = { 0: 'foo', 1: 'bar', 2: 'baz', length: 3 };

// bad
const arr = Array.prototype.slice.call(arrLike);

// good
const arr = Array.from(arrLike);
```

#### 4.6 用 `Array.from` 而不是 `...` 运算符去做map遍历。 因为这样可以避免创建一个临时数组。

```javascript
// bad
const baz = [...foo].map(bar);

// good
const baz = Array.from(foo, bar);
```

#### 4.7 在数组方法的回调函数中使用 `return` 语句。 如果函数体由一条返回一个表达式的语句组成， 并且这个表达式没有副作用， 这个时候可以忽略`return`，详见 8.2. eslint: `array-callback-return`。

```javascript
// good
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});

// good 函数只有一个语句
[1, 2, 3].map(x => x + 1);

// bad - 没有返回值， 因为在第一次迭代后acc 就变成undefined了
[[0, 1], [2, 3], [4, 5]].reduce((acc, item, index) => {
  const flatten = acc.concat(item);
  acc[index] = flatten;
});

// good
// reduce没有设置初始值，则默认取数组第一个值为初始值
[[0, 1], [2, 3], [4, 5]].reduce((acc, item, index) => {
  const flatten = acc.concat(item);
  acc[index] = flatten;
  return flatten;
});

// bad
inbox.filter((msg) => {
  const { subject, author } = msg;
  if (subject === 'Mockingbird') {
    return author === 'Harper Lee';
  } else {
    return false;
  }
});

// good
inbox.filter((msg) => {
  const { subject, author } = msg;
  if (subject === 'Mockingbird') {
    return author === 'Harper Lee';
  }

  return false;
});
```

#### 4.8 如果一个数组有很多行，在数组的 [ 后和 ] 前断行。 请看下面示例

```javascript
// bad
const arr = [
  [0, 1], [2, 3], [4, 5],
];

const objectInArray = [{
  id: 1,
}, {
  id: 2,
}];

const numberInArray = [
  1, 2,
];

// good
const arr = [[0, 1], [2, 3], [4, 5]];

const objectInArray = [
  {
    id: 1,
  },
  {
    id: 2,
  },
];

const numberInArray = [
  1,
  2,
];
```

### Destructuring

#### 5.1 用对象的解构赋值来获取和使用对象某个或多个属性值。 eslint: `prefer-destructuring`
> Why? 解构使您不必为这些属性创建临时引用

```javascript
// bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;

  return `${firstName} ${lastName}`;
}

// good
function getFullName(user) {
  const { firstName, lastName } = user;
  return `${firstName} ${lastName}`;
}

// best
function getFullName({ firstName, lastName }) {
  return `${firstName} ${lastName}`;
}
```

#### 5.2 用数组解构。

```javascript
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr;
```

#### 5.3 多个返回值用对象的解构，而不是数组解构。
> Why? 你可以在后期添加新的属性或者变换变量的顺序而不会打破原有的调用

```javascript
// bad
function processInput(input) {
  // 然后就是见证奇迹的时刻
  return [left, right, top, bottom];
}

// 调用者需要想一想返回值的顺序
const [left, __, top] = processInput(input);

// good
function processInput(input) {
  // oops， 奇迹又发生了
  return { left, right, top, bottom };
}

// 调用者只需要选择他想用的值就好了
const { left, top } = processInput(input);
```

### Strings

#### 6.1 对`string`用单引号 '' 。 eslint: `quotes`

```javascript
// bad
const name = "Capt. Janeway";

// bad - 样例应该包含插入文字或换行
const name = `Capt. Janeway`;

// good
const name = 'Capt. Janeway';
```

#### 6.2 超过100个字符的字符串不应该用`string`串联成多行。
> Why? 被折断的字符串工作起来是糟糕的而且使得代码更不易被搜索。

```javascript
// bad
const errorMessage = 'This is a super long error that was thrown because \
of Batman. When you stop to think about how Batman had anything to do \
with this, you would get nowhere \
fast.';

// bad
const errorMessage = 'This is a super long error that was thrown because ' +
  'of Batman. When you stop to think about how Batman had anything to do ' +
  'with this, you would get nowhere fast.';

// good
const errorMessage = 'This is a super long error that was thrown because of Batman. When you stop to think about how Batman had anything to do with this, you would get nowhere fast.';
```

#### 6.3 用字符串模板而不是字符串拼接来组织可编程字符串。 eslint: `prefer-template` `template-curly-spacing`
> Why? 模板字符串更具可读性、语法简洁、字符串插入参数。

```javascript
// bad
function sayHi(name) {
  return 'How are you, ' + name + '?';
}

// bad
function sayHi(name) {
  return ['How are you, ', name, '?'].join();
}

// bad
function sayHi(name) {
  return `How are you, ${ name }?`;
}

// good
function sayHi(name) {
  return `How are you, ${name}?`;
}
```

#### 6.4 永远不要在字符串中用`eval()`，他就是潘多拉盒子。 eslint: `no-eval`

#### 6.5 不要使用不必要的转义字符。eslint: `no-useless-escape`
> Why? 反斜线可读性差，所以他们只在必须使用时才出现哦

```javascript
// bad
const foo = '\'this\' \i\s \"quoted\"';

// good
const foo = '\'this\' is "quoted"';

//best
const foo = `my name is '${name}'`;
```

### Functions

#### 7.1 用命名函数表达式而不是函数声明。eslint: `func-style`
> 函数表达式： const func = function () {}

> 函数声明： function func() {}

> Why? 函数声明时作用域被提前了，这意味着在一个文件里函数很容易（太容易了）在其定义之前被引用。这样伤害了代码可读性和可维护性。如果你发现一个函数又大又复杂，这个函数妨碍这个文件其他部分的理解性，这可能就是时候把这个函数单独抽成一个模块了。别忘了给表达式显示的命名，不用管这个名字是不是由一个确定的变量推断出来的，这消除了由匿名函数在错误调用栈产生的所有假设，这在现代浏览器和类似babel编译器中很常见 (Discussion)

```javascript
// bad
function foo() {
  // ...
}

// bad
const foo = function () {
  // ...
};

// good
// lexical name distinguished from the variable-referenced invocation(s)
// 函数表达式名和声明的函数名是不一样的
const short = function longUniqueMoreDescriptiveLexicalFoo() {
  // ...
};
```

#### 7.2 把立即执行函数包裹在圆括号里。 eslint: `wrap-iife`

> Why? immediately invoked function expression = IIFE Why? 一个立即调用的函数表达式是一个单元 - 把它和他的调用者（圆括号）包裹起来，在括号中可以清晰的地表达这些。 Why? 注意：在模块化世界里，你几乎用不着 IIFE

```javascript
// immediately-invoked function expression (IIFE)
(function () {
  console.log('Welcome to the Internet. Please follow me.');
}());
```

#### 7.2.1 IIFE

IIFE: Immediately Invoked Function Expression，意为立即调用的函数表达式，也就是说，声明函数的同时立即调用这个函数。

* 不采用IIFE时的函数声明和函数调用：

``` javascript
function foo(){
  var a = 10;
  console.log(a);
}
foo();
```

* IIFE形式的函数调用：

``` javascript
(function foo(){
  var a = 10;
  console.log(a);
})();
```

函数的声明和IIFE的区别在于，在函数的声明中，我们首先看到的是function关键字，而IIFE我们首先看到的是左边的（。也就是说，使用一对（）将函数的声明括起来，使得JS编译器不再认为这是一个函数声明，而是一个IIFE，即需要立刻执行声明的函数。
两者达到的目的是相同的，都是声明了一个函数foo并且随后调用函数foo。

#### 7.3 不要在非函数块（`if`、`while`等等）内声明函数。把这个函数分配给一个变量。浏览器会允许你这样做，但浏览器解析方式不同，这是一个坏消息。【详见no-loop-func】 eslint: `no-loop-func`

#### 7.4 Note: 在ECMA-262中 [块 `block`] 的定义是： 一系列的语句； 但是函数声明不是一个语句。 函数表达式是一个语句。

```javascript
// bad
if (currentUser) {
  function test() {
    console.log('Nope.');
  }
}

// good
let test;
if (currentUser) {
  test = () => {
    console.log('Yup.');
  };
}
```

#### 7.5 不要用`arguments`命名参数。他的优先级高于每个函数作用域自带的`arguments`对象， 这会导致函数自带的`arguments`值被覆盖

```javascript
// bad
function foo(name, options, arguments) {
  // ...
}

// good
function foo(name, options, args) {
  // ...
}
```

#### 7.6 不要使用`arguments`，用rest语法`...`代替。 eslint: `prefer-rest-params`

> Why? ...明确你想用那个参数。而且rest参数是真数组，而不是类似数组的arguments

```javascript
// bad
function concatenateAll() {
  const args = Array.prototype.slice.call(arguments);
  return args.join('');
}

// good
function concatenateAll(...args) {
  return args.join('');
}
```

#### 7.7 用默认参数语法而不是在函数里对参数重新赋值。

```javascript
// really bad
function handleThings(opts) {
  // 不， 我们不该改arguments
  // 第二： 如果 opts 的值为 false, 它会被赋值为 {}
  // 虽然你想这么写， 但是这个会带来一些细微的bug
  opts = opts || {};
  // ...
}

// still bad
function handleThings(opts) {
  if (opts === void 0) {
    opts = {};
  }
  // ...
}

// good
function handleThings(opts = {}) {
  // ...
}
```

#### 7.8 默认参数避免副作用

> Why? 他会令人迷惑不解， 比如下面这个， a到底等于几， 这个需要想一下。

```javascript
var b = 1;
// bad
function count(a = b++) {
  console.log(a);
}
count();  // 1
count();  // 2
count(3); // 3
count();  // 3
```

#### 7.9 把默认参数赋值放在最后

```javascript
// bad
function handleThings(opts = {}, name) {
  // ...
}

// good
function handleThings(name, opts = {}) {
  // ...
}
```

#### 7.10 不要用函数构造器创建函数。 eslint: `no-new-func`

> Why? 以这种方式创建函数将类似于字符串 eval()，这会打开漏洞。

```javascript
// bad
var add = new Function('a', 'b', 'return a + b');

// still bad
var subtract = Function('a', 'b', 'return a - b');
```

#### 7.11 函数签名部分要有空格。eslint: `space-before-function-paren` `space-before-blocks`

> Why? 统一性好，而且在你添加/删除一个名字的时候不需要添加/删除空格

```javascript
// bad
const f = function(){};
const g = function (){};
const h = function() {};

// good
const x = function () {};
const y = function a() {};
```

#### 7.12 不要改参数. eslint: `no-param-reassign`

> Why? 操作参数对象对原始调用者会导致意想不到的副作用。 就是不要改参数的数据结构，保留参数原始值和数据结构。

```javascript
// bad
function f1(obj) {
  obj.key = 1;
};

// good
function f2(obj) {
  const key = Object.prototype.hasOwnProperty.call(obj, 'key') ? obj.key : 1;
};
```

#### 7.13 不要对参数重新赋值。 eslint: `no-param-reassign`

> Why? 参数重新赋值会导致意外行为，尤其是对 arguments。这也会导致优化问题，特别是在V8里

```javascript
// bad
function f1(a) {
  a = 1;
  // ...
}

function f2(a) {
  if (!a) { a = 1; }
  // ...
}

// good
function f3(a) {
  const b = a || 1;
  // ...
}

function f4(a = 1) {
  // ...
}
```

#### 7.14 用spread操作符...去调用多变的函数更好。 eslint: `prefer-spread`

> Why? 这样更清晰，你不必提供上下文，而且你不能轻易地用apply来组成new

```javascript
// bad
const x = [1, 2, 3, 4, 5];
console.log.apply(console, x);

// good
const x = [1, 2, 3, 4, 5];
console.log(...x);

// bad
new (Function.prototype.bind.apply(Date, [null, 2016, 8, 5]));

// good
new Date(...[2016, 8, 5]);
```

#### 7.15 调用或者书写一个包含多个参数的函数应该像这个指南里的其他多行代码写法一样： 每行值包含一个参数，每行逗号结尾。

```javascript
// bad
function foo(bar,
             baz,
             quux) {
  // ...
}

// good 缩进不要太过分
function foo(
  bar,
  baz,
  quux,
) {
  // ...
}

// bad
console.log(foo,
  bar,
  baz);

// good
console.log(
  foo,
  bar,
  baz,
);
```

### Arrow Functions

#### 8.1 当你一定要用函数表达式（在回调函数里）的时候就用箭头表达式吧。 eslint: `prefer-arrow-callback`, `arrow-spacing`

> Why? 他创建了一个this的当前执行上下文的函数的版本，这通常就是你想要的；而且箭头函数是更简洁的语法

> Why? 什么时候不用箭头函数： 如果你有一个相当复杂的函数，你可能会把这个逻辑移出到他自己的函数声明里。

```javascript
// bad
[1, 2, 3].map(function (x) {
  const y = x + 1;
  return x * y;
});

// good
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});
```

#### 8.2 如果函数体由一个没有副作用的表达式语句组成，删除大括号和`return`。否则，继续用大括号和 `return` 语句。 eslint: `arrow-parens`, `arrow-body-style`

> Why? 语法糖，当多个函数链在一起的时候好读

```javascript
// bad
[1, 2, 3].map(number => {
  const nextNumber = number + 1;
  `A string containing the ${nextNumber}.`;
});

// good
[1, 2, 3].map(number => `A string containing the ${number}.`);

// good
[1, 2, 3].map((number) => {
  const nextNumber = number + 1;
  return `A string containing the ${nextNumber}.`;
});

// good
[1, 2, 3].map((number, index) => ({
  [index]: number
}));

// 表达式有副作用就不要用隐式return
function foo(callback) {
  const val = callback();
  if (val === true) {
    // Do something if callback returns true
  }
}

let bool = false;

// bad
// 这种情况会return bool = true, 不好
foo(() => bool = true);

// good
foo(() => {
  bool = true;
});
```

#### 8.3 万一表达式涉及多行，把他包裹在圆括号里更可读。

> Why? 这样清晰的显示函数的开始和结束

```javascript
// bad
['get', 'post', 'put'].map(httpMethod => Object.prototype.hasOwnProperty.call(
    httpMagicObjectWithAVeryLongName,
    httpMethod
  )
);

// good
['get', 'post', 'put'].map(httpMethod => (
  Object.prototype.hasOwnProperty.call(
    httpMagicObjectWithAVeryLongName,
    httpMethod
  )
));
```

#### 8.4 如果你的函数只有一个参数并且函数体没有大括号，就删除圆括号。否则，参数总是放在圆括号里。 注意： 一直用圆括号也是没问题，只需要配置 “always” option for eslint. eslint: `arrow-parens`

> Why? 这样少一些混乱， 其实没啥语法上的讲究，就保持一个风格。

```javascript
// bad
[1, 2, 3].map((x) => x * x);

// good
[1, 2, 3].map(x => x * x);

// good
[1, 2, 3].map(number => (
  `A long string with the ${number}. It’s so long that we don’t want it to take up space on the .map line!`
));

// bad
[1, 2, 3].map(x => {
  const y = x + 1;
  return x * y;
});

// good
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});
```

#### 8.5 避免箭头函数(`=>`)和比较操作符（`<=`, `>=`）混淆. eslint: `no-confusing-arrow`

```javascript
// bad
const itemHeight = (item) => item.height <= 256 ? item.largeSize : item.smallSize;

// bad
const itemHeight = (item) => item.height >= 256 ? item.largeSize : item.smallSize;

// good
const itemHeight = (item) => (item.height <= 256 ? item.largeSize : item.smallSize);

// good
const itemHeight = (item) => {
  const { height, largeSize, smallSize } = item;
  return height <= 256 ? largeSize : smallSize;
};
```

#### 8.6 在隐式`return`中强制约束函数体的位置， 就写在箭头后面。 eslint: `implicit-arrow-linebreak`

```javascript
// bad
(foo) =>
  bar;

(foo) =>
  (bar);

// good
(foo) => bar;
(foo) => (bar);
(foo) => (
   bar
)
```

### Classes & Constructors

#### 9.1 常用class，避免直接操作prototype

> Why? class语法更简洁更易理解

```javascript
// bad
function Queue(contents = []) {
  this.queue = [...contents];
}
Queue.prototype.pop = function () {
  const value = this.queue[0];
  this.queue.splice(0, 1);
  return value;
};


// good
class Queue {
  constructor(contents = []) {
    this.queue = [...contents];
  }
  pop() {
    const value = this.queue[0];
    this.queue.splice(0, 1);
    return value;
  }
}
```

#### 9.2 用extends实现继承

> Why? 它是一种内置的方法来继承原型功能而不打破instanceof

```javascript
// bad
const inherits = require('inherits');
function PeekableQueue(contents) {
  Queue.apply(this, contents);
}
inherits(PeekableQueue, Queue);
PeekableQueue.prototype.peek = function () {
  return this.queue[0];
}

// good
class PeekableQueue extends Queue {
  peek() {
    return this.queue[0];
  }
}
```

#### 9.3 方法可以返回this来实现方法链

```javascript
// bad
Jedi.prototype.jump = function () {
  this.jumping = true;
  return true;
};

Jedi.prototype.setHeight = function (height) {
  this.height = height;
};

const luke = new Jedi();
luke.jump(); // => true
luke.setHeight(20); // => undefined

// good
class Jedi {
  jump() {
    this.jumping = true;
    return this;
  }

  setHeight(height) {
    this.height = height;
    return this;
  }
}

const luke = new Jedi();

luke.jump()
  .setHeight(20);
```

#### 9.4 写一个定制的`toString()`方法是可以的，只要保证它是可以正常工作且没有副作用的

```javascript
class Jedi {
  constructor(options = {}) {
    this.name = options.name || 'no name';
  }

  getName() {
    return this.name;
  }

  toString() {
    return `Jedi - ${this.getName()}`;
  }
}
```

#### 9.5 如果没有具体说明，类有默认的构造方法。一个空的构造函数或只是代表父类的构造函数是不需要写的。 eslint: `no-useless-constructor`

```javascript
// bad
class Jedi {
  constructor() {}

  getName() {
    return this.name;
  }
}

// bad
class Rey extends Jedi {
  // 这种构造函数是不需要写的
  constructor(...args) {
    super(...args);
  }
}

// good
class Rey extends Jedi {
  constructor(...args) {
    super(...args);
    this.name = 'Rey';
  }
}
```

#### 9.6 避免重复类成员。 eslint: `no-dupe-class-members`

> Why? 重复类成员会默默的执行最后一个 —— 重复本身也是一个bug

```javascript
// bad
class Foo {
  bar() { return 1; }
  bar() { return 2; }
}

// good
class Foo {
  bar() { return 1; }
}

// good
class Foo {
  bar() { return 2; }
}
```

#### 9.7 除非外部库或框架需要使用特定的非静态方法，否则类方法应该使用this或被做成静态方法。 作为一个实例方法应该表明它根据接收者的属性有不同的行为。eslint: `class-methods-use-this`

```javascript
// bad
class Foo {
  bar() {
    console.log('bar');
  }
}

// good - this 被使用了
class Foo {
  bar() {
    console.log(this.bar);
  }
}

// good - constructor 不一定要使用this
class Foo {
  constructor() {
    // ...
  }
}

// good - 静态方法不需要使用 this
class Foo {
  static bar() {
    console.log('bar');
  }
}
```

### Modules

#### 10.1 用(`import/export`) 模块而不是无标准的模块系统。你可以随时转到你喜欢的模块系统。

> Why? 模块化是未来，让我们现在就开启未来吧。

```javascript
// bad
const AirbnbStyleGuide = require('./AirbnbStyleGuide');
module.exports = AirbnbStyleGuide.es6;

// ok
import AirbnbStyleGuide from './AirbnbStyleGuide';
export default AirbnbStyleGuide.es6;

// best
import { es6 } from './AirbnbStyleGuide';
export default es6;
```

#### 10.2 不要用import通配符， 就是 * 这种方式

> Why? 这确保你有单个默认的导出

```javascript
// bad
import * as AirbnbStyleGuide from './AirbnbStyleGuide';

// good
import AirbnbStyleGuide from './AirbnbStyleGuide';
```

#### 10.3 不要直接从`import`中直接`export`

> Why? 虽然一行是简洁的，有一个明确的方式进口和一个明确的出口方式来保证一致性。

```javascript
// bad
// filename es6.js
export { es6 as default } from './AirbnbStyleGuide';

// good
// filename es6.js
import { es6 } from './AirbnbStyleGuide';
export default es6;
```

#### 10.4 一个路径只 `import` 一次。 eslint: `no-duplicate-imports`

> Why? 从同一个路径下import多行会使代码难以维护

```javascript
// bad
import foo from 'foo';
// … some other imports … //
import { named1, named2 } from 'foo';

// good
import foo, { named1, named2 } from 'foo';

// good
import foo, {
  named1,
  named2,
} from 'foo';
```

#### 10.5 不要导出可变的东西 eslint: `import/no-mutable-exports`

> Why? 变化通常都是需要避免，特别是当你要输出可变的绑定。虽然在某些场景下可能需要这种技术，但总的来说应该导出常量。

```javascript
// bad
let foo = 3;
export { foo }

// good
const foo = 3;
export { foo }
```

#### 10.6 在一个单一导出模块里，用 `export default` 更好。 eslint: `import/prefer-default-export`

> Why? 鼓励使用更多文件，每个文件只做一件事情并导出，这样可读性和可维护性更好。

```javascript
// bad
export function foo() {}

// good
export default function foo() {}
```

#### 10.7 import 放在其他所有语句之前。 eslint: `import/first`

> Why? 让import放在最前面防止意外行为。

```javascript
// bad
import foo from 'foo';
foo.init();

import bar from 'bar';

// good
import foo from 'foo';
import bar from 'bar';

foo.init();
```

#### 10.8 多行`import`应该缩进，就像多行数组和对象字面量

> Why? 花括号与样式指南中每个其他花括号块遵循相同的缩进规则，逗号也是。

```javascript
// bad
import {longNameA, longNameB, longNameC, longNameD, longNameE} from 'path';

// good
import {
  longNameA,
  longNameB,
  longNameC,
  longNameD,
  longNameE,
} from 'path';
```

####  10.9 在import语句里不允许Webpack loader语法 eslint: `import/no-webpack-loader-syntax`

> Why? 一旦用Webpack语法在import里会把代码耦合到模块绑定器。最好是在webpack.config.js里写webpack loader语法

```javascript
// bad
import fooSass from 'css!sass!foo.scss';
import barCss from 'style!css!bar.css';

// good
import fooSass from 'foo.scss';
import barCss from 'bar.css';
```

### Iterators and Generators

#### 11.1 不要用遍历器。用JavaScript高级函数代替for-in、 for-of。 eslint: `no-iterator no-restricted-syntax`

> Why? 这强调了我们不可变的规则。 处理返回值的纯函数比副作用更容易。

> Why?用数组的这些迭代方法： 
>  - map()
>  - every()
>  - filter() 
>  - find()
>  - findIndex()
>  - reduce()
>  - some()
>  - ... , 
>
>  用对象的这些方法 
>  - Object.keys()
>  - Object.values()
>  - Object.entries() 
>
>  去产生一个数组， 这样你就能去遍历对象了。
>

```javascript
const numbers = [1, 2, 3, 4, 5];

// bad
let sum = 0;
for (let num of numbers) {
  sum += num;
}
sum === 15;

// good
let sum = 0;
numbers.forEach(num => sum += num);
sum === 15;

// best (use the functional force)
const sum = numbers.reduce((total, num) => total + num, 0);
sum === 15;

// bad
const increasedByOne = [];
for (let i = 0; i < numbers.length; i++) {
  increasedByOne.push(numbers[i] + 1);
}

// good
const increasedByOne = [];
numbers.forEach(num => increasedByOne.push(num + 1));

// best (keeping it functional)
const increasedByOne = numbers.map(num => num + 1);
```

#### 11.2 现在不要用generator

> Why? 它在es5上支持的不好

#### 11.3 如果你一定要用，或者你忽略我们的建议, 请确保它们的函数签名空格是得当的。 eslint: `generator-star-spacing`

> Why? function 和 * 是同一概念关键字 - *不是function的修饰符，function*是一个和function不一样的独特结构

```javascript
// bad
function * foo() {
  // ...
}

// bad
const bar = function * () {
  // ...
}

// bad
const baz = function *() {
  // ...
}

// bad
const quux = function*() {
  // ...
}

// bad
function*foo() {
  // ...
}

// bad
function *foo() {
  // ...
}

// very bad
function
*
foo() {
  // ...
}

// very bad
const wat = function
*
() {
  // ...
}

// good
function* foo() {
  // ...
}

// good
const foo = function* () {
  // ...
}
```