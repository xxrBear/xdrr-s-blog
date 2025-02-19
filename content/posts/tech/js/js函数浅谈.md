---
title: "浅谈 JavaScript 函数"
date: 2025-02-19T21:46:28+08:00
lastmod: 2025-02-19T21:46:28+08:00
author: ["熊大如如"]
tags: # 标签
  - "javascript"
description: ""
weight:
slug: ""
summary: "javascript的函数总结"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202502151647700.png" # 文章的图片
---

## 一、创建函数

JS 中的函数实际上是对象，每个函数都是 `Function`类型的实例

JS 中有多种创建函数的方法

### 1.1 函数声明

```javascript
function sum(n1, n2) {
  return n1 + n2;
}
```

### 1.2 函数表达式

```javascript
let sum = function(n1, n2) => {
  return n1 + n2;
};
```

### 1.3 构造函数

```javascript
let sum = new Function("n1", "n2", "return n1+n2"); // 不推荐
```

构造函数创建函数，代码会被解释两次，不推荐，影响性能。

## 二、箭头函数

ES6 新增胖箭头函数，简洁且好用

```javascript
let sum = (a, b) => {
  return a + b;
};
```

箭头函数非常适合用在嵌入函数的场景，例如数组的 `map`、`filter`方法中。

虽然箭头函数语法简洁，但是它没有 `arguments`、`super`、`new.target`等参数，也没有 `propotype`属性。

## 三、函数名

ES6 中的函数都有一个只读的属性 `name`, 其中包含了函数的信息。

```javascript
function foo() {}
let bar = function () {};
let baz = () => {};

console.log(foo.name); // foo
console.log(bar.name); // bar
console.log(baz.name); // baz
console.log((() => {}).name); //
console.log(new Function().name); // anonymous
```

## 四、函数参数

ES 的函数参数与大多数编程语言的参数都不同，它不关心你定义的函数数量，也不关心你传入的参数数量，也就是说，无论你定义或者不定义参数，传入或者不传入参数，解释器都不会报错。

### 4.1 函数未定义参数

```javascript
function greet() {
  console.log("Hello!");
}

greet("Alice"); // 输出: Hello!
```

### 4.2 调用时不传入参数

```javascript
function greet(name) {
  console.log("Hello, " + name + "!");
}

greet(); // 输出: Hello, undefined!
```

### 4.3 调用时传入多个参数

```javascript
function greet(name) {
  console.log("Hello, " + name + "!");
}

greet("Alice", "Bob", "Charlie"); // 输出: Hello, Alice!
```

### 4.4 arguments 对象

JS 函数中的 `arguments`对象是一个类数组对象

你可以访问它的一些属性

- arguments.length：返回传入参数的数量

```javascript
function greet(name) {
  console.log(arguments); // ['Alice', 'Bob', 'Charlie', callee: ƒ, Symbol(Symbol.iterator): ƒ]
  console.log(arguments.length); // 3
}

greet("Alice", "Bob", "Charlie");
```

注意：arguments 参数只有在 function 关键字定义的函数中存在，在箭头函数中是不能使用的。

```javascript
let sum = () => arguments;

sum(); // Uncaught ReferenceError: arguments is not defined
```

## 五、默认参数

ES6 支持显示定义参数默认值

```javascript
function makeKing(name = "Bear") {
  return `King is ${name}`;
}

console.log(makeKing()); // King is Bear
```

## 六、参数扩展与收集

### 6.1 参数扩展

ES6 新增扩展操作符，使用它可以轻松的操作和组合集合数据。

我们定义一个 `getSum`函数

```javascript
let values = [1, 2, 3, 4, 5];

function getSum() {
  let sum = 0;
  for (let i = 0; i < arguments.length; ++i) {
    sum += arguments[i];
  }
  return sum;
}

console.log(getSum(...values)); // 15
```

我们想再添加数据，可以使用扩展操作符

```javascript
console.log(getSum(-1, ...values)); // 14
```

### 6.2 参数收集

扩展操作符可以把参数收集起来，在函数内部得到一个数组实例。但是有一定规则限制

```javascript
function getSum(...values) {
  return values.reduce((x, y) => x + y, 0);
}

console.log(getSum(1, 2, 3)); // 6
```

## 七、函数声明和函数表达式

JavaScript 在加载函数声明和函数表达式时有区别！

JS 引擎在任何代码执行前，会先读取函数声明，并在执行上下文中生成函数定义（啥 🤔）。而函数表达式必须等到代码执行到它那一行，才会在执行上下文中生成函数定义。

