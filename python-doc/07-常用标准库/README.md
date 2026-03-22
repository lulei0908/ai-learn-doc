# 07. 常用标准库

## 概述

Python 标准库提供了丰富的功能，无需安装第三方包即可完成大部分任务。本章介绍最常用的标准库模块。

---

## 1. 系统与路径

### 1.1 os 模块

```python
import os

# 路径操作
os.getcwd()           # 当前工作目录
os.chdir('/path')     # 切换目录
os.listdir('.')       # 列出目录内容
os.mkdir('dir')       # 创建目录
os.makedirs('a/b/c') # 递归创建目录
os.rmdir('dir')       # 删除空目录
os.remove('file')     # 删除文件
os.rename('old', 'new')  # 重命名

# 文件和目录信息
os.path.exists('file')
os.path.isfile('file')
os.path.isdir('path')
os.path.getsize('file')
os.path.getmtime('file')

# 环境变量
os.environ['HOME']
os.getenv('PATH', 'default')  # 获取，可设置默认值
os.putenv('KEY', 'value')     # 设置环境变量

# 执行命令
os.system('ls -la')  # 返回退出码

# 路径拼接
os.path.join('dir', 'subdir', 'file.txt')
os.path.split('/path/to/file.txt')  # ('/path/to', 'file.txt')
os.path.splitext('file.txt')  # ('file', '.txt')
os.path.dirname('/path/to/file')  # '/path/to'
os.path.basename('/path/to/file')  # 'file'
```

### 1.2 pathlib 模块 (推荐)

```python
from pathlib import Path

p = Path('/home/user/project/file.txt')

# 属性
p.name      # 'file.txt'
p.stem      # 'file'
p.suffix    # '.txt'
p.parent    # PosixPath('/home/user/project')
p.parts     # ('/home', 'user', 'project', 'file.txt')

# 操作
p.exists()
p.is_file()
p.is_dir()
p.is_absolute()
p.resolve()

# 创建
Path('new_dir').mkdir()
Path('new/nested').mkdir(parents=True, exist_ok=True)

# 遍历
list(Path('.').glob('*.py'))          # 当前目录的 .py 文件
list(Path('.').rglob('**/*.py'))      # 递归所有 .py 文件
list(Path('.').glob('**/test_*.py')) # 递归匹配 test_ 开头的

# 读写文件
p.read_text()
p.write_text('content')
p.read_bytes()
p.write_bytes(b'content')

# 路径拼接
Path('dir') / 'subdir' / 'file.txt'
```

---

## 2. 数据处理

### 2.1 json 模块

```python
import json

# 序列化
data = {"name": "Alice", "age": 30, "skills": ["Python", "JS"]}

json_str = json.dumps(data, indent=2, ensure_ascii=False)
# {
#   "name": "Alice",
#   "age": 30,
#   "skills": ["Python", "JS"]
# }

# 序列化到文件
with open('data.json', 'w', encoding='utf-8') as f:
    json.dump(data, f, indent=2)

# 反序列化
parsed = json.loads(json_str)

# 从文件读取
with open('data.json', 'r', encoding='utf-8') as f:
    data = json.load(f)

# 自定义编码
class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

json.dumps(data, cls=CustomEncoder)

# 自定义解码
def custom_decoder(dct):
    if 'date' in dct:
        return datetime.fromisoformat(dct['date'])
    return dct

json.loads(json_str, object_hook=custom_decoder)
```

### 2.2 datetime 模块

```python
from datetime import datetime, date, time, timedelta

# 当前时间
now = datetime.now()
utc_now = datetime.utcnow()
timestamp = datetime.now().timestamp()

# 创建时间
dt = datetime(2024, 1, 15, 10, 30, 0)
d = date(2024, 1, 15)
t = time(10, 30, 0)

# 时间格式化
dt.strftime('%Y-%m-%d %H:%M:%S')  # '2024-01-15 10:30:00'
dt.isoformat()                     # '2024-01-15T10:30:00'

# 解析字符串
datetime.strptime('2024-01-15 10:30:00', '%Y-%m-%d %H:%M:%S')
datetime.fromisoformat('2024-01-15T10:30:00')

# 时间运算
delta = timedelta(days=7, hours=2)
new_dt = dt + delta
diff = dt2 - dt1

# 时区
from datetime import timezone
tz = timezone(timedelta(hours=8))
dt_aware = datetime(2024, 1, 15, 10, 30, tzinfo=tz)
```

