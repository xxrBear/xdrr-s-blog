---
title: "Java 异常"
date: 2025-05-08T20:10:52+08:00
lastmod: 2025-05-08T20:10:52+08:00
author: ["熊大如如"]
tags: # 标签
  - "java"
description: ""
weight:
slug: ""
summary: "Java 异常"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202505062222488.png" # 文章的图片
---

## Java 异常的体系结构

```
java.lang.Throwable
├── Error（严重错误，程序无法处理）
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── ...
└── Exception（程序可以处理的异常）
    ├── Checked Exception（编译时异常）
    │   ├── IOException
    │   ├── SQLException
    │   └── ...
    └── Unchecked Exception（运行时异常，RuntimeException）
        ├── NullPointerException
        ├── ArrayIndexOutOfBoundsException
        ├── ClassCastException
        └── ...
```

### Error（错误）

不建议捕获，如 OutOfMemoryError、StackOverflowError 通常是 JVM 层级的错误

### Checked Exception（受检查异常）

编译阶段必须处理（捕获或抛出）常见如：IOException、SQLException

### Unchecked Exception（非受检查异常）

编译器不会强制处理 多为编程错误：NullPointerException、ArithmeticException、ArrayIndexOutOfBoundsException

## 异常处理机制

### try-catch-finally

```java
try {
    // 可能抛出异常的代码
} catch (ExceptionType e) {
    // 处理异常
} finally {
    // 无论是否发生异常都会执行
}
```

### 多异常处理

```java
catch (IOException | SQLException e) {
    // 多个异常的统一处理
}
```

### try-with-resources

`try-with-resources` 是 Java 7 引入的一个语法结构，用于 自动关闭资源，避免忘记关闭资源导致内存泄漏或句柄耗尽

```java
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
}
// 不需要手动调用 br.close()！
```

## 抛出异常

### throw 语句

```java
throw new IllegalArgumentException("参数非法");
```

### throws

`throws` **的作用**

将异常抛给调用者处理：如果一个方法内部可能抛出异常，但该方法自己不处理（没有 try-catch），就需要在方法签名中使用 `throws` 把异常抛出去，让调用者负责处理

让编译器检查异常是否被处理：对 Checked Exception（受检异常），Java 编译器会强制要求你

- 要么用 `try-catch` 捕获
- 要么用 `throws` 向上传递

```java
public void readFile(String path) throws IOException {
    FileReader reader = new FileReader(path); // 可能抛出 IOException
}
```

调用者必须处理异常：

```java
try {
    readFile("test.txt");
} catch (IOException e) {
    e.printStackTrace();
}
```
