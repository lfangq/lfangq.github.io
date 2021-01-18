---
title: ECMAScript历年新特性
date: 2020-09-21 10:01:52
tags: [javascript]
---

### [ES6 入门教程](https://es6.ruanyifeng.com/)

### ES11新特性 (ECMAScript2020)
<!-- more -->

#### String.prototype.matchAll
> 引用： https://blog.csdn.net/weixin_41849462/article/details/105813995

`String.prototype.match()`方法仅返回完整的匹配结果，却不会返回特定正则表达式组（Regex groups）的信息, 而`String.prototype.matchAll`它返回的迭代器不仅包括精确的匹配结果，还有全部的正则模式捕获结果。

```javascript
// match() 方法
const text = "From 2019.01.29 to 2019.01.30";

const regexp = /(?<year>\d{4}).(?<month>\d{2}).(?<day>\d{2})/gu;

const results = text.match(regexp);

console.log(results);
// [ '2019.01.29', '2019.01.30' ]
```

```javascript
// matchAll() 方法，可以看到结果的 groups 为命名捕获组
const text = "From 2019.01.29 to 2019.01.30";

const regexp = /(?<year>\d{4}).(?<month>\d{2}).(?<day>\d{2})/gu;

const results = Array.from(text.matchAll(regexp));

console.log(results);
// [
//   [
//     '2019.01.29',
//     '2019',
//     '01',
//     '29',
//     index: 5,
//     input: 'From 2019.01.29 to 2019.01.30',
//     groups: [Object: null prototype] { year: '2019', month: '01', day: '29' }
//   ],
//   [
//     '2019.01.30',
//     '2019',
//     '01',
//     '30',
//     index: 19,
//     input: 'From 2019.01.29 to 2019.01.30',
//     groups: [Object: null prototype] { year: '2019', month: '01', day: '30' }
//   ]
// ]
```

#### import()

可以按需获取的动态 `import.` 该类函数格式（它并非继承自 Function.prototype）返回了一个强大的 promise 函数，使得诸如按需引入、可计算模块名以及脚本内计算均成为可能。

```javascript
const modulePage = 'page.js'; 

import(modulePage)
     .then((module) => {
        module.default();
     });
```

```javascript
(async () => {
  const helpersModule = 'helpers.js';

  const module = await import(helpersModule)
  
  const total = module.sum(2, 2);
})();
```

#### BigInt – 任意精度整数

Js 中 Number类型只能安全的表示-(2^53-1)至 2^53-1 范的值，即Number.MINSAFEINTEGER 至Number.MAXSAFEINTEGER，超出这个范围的整数计算或者表示会丢失精度。

```javascript
var num = Number.MAX_SAFE_INTEGER;  // -> 9007199254740991

num = num + 1; // -> 9007199254740992

// 再次加 +1 后无法正常运算
num = num + 1; 
// -> 9007199254740992

// 两个不同的值，却返回了true
9007199254740992 === 9007199254740993 // -> true
```

为解决此问题，ES2020提供一种新的数据类型：BigInt。使用 BigInt 有两种方式：
- 在整数字面量后面加n

```javascript
var bigIntNum = 9007199254740993n;
```

- 使用 BigInt 函数
```javascript
var bigIntNum = BigInt(9007199254740);

var anOtherBigIntNum = BigInt('9007199254740993');
```

BigInt 是一种新的数据原始（primitive）类型。
```javascript
typeof 9007199254740993n; // -> 'bigint'
```
> 注意：尽可能避免通过调用函数 BigInt 方式来实例化超大整型。因为参数的字面量实际也是 Number 类型的一次实例化，超出安全范围的数字，可能会引起精度丢失。

#### Promise.allSettled

Promise.all 缺陷

都知道 Promise.all 具有并发执行异步任务的能力。但它的最大问题就是如果其中某个任务出现异常(reject)，所有任务都会挂掉，Promise直接进入 reject 状态。

想象这个场景：你的页面有三个区域，分别对应三个独立的接口数据，使用 Promise.all 来并发三个接口，如果其中任意一个接口服务异常，状态是reject,这会导致页面中该三个区域数据全都无法渲染出来，因为任何 reject 都会进入catch回调, 很明显，这是无法接受的，如下：

