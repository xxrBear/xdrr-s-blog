---
title: "Python 元类"
date: 2025-08-20T15:01:41+08:00
lastmod: 2025-08-20T15:01:41+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "Python 元类的知识点梳理"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "https://cdn.jsdelivr.net/gh/xxrBear/image/icons8-python-500.png"  # 文章的图片
---

## 为什么需要元类
在 Python 里：

+ 实例是由类创建的
+ 类本身也是对象，它是由元类创建的

也就是说：

```python
obj = MyClass()      # 用类创建实例
MyClass = MetaClass()  # 用元类创建类
```

默认情况下，Python 里的类是由内建的 `type` 这个元类创建的  
因此：

```python
class Foo: pass

print(type(Foo))     # <class 'type'>
print(type(int))     # <class 'type'>
```

你自己写的类和内置的类（如 `int`、`dict`）其实都是 `type` 的实例

## 元类的作用
如果把类比作工厂：

+ 类：是生产实例的工厂
+ 元类：是生产工厂的工厂

通过元类，你可以：

1. 拦截类的创建过程，在类生成前修改它；
2. 自动注入属性或方法；
3. 做约束检查，比如强制子类必须实现某些方法；

## 元类的定义与用法
元类就是继承 `type` 的类  
创建类的流程是：

1. 先调用元类的 `__new__` 来 创建类对象；
2. 再调用元类的 `__init__` 来 初始化类对象

### 基本写法：
```python
# 定义一个元类
class MyMeta(type):
    def __new__(mcs, name, bases, attrs):
        print("创建类:", name)
        attrs["added_attr"] = 123
        return super().__new__(mcs, name, bases, attrs)

    def __init__(cls, name, bases, attrs):
        print("初始化类:", name)
        super().__init__(name, bases, attrs)

# 使用元类创建类
class Foo(metaclass=MyMeta):
    pass

print(Foo.added_attr)  # 123
```

执行过程：

1. Python 遇到 `class Foo(...)` 时，并不是直接生成 `Foo`；
2. 它会调用 `MyMeta.__new__`；
3. 在里面你可以修改、增强属性字典；
4. 返回类对象；
5. 然后 `__init__` 会进一步初始化

## 元类的三个关键钩子方法

### __new__(mcs, name, bases, attrs)
+ `mcs`：元类本身（注意是 metaclass class，不是实例）
+ `name`：类名，比如 `"Foo"`
+ `bases`：父类
+ `attrs`：类的属性字典

这是真正创建类对象的地方，你可以修改属性

### __init__(cls, name, bases, attrs)
+ `cls`：刚刚创建的类对象
+ 用来进一步定制类对象

### __call__(cls, *args, **kwargs)
+ 定义当类被调用时的行为，也就是 `Foo()` 的时候
+ 默认行为是：先 `__new__`，再 `__init__`
+ 如果你想控制实例化过程，可以在元类里改 `__call__`

## 元类与 `type` 的关系
直接使用 `type` 也能动态创建类：

```python
# 动态创建类
Foo = type("Foo", (object,), {"bar": 42})
print(Foo)           # <class '__main__.Foo'>
print(Foo().bar)     # 42
```

其实 `class Foo: ...` 只是 `type()` 的语法糖而已  
写：

```python
class Foo:
    bar = 42
```

等价于：

```python
Foo = type("Foo", (object,), {"bar": 42})
```

## 元类应用案例
### 自动注册子类
```python
class Registry(type):
    registry = {}
    def __new__(mcs, name, bases, attrs):
        cls = super().__new__(mcs, name, bases, attrs)
        mcs.registry[name] = cls
        return cls

class Base(metaclass=Registry): pass
class A(Base): pass
class B(Base): pass

print(Registry.registry)
# {'Base': <class '__main__.Base'>,
#  'A': <class '__main__.A'>,
#  'B': <class '__main__.B'>}
```

用于插件系统、自动收集类

### 接口检查
```python
class InterfaceMeta(type):
    def __init__(cls, name, bases, attrs):
        if "do_something" not in attrs:
            raise TypeError(f"{name} 必须实现 do_something 方法")
        super().__init__(name, bases, attrs)

class Base(metaclass=InterfaceMeta):
    def do_something(self): pass

class Good(Base):
    def do_something(self): print("OK")

# class Bad(Base):
#     pass  # 会报错
```

### 单例模式
```python
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class MyClass(metaclass=Singleton):
    pass

a = MyClass()
b = MyClass()
print(a is b)  # True
```

## 元类的注意事项
1. 不要过度使用元类，很多时候用装饰器或类继承即可解决
2. 元类影响可读性，框架级代码常见，但业务代码最好避免
3. 避免多继承冲突，一个类只能有一个最终元类，否则会报错：

```plain
TypeError: metaclass conflict
```

解决方案是：写一个共同的元类，或者继承合并

