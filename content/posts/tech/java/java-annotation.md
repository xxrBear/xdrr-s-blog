---
title: "Java 注解"
date: 2025-05-16T22:53:29+08:00
lastmod: 2025-05-16T22:53:29+08:00
author: ["熊大如如"]
keywords: 
  - 
categories: # 分类
  - # 在这儿写分类
tags: # 标签
  - "java"
description: ""
weight:
slug: ""
summary: "Java 注解"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202505062222488.png"  # 文章的图片
---

## 什么是注解
+ 注解是 代码级的元数据，用来修饰类、方法、字段、参数等。
+ 作用：
    - 编译检查
    - 代码生成
    - 运行时反射增强
    - 配置替代（如 Spring、JPA）

## 注解的基本语法
```java
// 定义一个简单注解
public @interface MyAnnotation {
    String value();
    int count() default 1; // 带默认值
}
```

+ `@interface` 用来声明注解
+ 属性（成员）可以有默认值
+ 使用时：

```java
@MyAnnotation(value = "Hello", count = 3)
public class TestClass {}
```

> 注解本身啥也不做，但它是个信号。关键在于，你写代码去识别这个信号，然后做想做的事
>

```java
public class Main {
    public static void main(String[] args) {
        // 获取 Demo 类的 Class 对象
        Class<Demo> clazz = Demo.class;

        // 读取注解
        MyAnnotation annotation = clazz.getAnnotation(MyAnnotation.class);

        if (annotation != null) {
            System.out.println("value = " + annotation.value());
            System.out.println("count = " + annotation.count());
        }
    }
}

// value = Hi
// count = 1
```

## 常见的元注解
| 元注解 | 作用 |
| --- | --- |
| `@Target` | 指定注解能作用的目标（类、方法、字段等） |
| `@Retention` | 指定注解的生命周期（SOURCE、CLASS、RUNTIME） |
| `@Documented` | 生成 Javadoc 时包含注解信息 |
| `@Inherited` | 子类是否继承父类的注解 |
| `@Repeatable` | 允许注解可以重复使用 |


示例：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {}
```

## 注解的生命周期
| 生命周期 | 含义 |
| --- | --- |
| SOURCE | 源码阶段有效，编译后丢弃 |
| CLASS | 编译后有效，运行时不可用 |
| RUNTIME | 运行时仍然有效，可通过反射读取 |


## 注解的使用位置
| 元素类型 | 说明 |
| --- | --- |
| TYPE | 类、接口、枚举 |
| FIELD | 成员变量 |
| METHOD | 方法 |
| PARAMETER | 方法参数 |
| CONSTRUCTOR | 构造方法 |
| LOCAL_VARIABLE | 局部变量 |
| ANNOTATION_TYPE | 注解本身 |
| TYPE_USE | 类型使用位置（Java 8） |
| MODULE | 模块（Java 9） |


## 注解的使用示例
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Author {
    String name();
    String date();
}

// 使用
@Author(name = "Tom", date = "2025-05-16")
public class Demo {}
```

## 注解的高级用法
### 反射获取注解信息
```java
Class<?> clazz = Demo.class;
Author author = clazz.getAnnotation(Author.class);
System.out.println(author.name());
System.out.println(author.date());
```

### 嵌套注解
```java
public @interface Roles {
    Role[] value();
}

public @interface Role {
    String name();
}
```

### 重复注解（@Repeatable）
```java
@Repeatable(Roles.class)
public @interface Role {
    String name();
}

@Role(name = "Admin")
@Role(name = "User")
public class UserClass {}
```

## 编译器内置注解
| 注解 | 作用 |
| --- | --- |
| @Override | 表示重写父类方法 |
| @Deprecated | 标记已过时 |
| @SuppressWarnings | 屏蔽编译器警告 |


## 常见框架中的注解
+ Spring 注解：`@Component`、`@Autowired`、`@RestController`
+ JPA 注解：`@Entity`、`@Table`、`@Id`
+ Lombok 注解：`@Data`、`@Builder`、`@Getter`
+ 自定义 AOP 注解、校验注解

## 注解处理器（APT - Annotation Processing Tool）
+ 编译时对注解进行处理（如自动生成代码）
+ 关键类：`javax.annotation.processing.AbstractProcessor`
+ 常用框架：Lombok、Dagger2、ButterKnife

简单示例：

```java
@SupportedAnnotationTypes("com.example.MyAnnotation")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class MyProcessor extends AbstractProcessor {
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        // 处理注解逻辑
        return true;
    }
}
```

## 类型注解
+ `@Target(ElementType.TYPE_USE)` 支持更细粒度的类型位置

```java
@NotNull String name;
List<@NonNull String> list;
```