```javascript
Promise.all([
Promise.reject({
    code: 500,
    msg: '服务异常'
}),
Promise.resolve({
    code: 200,
    list: []
}),
Promise.resolve({
    code: 200,
    list: []})
]).then((ret) => {
// 如果其中一个任务是 reject，则不会执行到这个回调。
    RenderContent
        (ret);
}).catch((error) => {
// 本例中会执行到这个回调
// error: {code: 500, msg: "服务异常"}
})
```

我们需要一种机制，如果并发任务中，无论一个任务正常或者异常，都会返回对应的的状态（fulfilled 或者 rejected）与结果（业务value 或者 拒因 reason），在 then 里面通过 filter 来过滤出想要的业务逻辑结果，这就能最大限度的保障业务当前状态的可访问性，而 Promise.allSettled 就是解决这问题的。

```javascript
Promise.allSettled([
  fetch("https://api.github.com/users/pawelgrzybek").then(data => data.json()),
  fetch("https://api.github.com/users/danjordan").then(data => data.json())
])
  .then(result => console.log(`All profile settled`));
```

#### globalThis

所以在 JavaScript 中，全局的 `this` 到底是什么？在浏览器中它是 window, 在 worker 中它是 self, 在 Node.js 中它是 global, 在… 如今这种混乱终于结束了！感谢 Jordan Harband 为我们带来的 `globalThis` 关键字`。

#### for-in 机制

#### 可选链

可选链 可让我们在查询具有多层级的对象时，不再需要进行冗余的各种前置校验。

日常开发中，我们经常会遇到这种查询

```javascript
var name = user && user.info && user.info.name;
```

又或是这种

```javascript
var age = user && user.info && user.info.getAge && user.info.getAge();
```

这是一种丑陋但又不得不做的前置校验，否则很容易命中 `Uncaught TypeError: Cannot read property… `这种错误，这极有可能让你整个应用挂掉。

用了 Optional Chaining ，上面代码会变成

```javascript
var name = user?.info?.name;
var age = user?.info?.getAge?.();
```

可选链中的 `?`表示如果问号左边表达式有值, 就会继续查询问号后面的字段。根据上面可以看出，用可选链可以大量简化类似繁琐的前置校验操作，而且更安全。

#### 空值合并运算符

当我们查询某个属性时，经常会遇到，如果没有该属性就会设置一个默认的值。比如下面代码中查询玩家等级。

```javascript
var level = (user.data && user.data.level) || '暂无等级';
```

在JS中，空字符串、0 等，当进行逻辑操作符判时，会自动转化为 false。在上面的代码里，如果玩家等级本身就是 0 级, 变量 level 就会被赋值 暂无等级 字符串，这是逻辑错误。

```javascript
var level = (user.data && user.data.level) || '暂无等级';var level;

if(typeof user.level === 'number') {
    level = user.level;
} 
else if(!user.level) {
    level = '暂无等级';
} else{
    level = user.level;
}
```

来看看用空值合并运算符如何处理

```javascript
// {
//   "level": 0
// }
var level = `${user.level}级`?? '暂无等级';// level -> '0级'
```

用空值合并运算在逻辑正确的前提下，代码更加简洁。
空值合并运算符 与 可选链 相结合，可以很轻松处理多级查询并赋予默认值问题。

```javascript
var level = user.data?.level ?? '暂无等级';
```

#### import.meta

import.meta 提议为当前运行的模块添加了一个特定 host 元数据对象

```javascript
console.log(import.meta.url)
// file:///Users/pawelgrzybek/main.js
```

#### export * as ns from “mod”

这是对 ES 规范的有力补充，它允许开发者以新名称导出另一模块的命名空间外部对象。

```javascript
export * as ns from "mod"
// file:///Users/pawelgrzybek/main.js
```

### ES10新特性 (ECMAScript2019)

#### Object.fromEntries()

es8中对象添加了一个entries()静态方法，这个方法返回一个给定对象自身可枚举属性的键值对数组 ，Object.fromEntries()方法与 Object.entries() 正好相对，可以将键值对列表转换为一个对象 。

```javascript
const obj = {
  x: 1,
  y: 2,
};

