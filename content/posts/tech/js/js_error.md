---
title: "JavaScript 错误处理"
date: 2025-03-26T15:11:25+08:00
lastmod: 2025-03-26T15:11:25+08:00
author: ["熊大如如"]
tags: # 标签
  - "javascript"
description: ""
weight:
slug: ""
summary: "javascript 错误处理入门"
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

JavaScript 中的错误处理使用 `try/catch`关键字

## 入门

```javascript
try {
  let result = 10 / 0;
  console.log(result);
  throw new Error("自定义错误");
} catch (error) {
  console.error("捕获错误:", error.message);
} finally {
  console.log("无论是否有错误，都会执行");
}
```

- `try` 代码块包含可能发生错误的代码
- `catch` 代码块捕获错误并处理
- `finally`（可选）无论是否发生错误，都会执行

## 内置错误类型

| 错误类型         | 触发场景                                  |
| ---------------- | ----------------------------------------- |
| `Error`          | 通用错误基类                              |
| `SyntaxError`    | 代码语法错误（`eval()`）                  |
| `ReferenceError` | 访问未定义的变量                          |
| `TypeError`      | 操作 `null` / `undefined`，或使用错误类型 |
| `RangeError`     | 数值超出允许范围（数组长度、递归）        |
| `URIError`       | `encodeURI()` / `decodeURI()` 解析失败    |
| `EvalError`      | `eval()` 相关错误（已不常用）             |

## 自定义错误

```javascript
class CustomError extends Error {
  constructor(message) {
    super(message);
    this.name = "CustomError";
  }
}

try {
  throw new CustomError("这是一个自定义错误");
} catch (error) {
  console.error(error.name + ":", error.message);
}
```
