---
title: "Python dataclasses 详解：简化类定义与高级技巧"
date: 2025-06-04T08:30:59+08:00
lastmod: 2025-06-04T08:30:59+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "Python dataclasses 详解"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image/icons8-python-500.png"
---

`dataclasses` 模块是 Python 3.7 引入的一个非常有用的模块，它提供了一个装饰器和一组函数来简化类的定义

### 1. 基本用法
最简单的用法是使用 `@dataclass` 装饰器来自动为类生成常见的特殊方法，比如 `__init__`、`__repr__`、`__eq__` 等

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

# 创建实例
p = Point(1, 2)
print(p)  # 输出: Point(x=1, y=2)
```

### 2. 字段默认值
你可以为字段提供默认值，或者使用 `field` 来指定更复杂的默认值

```python
from dataclasses import dataclass, field

@dataclass
class Person:
    name: str
    age: int = 30  # 默认值
    email: str = field(default="example@example.com")  # 使用 field 定义默认值

# 创建实例
p1 = Person("Alice")
p2 = Person("Bob", 25)

print(p1)  # Person(name='Alice', age=30, email='example@example.com')
print(p2)  # Person(name='Bob', age=25, email='example@example.com')
```

### 3. field 参数
`field` 函数可以用来为数据字段提供更详细的控制，例如设置默认值、指定字段不可变、为字段提供比较等

常见参数：

+ `default`: 设置默认值
+ `default_factory`: 使用工厂函数为字段提供默认值
+ `init`: 控制字段是否在初始化方法中作为参数
+ `repr`: 控制是否在 `__repr__` 输出中显示该字段
+ `compare`: 控制字段是否参与对象比较
+ `hash`: 控制是否为字段生成 `__hash__` 方法

#### 3.1 default
+ **作用**：为字段设置一个默认值，如果创建对象时没有显式传递该字段的值，则使用该默认值。

```python
from dataclasses import dataclass, field

@dataclass
class Product:
    name: str
    price: float = field(default=10.0)

p = Product("Apple")
print(p)  # Product(name='Apple', price=10.0)
```

#### 3.2 default_factory
+ 作用：使用工厂函数为字段提供默认值。相比 `default`，`default_factory` 用于为可变类型（如列表、字典等）字段提供默认值，因为直接赋默认值会导致所有实例共享同一个对象。

```python
from dataclasses import dataclass, field

@dataclass
class ShoppingCart:
    items: list = field(default_factory=list)

cart1 = ShoppingCart()
cart2 = ShoppingCart()

cart1.items.append("apple")
cart2.items.append("banana")

print(cart1.items)  # ['apple']
print(cart2.items)  # ['banana']
```

#### 3.3 init
+ 作用：控制字段是否在生成的 `__init__` 方法中作为参数。默认为 `True`，即字段会作为构造函数的参数。如果设置为 `False`，字段就不会出现在构造函数中，但可以在类内部使用

```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: int
    y: int
    z: int = field(init=False)  # z 不会出现在 __init__ 方法中

    def __post_init__(self):
        self.z = self.x + self.y

p = Point(3, 4)
print(p)  # Point(x=3, y=4, z=7)
```

这里 `z` 没有出现在 `__init__` 方法中，但我们可以在 `__post_init__` 中手动设置它的值

#### 3.4 repr
+ 作用：控制字段是否在自动生成的 `__repr__` 方法中显示。如果设置为 `False`，则该字段不会出现在 `__repr__` 输出中

```python
from dataclasses import dataclass, field

@dataclass
class Person:
    name: str
    age: int
    _password: str = field(repr=False)  # 密码字段不显示在 repr 中

p = Person("Alice", 30, "secret")
print(p)  # Person(name='Alice', age=30)
```

在上面的例子中，`_password` 字段不会显示在 `repr` 输出中

#### 3.5 compare
+ 作用：控制该字段是否参与对象比较（如使用 `==` 或 `<` 进行比较）。默认为 `True`，即该字段会参与比较。设置为 `False`，则该字段不会影响对象的比较结果

```python
from dataclasses import dataclass, field

@dataclass
class Product:
    name: str
    price: float
    discount: float = field(compare=False)  # 折扣字段不参与比较

p1 = Product("Laptop", 1000, 100)
p2 = Product("Laptop", 1000, 200)

