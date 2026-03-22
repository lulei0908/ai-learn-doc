# 11. 性能优化

## 概述

Python 性能优化涉及多方面，从算法选择到语言特性，再到系统级优化。本章介绍 Python 性能优化的技巧和最佳实践。

---

## 1. 性能分析

### 1.1 timeit

```python
import timeit

# 测量函数执行时间
def slow_function():
    return sum(range(1000000))

t = timeit.timeit(slow_function, number=100)
print(f"平均时间: {t/100:.6f}s")

# 使用上下文
t = timeit.timeit(lambda: sum(range(100000)), number=10)

# 命令行用法
# python -m timeit "sum(range(100000))"
```

### 1.2 profile/cProfile

```python
import cProfile
import pstats

def my_function():
    # 复杂函数
    result = 0
    for i in range(100000):
        result += i
    return result

# 性能分析
pr = cProfile.Profile()
pr.enable()
my_function()
pr.disable()

# 输出报告
stats = pstats.Stats(pr)
stats.sort_stats('cumulative')  # 按累计时间排序
stats.print_stats()

# 保存到文件
stats.dump_stats('profile.prof')
```

### 1.3 memory_profiler

```python
from memory_profiler import profile

@profile(precision=4)  # 精度为小数点后4位
def memory_intensive():
    data = [i for i in range(1000000)]
    return sum(data)
```

---

## 2. 算法优化

### 2.1 选择合适的数据结构

```python
# 列表 vs 集合 (成员检查)
data_list = [i for i in range(100000)]
value_to_check = 99999

# ❌ O(n)
if value_to_check in data_list:
    print("Found")

# ✅ O(1)
data_set = set(data_list)
if value_to_check in data_set:
    print("Found")

# 字典 vs 列表 (查找)
data_dict = {str(i): i for i in range(100000)}
key_to_find = "99999"

# ❌ O(n) 遍历列表
for item in data_list:
    if item == 99999:
        break

# ✅ O(1) 字典查找
value = data_dict[key_to_find]
```

### 2.2 缓存技术

```python
from functools import lru_cache

# LRU 缓存
@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(100))  # 快速计算

# 使用缓存信息
fibonacci.cache_info()
fibonacci.cache_clear()

# 自定义缓存
cache = {}
def cached_function(n):
    if n in cache:
        return cache[n]
    result = expensive_operation(n)
    cache[n] = result
    return result
```

---

## 3. 内存优化

### 3.1 列表和生成器

```python
# ❌ 创建完整列表
result = []
for i in range(1000000):
    result.append(i * 2)
print(sum(result))

# ✅ 使用生成器 (惰性计算)
def generate_numbers(n):
    for i in range(n):
        yield i * 2

print(sum(generate_numbers(1000000)))

# 或者直接使用生成器表达式
result = sum(i * 2 for i in range(1000000))

# 内存对比
import sys
list_size = [i for i in range(1000000)]
gen_size = (i for i in range(1000000))
print(sys.getsizeof(list_size))  # 约 8MB
print(sys.getsizeof(gen_size))   # 约 112 bytes
```

### 3.2 迭代器 vs 列表

```python
from itertools import islice

# 大数据集处理
with open('large_file.txt', 'r') as f:
    for line in islice(f, 1000):  # 只读取前1000行
        print(line.strip())

# 文件迭代 (惰性读取)
with open('large_file.txt', 'r') as f:
    for line in f:  # 逐行读取，不加载整个文件
        process(line)
```

---

## 4. 循环优化

### 4.1 本地变量访问

```python
# ❌ 每次访问 self.attr
def slow_method(self):
    for i in range(1000000):
        self.result += i

# ✅ 本地变量
def fast_method(self):
    result = self.result
    for i in range(1000000):
        result += i
    self.result = result

# ❌ 重复访问 len()
for i in range(len(data)):
    item = data[i]

# ✅ 预先计算长度
length = len(data)
for i in range(length):
    item = data[i]
```

### 4.2 循环展开

```python
# ❌ 小循环开销大
for i in range(100):
    data[i] += 1

# ✅ 适当展开
for i in range(0, 100, 2):
    data[i] += 1
    data[i+1] += 1
```

---

## 5. I/O 优化

### 5.1 缓冲写入

```python
# ❌ 频繁写入
with open('output.txt', 'w') as f:
    for i in range(1000000):
        f.write(str(i) + '\n')

# ✅ 批量写入
with open('output.txt', 'w') as f:
    buffer = []
    for i in range(1000000):
        buffer.append(str(i) + '\n')
    f.write('\n'.join(buffer))

# ✅ 使用 writelines
with open('output.txt', 'w') as f:
    lines = [str(i) + '\n' for i in range(1000000)]
    f.writelines(lines)
```

### 5.2 异步 I/O

```python
import asyncio
import aiofiles

# 异步读写
async def read_file(filename):
    async with aiofiles.open(filename, 'r') as f:
        return await f.read()

async def write_file(filename, content):
    async with aiofiles.open(filename, 'w') as f:
        await f.write(content)
```

