---
title: "JavaScript 期约"
date: 2025-03-25T09:10:08+08:00
lastmod: 2025-03-25T09:10:08+08:00
author: ["熊大如如"]
keywords:
  -
categories: # 分类
  -  # 在这儿写分类
tags: # 标签
  - "javascript"
description: ""
weight:
slug: ""
summary: "JavaScript 期约相关知识"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "" # 文章的图片
---

## 简介

### 异步函数

JavaScript 是单线程语言，默认情况下代码是同步执行的，但在处理 I/O 操作（如网络请求、定时器、数据库查询等） 时，使用 异步函数 可以避免阻塞主线程，提高程序的运行效率

JavaScript 实现异步主要有以下几种方法：

- 回调函数
- Promise
- async/await

本文主要介绍 Promise

### 定时器函数

- setTimeout

**作用：指定时间后运行代码**

**语法**

```javascript
setTimeout(callback, delay, param1, param2, ...);
```

**示例**

```javascript
console.log("开始");

setTimeout(() => {
  console.log("2秒后执行");
}, 2000);

console.log("结束");
```

- setInterval

**作用：重复执行某任务**

**语法：**

```javascript
setInterval(callback, delay, param1, param2, ...);
```

**示例**

```javascript
setInterval(() => {
  console.log("每秒执行一次");
}, 1000);
```

### 异步回调函数

异步回调不会立即执行，而是在任务完成后执行，通常用于异步操作（如 I/O、定时器、网络请求）

**示例**

```javascript
function fetchData(callback) {
  setTimeout(() => {
    console.log("数据已获取");
    callback("处理异步数据");
  }, 2000);
}

fetchData((data) => {
  console.log(data);
});
```

**回调地狱**

```javascript
setTimeout(() => {
  console.log("第一步完成");
  setTimeout(() => {
    console.log("第二步完成");
    setTimeout(() => {
      console.log("第三步完成");
    }, 1000);
  }, 1000);
}, 1000);
```

- 代码嵌套层次太深，难以管理
- 代码可读性差，难以维护

**Promise 用以解决回调地狱！**

## Promise

### 简介

`Promise` 是 JavaScript 异步编程的重要概念，它用于处理异步操作的结果，避免回调地狱（Callback Hell），提高代码的可读性和可维护性

**Promise 是什么**

- `Promise` 是一个表示异步操作最终完成（成功或失败）的对象
- `Promise` 解决了回调地狱的问题，使异步代码更加清晰、易读
- `Promise` 的状态：
  - pending（进行中）：Promise 初始状态，异步操作尚未完成
  - fulfilled（已完成）：异步操作成功，返回 `resolve(value)`
  - rejected（已拒绝）：异步操作失败，返回 `reject(error)`

**示例**

```javascript
let promise = new Promise((resolve, reject) => {
  // 异步操作
  if (成功) {
    resolve("成功的结果");
  } else {
    reject("错误信息");
  }
});

promise.then(
  (result) => console.log("成功:", result),
  (error) => console.log("失败:", error)
);
```

### 示例：模拟异步请求

```javascript
let fetchData = new Promise((resolve, reject) => {
  setTimeout(() => {
    let success = true;
    if (success) {
      resolve("数据获取成功！");
    } else {
      reject("数据获取失败！");
    }
  }, 2000);
});

fetchData
  .then((result) => console.log(result))
  .catch((error) => console.log(error));
```

执行流程

1. `new Promise` 立即执行，其中 `setTimeout()`模拟异步操作
2. `resolve("数据获取成功！")` 触发 `then()`
3. `reject("数据获取失败！")` 触发 `catch()`

## Promise 方法

### then 处理成功

`then()` 方法接收一个回调，在 `Promise` **成功（resolved）**时调用

```javascript
let p = new Promise((resolve) => resolve("成功的数据"));
p.then((result) => console.log(result)); // "成功的数据"
```

### catch 处理失败

`catch()` 方法接收一个回调，在 `Promise` **失败（rejected）**时调用

```javascript
let p = new Promise((_, reject) => reject("出错了"));
p.catch((error) => console.log(error)); // "出错了"
```

### finally 无论成功或失败都会执行

```javascript
let p = new Promise((resolve) => resolve("OK"));
p.finally(() => console.log("操作完成")).then((result) => console.log(result));
```

### 链式调用

```javascript
new Promise((resolve) => {
  setTimeout(() => resolve(1), 1000);
})
  .then((result) => {
    console.log(result); // 1
    return result * 2;
  })
  .then((result) => {
    console.log(result); // 2
    return result * 2;
  })
  .then((result) => {
    console.log(result); // 4
  });
```

## Promise 并行执行

有时我们需要同时执行多个异步任务，这时可以使用 `Promise.all()` 或 `Promise.race()`

### Promise.all 全部成功才返回

```javascript
let p1 = new Promise((resolve) => setTimeout(() => resolve("任务1完成"), 1000));
let p2 = new Promise((resolve) => setTimeout(() => resolve("任务2完成"), 2000));

Promise.all([p1, p2]).then((results) => console.log(results));
```

**输出**

```javascript
（2秒后）
  ["任务1完成", "任务2完成"]
```

所有 Promise 完成才返回，否则返回 `reject`。

### Promise.race 最快的 Promise 先返回

```javascript
let p1 = new Promise((resolve) => setTimeout(() => resolve("任务1完成"), 1000));
let p2 = new Promise((resolve) => setTimeout(() => resolve("任务2完成"), 2000));

Promise.race([p1, p2]).then((result) => console.log(result));
```

**输出**

```plain
（1秒后）
任务1完成
```

特点：只返回最先完成的 Promise。