print(p1 == p2)  # True, 因为 discount 不参与比较
```

在上面的例子中，`discount` 字段的值虽然不同，但由于我们设置了 `compare=False`，它不会影响比较结果

#### 3.6 hash
+ 作用：控制该字段是否用于生成对象的 `__hash__` 方法。如果设置为 `False`，则该字段不会参与 `__hash__` 的计算。默认为 `True`（如果数据类是不可变的）

```python
from dataclasses import dataclass, field

@dataclass(frozen=True)
class Point:
    x: int
    y: int
    z: int = field(hash=False)  # z 不参与 hash 计算

p1 = Point(1, 2, 3)
p2 = Point(1, 2, 4)

# 可以将 p1 和 p2 用作字典的键，因为 z 不参与 hash 计算
d = {p1: "point1", p2: "point2"}
print(d)
```

在上面的例子中，即使 `z` 字段不同，`p1` 和 `p2` 的哈希值仍然相同，因为我们将 `z` 设置为 `hash=False`

### 4. 不可变类 (frozen=True)
你可以使数据类变为不可变的类，即所有字段都成为常量

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: int
    y: int

p = Point(1, 2)
# p.x = 3  # 会抛出错误: dataclasses.FrozenInstanceError
```

### 5. 自定义方法
你仍然可以在数据类中定义自定义方法

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

    def distance_from_origin(self) -> float:
        return (self.x ** 2 + self.y ** 2) ** 0.5

p = Point(3, 4)
print(p.distance_from_origin())  # 输出: 5.0
```

### 6. __post_init__ 方法
`__post_init__` 是一个特殊的方法，它在 `__init__` 方法调用后执行。你可以用它来进行一些额外的初始化工作或验证数据

```python
from dataclasses import dataclass

@dataclass
class Product:
    name: str
    price: float
    quantity: int

    def __post_init__(self):
        if self.price < 0:
            raise ValueError("Price cannot be negative")
        if self.quantity < 0:
            raise ValueError("Quantity cannot be negative")

p = Product("Apple", 1.5, 10)
# p_invalid = Product("Apple", -1.5, 10)  # 会抛出 ValueError
```

### 7. asdict 和  astuple 
`asdict` 将数据类转换为字典，而 `astuple` 将其转换为元组。

```python
from dataclasses import dataclass, asdict, astuple

@dataclass
class Point:
    x: int
    y: int

p = Point(1, 2)
print(asdict(p))  # 输出: {'x': 1, 'y': 2}
print(astuple(p))  # 输出: (1, 2)
```

### 8. 嵌套数据类
`dataclasses` 支持嵌套类，可以定义数据类的字段为另一个数据类

```python
from dataclasses import dataclass

@dataclass
class Address:
    city: str
    state: str

@dataclass
class Person:
    name: str
    address: Address

addr = Address("New York", "NY")
person = Person("John", addr)
print(person)  # Person(name='John', address=Address(city='New York', state='NY'))
```

### 9. 比较和排序
数据类支持比较和排序，默认情况下会基于字段的顺序进行比较。你可以通过 `order=True` 在装饰器中启用排序支持

```python
from dataclasses import dataclass

@dataclass(order=True)
class Point:
    x: int
    y: int

p1 = Point(1, 2)
p2 = Point(2, 1)

print(p1 < p2)  # 输出: True
```

### 10. dataclass 中的 init, repr, eq, order, unsafe_hash
+ `init`：是否自动生成 `__init__` 方法（默认为 `True`）
+ `repr`：是否自动生成 `__repr__` 方法（默认为 `True`）
+ `eq`：是否自动生成 `__eq__` 方法用于比较相等性（默认为 `True`）
+ `order`：是否自动生成用于排序的比较方法（默认为 `False`）
+ `unsafe_hash`：是否自动生成 `__hash__` 方法（默认为 `False`）

```python
from dataclasses import dataclass

@dataclass(eq=True, repr=True, order=False, unsafe_hash=False)
class Person:
    name: str
    age: int
```

### 11. dataclass 的 kw_only 参数（Python 3.10+）
从 Python 3.10 开始，`dataclasses` 允许你通过 `kw_only` 参数指定某些字段只能通过关键字参数进行初始化。这可以帮助你避免使用位置参数进行初始化时的混淆

```python
from dataclasses import dataclass

@dataclass(kw_only=True)
class Person:
    name: str
    age: int
    city: str

# p = Person("Alice", 30, "New York")  # 错误，必须使用关键字参数
p = Person(name="Alice", age=30, city="New York")
print(p)  # Person(name='Alice', age=30, city='New York')
```

### 12. 数据类的 __del__ 方法
尽管 `dataclass` 会自动生成大部分方法，但如果你需要在数据类销毁时做一些清理工作（如关闭文件、网络连接等），可以自定义 `__del__` 方法

```python
from dataclasses import dataclass

