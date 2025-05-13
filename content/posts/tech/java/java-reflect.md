---
title: "Java 反射"
date: 2025-05-13T21:12:47+08:00
lastmod: 2025-05-13T21:12:47+08:00
author: ["熊大如如"]
tags: # 标签
  - "java"
description: ""
weight:
slug: ""
summary: "Java 反射"
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
## 什么是反射
反射是 Java 提供的一种强大机制，允许在运行时动态获取类的信息、动态创建对象、调用方法、访问字段等

## 反射的核心类
### Class 类
#### 简介
Java 一切类在 JVM 中都是 `Class` 类的实例

作用：获取类的所有信息（字段、方法、构造器、注解、父类、接口等）

#### 获取 Class 对象
```java
Class<?> clazz = Class.forName("com.example.User");
```

#### 常用方法
```java
// 获取类的完全限定名（包含包名）
clazz.getName(); 
// 例：com.example.User

// 获取类的简单名（不包含包名）
clazz.getSimpleName(); 
// 例：User

// 获取类的包名（Java 9 以后推荐用 getPackageName）
clazz.getPackageName(); 
// 例：com.example

// 获取类中所有 public 的成员变量（包括继承自父类的 public 成员）
clazz.getFields(); 

// 获取类中声明的所有成员变量（不论权限，private、protected、public、default），但不包括继承自父类的
clazz.getDeclaredFields();

// 获取类中所有 public 的方法（包括继承自父类的 public 方法）
clazz.getMethods(); 

// 获取类中声明的所有方法（不论权限，private、protected、public、default），但不包括继承的
clazz.getDeclaredMethods(); 

// 获取类中所有 public 的构造方法
clazz.getConstructors(); 

// 获取类的直接父类（如果没有则返回 null，像 Object）
clazz.getSuperclass(); 

// 获取类实现的所有接口（不包括父类实现的接口）
clazz.getInterfaces(); 

```

### Constructor 类
#### 简介
+ 代表类的构造方法
+ 用于获取、操作类的构造器，动态创建实例
+ 即使构造器是私有的，也可以通过反射访问

#### 常用方法
| 方法 | 说明 |
| --- | --- |
| `getConstructor(参数类型...)` | 获取 `public`构造器 |
| `getDeclaredConstructor(参数类型...)` | 获取指定参数的构造器（包括 `private`<br/>） |
| `getConstructors()` | 获取所有 `public`构造器 |
| `getDeclaredConstructors()` | 获取所有构造器（包括 `private`） |


#### 示例
```java
class User {
    private String name;
    private int age;

    // 无参构造
    public User() {
        this.name = "默认名";
    }

    // 有参构造
    private User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void show() {
        System.out.println("User{name='" + name + "', age=" + age + "}");
    }
}
```

#### 获取 public 无参构造器
```java
Class<User> clazz = User.class;
Constructor<User> constructor = clazz.getConstructor();
User user = constructor.newInstance();
user.show();
```

#### 获取 private 有参构造器
```java
Constructor<User> privateConstructor = clazz.getDeclaredConstructor(String.class, int.class);
privateConstructor.setAccessible(true);
User user2 = privateConstructor.newInstance("张三", 20);
user2.show();
```

#### 获取所有构造器
```java
Constructor<?>[] constructors = clazz.getDeclaredConstructors();
for (Constructor<?> cons : constructors) {
    System.out.println(cons);
}
```

### Field 类
#### 简介
+ 用于表示并操作类的成员变量（包括 `public`、`private`、`protected`、`static` 等）
+ 支持对字段进行 动态获取、读取、修改、访问控制破坏

#### 获取 Field 对象
| 方法 | 说明 |
| --- | --- |
| `getField(String name)` | 获取 `public`字段 |
| `getDeclaredField(String name)` | 获取任意字段（包括 `private`、`protected`） |
| `getFields()` | 获取所有 `public`字段 |
| `getDeclaredFields()` | 获取所有字段（包括 `private`、`protected`） |


#### 常用方法
| 方法 | 说明 |
| --- | --- |
| `get(Object obj)` | 获取字段值 |
| `set(Object obj, Object value)` | 设置字段值 |
| `getType()` | 获取字段类型 |
| `getModifiers()` | 获取字段修饰符 |
| `setAccessible(true)` | 破坏封装，允许访问 `private`字段 |
| `getName()` | 获取字段名 |
| `isAnnotationPresent()` | 判断字段上是否有某个注解 |


