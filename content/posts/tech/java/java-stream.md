---
title: "Java 流"
date: 2025-05-08T09:16:02+08:00
lastmod: 2025-05-08T09:16:02+08:00
author: ["熊大如如"]
tags: # 标签
  - "java"
description: ""
weight:
slug: ""
summary: "Java 流简介"
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

Java 中的“流”主要指的是 Java Stream API（自 Java 8 引入），它为处理集合数据提供了一种高效、可读、函数式的方式

## 什么是 Stream

- 是对集合对象功能的增强
- 不是数据结构，不存储数据，只是数据的传输渠道（数据流水线）
- 操作是惰性执行的，只有在需要结果时才执行（如 `collect()`）

## 三大操作步骤

Java 流操作可分为三种类型，创建流、中间操作、终结操作

## 创建流

### 简单的示例

```java
import java.util.stream.*;

public class StreamOf {
    public static void main(String[] args) {

        Stream.of("It's ", "a ", "wonderful ", "day ", "for ", "pie!").forEach(System.out::print);

        System.out.println();

        Stream.of(3.14159, 2.718, 1.618).forEach(System.out::println);
    }
}
```

使用`Stream.of`可以轻松将一组条目变成一个流

```java
import java.util.*;

public class CollectionToStream {
    public static void main(String[] args) {

        Set<String> w = new HashSet<>(Arrays.asList(
                "It's a wonderful day for pie!".split(" ")
        ));
        w.stream().map(x -> x + " ")
                .forEach(System.out::print);
        System.out.println();

        Map<String, Double> m = new HashMap<>();
        m.put("pi", 3.14159);
        m.put("e", 2.718);
        m.put("phi", 1.618);
        m.entrySet().stream().map(e -> e.getKey() + ":" + e.getValue()).forEach(System.out::println);
    }
}
```

为了从 Map 集合生成一个流，我们首先调用 entrySet 方法来生成一个对象流，List 和 Set 类使用 stream 方法就可以了

### 随机数流

```java
Random random = new Random();

// 生成 IntStream（整数流）
IntStream intStream = random.ints();              // 无限整数流
IntStream intStreamLimited = random.ints(10);     // 限定长度
IntStream intBounded = random.ints(10, 1, 100);    // 10个 [1, 100) 的随机整数

// 生成 LongStream（长整型流）
LongStream longStream = random.longs(5, 10, 100);

// 生成 DoubleStream（浮点型流）
DoubleStream doubleStream = random.doubles(5, 0.0, 1.0);
```

### 文件流

使用 Files.lines 方法可以创建文件流

```java
Path path = Paths.get("test.txt");
try (Stream<String> lines = Files.lines(path)) {
    lines.forEach(System.out::println);
}
```

### 无限流

`Stream.generate()` 是 Java 8 引入的 Stream API 中用来 生成无限流（infinite stream） 的方法之一，非常适合构造随机数、常量流、自定义对象等。但是必须搭配 `.limit(n)` 使用，否则会无限执行下去。

```java
import java.util.stream.Stream;

public class GenerateStream {
    public static void main(String[] args) {
        Stream<String> helloStream = Stream.generate(() -> "hello");
        helloStream.limit(10).forEach(System.out::println);
        // 输出 10 次 hello
    }
}
```

### iterate

```java
public static void main(String[] args) {
    Stream.iterate(0, n -> n + 2)
            .limit(10)
            .forEach(System.out::println);
}
```

> 第一个参数是种子 n = 0，然后每次用 `n -> n + 2` 计算下一个。

### 流生成器

`Stream.builder()` 是 Java 8 中 `Stream` 提供的一种用于动态构建流的方法，适合你事先不知道流中有多少个元素，但想一个一个手动添加的场景

```java
import java.util.stream.Stream;

public class StreamBuilderExample {
    public static void main(String[] args) {
        Stream.Builder<String> builder = Stream.builder();

        builder.add("Java");
        builder.add("Python");
        builder.add("Go");

        Stream<String> stream = builder.build();
        stream.forEach(System.out::println);
    }
}
```

## 中间操作

Java 中 Stream 的中间操作（Intermediate Operations） 是指：在 Stream 处理流程中，返回一个新的 Stream 的操作，这些操作不会立即执行，只有当终端操作（Terminal Operation）调用时，才会触发实际的处理

| 操作                              | 含义                   | 示例                                                            |
| --------------------------------- | ---------------------- | --------------------------------------------------------------- |
| `filter(Predicate)`               | 过滤元素               | `stream.filter(x -> x > 5)`                                     |
| `map(Function)`                   | 元素映射（转换）       | `stream.map(String::toUpperCase)`                               |
| `flatMap(Function)`               | 扁平化嵌套结构         | `stream.flatMap(x -> Arrays.stream(x))`                         |
| `distinct()`                      | 去重                   | `stream.distinct()`                                             |
| `sorted()` / `sorted(Comparator)` | 排序                   | `stream.sorted()` or `stream.sorted(Comparator.reverseOrder())` |
| `peek(Consumer)`                  | 查看每个元素（调试用） | `stream.peek(System.out::println)`                              |
| `limit(n)`                        | 截取前 n 个元素        | `stream.limit(10)`                                              |
| `skip(n)`                         | 跳过前 n 个元素        | `stream.skip(5)`                                                |