@dataclass
class FileHandler:
    filename: str

    def __del__(self):
        print(f"Cleaning up the file: {self.filename}")

# 创建对象并销毁时会调用 __del__
f = FileHandler("example.txt")
del f  # 会触发 __del__ 输出
```

### 13. __eq__方法的定制
默认情况下，数据类会自动生成 `__eq__` 方法来比较两个实例是否相等。你可以通过继承并重写 `__eq__` 来定制比较行为

```python
from dataclasses import dataclass

@dataclass
class Person:
    name: str
    age: int

    def __eq__(self, other):
        if isinstance(other, Person):
            return self.name == other.name
        return False

p1 = Person("Alice", 30)
p2 = Person("Alice", 25)
p3 = Person("Bob", 30)

print(p1 == p2)  # True
print(p1 == p3)  # False
```

### 14. __hash__ 方法的定制
默认情况下，如果数据类是不可变的（`frozen=True`），它会自动生成 `__hash__` 方法。如果需要自定义 `__hash__`，可以直接重写

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Person:
    name: str
    age: int

    def __hash__(self):
        return hash(self.name)

p1 = Person("Alice", 30)
p2 = Person("Bob", 25)

print(hash(p1))
print(hash(p2))
```

### 15. 类的继承
数据类也可以继承。子类会继承父类的字段，并且能够自动处理父类字段的初始化

```python
from dataclasses import dataclass

@dataclass
class Employee:
    name: str
    salary: float

@dataclass
class Manager(Employee):
    department: str

manager = Manager(name="Alice", salary=100000, department="HR")
print(manager)  # Manager(name='Alice', salary=100000, department='HR')
```

### 16. dataclass 与 TypeVar 和 Generic 配合使用
`dataclass` 也可以和 Python 的泛型（`TypeVar`、`Generic`）配合使用，允许你在数据类中使用类型参数

```python
from dataclasses import dataclass
from typing import TypeVar, Generic

T = TypeVar('T')

@dataclass
class Box(Generic[T]):
    value: T

box_int = Box(123)
box_str = Box("hello")
print(box_int)  # Box(value=123)
print(box_str)  # Box(value='hello')
```

### 17. dataclass 与 __slots__
`dataclass` 默认会创建一个 `__dict__` 用于存储实例的属性，这会占用内存。如果你希望节省内存，可以使用 `__slots__` 来限制类的属性

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int
    __slots__ = ('x', 'y')  # 限制类的属性

# 不会创建 __dict__，节省内存
p = Point(1, 2)
print(p.__dict__)  # 抛出 AttributeError: 'Point' object has no attribute '__dict__'
```

### 18. dataclass 和 Enum 配合使用
数据类可以与 `Enum` 一起使用，可以更好地表示一些枚举值。

```python
from dataclasses import dataclass
from enum import Enum

class Status(Enum):
    ACTIVE = "active"
    INACTIVE = "inactive"

@dataclass
class User:
    name: str
    status: Status

user = User("John", Status.ACTIVE)
print(user)  # User(name='John', status=<Status.ACTIVE: 'active'>)
```

### 19. dataclass 与 dataclasses.field 的组合使用
使用 `dataclasses.field` 可以进行更精细的字段控制，例如当字段需要被初始化但不参与比较时

```python
from dataclasses import dataclass, field

@dataclass
class Data:
    name: str
    value: int = field(compare=False)

data = Data("example", 100)
print(data)  # Data(name='example', value=100)
```

### 20. dataclass 与 __new__
在某些情况下，你可能想要在实例化类时做一些额外的工作（比如单例模式）。你可以在数据类中重写 `__new__` 方法

```python
from dataclasses import dataclass

@dataclass
class Singleton:
    name: str
    _instances = {}

    def __new__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__new__(cls)
        return cls._instances[cls]

s1 = Singleton("instance1")
s2 = Singleton("instance2")
print(s1 == s2)  # True
```

### 21. __str__ 方法定制
如果你希望自定义输出格式而不是使用默认的 `__repr__`，你可以实现 `__str__` 方法

```python
from dataclasses import dataclass

@dataclass
class Product:
    name: str
    price: float

    def __str__(self):
        return f"{self.name} costs ${self.price}"

p = Product("Laptop", 999.99)
print(str(p))  # Laptop costs $999.99
```
