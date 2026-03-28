# 02. 数据类型与结构

## 概述

Python 提供丰富的数据类型，包括基础类型（整数、浮点数、字符串、布尔值）和容器类型（列表、元组、字典、集合）。本章详细介绍这些数据类型的特性和用法。

---

## 1. 基础数据类型

### 1.1 整数 (int)

```python
# 整数表示
a = 10          # 十进制
b = 0b1010      # 二进制 (10)
c = 0o12        # 八进制 (10)
d = 0xA         # 十六进制 (10)

# 大整数支持
big_num = 10**100  # Python 自动处理大整数

# 常用方法
num = 12345
print(len(str(num)))  # 5 (数字位数)

# 进制转换
print(bin(10))    # '0b1010'
print(oct(10))    # '0o12'
print(hex(10))    # '0xa'
```

### 1.2 浮点数 (float)

```python
# 浮点数
pi = 3.14159
scientific = 1.5e10  # 科学计数法: 15000000000.0

# 精度问题
0.1 + 0.2  # 0.30000000000000004

# 解决精度问题
from decimal import Decimal
Decimal('0.1') + Decimal('0.2')  # Decimal('0.3')

# 常用函数
print(round(3.14159, 2))  # 3.14
print(abs(-3.14))         # 3.14

# 数学模块
import math
print(math.floor(3.7))  # 3
print(math.ceil(3.7))   # 4
print(math.sqrt(16))    # 4.0
```

### 1.3 复数 (complex)

```python
# 复数
c = 3 + 4j
print(c.real)  # 3.0
print(c.conjugate())  # (3-4j)
abs(c)         # 5.0 (模)
```

### 1.4 布尔值 (bool)

```python
# 布尔值
is_active = True
is_deleted = False

# 布尔转换
bool(1)      # True
bool(0)      # False
bool("")     # False
bool([])     # False
bool(None)   # False

# 逻辑运算
True and False  # False
True or False   # True
not True        # False
```

### 1.5 字符串 (str)

```python
# 字符串创建
s1 = 'hello'
s2 = "hello"
s3 = '''multiple
lines'''
s4 = """also multiple
lines"""

# f-string (格式化字符串)
name = "Python"
version = 3.11
print(f"Welcome to {name} {version}!")  # Welcome to Python 3.11!

# 常用操作
s = "Hello, World!"

# 长度
len(s)  # 13

# 索引和切片
s[0]       # 'H'
s[-1]      # '!'
s[0:5]     # 'Hello'
s[::2]     # 'Hlo ol!' (步长为2)
s[::-1]    # '!dlroW ,olleH' (反转)

# 字符串方法
s.upper()        # 'HELLO, WORLD!'
s.lower()        # 'hello, world!'
s.title()        # 'Hello, World!'
s.strip()        # 去除首尾空白
s.split(',')     # ['Hello', ' World!']
s.join(['a', 'b'])  # 'aHellob'

# 查找和替换
s.find('World')  # 7 (索引位置)
s.replace('World', 'Python')  # 'Hello, Python!'
s.count('l')    # 3

# 字符串格式化
"Hello, {}".format("Python")     # 'Hello, Python'
"{} + {} = {}".format(1, 2, 3)  # '1 + 2 = 3'

# 原始字符串 (不转义)
path = r"C:\Users\Name"  # 'C:\\Users\\Name'
```

---

## 2. 列表 (list)

### 2.1 创建和访问

```python
# 创建列表
fruits = ["apple", "banana", "cherry"]
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", True, 3.14]

# 访问元素
fruits[0]     # 'apple'
fruits[-1]    # 'cherry'

# 切片
numbers[1:4]    # [2, 3, 4]
numbers[::2]    # [1, 3, 5] (偶数位)
```

### 2.2 修改列表

```python
fruits = ["apple", "banana", "cherry"]

# 添加元素
fruits.append("orange")      # 末尾添加
fruits.insert(1, "mango")     # 指定位置插入
fruits.extend(["grape", "kiwi"])  # 扩展列表

# 删除元素
fruits.remove("banana")        # 删除指定值
fruits.pop()                  # 删除并返回末尾元素
fruits.pop(0)                # 删除指定位置
del fruits[0]                # 删除元素
fruits.clear()                # 清空列表

# 修改元素
fruits[0] = "strawberry"
```

### 2.3 列表操作

```python
numbers = [3, 1, 4, 1, 5, 9, 2, 6]

# 排序
numbers.sort()          # 原地排序
sorted_numbers = sorted(numbers)  # 返回新列表
numbers.sort(reverse=True)  # 降序

# 反转
numbers.reverse()      # 原地反转
reversed_numbers = numbers[::-1]

# 列表推导式
squares = [x**2 for x in range(10)]  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# 条件过滤
even = [x for x in range(10) if x % 2 == 0]  # [0, 2, 4, 6, 8]

# 嵌套列表推导式
matrix = [[i*3+j for j in range(3)] for i in range(3)]
# [[0, 1, 2], [3, 4, 5], [6, 7, 8]]
```