#### 示例
```java
class User {
    public String username;
    private int age;

    @Deprecated
    private String secret = "秘密";
}
```

获取并操作`public`字段

```java
Class<User> clazz = User.class;
Field field = clazz.getField("username");

User user = new User();
field.set(user, "张三");
String value = (String) field.get(user);

System.out.println("用户名: " + value);
```

获取并操作`private`字段

```java
Field privateField = clazz.getDeclaredField("age");
privateField.setAccessible(true);  // 破坏封装
privateField.set(user, 20);
int age = (int) privateField.get(user);

System.out.println("年龄: " + age);
```

获取字段信息

```java
Field field = clazz.getDeclaredField("secret");
System.out.println("字段名: " + field.getName());
System.out.println("字段类型: " + field.getType());
System.out.println("修饰符: " + Modifier.toString(field.getModifiers()));
```

判断注解

```java
if (field.isAnnotationPresent(Deprecated.class)) {
    System.out.println("该字段已被标记为 Deprecated");
}
```

#### 操作静态字段
```java
Field staticField = MyClass.class.getDeclaredField("STATIC_FIELD");
staticField.setAccessible(true);
staticField.set(null, "新值");  // 静态字段 obj 传 null
```

### Method 类
#### 作用
`Method` 是 `java.lang.reflect` 包中的类

表示类中的一个成员方法（实例方法、静态方法）

可以通过 `Method` 类动态调用对象的方法，即使是私有方法

#### 获取 Method 对象的方式
| 方法 | 说明 |
| --- | --- |
| `getMethod(String name, Class<?>... parameterTypes)` | 获取 `public`方法（包括父类方法） |
| `getDeclaredMethod(String name, Class<?>... parameterTypes)` | 获取当前类声明的方法（包括 `private`） |
| `getMethods()` | 获取所有 `public`方法（包括继承自父类） |
| `getDeclaredMethods()` | 获取当前类声明的所有方法（包括 `private`） |


#### 常用方法
| 方法 | 说明 |
| --- | --- |
| `invoke(Object obj, Object... args)` | 调用该方法 |
| `getName()` | 获取方法名 |
| `getReturnType()` | 获取返回类型 |
| `getParameterTypes()` | 获取参数类型数组 |
| `getModifiers()` | 获取方法修饰符 |
| `setAccessible(true)` | 破坏封装，允许访问 `private`<br/> 方法 |
| `isAnnotationPresent()` | 判断方法上是否存在指定注解 |


#### 示例
```java
class User {
    public void sayHello(String name) {
        System.out.println("Hello, " + name);
    }

    private int add(int a, int b) {
        return a + b;
    }
}
```

获取并调用 `public` 方法

```java
Class<User> clazz = User.class;
Method method = clazz.getMethod("sayHello", String.class);

User user = new User();
method.invoke(user, "张三");
```

获取并调用 `private` 方法

```java
Method privateMethod = clazz.getDeclaredMethod("add", int.class, int.class);
privateMethod.setAccessible(true);
int result = (int) privateMethod.invoke(user, 5, 3);

System.out.println("加法结果: " + result);
```

获取方法信息

```java
Method method = clazz.getDeclaredMethod("add", int.class, int.class);
System.out.println("方法名: " + method.getName());
System.out.println("返回类型: " + method.getReturnType());
System.out.println("参数类型: " + Arrays.toString(method.getParameterTypes()));
System.out.println("修饰符: " + Modifier.toString(method.getModifiers()));
```

判断注解

```java
if (method.isAnnotationPresent(Deprecated.class)) {
    System.out.println("该方法已过时");
}
```

### Modifier 类
#### 作用
位于 `java.lang.reflect` 包。

用于解析类、方法、字段、构造器等的访问修饰符（例如 `public`、`private`、`static`、`final`、`abstract` 等）。

通过该类可以将修饰符的 int 值转换为可读的字符串，并进行修饰符检查。