### 示例

```java
import java.util.stream.Stream;

public class IntermediateExample {
    public static void main(String[] args) {
        Stream.of("apple", "apple", "banana", "apricot", "cherry")
                .filter(s -> s.startsWith("a"))          // 只保留以 a 开头的
                .map(String::toUpperCase)                // 转成大写
                .sorted()                                // 排序
//                .peek(System.out::println)               // 中间观察
                .distinct()                              // 过滤
                .limit(10)                        // 最多两个
                .forEach(System.out::println);           // 终止操作
    }
}
```

## Optional 类型

Optional 类型解决了流中没有数据，产生空指针异常的问题。

### 常用方法

| 操作          | 返回类型      | 含义                               |
| ------------- | ------------- | ---------------------------------- |
| `findFirst()` | `Optional<T>` | 获取第一个元素，可能为空           |
| `findAny()`   | `Optional<T>` | 获取任意元素，适用于并行流         |
| `max()`       | `Optional<T>` | 获取最大元素，可能集合为空         |
| `min()`       | `Optional<T>` | 获取最小元素                       |
| `reduce()`    | `Optional<T>` | 合并操作，若无初始值则结果可能为空 |

### 判断是否空流

```java
import java.util.Optional;
import java.util.stream.Stream;

public class OptionalBasics {
    static void test(Optional<String> optString) {
        if (optString.isPresent())
            System.out.println(optString.get());
        else
            System.out.println("Nothing inside!");
    }

    public static void main(String[] args) {
        test(Stream.of("Epithets").findFirst());
        test(Stream.<String> empty().findFirst());
    }
}
```

### 便捷函数

在 Java 的 `Optional` 中，有一组常用的便捷函数（Convenience Methods），用于优雅、安全地处理可能为空的值。它们是函数式编程风格的体现，广泛用于 Stream、数据库结果处理等场景

#### 构造函数

| 方法                         | 含义                                           |
| ---------------------------- | ---------------------------------------------- |
| `Optional.of(value)`         | 创建非空 Optional，值不能为 null，否则抛出异常 |
| `Optional.ofNullable(value)` | 创建 Optional，值可以为 null                   |
| `Optional.empty()`           | 创建空的 Optional 实例                         |

```java
Optional<String> opt1 = Optional.of("hello");
Optional<String> opt2 = Optional.ofNullable(null);
Optional<String> opt3 = Optional.empty();
```

#### 判断与获取值

| 方法                     | 说明                                       |
| ------------------------ | ------------------------------------------ |
| `isPresent()`            | 判断是否有值                               |
| `isEmpty()` _(Java 11+)_ | 判断是否为空                               |
| `get()`                  | 获取值（不推荐直接使用，值为空时会抛异常） |
| `orElse(T)`              | 如果为空返回默认值                         |
| `orElseGet(Supplier)`    | 如果为空返回 Lambda 提供的值（延迟执行）   |
| `orElseThrow()`          | 若为空则抛 `NoSuchElementException`        |
| `orElseThrow(Supplier)`  | 若为空抛出自定义异常                       |

```java
public class OptionalExamples {
    public static void main(String[] args) {

        // 1. isPresent()：判断是否有值
        Optional<String> opt1 = Optional.of("hello");
        if (opt1.isPresent()) {
            System.out.println("有值: " + opt1.get()); // 输出：有值: hello
        }

        // 2. isEmpty()（Java 11+）
        Optional<String> opt2 = Optional.empty();
        if (opt2.isEmpty()) {
            System.out.println("值是空的"); // 输出：值是空的
        }

        // 3. get()（不推荐直接使用，除非你确定值一定存在）
        Optional<String> opt3 = Optional.of("world");
        String value3 = opt3.get(); // 安全获取
        System.out.println("使用 get(): " + value3); // 输出：world

        // 4. orElse(T)：为空返回默认值
        Object name4 = Optional.ofNullable(null).orElse("默认值");
        System.out.println("orElse 返回: " + name4); // 输出：默认值

        // 5. orElseGet(Supplier)：为空返回 Lambda 提供的值（延迟执行）
        Object name5 = Optional.ofNullable(null)
                .orElseGet(() -> {
                    System.out.println("调用了 orElseGet");
                    return "延迟默认值";
                });
        System.out.println("orElseGet 返回: " + name5); // 输出：延迟默认值

        // 6. orElseThrow()：为空抛出 NoSuchElementException
        Optional<String> opt6 = Optional.of("存在值");
        String value6 = opt6.orElseThrow(); // 正常返回
        System.out.println("orElseThrow 返回: " + value6);

        Optional<String> emptyOpt = Optional.empty();
//        String error6 = emptyOpt.orElseThrow(); // 抛 NoSuchElementException
//        System.out.println(error6);

        // 7. orElseThrow(Supplier)：为空时抛出自定义异常
        Optional<String> opt7 = Optional.ofNullable(null);
        try {
            String value7 = opt7.orElseThrow(() -> new IllegalArgumentException("值不存在"));
            System.out.println("value7 = " + value7);
        } catch (IllegalArgumentException e) {
            System.out.println("捕获异常: " + e.getMessage()); // 输出：值不存在
        }
    }
}
```

