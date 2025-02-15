---
title: "JavaScript 数组浅谈"
date: 2025-02-15T16:54:12+08:00
lastmod: 2025-02-15T16:54:12+08:00
author: ["熊大如如"]
tags: # 标签
  - "javascript"
description: ""
weight:
slug: ""
summary: "JavaScript 数组方法的总结"
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

## 一、创建数组

### 1.字面量

```javascript
let nameArray = []; // 字面量创建数组

console.log(nameArray); // []
```

非常直观

### 2.构造函数方法

```javascript
let newNameArray = Array();

console.log(newNameArray); // []

// Array.of 方法创建数组 ES6
let arr = Array.of(1, 2, 3);
console.log(arr);
```

### 3.ES6 新增静态方法

**Array.of**

用于将一组参数转化为数组实例

```javascript
let arr = Array.of(1, 2, 3);
console.log(arr); // [1, 2, 3]
```

**Array.from**

接收两个参数，第一个参数接收类数组对象(所有可迭代对象)，第二个参数可选，接收映射函数参数

```javascript
// 字符换拆分
let arr1 = Array.from("name");
console.log(arr1); // ['n', 'a', 'm', 'e']

let a1 = [
  [1, 2],
  [3, 4],
];
console.log(Array.from(o1)); // [Array(2), Array(2)]

// 现有数组的浅复制
let a1 = [
  [1, 2],
  [3, 4],
];

a2 = Array.from(a1);
console.log(a1 === a2); // false

// 替代 Array.from().map()
const a3 = [1, 2, 3, 4];
a4 = Array.from(a3, (x) => x ** 2);
console.log(a4); // [1, 4, 9, 16]
```

## 二、检查数组

判断一个对象是不是数组

### 1.instanseof

```javascript
let a1 = [];
console.log(a1 instanceof Array); // true

let a2 = "hello";
console.log(a2 instanceof Array); // false
```

### 2.isArray

解决了不同上下文之间传递的问题，因为它不检查值的原型链，`instanceof Array` 会检查（没懂 🥲）

```javascript
console.log(Array.isArray([])); // true
```

## 三、检索内容

ES6 新增的三种方法，可以检索数组的索引和值。分别是 `keys()`、`values()`、`entries()`，它们都返回迭代器对象，可以使用 `Array.from`方法转化为数组实例

### 1.keys

返回数组索引

```javascript
const a1 = [1, 2, 3];
console.log(Array.from(a1.keys())); //[0, 1, 2]
```

### 2.values

返回数组值

```javascript
const a1 = [1, 2, 3];
console.log(Array.from(a1.values())); // [1, 2, 3]
```

### 3.entries

返回 索引/值 对

```javascript
const a1 = [1, 2, 3];
console.log(Array.from(a1.entries())); // [[0, 1], [1, 2], [2, 3]]
```

## 四、复制和填充

### 1.fill

使用 `fill()`方法，会将可以向一个存在的数组中，批量插入相同的值。

它接收三个参数，第一个参数是要批量填充的值，第二个参数开始索引，第三个参数结束索引。

它会静默忽略超出索引长度的部分

```javascript
const zeroes = [0, 0, 0, 0, 0];
zeroes.fill(5);
console.log(zeroes); // [5, 5, 5, 5, 5]

zeroes.fill(3, 3, 10);
console.log(zeroes); // [5, 5, 5, 3, 3]
```

### 2.copyWithin

使用 `copyWithin`方法，会按照指定范围浅复制数组中的部分内容，然后将它们插入索引开始的地方

- target (必需): 表示复制元素的目标起始位置（索引）
- start (可选): 表示复制元素的起始位置（索引）
- end (可选): 表示复制元素的结束位置（索引）

```javascript
const arr = [1, 2, 3, 4, 5];
arr.copyWithin(0, 1); // 将索引 3 到末尾的元素复制到索引 0 的位置
console.log(arr); // [4, 5, 3, 4, 5]
```

使用负数索引

```javascript
const arr = [1, 2, 3, 4, 5];
arr.copyWithin(-2, 0); // 将索引 0 到末尾的元素复制到倒数第二个位置
console.log(arr); // [1, 2, 3, 1, 2]
```

超出数组长度的数据，静默忽略

```javascript
const arr = [1, 2, 3, 4, 5];
arr.copyWithin(0, 6); // start 超出数组长度，不会复制任何元素
console.log(arr); // [1, 2, 3, 4, 5]
```

## 五、转换方法

每个对象都有 `toLocaleString`和 `toString`方法，分别调用对象对应的同名方法。

`valueOf`方法返回自身

对于数组来说，`toLocaleString`和 `toString`都返回逗号拼接的字符串

### toString/toLocaleString/valueOf

```javascript
const arr1 = [1, 2, 3];
const arr2 = [1, 2, 3];

console.log(arr1.valueOf()); // [1, 2, 3]
console.log(arr1.toLocaleString()); // 1,2,3
console.log(arr2.toString()); // 1,2,3
```

## 六、栈与队列

### 1.push

推送数据至数组

```javascript
let arr = [];
arr.push(1);
```

### 2.pop

尾部弹出一项

```javascript
arr.pop();
```

### 3.shift

删除数组第一项，并且返回

```javascript
let arr = [1, 2, 3];
arr.shift();
```

### 4.unshift

数组开头推入数据

```javascript
arr.unshift(6, 7);
```

## 七、排序方法

### 1.reserve

数组反向排序

```javascript
let arr = [1, 2, 4, 3, 5];
arr.reverse();
console.log(arr); // [5, 3, 4, 2, 1]
```

