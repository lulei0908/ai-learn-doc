# 03. 函数式编程

## 概述

Python 虽然不是纯函数式语言，但支持丰富的函数式编程特性。本章介绍函数定义、高阶函数、装饰器、匿名函数等核心概念。

---

## 1. 函数基础

### 1.1 函数定义

```python
# 基本函数
def greet(name: str) -> str:
    """问候函数"""
    return f"Hello, {name}!"

# 多返回值
def get_stats(numbers: list[int]) -> tuple[int, int, float]:
    return min(numbers), max(numbers), sum(numbers) / len(numbers)

min_val, max_val, avg_val = get_stats([1, 2, 3, 4, 5])

# 默认参数
def greet(name: str, greeting: str = "Hello") -> str:
    return f"{greeting}, {name}!"

greet("Alice")           # "Hello, Alice!"
greet("Bob", "Hi")       # "Hi, Bob!"
```

### 1.2 参数类型

```python
# 关键字参数
def connect(host: str, port: int, timeout: int = 30):
    pass

connect(host="localhost", port=8080)
connect("localhost", port=8080, timeout=60)

# *args 和 **kwargs
def func(*args, **kwargs):
    print(f"Positional: {args}")
    print(f"Keyword: {kwargs}")

func(1, 2, 3, name="Alice", age=30)
# Positional: (1, 2, 3)
# Keyword: {'name': 'Alice', 'age': 30}

# 参数解包
def send_message(to, subject, body):
    print(f"To: {to}, Subject: {subject}, Body: {body}")

msg = {"to": "user@example.com", "subject": "Hello", "body": "Hi!"}
send_message(**msg)
```

### 1.3 特殊参数

```python
# 位置-only 参数 (/)
def func(a, b, /, c, d, *, e, f):
    """位置参数在 / 之前，关键字参数在 * 之后"""
    pass

func(1, 2, 3, d=4, e=5, f=6)  # 正确
func(a=1, b=2, c=3, d=4, e=5, f=6)  # 错误
```

---

## 2. 高阶函数

高阶函数是指接受函数作为参数或返回函数的函数。

### 2.1 内置高阶函数

```python
# map - 对每个元素应用函数
numbers = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x**2, numbers))
# [1, 4, 9, 16, 25]

# filter - 过滤元素
evens = list(filter(lambda x: x % 2 == 0, numbers))
# [2, 4]

# reduce - 累积计算 (需要 import)
from functools import reduce
product = reduce(lambda x, y: x * y, numbers)
# 120

# any / all
any(map(lambda x: x > 3, numbers))  # True
all(map(lambda x: x > 0, numbers))  # True

# sorted - 排序
words = ["banana", "apple", "cherry"]
sorted_words = sorted(words, key=len)  # 按长度排序
```

### 2.2 自定义高阶函数

```python
# 接受函数作为参数
def apply_twice(func: Callable[[int], int], x: int) -> int:
    """对 x 两次应用函数"""
    return func(func(x))

result = apply_twice(lambda x: x + 1, 5)  # 7

# 返回函数
def make_multiplier(factor: int) -> Callable[[int], int]:
    """返回一个乘以指定因子的函数"""
    def multiplier(x: int) -> int:
        return x * factor
    return multiplier

double = make_multiplier(2)
triple = make_multiplier(3)
print(double(5))  # 10
print(triple(5))  # 15
```

---

## 3. 匿名函数 (lambda)

### 3.1 基本语法

```python
# lambda 参数: 表达式
square = lambda x: x ** 2
print(square(5))  # 25

# 多参数
add = lambda x, y: x + y
print(add(3, 4))  # 7

# 立即调用
result = (lambda x, y: x + y)(3, 4)  # 7
```

### 3.2 应用场景

```python
# 与内置函数配合
numbers = [1, 2, 3, 4, 5]

# sorted
pairs = [(1, 'one'), (2, 'two'), (3, 'three')]
sorted_pairs = sorted(pairs, key=lambda x: len(x[1]))
# [(1, 'one'), (2, 'two'), (3, 'three')] 按单词长度排序

# max / min
students = [
    {"name": "Alice", "score": 90},
    {"name": "Bob", "score": 85},
    {"name": "Charlie", "score": 95}
]
top_student = max(students, key=lambda s: s["score"])
# {'name': 'Charlie', 'score': 95}

# groupby
from itertools import groupby
data = [("A", 1), ("B", 2), ("A", 3), ("B", 4)]
grouped = {k: list(v) for k, v in groupby(sorted(data), key=lambda x: x[0])}
# {'A': [('A', 1), ('A', 3)], 'B': [('B', 2), ('B', 4)]}
```

---

## 4. 装饰器

装饰器是修改函数行为的强大工具，本质是高阶函数。

### 4.1 基础装饰器

