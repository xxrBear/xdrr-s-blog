---
title: "Java 权限修饰符"
date: 2025-05-14T17:07:39+08:00
lastmod: 2025-05-14T17:07:39+08:00
author: ["熊大如如"]
tags: # 标签
  - "java"
description: ""
weight:
slug: ""
summary: "Java 权限修饰符"
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

Java 提供 4 种访问控制级别，用于控制类、方法、成员变量、构造器等的访问范围。

## 四种权限修饰符

| 修饰符      | 同类（自己） | 同包（包内其他类） | 子类（即使不在同包） | 其他包 |
| ----------- | ------------ | ------------------ | -------------------- | ------ |
| `public`    | ✅           | ✅                 | ✅                   | ✅     |
| `protected` | ✅           | ✅                 | ✅                   | ❌     |
| 无修饰符    | ✅           | ✅                 | ❌                   | ❌     |
| `private`   | ✅           | ❌                 | ❌                   | ❌     |

## 逐个解释

### public

- 可以被任何类访问
- 应用场景：API、工具类、公共接口等

```java
public class User {
    public String name;
    public void sayHello() {}
}
```

### protected

- 同包可以访问
- 不同包的子类可以访问（继承时）
- 应用场景：父类想要对子类开放，但不希望其他包随便访问

```java
protected String name;
protected void sayHello() {}
```

### 默认

- 只有同包内的类可以访问
- 不同包的类无法访问，即使是子类
- 应用场景：包内工具、包内部类

```java
String name;  // 无修饰符
void sayHello() {}
```

### private

- 只能在本类内部访问
- 应用场景：封装数据、内部方法

```java
private String name;
private void sayHello() {}
```

## 三、在类、成员、构造器中的应用

| 成员位置 | 可用修饰符                                 |
| -------- | ------------------------------------------ |
| 顶级类   | `public`、（默认）                         |
| 成员变量 | `public`、`protected`、（默认）、`private` |
| 方法     | 同上                                       |
| 构造器   | 同上                                       |
| 内部类   | 同上                                       |

## 重点陷阱与注意点

1. 类只能是 `public` 或者默认，不能是 `protected` 或 `private`否则编译器报错
2. 内部类可以使用所有修饰符
3. `private` 方法不会被子类继承，也不能重写，但可以重新定义同名方法

## 典型应用场景总结

| 修饰符      | 应用场景                             |
| ----------- | ------------------------------------ |
| `public`    | API、工具、对外开放的类和方法        |
| `protected` | 框架内部类、希望子类可以使用但不对外 |
| 默认        | 包内部封装，不希望包外访问           |
| `private`   | 完全封装，内部数据、内部逻辑         |
