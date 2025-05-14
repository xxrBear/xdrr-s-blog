---
title: "Java 泛型"
date: 2025-05-14T16:32:57+08:00
lastmod: 2025-05-14T16:32:57+08:00
author: ["熊大如如"]
tags: # 标签
  - "java"
description: ""
weight:
slug: ""
summary: "Java 泛型"
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

## 泛型简介

- 定义：泛型（Generics）是 Java 提供的一种参数化类型机制，可以让类、接口、方法操作任意类型的数据
- 目的：
  - 编译时类型检查，提升类型安全
  - 消除强制类型转换
  - 代码复用，提高可读性和维护性

## 泛型基础

### 1. 泛型类

```java
public class Box<T> {
    private T value;

    public void setValue(T value) {
        this.value = value;
    }

    public T getValue() {
        return value;
    }

    public static void main(String[] args) {
        Box<String> box = new Box<>();
        box.setValue("hello");
        System.out.println(box.getValue());
    }
}
```

### 2. 泛型接口

```java
public interface Generator<T> {
    T next();
}

public class StringGenerator implements Generator<String> {
    public String next() {
        return "Hello";
    }
}
```

### 3. 泛型方法

```java
public class Util {
    public static <T> void printArray(T[] array) {
        for (T item : array) {
            System.out.println(item);
        }
    }
}

// 使用
Util.printArray(new Integer[]{1, 2, 3});
```

## 泛型进阶

### 4. 通配符 `?`

#### 4.1 无限定通配符

`List<?>` 表示：这是一个元素类型未知的 `List`，它可以是任何类型的 `List`

```java
public void printList(List<?> list) {
    for (Object obj : list) {
        System.out.println(obj);
    }
}
```

#### 4.2 上界通配符 `? extends`

表示参数是某个类或其子类（只读，不能添加元素，安全）

这时，`list` 可能是：

- `List<Integer>`
- `List<Double>`
- `List<Number>`

```java
public void printNumberList(List<? extends Number> list) {
    for (Number n : list) {
        System.out.println(n);
    }
}
```

#### 4.3 下界通配符 `? super`

表示参数是某个类或其父类（可写，添加时安全）

```java
public void addNumber(List<? super Integer> list) {
    list.add(1);
    list.add(2);
}
```

### 5. 通配符与泛型方法对比

- 通配符常用于方法参数
- 泛型方法常用于方法返回类型和参数之间建立类型关系

## 四、泛型的类型擦除

### 类型擦除原理

- Java 泛型是伪泛型，编译后会擦除泛型信息
- 类型擦除后：
  - 泛型类型变为 `Object`（或上限类型）
  - 插入强制类型转换
  - 不支持泛型数组、泛型的运行时反射

### 泛型擦除带来的限制

- 不能使用 `instanceof` 判断泛型类型
- 不能直接创建泛型数组
- 不能在静态方法、静态变量使用类的泛型参数

### 获取泛型类型的方式

```java
public class Demo<T> {
    public Class<?> getGenericType() {
        Type type = getClass().getGenericSuperclass();
        if (type instanceof ParameterizedType) {
            Type[] actualTypes = ((ParameterizedType) type).getActualTypeArguments();
            return (Class<?>) actualTypes[0];
        }
        return Object.class;
    }
}
```

## 泛型高级用法

### 泛型约束（限制类型范围）

```java
public class Calculator<T extends Number> {
    public double add(T a, T b) {
        return a.doubleValue() + b.doubleValue();
    }
}
```

### 泛型接口的实现

```java
public class MyList<T> implements List<T> {
    // 必须全部使用 T，而不能具体化为某个类型
}
```

### 泛型方法返回泛型类型

```java
public static <T> T copy(T value) {
    return value;
}
```

### 多个泛型参数

```java
public class Pair<K, V> {
    private K key;
    private V value;
}
```

### 泛型嵌套

```java
Box<List<String>> box = new Box<>();
```

## 泛型的常见陷阱和注意点

- 不支持泛型数组：`new T[]` 错误
- 泛型信息在运行时会被擦除：`List<String>` 和 `List<Integer>` 是同一个类
- `instanceof` 时不能直接使用泛型
- 静态上下文不能使用泛型参数