```python
# 定义装饰器
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before function call")
        result = func(*args, **kwargs)
        print("After function call")
        return result
    return wrapper

# 使用装饰器
@my_decorator
def say_hello(name):
    print(f"Hello, {name}!")

# 等价于
say_hello = my_decorator(say_hello)

say_hello("Alice")
# Before function call
# Hello, Alice!
# After function call
```

### 4.2 带参数的装饰器

```python
# 装饰器工厂
def repeat(times: int):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(times=3)
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")
# Hello, Alice!
# Hello, Alice!
# Hello, Alice!
```

### 4.3 内置装饰器

```python
# @staticmethod - 静态方法
class MyClass:
    @staticmethod
    def static_method():
        print("Static method")

MyClass.static_method()

# @classmethod - 类方法
class MyClass:
    @classmethod
    def class_method(cls):
        print(f"Class method on {cls.__name__}")

MyClass.class_method()

# @property - 属性方法
class Circle:
    def __init__(self, radius):
        self._radius = radius
    
    @property
    def radius(self):
        return self._radius
    
    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value
    
    @property
    def area(self):
        return 3.14159 * self._radius ** 2

c = Circle(5)
print(c.area)  # 78.53975
c.radius = 10
print(c.area)  # 314.159
```

### 4.4 functools.wraps

```python
import functools

def my_decorator(func):
    @functools.wraps(func)  # 保留原函数元信息
    def wrapper(*args, **kwargs):
        print("Decorated!")
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def original_function():
    """这是原始函数的文档"""
    pass

print(original_function.__name__)  # 'original_function'
print(original_function.__doc__)   # '这是原始函数的文档'
```

### 4.5 常用装饰器示例

```python
import functools
import time

# 计时装饰器
def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.4f}s")
        return result
    return wrapper

# 日志装饰器
def log(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

# 缓存装饰器
def cache(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        cache_key = (args, tuple(sorted(kwargs.items())))
        if cache_key not in wrapper.cache:
            wrapper.cache[cache_key] = func(*args, **kwargs)
        return wrapper.cache[cache_key]
    wrapper.cache = {}
    return wrapper

@timer
@cache
def slow_function(n):
    time.sleep(1)
    return n * 2
```

---

## 5. 闭包

闭包是引用了自由变量的函数。

```python
# 基础闭包
def outer(x):
    def inner(y):
        return x + y
    return inner

closure = outer(10)
print(closure(5))  # 15

# 闭包的应用
def make_counter():
    count = 0
    def counter():
        nonlocal count  # 声明使用外层变量
        count += 1
        return count
    return counter

counter1 = make_counter()
counter2 = make_counter()

print(counter1())  # 1
print(counter1())  # 2
print(counter2())  # 1 (独立的计数器)
```

---

## 6. 生成器

生成器是惰性计算的函数，使用 yield 返回值。

### 6.1 生成器函数

```python
# 基本生成器
def count_up_to(n):
    count = 1
    while count <= n:
        yield count
        count += 1

for i in count_up_to(5):
    print(i)  # 1, 2, 3, 4, 5

# 生成器是迭代器
gen = count_up_to(3)
print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # 3
# next(gen)  # StopIteration
```

### 6.2 生成器表达式

```python
# 列表推导式
squares = [x**2 for x in range(1000000)]  # 创建完整列表

# 生成器表达式 (惰性)
squares_gen = (x**2 for x in range(1000000))  # 惰性生成
print(next(squares_gen))  # 0
print(next(squares_gen))  # 1

# 与函数配合
def process_large_file(filename):
    with open(filename) as f:
        for line in (l.strip() for l in f):  # 惰性处理每行
            yield process(line)
```

### 6.3 生成器链

```python
# 使用 yield from 委托生成
def gen1():
    yield 1
    yield 2

def gen2():
    yield 3
    yield 4

def combined():
    yield from gen1()
    yield from gen2()

list(combined())  # [1, 2, 3, 4]
```

---

## 7. 柯里化

柯里化是将多参数函数转换为一系列单参数函数的技术。

```python
# 普通函数
def add(a, b, c):
    return a + b + c

# 柯里化版本
def curry_add(a):
    def curry_b(b):
        def curry_c(c):
            return a + b + c
        return curry_c
    return curry_b

curry_add(1)(2)(3)  # 6

# 使用 functools.partial
from functools import partial

def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(5))  # 25
print(cube(5))    # 125
```

---

## 下一步

- [面向对象编程](../04-面向对象编程/README.md) — 学习 Python 的 OOP 特性
- [异步编程](../08-异步编程/README.md) — 深入 Python 的异步机制

---

*参考资料：[Python 函数](https://docs.python.org/3/tutorial/controlflow.html) | [functools 模块](https://docs.python.org/3/library/functools.html)*
