# 04. 面向对象编程

## 概述

Python 是一种多范式语言，完整支持面向对象编程（OOP）。本章介绍类与对象、继承、多态、特殊方法等核心概念。

---

## 1. 类与对象基础

### 1.1 定义类

```python
class Person:
    """人物类"""
    
    # 类属性 (所有实例共享)
    species = "Homo sapiens"
    
    # 初始化方法
    def __init__(self, name: str, age: int):
        # 实例属性
        self.name = name
        self.age = age
    
    # 实例方法
    def greet(self) -> str:
        return f"Hello, I'm {self.name}"
    
    # 字符串表示
    def __str__(self) -> str:
        return f"Person({self.name}, {self.age})"
    
    def __repr__(self) -> str:
        return f"Person(name={self.name!r}, age={self.age!r})"

# 创建实例
person = Person("Alice", 30)
print(person.name)      # Alice
print(person.greet())   # Hello, I'm Alice
print(person)           # Person(Alice, 30)
```

### 1.2 self 参数

```python
class MyClass:
    def __init__(self, value):
        self.value = value
    
    def get_value(self):
        return self.value  # 访问实例属性
    
    @classmethod
    def create(cls, value):
        """类方法，第一个参数是类本身"""
        return cls(value)
    
    @staticmethod
    def static_method():
        """静态方法，不需要 self 或 cls"""
        return "static"
```

---

## 2. 属性与访问控制

### 2.1 属性装饰器

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius  # 私有属性 (约定)
    
    @property
    def radius(self):
        """获取半径"""
        return self._radius
    
    @radius.setter
    def radius(self, value):
        """设置半径，包含验证"""
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value
    
    @property
    def area(self):
        """计算面积 (只读属性)"""
        return 3.14159 * self._radius ** 2

c = Circle(5)
print(c.radius)  # 5
c.radius = 10    # 设置 (触发 setter)
print(c.area)    # 314.159 (只读)
```

### 2.2 私有属性

```python
class BankAccount:
    def __init__(self, initial_balance):
        self.__balance = initial_balance  # 名称重整 (Name Mangling)
    
    def deposit(self, amount):
        if amount > 0:
            self.__balance += amount
            return True
        return False
    
    def withdraw(self, amount):
        if 0 < amount <= self.__balance:
            self.__balance -= amount
            return True
        return False
    
    @property
    def balance(self):
        return self.__balance

account = BankAccount(1000)
account.deposit(500)
print(account.balance)  # 1500

# 访问私有属性 (不推荐)
# account._BankAccount__balance
```

---

## 3. 继承

### 3.1 基本继承

```python
class Animal:
    def __init__(self, name: str):
        self.name = name
    
    def speak(self) -> str:
        raise NotImplementedError

class Dog(Animal):
    def speak(self) -> str:
        return "Woof!"

class Cat(Animal):
    def speak(self) -> str:
        return "Meow!"

dog = Dog("Buddy")
print(dog.name)    # Buddy
print(dog.speak())  # Woof!
```

### 3.2 super() 函数

```python
class Person:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age

class Employee(Person):
    def __init__(self, name: str, age: int, employee_id: str):
        super().__init__(name, age)  # 调用父类 __init__
        self.employee_id = employee_id

class Manager(Employee):
    def __init__(self, name: str, age: int, employee_id: str, department: str):
        super().__init__(name, age, employee_id)
        self.department = department
```

### 3.3 多继承

```python
class Flyer:
    def fly(self):
        return "Flying!"

class Swimmer:
    def swim(self):
        return "Swimming!"

class Duck(Flyer, Swimmer):
    pass

duck = Duck()
print(duck.fly())   # Flying!
print(duck.swim())  # Swimming!

# MRO (Method Resolution Order)
print(Duck.__mro__)
# (<class 'Duck'>, <class 'Flyer'>, <class 'Swimmer'>, <class 'object'>)
```

---

## 4. 多态

多态允许不同类的对象对同一消息做出不同响应。

```python
# 协议/抽象基类
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass
    
    @abstractmethod
    def perimeter(self) -> float:
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self) -> float:
        return self.width * self.height
    
    def perimeter(self) -> float:
        return 2 * (self.width + self.height)

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def area(self) -> float:
        return 3.14159 * self.radius ** 2
    
    def perimeter(self) -> float:
        return 2 * 3.14159 * self.radius

# 多态使用
def print_shape_info(shape: Shape):
    print(f"Area: {shape.area():.2f}")
    print(f"Perimeter: {shape.perimeter():.2f}")

shapes: list[Shape] = [Rectangle(3, 4), Circle(5)]
for shape in shapes:
    print_shape_info(shape)
    print()
```

---

## 5. 特殊方法 (魔术方法)

### 5.1 对象表示

```python
class Book:
    def __init__(self, title: str, author: str, pages: int):
        self.title = title
        self.author = author
        self.pages = pages
    
    def __str__(self) -> str:
        """人类可读字符串"""
        return f"'{self.title}' by {self.author}"
    
    def __repr__(self) -> str:
        """开发者字符串 (可重建对象)"""
        return f"Book({self.title!r}, {self.author!r}, {self.pages!r})"
    
    def __format__(self, format_spec):
        return f"'{self.title}' ({self.pages} pages)"