#### 常见修饰符
| 常量 | 十进制值 | 说明 |
| --- | --- | --- |
| `Modifier.PUBLIC` | 1 | `public` |
| `Modifier.PRIVATE` | 2 | `private` |
| `Modifier.PROTECTED` | 4 | `protected` |
| `Modifier.STATIC` | 8 | `static` |
| `Modifier.FINAL` | 16 | `final` |
| `Modifier.SYNCHRONIZED` | 32 | `synchronized` |
| `Modifier.VOLATILE` | 64 | `volatile` |
| `Modifier.TRANSIENT` | 128 | `transient` |
| `Modifier.ABSTRACT` | 1024 | `abstract` |
| `Modifier.INTERFACE` | 512 | `interface` |
| `Modifier.NATIVE` | 256 | `native` |


#### 常用方法
| 方法 | 说明 |
| --- | --- |
| `toString(int mod)` | 将修饰符 int 值转为字符串 |
| `isPublic(int mod)` | 判断是否是 `public` |
| `isPrivate(int mod)` | 判断是否是 `private` |
| `isProtected(int mod)` | 判断是否是 `protected` |
| `isStatic(int mod)` | 判断是否是 `static` |
| `isFinal(int mod)` | 判断是否是 `final` |
| `isAbstract(int mod)` | 判断是否是 `abstract` |
| `isSynchronized(int mod)` | 判断是否是 `synchronized` |
| `isNative(int mod)` | 判断是否是 `native` |
| `isVolatile(int mod)` | 判断是否是 `volatile` |
| `isTransient(int mod)` | 判断是否是 `transient` |
| `isInterface(int mod)` | 判断是否是接口 |


#### 示例
```java
public class User {
    private static final String NAME = "张三";

    public void sayHello() {}
}
```

示例代码

```java
Class<User> clazz = User.class;

// 获取字段修饰符
Field field = clazz.getDeclaredField("NAME");
int fieldModifiers = field.getModifiers();
System.out.println("字段修饰符: " + Modifier.toString(fieldModifiers));
System.out.println("是否是 static: " + Modifier.isStatic(fieldModifiers));
System.out.println("是否是 final: " + Modifier.isFinal(fieldModifiers));

// 获取方法修饰符
Method method = clazz.getMethod("sayHello");
int methodModifiers = method.getModifiers();
System.out.println("方法修饰符: " + Modifier.toString(methodModifiers));
System.out.println("是否是 public: " + Modifier.isPublic(methodModifiers));
```

输出

```plain
字段修饰符: private static final
是否是 static: true
是否是 final: true
方法修饰符: public
是否是 public: true
```

### Array 类
#### 作用
+ 提供在运行时动态创建、访问、修改数组的方法
+ 支持任意类型、任意维度的数组操作
+ 解决反射场景下无法直接操作数组的问题

#### 常用方法
| 方法 | 说明 |
| --- | --- |
| `newInstance(Class<?> componentType, int length)` | 创建一维数组 |
| `newInstance(Class<?> componentType, int... dimensions)` | 创建多维数组 |
| `get(Object array, int index)` | 获取数组中的元素 |
| `set(Object array, int index, Object value)` | 设置数组中的元素 |
| `getLength(Object array)` | 获取数组长度 |


#### 使用示例
动态创建一维数组

```java
int[] intArray = (int[]) Array.newInstance(int.class, 5);

Array.set(intArray, 0, 10);
Array.set(intArray, 1, 20);

System.out.println(Array.get(intArray, 0));  // 10
System.out.println(Array.get(intArray, 1));  // 20
System.out.println("数组长度: " + Array.getLength(intArray));  // 5
```

动态创建多维数组

```java
int[][] matrix = (int[][]) Array.newInstance(int.class, 3, 3);

Array.set(Array.get(matrix, 0), 0, 100);
System.out.println(((int[]) Array.get(matrix, 0))[0]);  // 100
```

获取数组信息

```java
String[] strs = new String[]{"A", "B", "C"};
System.out.println("数组类型: " + strs.getClass().getName());
System.out.println("数组长度: " + Array.getLength(strs));
```

## 反射的性能影响
+ 反射比直接调用慢（通常慢 10-20 倍），但合理使用影响可以接受
+ 使用缓存可以降低反射带来的性能损耗
+ `MethodHandle`、`LambdaMetafactory` 可替代部分反射，性能更优

## 反射的底层实现机制
+ 反射通过 JVM 提供的元数据实现（类的元信息、方法表、字段表）
+ 访问字段、方法底层通过 `Unsafe`、JNI 实现直接操作内存
+ `Class` 类是所有类的元数据载体