const entries = Object.entries(obj);

console.log(entries);
console.log(Object.fromEntries(entries));

// [["x",1],["y":2]]
// {x:1,y:2}
```

只要符合entries结构的都可以使用Object.fromEntries(entries)将键值对列表转换为一个对象，比如Map

#### String.prototype.trimStart()/String.prototype.trimEnd()

- trimStart() /trimLeft()

  trimLeft是trimStart的别名,作用是去掉字符串左边的空格

- trimEnd() / trimRight()

  trimEnd是trimRight的别名,作用是去掉字符串右边的空格

```javascript
const str = "   hello world   ";
console.log(str.trimStart());
console.log(str.trimEnd());
console.log(str.trim());

// "hello world   "
// "   hello world"
// "hello world"
```

#### Array.prototype.flat()/Array.prototype.flatMap()

- Array.prototype.flat()

  flat() 方法会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回 。

```javascript
const arr = [1, [2, [3, [4, [5, [6, 7], 8], 9]]]];
console.log(arr.flat(1));
console.log(arr.flat(5));
console.log(arr.flat(Infinity));

// [1,2,[3, [4, [5, [6, 7], 8], 9]]]
// [1,2,3,4,5,6,7,8,9]
// [1,2,3,4,5,6,7,8,9]
```

- Array.prototype.flatMap()

  flatMap实质上包含两部分功能，一是map,二是flat

```javascript
const numbers = [1, 2, 3];

console.log(numbers.map((x) => [x ** 2]).flat());
console.log(numbers.flatMap((x) => [x ** 2]));

// [1,4,9]
// [1,4,9]
```

#### Symbol.description

可以通过 description 获取 Symbol 的描述

```javascript
const symbol = Symbol("symbol");
console.log(symbol.description); // symbol
console.log(symbol.description === "symbol"); // true
```

在es10以前，我们只能通过调用 Symbol 的 toString() 时才可以读取这个属性

```javascript
console.log(symbol.toString() === "Symbol(symbol)");
```

#### Function.prototype.toString()

Function.prototype.toString() 方法返回一个表示当前函数源代码的字符串

```javascript
function test(a) {
  // es10以前不返回注释部分
  console.log(a);
}
console.log(test.toString());

// function test(a) {
//  // es10以前不返回注释部分
//  console.log(a);
// }
```

#### catch Building

es10允许我们在捕获异常时省略catch的参数

```javascript
// es10以前
try {
  throw new Error();
} catch (error) {
  console.log("fail");
}

// es10
try {
  throw new Error();
} catch {
  console.log("fail");
}
```

#### JSON扩展

- JSON 内容可以支持包含 U+2028行分隔符 与 U+2029段分隔符
- 在 ES10 JSON.stringify 会用转义字符的方式来处理 超出范围的 Unicode 展示错误的问题 而非编码的方式

```javascript
console.log(JSON.stringify('\uD83D\uDE0E')) // 笑脸

// 单独的\uD83D其实是个无效的字符串
// 之前的版本 ，这些字符将替换为特殊字符，而现在将未配对的代理代码点表示为JSON转义序列
console.log(JSON.stringify('\uD83D')) // "\ud83d"
```

### ES9新特性(ECMAScript2018)

#### for await of/Symbol.asyncIterator

es6中有一个新特性Iteartor,只要元素符合两个协议：

- 可迭代协议：对象包含Symbol.iterator属性；
- 迭代器协议：Symbol.iterator属性必须返回一个对象，这个对象包含一个next方法，且next方法也返回一个对象，此对象包含value,done两个属性
我们就可以使用for…of去遍历这个元素。

我们知道 for…of 可以遍历同步运行的任务，那如果是异步任务呢，如下：
```javascript
function getPromise(time) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(time)
    }, time)
  })
}

const asyncArr = [getPromise(1000), getPromise(200), getPromise(3000)]

