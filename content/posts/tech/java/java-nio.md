---
title: "Java NIO 文件处理接口"
date: 2025-05-11T11:59:00+08:00
lastmod: 2025-05-11T11:59:00+08:00
author: ["熊大如如"]
keywords:
  -
categories: # 分类
  -  # 在这儿写分类
tags: # 标签
  - "java"
description: ""
weight:
slug: ""
summary: "Java NIO 文件处理接口相关知识"
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

## 简介

Java NIO.2 是 Java 7 引入的新一代文件系统 API，`java.nio.file` 是它的核心包，功能上远超 `java.io.File`，提供了：

- 更强大的路径处理能力（跨平台）
- 文件读写、复制、移动、删除等操作
- 文件系统访问
- 符号链接支持
- 文件监控（WatchService）

## 核心类

| 类/接口名             | 简介                                             |
| --------------------- | ------------------------------------------------ |
| `Path`（接口）        | 抽象表示一个路径，替代 `File`                    |
| `~~Paths~~`~~（类）~~ | ~~工具类，用于创建 ~~`~~Path~~`~~ 对象~~         |
| `Files`（类）         | 文件和目录操作的工具类（全部为静态方法）         |
| `FileSystem`          | 表示整个文件系统，可获取根路径等                 |
| `FileSystems`         | 获取 `FileSystem` 的工厂类                       |
| `StandardOpenOption`  | 用于指定打开文件时的选项（读、写、追加、创建等） |
| `LinkOption`          | 是否跟随符号链接                                 |
| `StandardCopyOption`  | 复制/移动文件时的选项                            |
| `WatchService`        | 文件系统事件监听服务                             |
| `WatchKey`            | 表示一个可轮询的事件键                           |
| `WatchEvent`          | 表示目录变更事件                                 |
| `DirectoryStream`     | 更传统地迭代目录内容                             |
| `BasicFileAttributes` | 获取文件属性（大小、时间、是否为目录等）         |

## 常用核心类

### Path 与 Paths

#### 创建 Path 实例

```java
Path path1 = Paths.get("file.txt");  // 相对路径
Path path2 = Paths.get("/home/user/file.txt"); // 绝对路径
Path path3 = Paths.get("folder", "sub", "file.txt"); // 多段构建
```

#### 常用方法

```java
path.getFileName();     // 文件名
path.getParent();       // 上级路径
path.getRoot();         // 根路径（如 / 或 C:\）
path.toAbsolutePath();  // 转为绝对路径
path.normalize();       // 规范化路径（处理 ../）
path.resolve("other");  // 拼接路径
path.relativize(other); // 相对路径
path.startsWith("/");   // 是否以某段路径开始
```

### Files 类功能总结

#### 文件读写

```java
Files.readAllBytes(path);
Files.readAllLines(path);
Files.write(path, bytes);     // 覆盖
Files.write(path, bytes, StandardOpenOption.APPEND); // 追加
```

#### 文件管理

```java
Files.exists(path);
Files.isDirectory(path);
Files.copy(src, dest, StandardCopyOption.REPLACE_EXISTING);
Files.move(src, dest);
Files.delete(path);
```

#### 创建文件/目录

```java
Files.createFile(path);
Files.createDirectory(path);
Files.createDirectories(path); // 多层
```

#### 获取文件属性

```java
BasicFileAttributes attrs = Files.readAttributes(path, BasicFileAttributes.class);
attrs.size();
attrs.creationTime();
attrs.lastModifiedTime();
```

#### 遍历文件夹

```java
try (Stream<Path> stream = Files.list(path)) {
    stream.forEach(System.out::println);
}
```

#### 深度遍历（递归）

```java
Files.walk(path).filter(Files::isRegularFile).forEach(System.out::println);
```

### StandardOpenOption（文件写入控制）

| 枚举常量            | 含义             |
| ------------------- | ---------------- |
| `WRITE`             | 写入文件         |
| `APPEND`            | 追加写入         |
| `CREATE`            | 文件不存在则创建 |
| `CREATE_NEW`        | 文件存在则抛异常 |
| `TRUNCATE_EXISTING` | 清空文件         |
| `READ`              | 读取             |

### 文件系统操作（FileSystem / FileSystems）

```java
FileSystem fs = FileSystems.getDefault();
Path root = fs.getPath("/");
Iterable<Path> roots = fs.getRootDirectories();
```

可以用于获取多个挂载点（如 C:\、D:\）

### 文件监控（WatchService）

```java
Path dir = Paths.get(".");
WatchService watcher = FileSystems.getDefault().newWatchService();
dir.register(watcher, ENTRY_CREATE, ENTRY_DELETE, ENTRY_MODIFY);

while (true) {
    WatchKey key = watcher.take();
    for (WatchEvent<?> event : key.pollEvents()) {
        System.out.println("Event kind: " + event.kind());
        System.out.println("File affected: " + event.context());
    }
    key.reset();
}
```

## 应用场景示例

### 递归删除目录

```java
Files.walk(path)
     .sorted(Comparator.reverseOrder()) // 先删子目录
     .forEach(Files::delete);
```

### 搜索特定扩展名文件

```java
Files.walk(Paths.get("."))
     .filter(p -> p.toString().endsWith(".java"))
     .forEach(System.out::println);
```

### 读取大文件推荐方式（逐行）

```java
try (BufferedReader reader = Files.newBufferedReader(path)) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
}
```

## 易错点提示

| 问题场景             | 建议                                        |
| -------------------- | ------------------------------------------- |
| 符号链接问题         | 用 `LinkOption.NOFOLLOW_LINKS` 判断属性     |
| 文件已存在时写入失败 | 使用 `StandardOpenOption.CREATE` 或覆盖选项 |
| 文件不存在时报错     | 使用 `Files.exists()` 检查后处理            |
| 删除非空目录失败     | 使用 `walk` 并排序后从叶节点开始删除        |

## 学习建议

- 建议掌握 `Path` + `Files` + `StandardOpenOption` 的配套使用方式
- 学习使用 `try-with-resources` 来管理 `Stream<Path>` 等资源
- WatchService 是做日志监控、自动部署等实用场景的关键工具
- Java 的 NIO 文件 API 已足够替代老旧的 `File` 类，建议优先使用