也就是说函数声明会有 **函数声明提升，在执行代码前，JS 引擎会先执行一遍扫描，把发现的函数声明提升到源代码数的顶部。**

- 举个例子 🌰

```javascript
console.log(sum(10, 10)); // 20
function sum(n1, n2) {
  return n1 + n2;
}
```

- 举个函数表达式的例子

**报错**

```javascript
console.log(sum(10, 10)); // Uncaught TypeError: sum is not a function
var sum = function sum(n1, n2) {
  return n1 + n2;
};
```

## 八、函数作为值

函数可以当作另一个函数的参数

举个例子 🌰

```javascript
function callSomeFunction(someFunction, someArgument) {
  return someFunction(someArgument);
}
function add10(num) {
  return num + 10;
}

let result = callSomeFunction(add10, 10);
console.log(result); // 20
```

## 九、函数内部

ES5 中，函数内部存在两个特殊的对象：`arguments`和 `this`

ES6 新增一个 `new.target`属性

### 9.1 arguments.callee

`arguments`参数前面讨论过很多次了，它是一个类数组对象，我们要介绍的是它的一个属性 `callee`

`callee`属性指向 `arguments`对象所在函数的指针。

- 举个例子

一个经典的递归求阶乘函数

```javascript
function factorial(num) {
  if (num <= 1) {
    return 1;
  } else {
    return num * factorial(num - 1);
  }
}
console.log(factorial(3));
```

我们可以使用 `callee`参数重写

```javascript
function factorial(num) {
  if (num <= 1) {
    return 1;
  } else {
    return num * arguments.callee(num - 1);
  }
}
console.log(factorial(3));
```

这个函数调用和上面的一样，你可能觉得这个没啥用。只是用 `arguments.callee`代替了 `factorial`。好像确实是 😅，但是红宝书给出了这样一个例子

```javascript
let trueFac = factorial;

factorial = function () {
  return 0;
};

console.log(trueFac(5)); // 120
console.log(factorial(5)); // 0
```

依然觉得没啥用

### 9.2 this

this 是一个特殊的对象，它在标准函数和箭头函数中有不同的行为。

this 指代的对象根据上下文的不同会改变

- 举个例子

```javascript
window.color = "red";
let o = {
  color: "blue",
};

function sayColor() {
  console.log(this.color);
}

sayColor(); // red

o.sayColor = sayColor;
o.sayColor(); // blue
```

this 很复杂，后面会再写一篇总结。

### 9.3 caller

`caller`属性引用调用当前函数的函数

- 举个例子

```javascript
function outer() {
  inner();
}

function inner() {
  console.log(inner.caller);
}
outer();
```

显示 outer 的源代码

### 9.4 new.target

判断函数是不是使用 `new`关键字调用的

如果不是返回 `undefined`,如果是 `new`关键字调用，则引用被调用的构造函数。

## 十、函数属性和方法

### 10.1 prototype 属性

保存引用类型的所有实例方法

### 10.2 apply 方法

接收两个参数

- this
- 数组 或者 arguments

举个例子 🌰

```javascript
function sum(n1, n2) {
  return n1 + n2;
}

function callSum1(n1, n2) {
  return sum.apply(this, arguments);
}

function callSum2(n1, n2) {
  return sum.apply(this, [n1, n2]);
}

console.log(callSum1(10, 10)); // 20
console.log(callSum2(10, 10)); // 20
```

在这个例子中，`callSum1()` 会调用 `sum()` 函数

### 10.3 call 方法

`call`方法与 `apply`方法的作用一样，但是传参不一样，第一个参数都是 `this`,但是第二个参数是逐个传递的。

```javascript
function sum(n1, n2) {
  return n1 + n2;
}

function callSum1(n1, n2) {
  return sum.call(this, n1, n2);
}

console.log(callSum1(10, 10)); // 20
```

`call`和 `apply`**方法作用一致，如何使用取决你觉得如何传参方便**

### 10.4 bind 方法

`bind` 方法会创建一个新的函数实例，其 `this` 值会绑定到传给 `bind()` 的对象

```javascript
window.color = "red";
var o = {
  color: "blue",
};

function sayColor() {
  console.log(this.color);
}

let objectSayColor = sayColor.bind(o);
objectSayColor(); // blue
```

## 十一、尾调用优化

看不懂，PASS😅

## 十二、闭包

过于重要，单独写一篇总结 ❤️

## 十三、立即调用的函数

立即调用函数表达式：

```javascript
(function () {})();
```
