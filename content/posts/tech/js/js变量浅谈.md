---
title: "浅谈 JavaScript 中的变量"
date: 2025-02-19T17:08:54+08:00
lastmod: 2025-02-19T17:08:54+08:00
author: ["熊大如如"]
tags: # 标签
  - "javascript"
description: ""
weight:
slug: ""
summary: "JavaScript中的变量"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202502191717176.png"
---

## 一、声明变量

在 ES6 之后，JavaScript 声明变量有三种方式，分别是 `var`、`let`、`const`

当然，你也可以不使用关键字声明变量，这样会创建一个全局变量

### 1.1 var 关键字

在 ES6 以前，JS 引擎使用 `var` 关键字声明变量，在 `var` 时代，无论你把变量声明写到哪儿，最后都会在编译阶段提升到作用域的顶端，这个就是变量提升，这个我们稍后再谈。

`var` 关键字声明的变量，可以重复声明

- 简单的示例

```javascript
var message = "Hello World!";
console.log(message); // Hello World!

// 重复声明不会报错
var message = "x";
console.log(message); // x
```

### 1.2 let 关键字

ES6 新增，`let` 声明的变量作用域是块级作用域的，也就是 `{}`之内的。而且 `let` 关键字声明的变量不能再次声明

- 简单的示例

```javascript
let name = "xxx";
console.log(name); // xxx

// 重复声明会报错
let name = "1";
console.log(name); // 'name' has already been declared
```

- 超出作用域，报错

```javascript
if (true) {
  let xxx = "xxx";
}
console.log(xxx); // xxx is not defined
```

### 1.3 const 关键字

`const` 关键字与 `let` 基本一致，最重要的区别就是，`const` 声明后的关键字必须初始化一个值，而且尝试修改 const 声明的变量会报错

- 简单的示例

```javascript
// 尝试修改报错
const name = "xxx";
name = 1;
console.log(name); // Assignment to constant variable.
```

### 1.4 无声明的变量

没有使用关键字的变量，会成为全局变量，这可能导致许多问题

- 简单示例

```javascript
function test() {
  message = "xxx";
}
console.log(message); // xxx
```

如你所见，函数作用域中的变量会成为全局作用域的变量，这可能导致代码维护困难，我们无从知道变量的来源。

我们应该绝对避免这么做

### 1.5 JS 的变量作用域

在 ES6 之前，只有两种作用域

- 函数作用域
- 全局作用域

在 ES6 之后，新增加了

- 块级作用域

## 二、变量提升

### 2.1 什么是变量提升

**简单的演示**

```javascript
console.log(xxx); // undefined
var xxx = "Bear";
```

如你所见，变量还未声明，我们就可以使用它，这就是所谓的变量提升。

通俗来说，变量提升是指在 JavaScript 代码执行过程中，JavaScript 引擎把变量的声明部分和函数的声明部分提升到代码开头的行为。变量被提升后，会给变量设置默认值为 undefined。

### 2.2 变量提升的优点

**提高性能**

在 JS 代码执行之前，会进行语法检查和预编译，并且这一操作只进行一次。这么做就是为了提高性能，如果没有这一步，那么每次执行代码前都必须重新解析一遍该变量（函数），这是没有必要的，因为变量（函数）的代码并不会改变，解析一遍就够了。

在解析的过程中，还会为函数生成预编译代码。在预编译时，会统计声明了哪些变量、创建了哪些函数，并对函数的代码进行压缩，去除注释、不必要的空白等。这样做的好处就是每次执行函数时都可以直接为该函数分配栈空间（不需要再解析一遍去获取代码中声明了哪些变量，创建了哪些函数），并且因为代码压缩的原因，代码执行也更快了。

(不理解，以后再理解吧 😅)

**减少错误**

```javascript
a = 1;
var a;
console.log(a); // 1
```

如果没有变量提升，这代码就会报错，变量提升某种程度上可以提高容错率

### 2.3 变量提升的缺点

**变量被覆盖**

```javascript
var name = "JavaScript";
function showName() {
  console.log(name);
  if (0) {
    var name = "CSS";
  }
}
showName(); // undefined
```

日志打印，会优先从最近的作用域中找变量，这个函数中的赋值操作永远走不到，所以打印 undefined  
**变量未销毁**

```javascript
function foo() {
  for (var i = 0; i < 5; i++) {}
  console.log(i);
}
foo(); // 5
```

使用其他的大部分语言实现类似代码时，在 for 循环结束之后，i 就已经被销毁了，但是在 JavaScript 代码中，i 的值并未被销毁，所以最后打印出来的是 5。这也是由变量提升而导致的，在创建执行上下文阶段，变量 i 就已经被提升了，所以当 for 循环结束之后，变量 i 并没有被销毁。

### 2.4 禁用变量提升

`let`和 `const`不会导致变量提升，所以，为了避免变量提升导致的各种奇奇怪怪的问题，让我们不再使用 `var`来声明变量吧 🤪

## 三、暂时性死区

### 3.1 什么是暂时性死区？

前面我们说过了，使用 `let`或者 `const`声明的变量不会使变量提升，那就是说，我们不能在变量声明前，使用这个变量，不然会报错。

也就是说，在 `const`或者 `let`声明之前的区域叫做 暂时性死区，此时我们不能使用这些变量，否则会导致报错。

## 四、参考链接

- [浅谈 JavaScript 变量提升](https://juejin.cn/post/7007224479218663455)

欢迎指正~