for (let item of asyncArr) {
  console.log(item, item.then(res => {
    console.log(res)
  }))
}
// Promise {<pending>} 
// Promise {<pending>}
// Promise {<pending>}
// 200
// 1000
// 3000
```

在上述遍历的过程中可以看到三个任务是同步启动的，我们期望的是一个异步任务执行完，在执行下一个异步任务，然而从输出可以看出不是按任务的执行顺序输出的，这显然不太符合我们的要求，在 es9 中也可以用 for…await…of 来操作：

```javascript
function getPromise(time) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({
        value: time,
        done: false,
      });
    }, time);
  });
}

const asyncArr = [getPromise(1000), getPromise(200), getPromise(3000)];
asyncArr[Symbol.asyncIterator] = function () {
  let nextIndex = 0;
  return {
    next() {
      return nextIndex < asyncArr.length
        ? asyncArr[nextIndex++]
        : Promise.resolve({
            value: undefined,
            done: true,
          });
  },
  };
};

async function test() {
  for await (let item of asyncArr) {
    console.log(Date.now(), item);
  }
}

test();

// 1594374685156 1000
// 1594374685157 200
// 1594374687157 3000
```

> await需要在async 函数或者 async 生成器里面使用

同步迭代器/异步迭代器

| 类别 | 同步迭代器 | 异步迭代器 |
| ---- | --------- | --------- |
| 迭代器协议 | Symbol.iterator | Symbol.asyncIteartor |
| 遍历 | for…of | for…await…of |

#### 正则的扩展

- dotAll/s

  dotAll模式就是：在正则中使用(.)字符时使用s修饰符可以解决(.)字符不能匹配行终止符的例外

```javascript
console.log(/./.test(1));
console.log(/./.test("1"));
console.log(/./.test("\n"));
console.log(/./.test("\r"));
console.log(/./.test("\u{2028}"));
// true
// true
// false
// false
// false

// 使用s修饰符
console.log(/./s.test(1));
console.log(/./s.test("1"));
console.log(/./s.test("\n"));
console.log(/./s.test("\r"));
console.log(/./s.test("\u{2028}"));
// true
// true
// true
// true
// true
```

> 1. （.）是一个特殊字符，代表任意的单个字符，但是有两个例外。一个是四个字节的 UTF-16 字符，这个可以用u修饰符解决；另一个是行终止符
>
> 2. 正则中可以使用的修饰符有i, g, m, y, u, s

- 具名组匹配

```javascript
console.log("2020-07-10".match(/(\d{4})-(\d{2})-(\d{2})/));
// ["2020-07-10", "2020", "07", "10", index: 0, input: "2020-07-10", groups: undefined]
```

按照 match 的语法，没有使用 g 修饰符，所以返回值第一个数值是正则表达式的完整匹配，接下来的第二个值到第四个值是分组匹配（2020, 07, 10）,我们想要获取年月日的时候不得不通过数组的下标去获取，这样显得不灵活。仔细观察 match 返回值还有几个属性，分别是 index、input、groups。

> index [匹配的结果的开始位置]
>
> input [匹配的字符串]
>
> groups [捕获组]

所谓的具名组匹配就是命名捕获分组：

```javascript
console.log("2020-07-10".match(/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/));

// groups的值
groups: {year: "2020", month: "07", day: "10"}
```

这样我们就可以通过groups及命名分组获取对应的年月日的值了。

- 后行断言

```javascript
let test = 'world hello'
console.log(test.match(/(?<=world\s)hello/))
```

(?<)是后行断言的符号配合= 、！等使用。

#### 对象的Rest和Spread语法

一个例子对比理解对象的Rest和Spread语法

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// 数组合并
const arr = [...arr1, ...arr2];
console.log(arr);

const obj1 = { a: 1 };
const obj2 = { b: 2 };

// 对象合并
const obj = { ...obj1, ...obj2 };
console.log(obj);

// [1, 2, 3, 4, 5, 6]
// {a: 1, b: 2}
```

一句话总结就是(…)运算符在数组中可以怎样使用，在对象就可以怎样使用。

#### Promise.prototype.finally()

不管promise状态如何都会执行的回调函数