## 反射实战
### 框架实现 IOC 容器
+ 反射用于根据配置或注解，动态创建对象并注入到容器

示例：

```java
public class BeanFactory {
    public static Object createBean(String className) throws Exception {
        Class<?> clazz = Class.forName(className);
        return clazz.getDeclaredConstructor().newInstance();
    }

    public static void main(String[] args) throws Exception {
        Object obj = createBean("java.util.ArrayList");
        System.out.println(obj.getClass().getName()); // java.util.ArrayList
    }
}
```

### 实现 ORM 框架的实体与数据库映射
+ 反射读取实体类字段，自动生成 SQL 语句，自动赋值

示例

```java
public static void printInsertSQL(Object obj) throws Exception {
    Class<?> clazz = obj.getClass();
    StringBuilder sb = new StringBuilder("INSERT INTO ");
    sb.append(clazz.getSimpleName()).append(" VALUES(");

    Field[] fields = clazz.getDeclaredFields();
    for (Field field : fields) {
        field.setAccessible(true);
        sb.append("'").append(field.get(obj)).append("', ");
    }
    sb.setLength(sb.length() - 2); // 移除最后的逗号
    sb.append(")");
    System.out.println(sb);
}

class User {
    private String name = "张三";
    private int age = 20;
}

printInsertSQL(new User());
// INSERT INTO User VALUES('张三', '20')
```

### 通用 Bean 属性拷贝工具
+ 反射实现对象属性值的动态拷贝

示例

```java
public static void copyProperties(Object source, Object target) throws Exception {
    Class<?> clazz = source.getClass();
    for (Field field : clazz.getDeclaredFields()) {
        field.setAccessible(true);
        Object value = field.get(source);
        field.set(target, value);
    }
}

class Person {
    private String name = "李四";
    private int age = 30;
}

Person p1 = new Person();
Person p2 = new Person();
copyProperties(p1, p2);
```

### 动态调用插件类（插件机制、热加载）
+ 反射动态加载指定类并调用其方法，不需要预编译绑定

示例：

```java
public static void executePlugin(String className, String methodName) throws Exception {
    Class<?> clazz = Class.forName(className);
    Object obj = clazz.getDeclaredConstructor().newInstance();
    Method method = clazz.getDeclaredMethod(methodName);
    method.invoke(obj);
}

class MyPlugin {
    public void run() {
        System.out.println("插件启动");
    }
}

executePlugin("MyPlugin", "run");
// 插件启动
```

### JDK 动态代理（AOP、事务、日志）
+ 反射是动态代理的基础，动态创建代理对象

示例（简化版）：

```java
import java.lang.reflect.*;

interface Service {
    void doSomething();
}

class ServiceImpl implements Service {
    public void doSomething() {
        System.out.println("执行业务逻辑");
    }
}

Service proxy = (Service) Proxy.newProxyInstance(
    ServiceImpl.class.getClassLoader(),
    new Class[]{Service.class},
    (proxyObj, method, args1) -> {
        System.out.println("开始事务");
        Object result = method.invoke(new ServiceImpl(), args1);
        System.out.println("提交事务");
        return result;
    }
);

proxy.doSomething();
// 开始事务
// 执行业务逻辑
// 提交事务
```

### 实现通用 JSON 序列化工具
+ 反射获取字段并转成 JSON

示例：

```java
public static String toJson(Object obj) throws Exception {
    Class<?> clazz = obj.getClass();
    StringBuilder sb = new StringBuilder("{");
    for (Field field : clazz.getDeclaredFields()) {
        field.setAccessible(true);
        sb.append("\"").append(field.getName()).append("\":\"")
          .append(field.get(obj)).append("\",");
    }
    sb.setLength(sb.length() - 1);
    sb.append("}");
    return sb.toString();
}

class Book {
    private String title = "Java核心技术";
    private double price = 99.9;
}

System.out.println(toJson(new Book()));
// {"title":"Java核心技术","price":"99.9"}
```

### 反射操作数组（不确定类型的数组操作）
+ 例如构建任意类型、任意长度的数组

示例：

```java
Object array = Array.newInstance(String.class, 5);
Array.set(array, 0, "Hello");
System.out.println(Array.get(array, 0)); // Hello
```