---

## 6. 并发优化

### 6.1 多进程 vs 多线程 vs 异步

```python
import concurrent.futures
import asyncio
import threading
import multiprocessing

# CPU 密集型 → 多进程
def cpu_bound(n):
    return sum(i*i for i in range(n))

with concurrent.futures.ProcessPoolExecutor() as executor:
    results = executor.map(cpu_bound, [1000000, 2000000])

# I/O 密集型 → 多线程/异步
def io_bound(url):
    import requests
    return requests.get(url).text

# 多线程
with concurrent.futures.ThreadPoolExecutor() as executor:
    results = executor.map(io_bound, ['url1', 'url2'])

# 异步
async def fetch_all(urls):
    import aiohttp
    async with aiohttp.ClientSession() as session:
        async def fetch(url):
            async with session.get(url) as resp:
                return await resp.text()
        return await asyncio.gather(*[fetch(url) for url in urls])
```

### 6.2 协程和异步

```python
import asyncio

async def task_with_cpu_bound(n):
    # 在线程池中运行 CPU 密集型操作
    loop = asyncio.get_running_loop()
    return await loop.run_in_executor(None, cpu_bound, n)

async def task_with_io_bound(url):
    # 原生异步 I/O
    import aiohttp
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            return await resp.text()

async def mixed_task():
    cpu_result = await task_with_cpu_bound(1000000)
    io_result = await task_with_io_bound('https://example.com')
    return cpu_result, io_result
```

---

## 7. NumPy 和科学计算

```python
import numpy as np

# 使用 NumPy 代替循环
data = np.random.randn(1000000)

# ❌ Python 循环
result = 0
for x in data:
    result += x

# ✅ NumPy 向量化操作
result = np.sum(data)

# 矩阵运算
matrix = np.random.randn(1000, 1000)
# ❌ Python 循环
for i in range(1000):
    for j in range(1000):
        matrix[i, j] *= 2

# ✅ NumPy 广播
matrix *= 2

# 使用专用函数
from scipy import optimize, linalg
from sklearn.ensemble import RandomForestClassifier
```

---

## 8. Python 扩展

### 8.1 Cython

```python
# .pyx 文件
def cython_function(n):
    cdef int result = 0
    for i in range(n):
        result += i
    return result

# 编译
# pip install cython
# cythonize -i mymodule.pyx
```

### 8.2 Numba

```python
from numba import njit

@njit
def numba_function(n):
    result = 0
    for i in range(n):
        result += i * i
    return result

# 快速执行
print(numba_function(1000000))
```

---

## 9. PyPy

PyPy 是 Python 的 JIT 编译器，对于某些代码可以显著提升性能。

```bash
# 安装 PyPy
pypy3 myscript.py  # 使用 PyPy 运行
```

---

## 10. 内存管理技巧

```python
# 使用内存映射文件
import mmap

with open('large_file.txt', 'rb') as f:
    with mmap.mmap(f.fileno(), 0, access=mmap.ACCESS_READ) as m:
        data = m.read(1000)

# 循环引用
import weakref

class MyClass:
    def __init__(self):
        self.self_ref = weakref.ref(self)

# 使用 __slots__
class EfficientClass:
    __slots__ = ['x', 'y']
    def __init__(self):
        self.x = 0
        self.y = 0

# 对比内存使用
class NormalClass:
    def __init__(self):
        self.x = 0
        self.y = 0

import sys
print(sys.getsizeof(NormalClass()))  # 约 56 bytes
print(sys.getsizeof(EfficientClass()))  # 约 32 bytes
```

---

## 11. 编译优化

### 11.1 CPython 优化模式

```bash
# -O 优化级别 (移除断言)
python -O myscript.py

# -OO 优化级别 (移除断言和 __debug__)
python -OO myscript.py

# -m py_compile (预编译)
python -m py_compile myscript.py
```

---

## 12. 代码优化

### 12.1 预计算

```python
# ❌ 每次重新计算
def expensive_transform(data):
    for item in data:
        transformed = expensive_function(item)
        # 处理 transformed

# ✅ 预计算结果
precomputed = {item: expensive_function(item) for item in data}
for item in data:
    transformed = precomputed[item]
```

### 12.2 避免重复创建对象

```python
# ❌ 重复创建字符串
for i in range(10000):
    s = "prefix_" + str(i)

# ✅ 重用模板
template = "prefix_{}"
for i in range(10000):
    s = template.format(i)

# ❌ 重复创建函数
def process(data):
    for item in data:
        def expensive_function(x):
            return x * x
        result = expensive_function(item)

# ✅ 提前定义函数
def expensive_function(x):
    return x * x

def process(data):
    for item in data:
        result = expensive_function(item)
```

---

## 下一步

- [最佳实践与设计模式](../12-最佳实践与设计模式/README.md) — 学习设计模式

---

*参考资料：[Python Performance Tips](https://wiki.python.org/moin/PythonSpeed)*