---
title: "JavaScript async/await 异步函数入门"
date: 2025-03-25T11:47:05+08:00
lastmod: 2025-03-25T11:47:05+08:00
author: ["熊大如如"]
tags: # 标签
  - "javascript"
description: ""
weight:
slug: ""
summary: "JavaScript 异步函数入门"
draft: false # 是否为草稿
mermaid: true #是否开启 mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202502151647700.png" # 文章的图片
---

## 简介

`async/await` 是 JavaScript 用于处理异步操作的一种语法糖，它基于 Promise，使得异步代码的可读性更接近同步代码

### async 关键字

示例

```javascript
async function foo() {
  return "Hello";
}
foo().then(console.log); // 输出: Hello
```

等价于

```javascript
function foo() {
  return Promise.resolve("Hello");
}
foo().then(console.log); // 输出: Hello
```

### await 关键字

```javascript
async function fetchData() {
  let result = await new Promise((resolve) => {
    setTimeout(() => resolve("数据加载完毕"), 2000);
  });
  console.log(result);
}
fetchData();
```

**执行流程：**

1. `await` 让 JavaScript 引擎暂停 `fetchData` 函数的执行，并等待 Promise 解析
2. 其他任务可以继续执行，不会阻塞主线程
3. Promise 解析后，继续执行 `fetchData` 函数

### 处理错误

```javascript
async function fetchData() {
  try {
    let result = await new Promise((_, reject) => {
      setTimeout(() => reject("发生错误"), 2000);
    });
    console.log(result);
  } catch (error) {
    console.error("错误:", error);
  }
}
fetchData();
```

**执行流程：**

1. `await` 发生 `Promise` 拒绝（`reject`）
2. `try` 捕获不到时，错误会抛出到全局（`UnhandledPromiseRejection`）
3. `catch` 捕获到错误，打印 错误: 发生错误

## 底层原理

### 简介

- `async` 函数本质上 返回一个 Promise
- `await` 其实是 Promise.then 的语法糖，JavaScript 引擎会暂停 `async` 函数的执行，等待 Promise 解析后继续执行

### 示例

```javascript
async function foo() {
  let result = await someAsyncFunction();
  console.log(result);
}
```

**相当于**

```javascript
function foo() {
  return someAsyncFunction().then((result) => {
    console.log(result);
  });
}
```