#### 变换与过滤

| 方法                | 说明                                         |
| ------------------- | -------------------------------------------- |
| `map(Function)`     | 映射值，如果存在就应用函数                   |
| `flatMap(Function)` | 和 map 类似，但函数返回 Optional（避免嵌套） |
| `filter(Predicate)` | 如果值存在且满足条件则保留，否则变成 empty   |

```java
public class OptionalDemo {

    public static void main(String[] args) {

        Optional<String> name = Optional.of("bear");

        // 1. map：将字符串映射为大写
        Optional<String> upperName = name.map(String::toUpperCase);
        System.out.println("map 结果: " + upperName.orElse("空")); // 输出：BEAR

        // 2. flatMap：获取字符串长度（返回 Optional）
        Optional<Integer> nameLength = name.flatMap(n -> Optional.of(n.length()));
        System.out.println("flatMap 结果: " + nameLength.orElse(-1)); // 输出：4

        // 3. filter：只保留长度 > 3 的字符串
        Optional<String> longName = name.filter(n -> n.length() > 3);
        System.out.println("filter（>3） 结果: " + longName.orElse("不满足条件")); // 输出：bear

        // 4. filter：只保留长度 < 3 的字符串
        Optional<String> shortName = name.filter(n -> n.length() < 3);
        System.out.println("filter（<3） 结果: " + shortName.orElse("不满足条件")); // 输出：不满足条件
    }
}
```

## 终结操作

Java 流（Stream）中的终结操作（Terminal Operation） 是触发流中处理逻辑的最后一步，它将流的中间操作“收尾”，并返回一个最终结果或副作用（比如打印、求和等）。执行终结操作后，流就不能再使用了

### 常见

| 方法名                 | 说明                         |
| ---------------------- | ---------------------------- |
| `forEach(Consumer)`    | 遍历每个元素（有副作用）     |
| `toArray()`            | 转为数组                     |
| `reduce(...)`          | 归约操作（如求和、拼接等）   |
| `collect(...)`         | 收集结果（转集合、字符串等） |
| `count()`              | 统计数量                     |
| `min(Comparator)`      | 最小值                       |
| `max(Comparator)`      | 最大值                       |
| `anyMatch(Predicate)`  | 任意匹配                     |
| `allMatch(Predicate)`  | 全部匹配                     |
| `noneMatch(Predicate)` | 全不匹配                     |
| `findFirst()`          | 找第一个元素                 |
| `findAny()`            | 找任意一个（并行流更有效）   |

```java
import java.util.*;
import java.util.stream.*;

public class TerminalOperationsDemo {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

        // 1. forEach
        names.stream().forEach(System.out::println);

        // 2. toArray
        Object[] array = names.stream().toArray();
        System.out.println("数组长度: " + array.length);

        // 3. reduce
        String joined = names.stream().reduce((a, b) -> a + ", " + b).orElse("无内容");
        System.out.println("连接结果: " + joined);

        // 4. collect
        List<String> filtered = names.stream()
                .filter(n -> n.length() > 3)
                .collect(Collectors.toList());
        System.out.println("过滤后: " + filtered);

        // 5. count
        long count = names.stream().filter(n -> n.contains("a")).count();
        System.out.println("包含字母 a 的数量: " + count);

        // 6. min & max
        String minName = names.stream().min(Comparator.comparing(String::length)).orElse("无");
        System.out.println("最短名字: " + minName);

        // 7. anyMatch, allMatch, noneMatch
        boolean any = names.stream().anyMatch(n -> n.startsWith("A"));
        boolean all = names.stream().allMatch(n -> n.length() >= 3);
        boolean none = names.stream().noneMatch(n -> n.equals("Zoe"));
        System.out.println("任意以 A 开头: " + any);
        System.out.println("全部长度 >= 3: " + all);
        System.out.println("没有人叫 Zoe: " + none);

        // 8. findFirst & findAny
        Optional<String> first = names.stream().findFirst();
        first.ifPresent(n -> System.out.println("第一个: " + n));
    }
}
```
