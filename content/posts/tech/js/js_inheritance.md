---
title: "JavaScript 继承方式总结"
date: 2025-03-20T10:34:57+08:00
lastmod: 2025-03-20T10:34:57+08:00
author: ["熊大如如"]
tags: # 标签
  - "javascript"
description: ""
weight:
slug: ""
summary: "JavaScript高级程序设计 + ChatGPT 总结 JavaScript的继承方式"
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

## 原型继承

### 简介

在 JavaScript 中，原型继承（Prototype Inheritance）是一种基于 原型链（Prototype Chain） 的继承方式，它允许对象从其他对象继承属性和方法，而不需要类（class）机制。

### 示例

```javascript
function SuperType() {
  this.property = true;
}

SuperType.prototype.getSuperValue = function () {
  return this.property;
};

function SubType() {
  this.subproperty = false;
}

SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function () {
  return this.subproperty;
};

let instance = new SubType();
console.loinstance.getSuperValue()); // true
```

### 原型继承存在的问题

- **继承引用类型数据的问题**

当一个对象从原型继承 引用类型数据（如数组、对象）时，这些数据会被所有实例共享，而不是每个实例都有独立的副本

```javascript
function Person(name) {
  this.name = name;
  this.hobbies = ["reading", "coding"];
}

let p1 = new Person("Alice");
let p2 = new Person("Bob");

p1.hobbies.push("gaming");

console.log(p1.hobbies); // ["reading", "coding", "gaming"]
console.log(p2.hobbies); // ["reading", "coding", "gaming"] ❌ 受影响
```

`hobbies` 是一个数组，所有实例共享同一个 `hobbies`，导致修改一个实例的 `hobbies`，其他实例也会被影响

- **不能向父类构造函数传递参数**

在原型继承（Object.create()） 方式中，我们无法在子对象创建时向父类的构造函数传递参数

```javascript
let person = {
  init: function (name) {
    this.name = name;
  },
  greet: function () {
    console.log("Hello, I am " + this.name);
  },
};

let student = Object.create(person);
student.init("Alice");
student.greet(); // Hello, I am Alice

let student2 = Object.create(person);
console.log(student2.name); // undefined ❌
```

## 盗用构造函数

### 简介

盗用构造函数（Constructor Stealing），也称为经典继承（Classical Inheritance），是一种在 JavaScript 继承中用于解决原型继承问题的方法。它通过在子类构造函数内部调用父类构造函数，从而避免原型链继承的缺陷，尤其是共享引用类型属性的问题。

### 示例

```javascript
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"]; // 引用类型数据
}

function SubType(name, age) {
  SuperType.call(this, name); // 盗用 SuperType 构造函数
  this.age = age;
}

let instance1 = new SubType("Alice", 25);
instance1.colors.push("yellow");

let instance2 = new SubType("Bob", 30);

console.log(instance1.name); // Alice
console.log(instance1.age); // 25
console.log(instance1.colors); // ["red", "blue", "green", "yellow"]

console.log(instance2.name); // Bob
console.log(instance2.age); // 30
console.log(instance2.colors); // ["red", "blue", "green"] ✅ 没有被污染
```

### 缺点

- **无法继承父类的原型方法**

```javascript
function SuperType(name) {
  this.name = name;
}

SuperType.prototype.sayHello = function () {
  return "Hello, " + this.name;
};

function SubType(name) {
  SuperType.call(this, name);
}

let instance = new SubType("Alice");
console.log(instance.sayHello()); // ❌ TypeError: instance.sayHello is not a function
```

## 组合继承

### 简介

为了既能继承实例属性，又能继承父类原型方法，可以使用组合继承（Combination Inheritance）本质上是综合了原型链和盗用构造函数

组合继承是 JavaScript 中使用最多的继承模式

### 示例

```javascript
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayHello = function () {
  return "Hello, " + this.name;
};

function SubType(name, age) {
  SuperType.call(this, name); // 盗用构造函数，继承实例属性
  this.age = age;
}

// 继承原型方法
SubType.prototype = Object.create(SuperType.prototype);
SubType.prototype.constructor = SubType;

let instance = new SubType("Alice", 25);
console.log(instance.sayHello()); // "Hello, Alice" ✅
```