### 2.3 collections 模块

```python
from collections import Counter, defaultdict, OrderedDict, deque, namedtuple

# Counter - 计数
words = ['apple', 'banana', 'apple', 'cherry', 'banana', 'apple']
counter = Counter(words)
# Counter({'apple': 3, 'banana': 2, 'cherry': 1})
counter.most_common(2)  # [('apple', 3), ('banana', 2)]

# defaultdict - 默认字典
d = defaultdict(list)
d['fruits'].append('apple')
d['fruits'].append('banana')
# defaultdict(<class 'list'>, {'fruits': ['apple', 'banana']})

# deque - 双端队列
dq = deque([1, 2, 3])
dq.append(4)        # [1, 2, 3, 4]
dq.appendleft(0)    # [0, 1, 2, 3, 4]
dq.pop()           # 返回 4
dq.popleft()       # 返回 0
dq.extend([5, 6])
dq.rotate(1)       # 右旋一位

# namedtuple - 命名元组
Point = namedtuple('Point', ['x', 'y'])
p = Point(10, 20)
print(p.x, p.y)  # 10 20

# OrderedDict - 有序字典 (Python 3.7+ dict 已有序)
```

---

## 3. 函数式编程工具

### 3.1 functools 模块

```python
from functools import reduce, lru_cache, partial, wraps, singledispatch

# reduce - 累积计算
from functools import reduce
result = reduce(lambda x, y: x + y, [1, 2, 3, 4, 5])  # 15

# lru_cache - 记忆化缓存
@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

fibonacci.cache_info()  # CacheInfo(hits=..., misses=..., ...)
fibonacci.cache_clear()

# partial - 偏函数
from functools import partial
def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)
square(5)  # 25
cube(5)    # 125

# singledispatch - 函数重载
@singledispatch
def process(data):
    print(f"Processing: {data}")

@process.register(int)
def _(data):
    print(f"Integer: {data * 2}")

@process.register(str)
def _(data):
    print(f"String: {data.upper()}")
```

### 3.2 itertools 模块

```python
import itertools

# count - 无限计数器
for i in itertools.count(1):
    if i > 5: break
    print(i)  # 1, 2, 3, 4, 5

# cycle - 无限循环
counter = 0
for item in itertools.cycle(['A', 'B']):
    if counter > 3: break
    print(item)  # A, B, A, B
    counter += 1

# repeat - 重复
list(itertools.repeat('x', 3))  # ['x', 'x', 'x']

# chain - 连接
list(itertools.chain([1, 2], [3, 4], [5, 6]))  # [1, 2, 3, 4, 5, 6]

# islice - 切片迭代器
list(itertools.islice(range(10), 2, 8, 2))  # [2, 4, 6]

# groupby - 分组
data = [('A', 1), ('B', 2), ('A', 3), ('B', 4)]
for key, group in itertools.groupby(data, key=lambda x: x[0]):
    print(key, list(group))

# product - 笛卡尔积
list(itertools.product([1, 2], ['a', 'b']))  # [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]

# permutations - 排列
list(itertools.permutations('ABC', 2))  # [('A', 'B'), ('A', 'C'), ('B', 'A'), ...]

# combinations - 组合
list(itertools.combinations('ABC', 2))  # [('A', 'B'), ('A', 'C'), ('B', 'C')]
```

---

## 4. 类型提示

### 4.1 typing 模块

```python
from typing import List, Dict, Set, Tuple, Optional, Union, Any, Callable, Type

# 基础类型提示
def greet(name: str) -> str:
    return f"Hello, {name}!"

# 容器类型
def process(items: list[int]) -> dict[str, int]:
    return {"sum": sum(items), "count": len(items)}

# Optional 和 Union
def parse(value: str | None) -> int | None:
    if value is None:
        return None
    return int(value)

def parse_strict(value: str) -> int:
    try:
        return int(value)
    except ValueError:
        raise ValueError(f"Cannot parse: {value}")

# Callable
def apply(func: Callable[[int], int], value: int) -> int:
    return func(value)

# Type
def create_instance(cls: Type) -> object:
    return cls()

# TypeVar
from typing import TypeVar
T = TypeVar('T')
def first(lst: list[T]) -> T | None:
    return lst[0] if lst else None

# Protocol (结构子类型)
from typing import Protocol
class Drawable(Protocol):
    def draw(self) -> None: ...

# Literal
from typing import Literal
def move(direction: Literal["up", "down", "left", "right"]) -> None: ...

# TypedDict
from typing import TypedDict
class UserDict(TypedDict):
    name: str
    age: int

user: UserDict = {"name": "Alice", "age": 30}
```