### 2.4 常用函数

```python
numbers = [1, 2, 3, 4, 5]

len(numbers)     # 5 (长度)
max(numbers)     # 5 (最大值)
min(numbers)     # 1 (最小值)
sum(numbers)     # 15 (求和)
1 in numbers    # True (成员检查)
numbers.count(2)  # 1 (计数)
numbers.index(3)  # 2 (查找索引)
```

---

## 3. 元组 (tuple)

### 3.1 创建和使用

```python
# 创建元组
point = (10, 20)
single = (42,)  # 逗号结尾表示单元素元组
empty = ()

# 访问元素
point[0]    # 10
point[-1]   # 20

# 元组解包
x, y = point
a, b, c = (1, 2, 3)

# 交换变量
a, b = b, a

# 函数返回多值
def get_stats(numbers):
    return min(numbers), max(numbers), sum(numbers)/len(numbers)

min_val, max_val, avg_val = get_stats([1, 2, 3])
```

### 3.2 元组 vs 列表

| 特性 | 元组 | 列表 |
|:---|:---|:---|
| 可变性 | 不可变 | 可变 |
| 语法 | (1, 2, 3) | [1, 2, 3] |
| 性能 | 更快 | 稍慢 |
| 用途 | 固定数据、函数返回值 | 动态数据 |
| 哈希 | 可哈希(不可变时) | 不可哈希 |

---

## 4. 字典 (dict)

### 4.1 创建和访问

```python
# 创建字典
person = {
    "name": "Alice",
    "age": 30,
    "city": "Beijing"
}

# 访问值
person["name"]      # 'Alice'
person.get("name")  # 'Alice'
person.get("gender", "Unknown")  # 'Unknown' (默认值)

# 添加/修改
person["email"] = "alice@example.com"
person["age"] = 31
```

### 4.2 字典操作

```python
person = {"name": "Alice", "age": 30, "city": "Beijing"}

# 删除
del person["age"]
age = person.pop("age")

# 查看
person.keys()     # dict_keys(['name', 'city'])
person.values()   # dict_values(['Alice', 'Beijing'])
person.items()    # dict_items([('name', 'Alice'), ('city', 'Beijing')])

# 合并
d1 = {"a": 1, "b": 2}
d2 = {"b": 3, "c": 4}
d1.update(d2)  # {'a': 1, 'b': 3, 'c': 4}

# 字典推导式
squares = {x: x**2 for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

### 4.3 defaultdict

```python
from collections import defaultdict

# 自动创建默认值
dd = defaultdict(list)
dd["fruits"].append("apple")
dd["fruits"].append("banana")
# dd = {'fruits': ['apple', 'banana']}
```

---

## 5. 集合 (set)

### 5.1 创建和操作

```python
# 创建集合
fruits = {"apple", "banana", "cherry"}
numbers = {1, 2, 3, 2, 1}  # 自动去重: {1, 2, 3}

# 添加元素
fruits.add("orange")
fruits.update(["grape", "kiwi"])

# 删除元素
fruits.remove("banana")      # 不存在会报错
fruits.discard("banana")     # 不存在不报错
fruits.pop()                 # 随机删除一个

# 集合运算
set1 = {1, 2, 3, 4}
set2 = {3, 4, 5, 6}

set1 | set2      # 并集: {1, 2, 3, 4, 5, 6}
set1 & set2      # 交集: {3, 4}
set1 - set2      # 差集: {1, 2}
set1 ^ set2      # 对称差集: {1, 2, 5, 6}

# 子集和超集
{1, 2}.issubset({1, 2, 3})  # True
{1, 2, 3}.issuperset({1, 2})  # True
```

---

## 6. 类型注解

### 6.1 基础类型注解

```python
# 变量类型注解
name: str = "Alice"
age: int = 30
height: float = 1.65
is_active: bool = True

# 容器类型注解
numbers: list[int] = [1, 2, 3]
person: dict[str, str] = {"name": "Alice"}
coords: tuple[int, int] = (10, 20)
ids: set[int] = {1, 2, 3}

# 可选类型
name: str | None = None
from typing import Optional
name: Optional[str] = None

# 类型别名
Point = tuple[float, float]
Vector = list[float]
```

### 6.2 函数类型注解

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"

def process(items: list[int]) -> dict[str, int]:
    return {
        "sum": sum(items),
        "count": len(items)
    }

# 多类型
def parse(value: str | int) -> int:
    return int(value)
```

---

## 下一步

- [函数式编程](../03-函数式编程/README.md) — 学习 Python 的函数特性
- [面向对象编程](../04-面向对象编程/README.md) — 深入 Python 的 OOP 特性

---

*参考资料：[Python 数据类型](https://docs.python.org/3/library/stdtypes.html)*
