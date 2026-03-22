# 12. 最佳实践与设计模式

## 概述

本章介绍 Python 开发的最佳实践和常用设计模式，帮助你编写高质量、可维护的代码。

---

## 1. Python 代码规范

### 1.1 PEP 8 要点

```python
# 缩进：4 空格
if True:
    do_something()

# 行长度：最多 79 字符
long_line = (
    "This is a very long line that "
    "spans multiple lines"
)

# 导入顺序
import os               # 标准库
import sys

import numpy as np      # 第三方库
import pandas as pd

from mypackage import   # 本地导入
    mymodule

# 空格使用
x = 1           # ✅
x=1             # ❌
func(a, b)      # ✅
func(a , b)     # ❌
if x == 1:      # ✅
if x==1 :       # ❌

# 命名规范
MY_CONSTANT = 42         # 常量
my_variable = 1           # 变量
my_function()            # 函数
MyClass                   # 类
_private = 1              # 私有
__mangled = 1             # 名称重整
```

### 1.2 类型注解规范

```python
from typing import Optional, List, Dict, Callable

# 变量注解
name: str = "Alice"
age: int = 30
scores: List[int] = [90, 85, 88]
mapping: Dict[str, int] = {"a": 1}

# 函数注解
def greet(name: str, times: int = 1) -> str:
    return ", ".join([f"Hello, {name}!"] * times)

# 可调用类型
def apply(func: Callable[[int, int], int], x: int, y: int) -> int:
    return func(x, y)

# 联合类型
def parse(value: str | int | float) -> float:
    return float(value)

# 可选类型
def find(name: Optional[str] = None) -> str:
    return name or "Anonymous"
```

---

## 2. 创建型模式

### 2.1 单例模式

```python
# 方法一：模块级单例（推荐）
# mysingleton.py
class _Singleton:
    def __init__(self):
        self.value = None

singleton = _Singleton()

# 使用
from mysingleton import singleton

# 方法二：装饰器
def singleton(cls):
    instance = None
    def get_instance(*args, **kwargs):
        nonlocal instance
        if instance is None:
            instance = cls(*args, **kwargs)
        return instance
    return get_instance

@singleton
class MyClass:
    def __init__(self):
        self.value = None

# 方法三：元类
class SingletonMeta(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self):
        self.connection = None
```

### 2.2 工厂模式

```python
from abc import ABC, abstractmethod

# 抽象产品
class Button(ABC):
    @abstractmethod
    def render(self) -> str:
        pass

# 具体产品
class WindowsButton(Button):
    def render(self) -> str:
        return "Windows Button"

class MacButton(Button):
    def render(self) -> str:
        return "Mac Button"

# 工厂
class ButtonFactory:
    @staticmethod
    def create_button(platform: str) -> Button:
        buttons = {
            "windows": WindowsButton,
            "mac": MacButton
        }
        return buttons[platform]()

# 使用
button = ButtonFactory.create_button("windows")
print(button.render())  # Windows Button
```

### 2.3 构建者模式

```python
class Pizza:
    def __init__(self):
        self.dough = None
        self.sauce = None
        self.toppings = []

class PizzaBuilder:
    def __init__(self):
        self.pizza = Pizza()
    
    def set_dough(self, dough: str):
        self.pizza.dough = dough
        return self
    
    def set_sauce(self, sauce: str):
        self.pizza.sauce = sauce
        return self
    
    def add_topping(self, topping: str):
        self.pizza.toppings.append(topping)
        return self
    
    def build(self):
        return self.pizza

# 使用
pizza = (PizzaBuilder()
    .set_dough("thin crust")
    .set_sauce("tomato")
    .add_topping("cheese")
    .add_topping("pepperoni")
    .build())
```

---

## 3. 结构型模式

### 3.1 装饰器模式

```python
# 基础组件
class Coffee:
    def cost(self) -> float:
        return 2.0
    
    def description(self) -> str:
        return "Coffee"

# 装饰器基类
class CoffeeDecorator(Coffee):
    def __init__(self, coffee: Coffee):
        self._coffee = coffee
    
    def cost(self) -> float:
        return self._coffee.cost()
    
    def description(self) -> str:
        return self._coffee.description()

# 具体装饰器
class Milk(CoffeeDecorator):
    def cost(self) -> float:
        return super().cost() + 0.5
    
    def description(self) -> str:
        return super().description() + ", Milk"

class Sugar(CoffeeDecorator):
    def cost(self) -> float:
        return super().cost() + 0.2
    
    def description(self) -> str:
        return super().description() + ", Sugar"

# 使用
coffee = Coffee()
coffee = Milk(coffee)
coffee = Sugar(coffee)
print(coffee.description())  # Coffee, Milk, Sugar
print(coffee.cost())        # 2.7
```

