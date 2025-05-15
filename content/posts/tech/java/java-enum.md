---
title: "Java 枚举"
date: 2025-05-15T16:09:14+08:00
lastmod: 2025-05-15T16:09:14+08:00
author: ["熊大如如"]
tags: # 标签
  - "java"
description: ""
weight:
slug: ""
summary: "java 枚举"
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

## 一、Java 枚举基础语法

```java
public enum Color {
    RED, GREEN, BLUE
}
```

### 特点

- `enum` 声明的类，自动继承自 `java.lang.Enum`
- 不能显式继承其他类（因为 Java 只允许单继承，而 `enum` 已继承自 `Enum`）
- 枚举构造器只能是 `private`
- 枚举常量 `RED, GREEN, BLUE`每个常量都是 `Color` 类型的实例，系统会自动帮你生成：

```java
public static final Color RED = new Color("RED", 0);
public static final Color GREEN = new Color("GREEN", 1);
public static final Color BLUE = new Color("BLUE", 2);
// 这些实例在类加载时就会创建（饿汉式单例）
```

## 二、枚举高级用法

### 枚举带构造方法和属性

```java
public enum Color {
    RED("红色"), GREEN("绿色"), BLUE("蓝色");

    private String desc;

    Color(String desc) {
        this.desc = desc;
    }

    public String getDesc() {
        return desc;
    }
}
```

### 枚举实现接口

```java
interface Printable {
    void print();
}

public enum Status implements Printable {
    OK {
        public void print() { System.out.println("OK"); }
    },
    FAIL {
        public void print() { System.out.println("FAIL"); }
    }
}
```

### 枚举方法重写（每个枚举单独实现方法）

```java
public enum Operation {
    PLUS {
        public int apply(int a, int b) { return a + b; }
    },
    MINUS {
        public int apply(int a, int b) { return a - b; }
    };

    public abstract int apply(int a, int b);
}
```

### 枚举与 switch 配合

```java
Color c = Color.RED;
switch (c) {
    case RED:
        System.out.println("红色");
        break;
}
```

### 枚举遍历

```java
for (Color color : Color.values()) {
    System.out.println(color.name() + " : " + color.ordinal());
}
```

## 三、枚举底层机制

| 特点           | 说明                                                    |
| -------------- | ------------------------------------------------------- |
| 继承自 Enum    | 不能继承其他类，只能实现接口                            |
| 构造器私有     | 防止外部 new，枚举实例在类加载时就创建好了              |
| values 方法    | 编译器自动添加，返回枚举数组                            |
| name() 方法    | 返回定义的名称                                          |
| ordinal() 方法 | 返回定义时的位置（从 0 开始，不推荐持久化依赖 ordinal） |

> 本质上是一个 `final class`，带有静态常量和静态初始化块

## 四、枚举在业务中的高级应用

### 替代常量类（推荐）

```java
public enum HttpStatus {
    OK(200), NOT_FOUND(404);
    ...
}
```

### 枚举工厂模式

```java
public enum ShapeType {
    CIRCLE(() -> new Circle()),
    SQUARE(() -> new Square());

    private Supplier<Shape> factory;
    ShapeType(Supplier<Shape> factory) { this.factory = factory; }
    public Shape create() { return factory.get(); }
}
```

### 配合 Map 使用（高性能替代 if-else/switch）

```java
public enum Command {
    LOGIN, LOGOUT;
    private static final Map<String, Command> map = new HashMap<>();
    static {
        for (Command c : Command.values()) {
            map.put(c.name(), c);
        }
    }
    public static Command from(String name) {
        return map.get(name);
    }
}
```

## 五、常见坑与注意事项

| 坑点                     | 说明                                                                |
| ------------------------ | ------------------------------------------------------------------- |
| 枚举不可继承其他类       | 因为自动继承了 `Enum`                                               |
| 枚举实例是单例（饿汉式） | JVM 加载类时即初始化所有实例                                        |
| 不要持久化 ordinal       | 一旦枚举顺序变化，持久化数据就错乱，推荐使用 `name()` 或自定义 code |
| 枚举反序列化             | 默认基于 `name()`，而非实例本身                                     |
| 枚举不适合复杂状态机     | 复杂状态机建议用类和策略模式                                        |

## 六、进阶：枚举 + 反射 + 工厂

```java
public enum HandlerType {
    A(AHandler.class),
    B(BHandler.class);

    private Class<? extends Handler> clazz;

    HandlerType(Class<? extends Handler> clazz) {
        this.clazz = clazz;
    }

    public Handler getInstance() throws Exception {
        return clazz.getDeclaredConstructor().newInstance();
    }
}
```

## 七、最佳实践

- 推荐枚举代替常量类
- 不要依赖 `ordinal()` 做数据库映射
- 多用属性字段，提升可扩展性
- 配合接口+工厂，让枚举更灵活
- 配合 `Map` 替代复杂的 `if-else`

## 八、高级枚举知识点

### 枚举单例（线程安全的懒加载单例神器）

```java
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        System.out.println("做事");
    }
}
```

- 推荐理由：
  - JVM 保证枚举单例是线程安全的
  - 防止反射、反序列化攻击
- 这是 Joshua Bloch（Effective Java 作者）推荐的单例模式首选

### 反射枚举实例（受限但可以）

```java
Color color = Color.valueOf("RED");
```

> 通过反射无法创建新的枚举实例（JVM 限制）  
> 但可以通过 `Enum.valueOf()` 获取已有实例

### 反序列化安全

- 枚举的反序列化自动保证单例，不会像普通类一样生成新对象
- 不需要重写 `readResolve()`

### 枚举类 Class 特性

```java
Color.class.getSuperclass(); // java.lang.Enum
Color.class.isEnum(); // true
```

### 枚举泛型

```java
public enum ResultCode implements EnumInterface<String> {
    SUCCESS("200"),
    FAIL("500");

    private String code;
    ...
}
```

### 枚举扩展接口、行为（推荐用于策略、工厂）

```java
public interface IBehavior {
    void act();
}

public enum Animal implements IBehavior {
    CAT {
        public void act() { System.out.println("喵喵"); }
    },
    DOG {
        public void act() { System.out.println("汪汪"); }
    }
}
```

## 九、枚举源码内部机制解密

| 方法          | 源码行为说明                      |
| ------------- | --------------------------------- |
| values()      | 编译器自动生成，底层缓存数组      |
| valueOf(name) | 内部遍历 `values()` 找到匹配 name |
| name()        | 返回实例名（不可改）              |
| ordinal()     | 返回定义顺序（不可改，危险）      |

> 本质上 `values()` 和 `valueOf()` 都是编译器插入的静态方法，不在 Enum 父类里

## 十、最佳反模式

| 反模式                 | 推荐做法                        |
| ---------------------- | ------------------------------- |
| 枚举用于存储可变状态   | 枚举应该是不可变的              |
| 使用枚举做过多逻辑判断 | 推荐接口+多态策略模式           |
| 持久化时用 `ordinal`   | 推荐持久化 `name` 或自定义 code |
| 枚举类名与业务不匹配   | 保持语义明确，如 `OrderStatus`  |
