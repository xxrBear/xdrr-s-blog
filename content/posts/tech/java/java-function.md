---
title: "Java 函数式编程"
date: 2025-05-06T22:23:27+08:00
lastmod: 2025-05-06T22:23:27+08:00
author: ["熊大如如"]
tags: # 标签
  - "java"
description: ""
weight:
slug: ""
summary: "Java 函数式编程总结"
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

## 函数式编程的意义

函数式编程理念强调函数纯粹性和不可变性，这有助于写出更稳定、更易测试的代码，尤其在并发环境下减少 bug

## lambda 表达式

```java
import java.util.function.Function;

public class Strategize {
    Function<String, String> getString = h -> h + "hello";

    public static void main(String[] args) {
        Strategize s = new Strategize();
        System.out.println(s.getString.apply("Hi, "));
    }
}
```

与匿名内部类相比，lambda 表达式通常可读性更好。

## 方法引用

Java 方法引用（Method Reference）是 Lambda 表达式的一种简洁写法，用于直接引用已有的方法，使代码更加清晰、简洁

```java
interface Callable {
    void call(String s);
}

class Describe {
    void show(String msg) {
        System.out.println(msg);
    }
}

public class MethodReferences {
    static void hello(String name) {
        System.out.println("Hello " + name);
    }

    static class Description {
        String about;

        Description(String desc) {
            about = desc;
        }

        void help(String msg) {
            System.out.println(about + " " + msg);
        }
    }

    static class Helper {
        static void assist(String msg) {
            System.out.println(msg);
        }
    }

    public static void main(String[] args) {
        Describe d = new Describe();
        Callable c = d::show;
        c.call("call()");

        c = MethodReferences::hello;
        c.call("bob");

        c = new Description("valuable")::help;
        c.call("information");

        c = Helper::assist;
        c.call("Help");
    }
}
```

## 函数式接口

### Rubbable 接口

`Runnable` 是 Java 中最经典的函数式接口之一，用于表示一个“无参、无返回值”的任务，通常用于线程执行

```java
public class Work {
    public static void doWork() {
        System.out.println("工作进行中！");
    }

    public static void main(String[] args) {
        Runnable task = Work::doWork;
        new Thread(task).start();
    }
}
```

### 单一抽象方法注解

接口中只能有一个抽象方法。这样的接口被称为函数式接口，是 Lambda 表达式可以使用的基础。

```java
@FunctionalInterface
interface AnotherBadInterface {
    void doA();

    void doB();
}
```

`@FunctionalInterface`注解，当接口定义的方法非单一，编译报错

## 闭包

Java 中的闭包是通过 Lambda 表达式实现的，它允许函数访问其定义时的作用域变量，但要求这些变量是不可变的（effectively final），以保证线程安全与稳定性

```java
public class ClosureExample {
    public static void main(String[] args) {
        int x = 10;

        Runnable r = () -> {
            System.out.println("x = " + x);  // 捕获了外部变量 x
        };

        r.run(); // 输出 x = 10
    }
}
```

### 变量默认是 final

```java
int x = 10;
x = 20; // ❌ 编译错误
Runnable r = () -> System.out.println(x); // 会报错
```

### 可以修改捕获对象的字段，但不能重新赋值变量

```java
class Box {
    int value = 10;
}

public class ClosureDemo {
    public static void main(String[] args) {
        Box box = new Box();

        Runnable r = () -> {
            box.value = 99; // ✅ 允许修改对象字段
        };

        r.run();
        System.out.println(box.value); // 输出 99

        // box = new Box(); // ❌ 不允许重新赋值
    }
}
```

### Java 闭包的工作原理（底层理解）

Java 编译器会将 Lambda 表达式 编译成一个匿名内部类 或 方法句柄（在 JVM 层面优化），并将捕获的变量以常量或构造函数参数的形式传入

所以你看到的是“捕获变量”，实际上 Lambda 拿到的是 复制进去的值或引用，但不允许修改其指向

## 函数组合

在 Java 中，函数组合（Function Composition） 是函数式编程的一个强大能力，允许你将多个函数合并成一个新函数，以更简洁优雅的方式处理数据

### 多个方法

| 组合方式             | 含义                                         |
| -------------------- | -------------------------------------------- |
| `f.andThen(g)`       | 先执行 `f`，再执行 `g`（即 `g(f(x)`)）       |
| `f.compose(g)`       | 先执行 `g`，再执行 `f`（即 `f(g(x)`)）       |
| `predicate.and()`    | 逻辑与，组合两个条件为：条件 1 **且** 条件 2 |
| `predicate.or()`     | 逻辑或，组合两个条件为：条件 1 **或** 条件 2 |
| `predicate.negate()` | 条件取反，等价于 `!predicate.test(x)`        |

示例 1

```java
import java.util.function.Function;

public class FunctionComposeDemo {
    public static void main(String[] args) {
        Function<Integer, Integer> f = x -> x + 2;
        Function<Integer, Integer> g = x -> x * 3;

        // h(x) = g(f(x)) = (x + 2) * 3
        Function<Integer, Integer> h = f.andThen(g);

        // k(x) = f(g(x)) = (x * 3) + 2
        Function<Integer, Integer> k = f.compose(g);

        System.out.println("h(4) = " + h.apply(4)); // (4 + 2) * 3 = 18
        System.out.println("k(4) = " + k.apply(4)); // (4 * 3) + 2 = 14
    }
}
```

示例 2

```java
Predicate<String> notEmpty = s -> !s.isEmpty();
Predicate<String> startsWithA = s -> s.startsWith("A");

Predicate<String> complex = notEmpty.and(startsWithA);

System.out.println(complex.test("Apple")); // true
System.out.println(complex.test(""));      // false
```
