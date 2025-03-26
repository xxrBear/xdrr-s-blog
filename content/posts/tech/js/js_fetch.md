---
title: "JavaScript Fetch API"
date: 2025-03-26T15:11:32+08:00
lastmod: 2025-03-26T15:11:32+08:00
author: ["熊大如如"]
tags: # 标签
  - "javascript"
description: ""
weight:
slug: ""
summary: "javascript fetch api 总结"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
---

## 简介

`fetch()` API 是用于发送 HTTP 请求的现代异步方法，它基于 `Promise`，比传统的 `XMLHttpRequest` 更加简洁、强大

## 示例

### 基本语法

```javascript
fetch(url, options)
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => console.error("请求错误:", error));
```

- `url`：请求的地址（必须）
- `options`：可选的配置对象（如请求方法、头部信息、数据等）
- `fetch()` 返回一个 `Promise<Response>` 对象

### 最简单的示例

```javascript
fetch("https://jsonplaceholder.typicode.com/posts/1")
  .then((response) => response.json()) // 解析 JSON
  .then((data) => console.log("获取的数据:", data))
  .catch((error) => console.error("请求错误:", error));
```

### 如何解析响应

```javascript
fetch("https://jsonplaceholder.typicode.com/posts/1")
  .then((response) => {
    console.log("状态码:", response.status);
    console.log("响应头:", response.headers.get("Content-Type"));
    return response.text(); // 或者 response.json(), response.blob()
  })
  .then((data) => console.log("响应内容:", data))
  .catch((error) => console.error("请求错误:", error));
```

常用解析方式：

- `response.json()`：解析 JSON 数据
- `response.text()`：解析文本数据
- `response.blob()`：解析二进制数据（图片、文件）
- `response.arrayBuffer()`：解析底层二进制数据
- `response.formData()`：解析表单数据

### 发送 POST 请求

```javascript
fetch("https://jsonplaceholder.typicode.com/posts", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    title: "foo",
    body: "bar",
    userId: 1,
  }),
})
  .then((response) => response.json())
  .then((data) => console.log("返回的数据:", data))
  .catch((error) => console.error("请求错误:", error));
```

## 使用 `async/await` 方式

```javascript
async function fetchData() {
  try {
    let response = await fetch("https://jsonplaceholder.typicode.com/posts/1");
    if (!response.ok) throw new Error(`HTTP 错误: ${response.status}`);
    let data = await response.json();
    console.log(data);
  } catch (error) {
    console.error("请求失败:", error);
  }
}
fetchData();
```