### 3.2 适配器模式

```python
# 旧接口
class OldAPI:
    def get_user_data(self) -> dict:
        return {"name": "Alice", "age": 30}

# 新接口
class NewAPI:
    def fetch_user(self) -> dict:
        return {"full_name": "Alice", "years": 30}

# 适配器
class APIAdapter:
    def __init__(self, api):
        self._api = api
    
    def get_user_data(self) -> dict:
        data = self._api.fetch_user()
        return {
            "name": data["full_name"].split()[0],
            "age": data["years"]
        }

# 使用
adapter = APIAdapter(NewAPI())
print(adapter.get_user_data())  # {'name': 'Alice', 'age': 30}
```

### 3.3 代理模式

```python
class Image:
    def __init__(self, filename: str):
        self.filename = filename
        self._load()
    
    def _load(self):
        print(f"Loading {self.filename}")
    
    def display(self):
        print(f"Displaying {self.filename}")

class ProxyImage:
    def __init__(self, filename: str):
        self.filename = filename
        self._image = None
    
    def display(self):
        if self._image is None:
            self._image = Image(self.filename)
        self._image.display()

# 使用
image = ProxyImage("large_photo.jpg")  # 不加载
image.display()  # 加载并显示
image.display()  # 直接显示（已加载）
```

---

## 4. 行为型模式

### 4.1 策略模式

```python
from abc import ABC, abstractmethod

class SortStrategy(ABC):
    @abstractmethod
    def sort(self, data: list) -> list:
        pass

class QuickSort(SortStrategy):
    def sort(self, data: list) -> list:
        # 实际实现
        return sorted(data)  # 简化

class MergeSort(SortStrategy):
    def sort(self, data: list) -> list:
        return sorted(data)  # 简化

class Sorter:
    def __init__(self, strategy: SortStrategy):
        self._strategy = strategy
    
    def set_strategy(self, strategy: SortStrategy):
        self._strategy = strategy
    
    def sort(self, data: list) -> list:
        return self._strategy.sort(data)

# 使用
sorter = Sorter(QuickSort())
print(sorter.sort([3, 1, 2]))

sorter.set_strategy(MergeSort())
print(sorter.sort([3, 1, 2]))
```

### 4.2 观察者模式

```python
from abc import ABC, abstractmethod
from typing import List

class Observer(ABC):
    @abstractmethod
    def update(self, message: str):
        pass

class Subject:
    def __init__(self):
        self._observers: List[Observer] = []
    
    def attach(self, observer: Observer):
        self._observers.append(observer)
    
    def detach(self, observer: Observer):
        self._observers.remove(observer)
    
    def notify(self, message: str):
        for observer in self._observers:
            observer.update(message)

class NewsAgency(Subject):
    def __init__(self):
        super().__init__()
        self._news = ""
    
    def set_news(self, news: str):
        self._news = news
        self.notify(news)

class NewsChannel(Observer):
    def __init__(self, name: str):
        self.name = name
    
    def update(self, message: str):
        print(f"{self.name} received: {message}")

# 使用
agency = NewsAgency()
channel1 = NewsChannel("Channel 1")
channel2 = NewsChannel("Channel 2")

agency.attach(channel1)
agency.attach(channel2)
agency.set_news("Breaking news!")
```

### 4.3 命令模式

```python
from abc import ABC, abstractmethod

class Command(ABC):
    @abstractmethod
    def execute(self):
        pass
    
    @abstractmethod
    def undo(self):
        pass

class Light:
    def on(self):
        print("Light is ON")
    
    def off(self):
        print("Light is OFF")

class LightOnCommand(Command):
    def __init__(self, light: Light):
        self._light = light
    
    def execute(self):
        self._light.on()
    
    def undo(self):
        self._light.off()

class RemoteControl:
    def __init__(self):
        self._history: List[Command] = []
    
    def execute(self, command: Command):
        command.execute()
        self._history.append(command)
    
    def undo(self):
        if self._history:
            command = self._history.pop()
            command.undo()

# 使用
light = Light()
remote = RemoteControl()

remote.execute(LightOnCommand(light))
remote.undo()  # 撤销
```