---

## 5. 文件操作

### 5.1 文件读写

```python
# 基本读写
with open('file.txt', 'r', encoding='utf-8') as f:
    content = f.read()

with open('file.txt', 'w', encoding='utf-8') as f:
    f.write('content')

# 逐行读取
with open('file.txt', 'r') as f:
    for line in f:
        print(line.strip())

# 读所有行
with open('file.txt', 'r') as f:
    lines = f.readlines()

# 上下文管理器
with open('input.txt') as infile, open('output.txt', 'w') as outfile:
    for line in infile:
        outfile.write(line)
```

### 5.2 shutil 模块

```python
import shutil

# 复制
shutil.copy('src.txt', 'dst.txt')           # 复制文件
shutil.copy2('src.txt', 'dst.txt')          # 保留元数据
shutil.copytree('dir/', 'new_dir/')         # 复制目录

# 移动
shutil.move('src/', 'dst/')

# 删除
shutil.rmtree('dir/')  # 删除目录树

# 压缩
shutil.make_archive('archive', 'zip', 'dir/')
shutil.unpack_archive('archive.zip', 'extract_dir/')

# 文件操作
shutil.get_terminal_size()
shutil.which('python')
```

---

## 6. 正则表达式

### 6.1 re 模块

```python
import re

# 基本匹配
pattern = r'\d+'  # 一个或多个数字
text = 'abc 123 def 456'

re.findall(pattern, text)    # ['123', '456']
re.search(pattern, text)     # <re.Match object; span=(4, 7), match='123'>
re.match(pattern, text)      # None (从开头匹配)
re.split(pattern, text)      # ['abc ', ' def ', '']

# 替换
re.sub(r'\d+', 'NUM', text)  # 'abc NUM def NUM'
re.subn(r'\d+', 'NUM', text) # ('abc NUM def NUM', 2)

# 编译 (提高性能)
pattern = re.compile(r'\d+')
pattern.findall(text)

# 分组
pattern = r'(\d+)-(\d+)'
match = re.search(pattern, '123-456')
match.group(0)  # '123-456'
match.group(1)  # '123'
match.group(2)  # '456'
match.groups()  # ('123', '456')

# 命名分组
pattern = r'(?P<area>\d+)-(?P<number>\d+)'
match = re.search(pattern, '123-456')
match.group('area')   # '123'
match.group('number') # '456'
match.groupdict()    # {'area': '123', 'number': '456'}

# 常用模式
r'\d'       # 数字
r'\D'       # 非数字
r'\w'       # 单词字符 [a-zA-Z0-9_]
r'\W'       # 非单词字符
r'\s'       # 空白字符
r'^'        # 字符串开始
r'$'        # 字符串结束
r'\b'       # 单词边界
r'*?'       # 非贪婪匹配
r'(?:...)'  # 非捕获组
```

---

## 7. 哈希与加密

### 7.1 hashlib 模块

```python
import hashlib

# MD5 (不安全，仅用于校验)
md5 = hashlib.md5()
md5.update(b'hello')
print(md5.hexdigest())  # 5d41402abc4b2a76b9719d911017c592

# SHA
sha256 = hashlib.sha256()
sha256.update(b'hello')
print(sha256.hexdigest())

# 带盐值
salt = b'random_salt'
h = hashlib.pbkdf2_hmac('sha256', b'password', salt, 100000)

# 文件哈希
with open('file.txt', 'rb') as f:
    h = hashlib.file_digest(f, 'sha256')
print(h.hexdigest())
```

---

## 8. 并发与多进程

### 8.1 concurrent.futures

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

# 线程池
with ThreadPoolExecutor(max_workers=4) as executor:
    future = executor.submit(pow, 2, 3)
    result = future.result()  # 8

# 多进程
def cpu_bound(n):
    return sum(i*i for i in range(n))

with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(cpu_bound, [1000000, 2000000, 3000000]))

# 异步回调
def done_callback(future):
    print(f"Result: {future.result()}")

executor.submit(pow, 2, 3).add_done_callback(done_callback)
```

---

## 下一步

- [异步编程](../08-异步编程/README.md) — 深入 Python 异步机制
- [数据库与网络](../09-数据库与网络/README.md) — 学习网络编程

---

*参考资料：[Python 标准库](https://docs.python.org/3/library/)* *