### 2.sort

默认情况下，会将每一项的值转化为字符串，然后比较。也可以传入一个比较函数，按规则比较

```javascript
let arr = [1, 2, 4, 3, 5, 11];

function compare(v1, v2) {
  return v1 - v2;
}
arr.sort(compare);
console.log(arr); // [1, 2, 3, 4, 5, 11]
```

## 八、操作方法

### 1.concat

将现有数组的全部元素都拿到，组成一个新的数组

```javascript
let arr = [1, 2, 3];
let arr2 = arr.concat("xxx", ["sss", "fff"]);
console.log(arr2); // [1, 2, 3, 'xxx', 'sss', 'fff']
```

### 2.slice

根据索引位置，返回新数组

```javascript
let colors = ["red", "green", "blue", "yellow", "purple"];
let colors2 = colors.slice(1);
let colors3 = colors.slice(1, 4);

console.log(colors2); // ['green', 'blue', 'yellow', 'purple']
console.log(colors3); // ['green', 'blue', 'yellow']
```

### 3.splice

可能是最强大的数组方法，接收三个参数

1）数组开始位置（必填）

2）要删除的元素数量（必填）

3）要替换的元素（选填）

- 删除

```javascript
let arr = [1, 3, 5, 7, 9, 11];
let arrRemoved = arr.splice(0, 2);
console.log(arr); //[5, 7, 9, 11]
console.log(arrRemoved); //[1, 3]
```

- 插入

```javascript
let array1 = [22, 3, 31, 12];
array1.splice(1, 0, 12, 35); //[]

console.log(array1); // [22, 12, 35, 3, 31, 12]
```

- 替换

```javascript
const array1 = [22, 3, 31, 12];
array1.splice(1, 1, 8); //[3]

console.log(array1); // [22, 8, 31, 12]
```

## 九、搜索

### 1.indexOf

找出对应值所在的数组索引，不存在则返回 -1

```javascript
let arr = [1, 3, 5, 7, 7, 5, 3, 1];
console.log(arr.indexOf(5)); //2
console.log(arr.indexOf(5, 2)); //2
console.log(arr.indexOf("5")); //-1
```

### 2.lastIndexOf

与 `indexOf`相似，只是从末尾向前找

```javascript
let arr = [1, 3, 5, 7, 7, 5, 3, 1];
console.log(arr.lastIndexOf(5)); //5
console.log(arr.lastIndexOf(5, 4)); //2
```

### 3.includes

ES7 新增

是否找到一个与元素匹配的值，返回对应的布尔值

```javascript
const array1 = [22, 3, 31, 12, arr];
const includes = array1.includes(31);
console.log(includes); // true
const includes1 = array1.includes(31, 3); // 从索引3开始查找31是否存在
console.log(includes1); // false
```

### 4.find

接收三个参数，元素、索引、数组本身

总最小索引开始，找到匹配的第一个元素，返回

```javascript
let arr = [1, 2, 3, 5, 1, 9];

console.log(
  arr.find((element, index, array) => {
    return element > 2;
  })
); // 3 返回匹配的值
```

### 5.findIndex

与 `find`一致，但是返回的是匹配的第一个元素的索引

```javascript
let arr = [1, 2, 3, 5, 1, 9];

console.log(
  arr.findIndex((element, index, array) => {
    return element > 2;
  })
); // 3 返回匹配的值
```

## 十、迭代方法

ECMAScript 为数组定义了 5 个迭代方法，每个方法接收两个参数

1）以每一项为参数运行的函数（必填）

2）可选的作为函数运行上下文的作用域对象

必填的函数结束三个元素：

1.数组元素

2.元素索引

3.数组本身

### 1.every

数组的每一项都让传入的函数返回 true，则这个方法返回 true 否则返回 false

```javascript
let numbers = [1, 2, 3];

console.log(numbers.every((ele, index, arr) => ele >= 1)); // true
console.log(numbers.every((ele, index, arr) => ele > 2)); // false
```

### 2.some

数组只要有一项都让传入的函数返回 true，则这个方法返回 true 否则返回 false

```javascript
let numbers = [1, 2, 3];

console.log(numbers.some((ele, index, arr) => ele >= 1)); // true
console.log(numbers.some((ele, index, arr) => ele > 3)); // false
```

### 3.filter

基于给定的函数，判断数组中的每一项是否返回的新数组

```javascript
let numbers = [1, 2, 3];

console.log(numbers.filter((ele, index, arr) => ele > 1)); // [2, 3]
```

### 4.map

基于给定的函数，对数组的每一项运行该函数，得到返回结果的新数组

```javascript
let numbers = [1, 2, 3];

console.log(numbers.map((ele, index, arr) => ele ** 2)); // [1, 4, 9]
```

### 5.forEach

相当于 for 循环遍历数组

```javascript
let arr = [11, 22, 33, 44, 55];
arr.forEach(function (e, index, arrry) {
  console.log(e);
});
```

## 十一、归并方法

ECMAScript 提供两个归并方法，遍历数组每一项，返回一个最终值。

`reduce`和 `reductRight`分别接收四个参数

- 上一个归并值
- 当前项
- 当前项的索引
- 数组本身

### 1.reduce

reduct 从左到右

```javascript
let values = [1, 2, 3, 4, 5];
let sum = values.reduce((prev, cur, index, array) => prev + cur);

console.log(sum); // 15
```

### 2.reductRight

从右到左

```javascript
let values = [1, 2, 3, 4, 5];
let sum = values.reduceRight((prev, cur, index, array) => prev + cur);

console.log(sum); // 15
```
