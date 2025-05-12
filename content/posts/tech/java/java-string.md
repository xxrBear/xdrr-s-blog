---
title: "Java 字符串常用接口"
date: 2025-05-12T15:05:14+08:00
lastmod: 2025-05-12T15:05:14+08:00
author: ["熊大如如"]
tags: # 标签
  - "java"
description: ""
weight:
slug: ""
summary: "Java 字符串常用接口"
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

## 1. 基本属性与判断

```java
String s = "Hello World";

// 获取长度
System.out.println(s.length()); // 11

// 判断是否为空
System.out.println(s.isEmpty()); // false

// 判断是否全是空白
System.out.println("  \t\n".isBlank()); // true (Java 11+)

// 判断包含
System.out.println(s.contains("World")); // true

// 判断前缀
System.out.println(s.startsWith("Hello")); // true

// 判断后缀
System.out.println(s.endsWith("World")); // true

// 判断相等
System.out.println(s.equals("Hello World")); // true

// 忽略大小写比较
System.out.println(s.equalsIgnoreCase("hello world")); // true
```

## 2. 查找

```java
String s = "hello world hello";

// 查找第一个出现的位置
System.out.println(s.indexOf("hello")); // 0

// 查找最后一个出现的位置
System.out.println(s.lastIndexOf("hello")); // 12

// 查找指定字符
System.out.println(s.indexOf('o')); // 4

// 查找不存在返回 -1
System.out.println(s.indexOf("java")); // -1
```

## 3. 截取

```java
String s = "Hello World";

// 从索引开始到末尾
System.out.println(s.substring(6)); // World

// 从索引开始到结束索引（不包含结束索引）
System.out.println(s.substring(0, 5)); // Hello
```

## 4. 替换

```java
String s = "apple banana apple";

// 替换所有
System.out.println(s.replace("apple", "orange")); // orange banana orange

// 替换第一个匹配
System.out.println(s.replaceFirst("apple", "orange")); // orange banana apple

// 替换正则
System.out.println(s.replaceAll("\\s", "-")); // apple-banana-apple
```

## 5. 分割与拼接

```java
String s = "a,b,c,d";

// 分割
String[] parts = s.split(",");
System.out.println(Arrays.toString(parts)); // [a, b, c, d]

// 拼接
String joined = String.join("-", parts);
System.out.println(joined); // a-b-c-d
```

## 6. 大小写与去空格

```java
String s = "  Hello World  ";

// 去除前后空格
System.out.println(s.trim()); // "Hello World"

// 去除前后空白字符 (Java 11+)
System.out.println(s.strip()); // "Hello World"

// 转大写
System.out.println(s.toUpperCase()); // "  HELLO WORLD  "

// 转小写
System.out.println(s.toLowerCase()); // "  hello world  "
```

## 7. 类型转换

```java
int num = 123;

// 数值转字符串
String s = String.valueOf(num);
System.out.println(s); // "123"

// 对象转字符串
String s2 = String.valueOf(new Object()); // 默认 Object.toString()
System.out.println(s2);

// 字符串转字符数组
char[] chars = s.toCharArray();
System.out.println(Arrays.toString(chars)); // [1, 2, 3]

// 字符数组转字符串
String s3 = new String(chars);
System.out.println(s3); // 123
```

## 8. 格式化

```java
String s = String.format("Hello %s, you have %d messages.", "Tom", 5);
System.out.println(s); // Hello Tom, you have 5 messages.
```

## 9. intern

`intern()` 是 `String` 类的一个实例方法，用于将字符串对象放入 字符串常量池（String Constant Pool） 中，并返回常量池中的引用

### 作用

- 如果字符串常量池中已经存在一个相同内容的字符串，`intern()` 返回池中的引用
- 如果不存在，则将该字符串加入池，并返回池中的引用
- 节省内存（多个相同内容的字符串只保留一份）
- 需要对比字符串时使用 `==`，可以提高效率（因为常量池中的字符串地址是唯一的）
- 但要注意，频繁使用 `intern()` 可能会增加常量池压力，导致性能下降

```java
String s1 = new String("hello");
String s2 = "hello";
String s3 = s1.intern();

System.out.println(s1 == s2); // false （s1 在堆，s2 在常量池）
System.out.println(s3 == s2); // true （s3 被 intern 后指向常量池）
```

### 解释：

- `new String("abc")` 创建了两个对象（一个堆对象，一个常量池对象）
- `s.intern()` 返回常量池中的 `"abc"`

## 10. compareTo 比较

```java
System.out.println("abc".compareTo("abc")); // 0
System.out.println("abc".compareTo("abd")); // -1
System.out.println("abd".compareTo("abc")); // 1
```