```javascript
new Promise((resolve, reject) => {
  resolve(1);
})
  .then((res) => {
    console.log(res);
  })
  .catch((err) => {
    console.log(err);
  })
  .finally(() => {
    console.log("finally");
  });
// 1
// promise
```

#### 带标签的模板字符串扩展

es9 新特性中移除了对 ECMAScript带标签的模板字符串中转义序列的语法限制。 遇到不合法的字符串转义返回undefined，并且从raw上可获取原字符串

```javascript
function foo(str) {
  console.log(str);
}

foo`\undfdfdf`;
// es9以前报错
// es9:[undefined, raw:["\undfdfdf"]]
```

### ES8新特性(ECMAScript2017)

#### async/await

> 1. async/await是继es6中promise、generator后又一种更加优雅的异步编程的解决方案
> 
> 2. async函数是generator函数的语法糖

基本用法:

```javascript
// 不使用async/await
function getPromise() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log(1);
      resolve(2);
    }, 1000);
  });
}

function foo() {
  const res = getPromise();
  console.log(res);
  console.log(3);
}

foo();
// Promise {<pending>}
// 3
// 1

// 使用async/await
async function foo() {
  const res = await getPromise();
  console.log(res);
  console.log(3);
}

foo();

// 1
// 2
// 3
```

由上面两个例子的对比就能发现，async/await可以使异步任务处理起来像是同步任务，这是因为await关键字在执行的时候会停下来，等待异步任务执行完毕(await后面一般跟的是异步任务，否则没有意义)在继续执行同步任务。

更优雅的异步编程的解决方案:

在es6之前我们对于这个过程应该不陌生
```javascript
ajax('xxx/a', res => {
    console.log(res)
    ajax('xxx/b', res => {
        console.log(res)
        ajax('xxx/c', res => {
            console.log(res)
        })
    })
})
```

这种回调之后再回调的调用方式我们称之为“回调地狱”，这种回调方式在日常开发和项目维护当中很让人头疼。我们对比下es6中Promise的处理和es8中的async/await的处理方式就知道了为什么我们称async/await为更优雅的异步编程的解决方案。

```javascript
// 以下都是模拟接口请求的代码
// Promise
function getPromise(url) {
    return new Promise((resolve, reject) => {
        ajax(url, res => {
            resolve(res)
        }, err => {
            reject(err)
        })
    })
}

getPromise('xxx/a')
    .then(res => {
        console.log(res)
        return getPromise('xxx/b')
    }).then(res => {
        console.log(res)
        return getPromise('xxx/c')
    }).then(res => {
        console.log(res)
    }).catch(err => {
        console.log(err)
    })
    
    
 // async/await
 function request(url) {
    return new Promise(resolve => {
        ajax(url, res => {
            resolve(res)
        })
    })
}
async function getData() {
    let res1 = await request('xxx/a')
    console.log(res1)
    let res2 = await request('xxx/b')
    console.log(res2)
    let res3 = await request('xxx/c')
    console.log(res3)
}
getData()
```

从两者的对比可以看出，Promise虽然将回调嵌套回调的方式改成平级调用，但是这种调用方式相比于async/await还是显得繁琐，而且async/await不存在回调。

#### Object.values()/Object.entries()

- Object.values()

  Object.values() 返回一个数组，其元素是在对象上找到的可枚举属性值。

```javascript
const obj = {
  name: "张三",
  age: 18,
  sex: "male",
}
console.log(Object.values(obj)) // ["张三", 18, "male"]
```

- Object.entries

  Object.entries()方法返回一个给定对象自身可枚举属性的键值对数组。

```javascript
const obj = {
  name: "张三",
  age: 18,
  sex: "male",
}
console.log(Object.entries(obj))
// [["name", "张三"],["age", "18"], ["sex", "male"]]
```

for in可以遍历出原型链上的可枚举属性，而Object.keys()/Object.values()/Object.entries()只能遍历自身的可枚举属性