## 原型式继承

### 简介

原型式继承（Prototypal Inheritance）是基于 已有对象 创建新对象的一种方式，它的核心思想是 直接使用一个对象作为另一个对象的原型，而不是通过构造函数来创建实例。

### 示例

```javascript
let person = {
  name: "Alice",
  friends: ["Bob", "Charlie"],
  sayHello: function () {
    return "Hello, I am " + this.name;
  },
};

// 通过 Object.create() 创建新对象
let anotherPerson = Object.create(person);
anotherPerson.name = "David";
anotherPerson.friends.push("Eve");

console.log(anotherPerson.name); // "David"
console.log(anotherPerson.sayHello()); // "Hello, I am David"
console.log(person.friends); // ["Bob", "Charlie", "Eve"] ❌ 受影响
```

### 缺点

由于 `Object.create` 方法是浅复制，所以它有和原型链继承一样的问题：即，实例之间共享引用值

## 寄生式继承

### 简介

寄生式继承（Parasitic Inheritance）是基于 原型式继承（`Object.create()`） 的增强方式，它的核心思想是：

1. 创建一个基于某个对象的副本（原型式继承）
2. 在副本上添加新的方法或属性
3. 返回这个增强后的对象

寄生式继承适用于：

- 需要在继承的基础上增加额外功能
- 但不适用于性能敏感的场景，因为每次创建对象都会创建新的方法，而不是共享方法

### 示例

```javascript
function createPerson(original) {
  let clone = Object.create(original); // 基于原型式继承创建对象
  clone.sayHi = function () {
    // 添加新的方法
    console.log("Hi, I am " + this.name);
  };
  return clone; // 返回增强后的对象
}

let person = { name: "Alice", friends: ["Bob", "Charlie"] };
let anotherPerson = createPerson(person);

anotherPerson.sayHi(); // "Hi, I am Alice"
console.log(anotherPerson.name); // "Alice"
```

### 缺点

1. 实例之间还是有共享引用属性的问题
2. 寄生式继承无法共享方法，每次创建对象都会重新创建方法，影响性能

寄生式组合继承可以解决问题，我们继续看下去吧

## 寄生式组合继承

### 简介

解决 JavaScript 继承方式（如原型链继承、盗用构造函数继承）都存在一些缺陷：

- **原型链继承**：子类共享父类的引用类型数据，修改一个实例会影响其他实例。
- **盗用构造函数**：不能继承父类原型上的方法，导致方法需要重复定义在构造函数中，浪费内存。
- **组合继承**（构造函数 + 原型继承）：`SubType.prototype = new SuperType();`**会调用两次父类构造函数**，导致性能浪费

### 思路

- 使用盗用构造函数继承实例属性，避免共享引用类型
- 使用 `Object.create()` 继承原型方法，避免重复调用父类构造函数

### 示例

```javascript
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue"];
}

SuperType.prototype.sayName = function () {
  return "My name is " + this.name;
};

function SubType(name, age) {
  SuperType.call(this, name); // 盗用构造函数（继承实例属性）
  this.age = age;
}

// **关键：使用 Object.create() 继承方法**
SubType.prototype = Object.create(SuperType.prototype);
SubType.prototype.constructor = SubType; // 修正 constructor 指向

SubType.prototype.sayAge = function () {
  return "I am " + this.age + " years old.";
};

// 测试
let instance1 = new SubType("Alice", 25);
instance1.colors.push("green");

let instance2 = new SubType("Bob", 30);

console.log(instance1.sayName()); // "My name is Alice"
console.log(instance1.sayAge()); // "I am 25 years old."
console.log(instance1.colors); // ["red", "blue", "green"]

console.log(instance2.sayName()); // "My name is Bob"
console.log(instance2.sayAge()); // "I am 30 years old."
console.log(instance2.colors); // ["red", "blue"]
```