book = Book("1984", "George Orwell", 328)
print(str(book))   # '1984' by George Orwell
print(repr(book))  # Book('1984', 'George Orwell', 328)
print(f"{book:short}")  # '1984' (328 pages)
```

### 5.2 比较操作

```python
class Version:
    def __init__(self, major: int, minor: int, patch: int = 0):
        self.major = major
        self.minor = minor
        self.patch = patch
    
    def __eq__(self, other) -> bool:
        if not isinstance(other, Version):
            return NotImplemented
        return (self.major, self.minor, self.patch) == \
               (other.major, other.minor, other.patch)
    
    def __lt__(self, other) -> bool:
        if not isinstance(other, Version):
            return NotImplemented
        return (self.major, self.minor, self.patch) < \
               (other.major, other.minor, other.patch)
    
    def __le__(self, other) -> bool:
        return self == other or self < other
    
    def __gt__(self, other) -> bool:
        return not self <= other
    
    def __ge__(self, other) -> bool:
        return not self < other

v1 = Version(1, 2, 0)
v2 = Version(1, 2, 1)
print(v1 < v2)   # True
print(v1 == v2)  # False
```

### 5.3 算术运算

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
    
    def __sub__(self, other):
        return Vector(self.x - other.x, self.y - other.y)
    
    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)
    
    def __rmul__(self, scalar):
        return self.__mul__(scalar)
    
    def __neg__(self):
        return Vector(-self.x, -self.y)
    
    def __abs__(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5
    
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)   # Vector(4, 6)
print(v1 * 3)    # Vector(3, 6)
print(3 * v1)    # Vector(3, 6)
print(-v1)       # Vector(-1, -2)
print(abs(v1))   # 2.23606797749979
```

### 5.4 容器协议

```python
class Counter:
    def __init__(self, data: list):
        self.data = data
    
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, index):
        return self.data[index]
    
    def __setitem__(self, index, value):
        self.data[index] = value
    
    def __contains__(self, item):
        return item in self.data
    
    def __iter__(self):
        return iter(self.data)
    
    def __reversed__(self):
        return reversed(self.data)

c = Counter([1, 2, 3, 4, 5])
print(len(c))     # 5
print(c[0])       # 1
print(3 in c)     # True
print(list(c))    # [1, 2, 3, 4, 5]
print(list(reversed(c)))  # [5, 4, 3, 2, 1]
```

### 5.5 可调用对象

```python
class Adder:
    def __init__(self, n):
        self.n = n
    
    def __call__(self, x):
        return self.n + x

add_five = Adder(5)
print(add_five(10))  # 15
print(callable(add_five))  # True
```

---

## 6. 类方法与静态方法

```python
class MyClass:
    class_attr = "class attribute"
    
    def __init__(self, instance_attr):
        self.instance_attr = instance_attr
    
    # 实例方法: 第一个参数是 self
    def instance_method(self):
        return f"Instance: {self.instance_attr}"
    
    # 类方法: 第一个参数是 cls
    @classmethod
    def class_method(cls):
        return f"Class: {cls.class_attr}"
    
    # 静态方法: 无隐式参数
    @staticmethod
    def static_method():
        return "Static method"

print(MyClass.class_method())  # Class: class attribute
print(MyClass.static_method()) # Static method

obj = MyClass("value")
print(obj.instance_method())   # Instance: value
```

---

## 7. 数据类 (dataclass)

Python 3.7+ 引入了 dataclass，简化创建类。

```python
from dataclasses import dataclass, field
from typing import List
from datetime import datetime

@dataclass
class Person:
    name: str
    age: int
    email: str = ""  # 默认值
    created_at: datetime = field(default_factory=datetime.now)

@dataclass
class Company:
    name: str
    employees: List[str] = field(default_factory=list)
    employee_count: int = 0

# 创建实例
person = Person("Alice", 30, "alice@example.com")
print(person)
# Person(name='Alice', age=30, email='alice@example.com', created_at=datetime(...))

# 比较
p1 = Person("Alice", 30)
p2 = Person("Alice", 30)
print(p1 == p2)  # True (自动生成 __eq__)

# 不可变 dataclass
@dataclass(frozen=True)
class Point:
    x: float
    y: float
```

---

## 8. 协议与结构化类型

### 8.1 抽象基类 (ABC)

```python
from abc import ABC, abstractmethod

class Drawable(ABC):
    @abstractmethod
    def draw(self):
        pass
    
    @property
    @abstractmethod
    def color(self) -> str:
        pass

class Circle(Drawable):
    def __init__(self, color: str, radius: float):
        self.color_value = color
        self.radius = radius
    
    def draw(self):
        print(f"Drawing circle with radius {self.radius}")
    
    @property
    def color(self) -> str:
        return self.color_value
```

### 8.2 协议 (Protocol)

Python 3.8+ 支持结构子类型（Protocol）。

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...
    @property
    def color(self) -> str: ...

class Circle:
    def __init__(self, color: str):
        self.color_value = color
    
    def draw(self) -> None:
        print("Drawing circle")
    
    @property
    def color(self) -> str:
        return self.color_value

# 静态类型检查器会验证 Circle 实现了 Drawable
def render(d: Drawable) -> None:
    d.draw()

render(Circle("red"))  # OK
```

---

## 下一步

- [异常处理与调试](../05-异常处理与调试/README.md) — 学习异常处理机制
- [模块与包管理](../06-模块与包管理/README.md) — 深入 Python 模块系统

---

*参考资料：[Python 类](https://docs.python.org/3/tutorial/classes.html) | [dataclasses](https://docs.python.org/3/library/dataclasses.html)*