```javascript
const obj = {
  name: "张三",
  age: 18,
  sex: "male",
}
Object.prototype.test = "test"

for (let key in obj) {
  console.log(obj[key])
}
//  "张三", 18, "male","test"
console.log(Object.keys(obj).map(key => obj[key]))
// ["张三", 18, "male"]
console.log(Object.values(obj))
// ["张三", 18, "male"]
console.log(Object.entries(obj).map(([key, value]) => value))
// ["张三", 18, "male"]
```

如何实现一个Object.values()/Object.entries()

```javascript
const obj = {
  name: "张三",
  age: 18,
  sex: "male",
}
// Object.values
function values(obj) {
  return Object.keys(obj).map(key => obj[key])
}

// Object.entries
function entries(obj) {
  return Object.keys(obj).map(key => [key, obj[key]])
}

console.log(values(obj)) 
console.log(entries(obj))
```

#### Object.getOwnPropertyDescriptors()

Object.defineProperty()可以通过对描述符的设置进行更精准的控制对象属性，所谓描述符：

1. value [属性的值]
2. writable [属性的值是否可被改变]
3. enumerable [属性的值是否可被枚举]
4. configurable [描述符本身是否可被修改，属性是否可被删除]

```javascript
var test = {
  name: '测试',
  value: 5
}
Object.defineProperty(test, "name", {
  enumerable: false
})

for (let key in test) {
  console.log(key)
} 
// value
```

#### Object.getOwnPropertyDescriptors ()

Object.getOwnPropertyDescriptors ()可以返回对象属性的描述符

```javascript
let test = {
  name: '测试',
  value: 5
}
console.log(Object.getOwnPropertyDescriptors(test)) 
// {
//   name: {value: "测试", writable: true, enumerable: true, configurable: true}
//   value: {value: 5, writable: true, enumerable: true, configurable: true}
// }
```

Object.getOwnPropertyDescriptors(target,param)接收两个参数,返回某一个参数的描述符，通过这个方法可以实现一个Object.getOwnPropertyDescriptors ()

Object.getOwnPropertyDescriptors()实现

```javascript
let test = {
  name: '测试',
  value: 5
}

function getOwnPropertyDescriptors(obj) {
  const result = {};
  for (let key of Reflect.ownKeys(obj)) {
    result[key] = Object.getOwnPropertyDescriptor(obj, key);
  }
  return result;
}
getOwnPropertyDescriptors(test)
```

#### String.prototype.padStart()/String.prototype.padEnd()

- padStart()
  允许将空字符串或其他字符串添加到原始字符串的开头。

```javascript
console.log('13'.padStart(3, '0')); // => 013
console.log('13'.padStart(2, '0')); // => 13
```

- padEnd()
  允许将空字符串或其他字符串添加到原始字符串的结尾。

```javascript
console.log('13'.padStart(3, '0')); // => 130
console.log('13'.padStart(2, '0')); // => 13
```

#### 尾逗号

此前，函数定义和调用时，都不允许最后一个参数后面出现逗号,es8 允许函数的最后一个参数有尾逗号

```javascript
// es8以前
function foo(a, b, c, d) {
  console.log(a, b, c, d)
}

// es8
function foo(a, b, c, d,) {
  console.log(a, b, c, d)
}
```

### ES7新特性(ECMAScript2017)

#### Array.prototype.includes()

判断一个数组是否包含一个元素

```javascript
const arr = ["es6", "es7", "es8", "es9", "es10", "es11"，NaN]
console.log(arr.includes("es6"))   // true
console.log(arr.includes(NaN))   // true
console.log(arr.includes("es12"))  // false
```

includes的用法和indexOf用法相似，都可以用来判断数组中是否包含一个元素，唯一的区别在于includes可以识别NaN。

> 注意
> 1. Array.prototype.includes（）：可以接收两个参数，要搜索的值和搜索的开始索引。第二个参数可选，若为负数表示从末尾开始计数下标。
> 2. 只能判断简单类型的数据，对于复杂类型的数据，比如对象类型的数组，二维数组，这些是无法判断的。

#### 幂运算符

```javascript
console.log(2**53)
```

> 注意
> 1. 幂运算符的两个*号之间不能出现空格，前后有无空格都可以。
> 2. 注意最大安全数：Number.MAX_SAFE_INTEGER = (2**53)-1