## 元类与 __init_subclass__
从 Python 3.6 开始，很多元类的简单需求，可以用 `__init_subclass__` 来替代，不必显式写元类：

```python
class Base:
    def __init_subclass__(cls, **kwargs):
        print(f"子类 {cls.__name__} 被创建")
        super().__init_subclass__(**kwargs)

class A(Base): pass
class B(Base): pass

# 输出：
# 子类 A 被创建
# 子类 B 被创建
```

## Python 元类的进阶知识点
### object 和 type 的自举关系
+ 在 Python 里：

```python
type(object)   # <class 'type'>
type(type)     # <class 'type'>
```

+ 这意味着：
    - `object` 是所有类的基类
    - `type` 是所有类的元类
    - 但 `type` 自己也是 `type` 的实例
+ 这种循环关系让 Python 的类型系统保持统一

### __prepare__ 钩子
+ 元类可以定义一个 `__prepare__` 方法，控制类体的命名空间
+ 默认是 `dict`，但你可以换成 `OrderedDict`、自定义映射，甚至做校验

示例：

```python
class MyMeta(type):
    @classmethod
    def __prepare__(mcs, name, bases):
        print("准备命名空间:", name)
        return {}

    def __new__(mcs, name, bases, attrs):
        print("创建类:", name)
        return super().__new__(mcs, name, bases, attrs)

class Foo(metaclass=MyMeta):
    x = 1
    y = 2
```

应用：Django 就用过 `__prepare__` 来保证 `Model` 中字段的定义顺序

### 类的方法解析顺序
+ 元类可以控制继承时类的 `__mro__`（多继承搜索顺序）。
+ 默认用 C3 线性化算法。
+ 如果你写自定义元类，可以在 `mro()` 方法里修改。

```python
class MyMeta(type):
    def mro(cls):
        print("计算 MRO:", cls.__name__)
        return super().mro()

class Base(metaclass=MyMeta): pass
class A(Base): pass
```

### 元类与 __slots__
+ 如果类定义了 `__slots__`，元类会影响其内存布局
+ 有时 ORM 框架用元类+`__slots__` 来优化对象大小

### 元类冲突
+ 当一个类继承多个基类，而这些基类有不同的元类时，会触发冲突：

```plain
TypeError: metaclass conflict
```

+ 解决办法是写一个 共同的元类，继承所有父元类

### 与抽象基类的关系
+ Python 的抽象基类就是靠元类实现的
+ `ABCMeta` 会检查是否有抽象方法未实现：

```python
from abc import ABC, abstractmethod

class Base(ABC):
    @abstractmethod
    def run(self): pass

# 报错：Can't instantiate abstract class
class Sub(Base): pass
```

### 元类与 __getattribute__ / __getattr__
+ 元类可以拦截类属性的访问，就像普通对象可以拦截实例属性一样
+ 例如：

```python
class Meta(type):
    def __getattribute__(cls, item):
        print(f"访问类属性: {item}")
        return super().__getattribute__(item)

class Foo(metaclass=Meta):
    x = 42

print(Foo.x)  # 访问类属性: x → 42
```

+ 应用：动态计算类属性、惰性加载

### 元类与 __dir__
+ 可以自定义类的 `dir()` 输出
+ 用来实现动态 API，例如根据数据库表字段动态生成方法

```python
class Meta(type):
    def __dir__(cls):
        return ["dynamic_method", "another_method"]

class Foo(metaclass=Meta):
    pass

print(dir(Foo))  # ['dynamic_method', 'another_method']
```

### 元类与 __annotations__
+ 从 Python 3 开始，类体中的类型注解会被收集到 `__annotations__`
+ 元类可以利用这个机制做类型检查或自动代码生成

```python
class Meta(type):
    def __new__(mcs, name, bases, attrs):
        print(f"{name} 注解:", attrs.get("__annotations__"))
        return super().__new__(mcs, name, bases, attrs)

class Foo(metaclass=Meta):
    x: int
    y: str
```

应用：类似 Pydantic 的数据验证模型

### 元类与类装饰器的组合
+ 有时候单靠元类不够灵活，可以结合类装饰器使用：
    - 元类 → 统一拦截和增强；
    - 类装饰器 → 针对个别类再做修改

```python
def class_decorator(cls):
    cls.extra = "decorated"
    return cls

class Meta(type):
    def __new__(mcs, name, bases, attrs):
        attrs["from_meta"] = "yes"
        return super().__new__(mcs, name, bases, attrs)

@class_decorator
class Foo(metaclass=Meta):
    pass

print(Foo.from_meta, Foo.extra)  # yes decorated
```

### 动态修改元类
+ 元类本质上也是类，可以在运行时修改它，从而影响所有用它创建的类

```python
class Meta(type):
    pass

class Foo(metaclass=Meta):
    pass

# 动态修改元类
def new_repr(cls): return f"<Dynamic {cls.__name__}>"
Meta.__repr__ = new_repr

print(Foo)  # <Dynamic Foo>
```