### 4.4 迭代器模式

```python
from typing import Iterator

class TreeNode:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right

class TreeIterator(Iterator):
    def __init__(self, root):
        self._stack = []
        self._current = root
    
    def __iter__(self):
        return self
    
    def __next__(self):
        while self._stack or self._current:
            if self._current:
                self._stack.append(self._current)
                self._current = self._current.left
            else:
                node = self._stack.pop()
                result = node.value
                self._current = node.right
                return result
        raise StopIteration

# 使用
root = TreeNode(1,
    TreeNode(2, TreeNode(4), TreeNode(5)),
    TreeNode(3)
)

for value in TreeIterator(root):
    print(value)  # 1, 2, 4, 5, 3 (中序遍历)
```

---

## 5. 依赖注入

```python
# 依赖注入容器
class Container:
    def __init__(self):
        self._services = {}
    
    def register(self, name: str, factory):
        self._services[name] = factory
    
    def get(self, name: str):
        factory = self._services.get(name)
        if factory:
            return factory()
        raise KeyError(f"Service {name} not found")

# 使用
container = Container()
container.register('database', lambda: SQLDatabase())
container.register('cache', lambda: RedisCache())

db = container.get('database')
cache = container.get('cache')
```

---

## 6. 上下文管理器

```python
# 类式上下文管理器
class Timer:
    def __enter__(self):
        import time
        self.start = time.time()
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        import time
        elapsed = time.time() - self.start
        print(f"Elapsed: {elapsed:.2f}s")
        return False

# 使用
with Timer() as timer:
    # do something
    pass

# 生成器式
from contextlib import contextmanager

@contextmanager
def timer():
    import time
    start = time.time()
    try:
        yield
    finally:
        print(f"Elapsed: {time.time() - start:.2f}s")
```

---

## 7. SOLID 原则

```python
# S - 单一职责原则 (Single Responsibility)
class User:
    def __init__(self, name: str):
        self.name = name

class UserValidator:  # 只负责验证
    def validate(self, user: User) -> bool:
        return bool(user.name)

class UserRepository:  # 只负责存储
    def save(self, user: User):
        pass

# O - 开闭原则 (Open/Closed)
# 对扩展开放，对修改封闭
class Shape(ABC):
    @abstractmethod
    def area(self) -> float: pass

class Rectangle(Shape):
    def __init__(self, w, h):
        self.w = w
        self.h = h
    
    def area(self) -> float:
        return self.w * self.h

# 添加新形状无需修改现有代码
class Circle(Shape):
    def __init__(self, r):
        self.r = r
    
    def area(self) -> float:
        import math
        return math.pi * self.r ** 2

# L - 里氏替换原则 (Liskov Substitution)
# 子类必须可以替换父类
class Bird(ABC):
    @abstractmethod
    def fly(self): pass

# ❌ 错误：Penguin 无法飞行
# class Penguin(Bird):
#     def fly(self):
#         raise NotImplementedError

# ✅ 正确：分离接口
class FlyingBird(ABC):
    @abstractmethod
    def fly(self): pass

class WalkingBird(ABC):
    @abstractmethod
    def walk(self): pass

# I - 接口隔离原则 (Interface Segregation)
# 客户端不应依赖不需要的接口
class IMachine(ABC):
    def print(self): pass
    def scan(self): pass
    def fax(self): pass

# ❌ 太臃肿
# ✅ 分解为小接口
class IPrinter(ABC):
    @abstractmethod
    def print(self): pass

class IScanner(ABC):
    @abstractmethod
    def scan(self): pass

# D - 依赖倒置原则 (Dependency Inversion)
# 高层模块不应依赖低层模块
# ❌ 低级实现
class MySQL:
    def connect(self): pass

class App:
    def __init__(self):
        self.db = MySQL()  # 依赖具体实现

# ✅ 依赖抽象
class Database(ABC):
    @abstractmethod
    def connect(self): pass

class App:
    def __init__(self, db: Database):
        self.db = db
```

---

## 下一步

恭喜你完成 Python 学习！建议：

- 实践项目：将所学知识应用到实际项目中
- 阅读源码：学习优秀开源项目的代码
- 持续学习：关注 Python 最新特性和最佳实践

---

*参考资料：[Design Patterns](https://refactoring.guru/design-patterns) | [SOLID 原则](https://solidpython.com/)*
