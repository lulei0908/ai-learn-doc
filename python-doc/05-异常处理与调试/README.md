# 05. 异常处理与调试

## 概述

Python 提供了完善的异常处理机制和丰富的调试工具，帮助开发者发现和处理程序中的错误。

---

## 1. 异常基础

### 1.1 异常概念

```python
# 常见异常类型
print(dir(__builtins__))  # 查看所有内置异常

# 常见异常
ZeroDivisionError    # 除数为零
TypeError           # 类型错误
ValueError          # 值错误
KeyError            # 字典键不存在
IndexError          # 索引越界
AttributeError      # 属性不存在
FileNotFoundError   # 文件不存在
ImportError         # 导入错误
NameError           # 名称未定义
SyntaxError         # 语法错误
RuntimeError        # 运行时错误
```

### 1.2 异常捕获

```python
# 基本 try-except
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")

# 捕获多个异常
try:
    value = int("abc")
    result = 10 / 0
except ValueError:
    print("Invalid value")
except ZeroDivisionError:
    print("Cannot divide by zero")

# 捕获异常对象
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")
    print(f"Type: {type(e)}")

# 捕获所有异常
try:
    result = 10 / 0
except Exception as e:
    print(f"Unexpected error: {e}")
except BaseException as e:  # 包括系统异常
    print(f"Base exception: {e}")
```

### 1.3 else 和 finally

```python
try:
    file = open("data.txt", "r")
    content = file.read()
except FileNotFoundError:
    print("File not found")
else:
    print(f"Read {len(content)} bytes")  # 仅在没有异常时执行
finally:
    file.close()  # 无论是否有异常都会执行
```

---

## 2. 抛出异常

### 2.1 raise 语句

```python
# 抛出异常
def validate_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    if age > 150:
        raise ValueError("Age is unrealistic")
    return age

try:
    validate_age(-5)
except ValueError as e:
    print(e)  # Age cannot be negative

# 重新抛出异常
try:
    # 一些操作
    pass
except Exception as e:
    print("Logging error...")
    raise  # 重新抛出原始异常

# 链式异常 (Python 3)
try:
    int("abc")
except Exception as e:
    raise ValueError("Conversion failed") from e
# Traceback:
#   File "...", line 3, in <module>
#     int("abc")
# ValueError: invalid literal for int() with base 10: 'abc'
# 
# The above exception was the direct cause of the following exception:
#   File "...", line 6, in <module>
#     raise ValueError("Conversion failed") from e
# ValueError: Conversion failed
```

### 2.2 自定义异常

```python
# 定义自定义异常
class ValidationError(Exception):
    """验证错误"""
    pass

class PositiveNumberError(ValidationError):
    """必须是正数"""
    def __init__(self, value):
        self.value = value
        super().__init__(f"Expected positive number, got {value}")

# 使用自定义异常
def square_root(n):
    if n < 0:
        raise PositiveNumberError(n)
    return n ** 0.5

try:
    square_root(-4)
except PositiveNumberError as e:
    print(e.value)  # -4
```

---

## 3. 异常最佳实践

### 3.1 异常层次结构

```python
# 建议的异常层次
class AppError(Exception):
    """应用基础异常"""
    pass

class ValidationError(AppError):
    """验证错误"""
    pass

class DataError(AppError):
    """数据处理错误"""
    pass

class DatabaseError(DataError):
    """数据库错误"""
    pass

class ConnectionError(DatabaseError):
    """连接错误"""
    pass

class TimeoutError(ConnectionError):
    """超时错误"""
    pass
```

### 3.2 EAFP vs LBYL

```python
# LBYL (Look Before You Leap) - 先检查
def access_dict_lbyl(d, key):
    if key in d:  # 先检查键是否存在
        return d[key]
    return None

# EAFP (Easier to Ask Forgiveness than Permission) - 请求宽恕
def access_dict_eafp(d, key):
    try:
        return d[key]
    except KeyError:
        return None

# 建议: 两者结合
# - 已知可能失败的场景用 try-except
# - 条件检查适用于复杂验证
```

---

## 4. 调试基础

### 4.1 print 调试

```python
def buggy_function(n):
    result = 0
    for i in range(n):
        print(f"i={i}, result={result}")  # 调试输出
        result += i
    return result
```

### 4.2 assert 断言

```python
# 基本断言
assert condition, "Error message"
assert x > 0, "x must be positive"

# 断言使用场景
# - 开发时检查不应发生的情况
# - 文档化代码假设
# - 不要用于可恢复的错误

def divide(a, b):
    assert b != 0, "Divisor cannot be zero"  # 假设检查
    return a / b

# 禁用断言 (生产环境)
# python -O script.py
```

### 4.3 日志模块

```python
import logging

# 配置日志
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('app.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)

logger.debug("Debug message")
logger.info("Info message")
logger.warning("Warning message")
logger.error("Error message")
logger.critical("Critical message")

# 日志级别
# DEBUG < INFO < WARNING < ERROR < CRITICAL
```

### 4.4 日志配置

```python
import logging
from logging.handlers import RotatingFileHandler

# 创建 logger
logger = logging.getLogger('myapp')
logger.setLevel(logging.DEBUG)

# 文件处理器 (自动轮转)
file_handler = RotatingFileHandler(
    'app.log',
    maxBytes=1024*1024,  # 1MB
    backupCount=5
)
file_handler.setLevel(logging.DEBUG)

# 控制台处理器
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)

# 格式化器
formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
file_handler.setFormatter(formatter)
console_handler.setFormatter(formatter)

logger.addHandler(file_handler)
logger.addHandler(console_handler)
```

---

## 5. pdb 调试器

### 5.1 基本命令

```python
import pdb

def debug_function(n):
    pdb.set_trace()  # 设置断点
    result = 1
    for i in range(1, n + 1):
        result *= i
    return result

# pdb 命令:
# n (next) - 执行下一行
# s (step) - 进入函数
# c (continue) - 继续执行到下一个断点
# p <expr> - 打印表达式
# pp <expr> - 格式化打印
# l (list) - 显示当前代码上下文
# w (where) - 显示调用栈
# u (up) - 向上移动栈帧
# d (down) - 向下移动栈帧
# q (quit) - 退出调试器
# b (break) - 设置断点
# tbreak - 临时断点
# cl (clear) - 清除断点
# r (return) - 运行到当前函数返回
```

### 5.2 启动调试

```python
# 方式 1: 命令行启动
# python -m pdb script.py

# 方式 2: 在代码中设置断点
import pdb
pdb.set_trace()

# 方式 3: post-mortem 调试
try:
    # 可能出错的代码
    result = 10 / 0
except:
    pdb.post_mortem()

# 方式 4: IPython / Jupyter 中的调试
# %debug  # 开启交互式调试器
```

---

## 6. IDE 调试

### 6.1 VS Code

```json
// .vscode/launch.json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Current File",
            "type": "python",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal"
        },
        {
            "name": "Python: Module",
            "type": "python",
            "request": "launch",
            "module": "mymodule",
            "args": ["--arg1", "value"],
            "console": "integratedTerminal"
        }
    ]
}
```

### 6.2 PyCharm

- 设置断点：点击代码行号左侧
- 调试运行：`Shift + F9`
- Step Over：`F8`
- Step Into：`F7`
- Step Out：`Shift + F8`
- Resume：`F9`

---

## 7. 单元测试

### 7.1 unittest 模块

```python
import unittest

class TestMathOperations(unittest.TestCase):
    def setUp(self):
        """每个测试方法前执行"""
        self.data = [1, 2, 3, 4, 5]
    
    def tearDown(self):
        """每个测试方法后执行"""
        pass
    
    def test_addition(self):
        self.assertEqual(1 + 1, 2)
    
    def test_list_sum(self):
        self.assertEqual(sum(self.data), 15)
    
    def test_raises_exception(self):
        with self.assertRaises(ZeroDivisionError):
            1 / 0
    
    def test_almost_equal(self):
        self.assertAlmostEqual(0.1 + 0.2, 0.3, places=5)

if __name__ == '__main__':
    unittest.main()
```

### 7.2 pytest

```bash
# 安装
pip install pytest

# 运行
pytest tests.py
pytest tests/ -v
pytest tests/ -k "test_name"
```

```python
# test_example.py
import pytest

def test_addition():
    assert 1 + 1 == 2

def test_list_operations():
    data = [1, 2, 3]
    assert sum(data) == 6
    assert len(data) == 3

@pytest.fixture
def sample_data():
    return {"name": "test", "value": 42}

def test_with_fixture(sample_data):
    assert sample_data["value"] == 42

def test_raises():
    with pytest.raises(ZeroDivisionError):
        1 / 0

# 参数化测试
@pytest.mark.parametrize("input,expected", [
    (1, 1),
    (2, 4),
    (3, 9),
])
def test_square(input, expected):
    assert input ** 2 == expected
```

---

## 下一步

- [模块与包管理](../06-模块与包管理/README.md) — 学习 Python 模块系统
- [常用标准库](../07-常用标准库/README.md) — 掌握标准库的使用

---

*参考资料：[Python 异常](https://docs.python.org/3/tutorial/errors.html) | [logging 模块](https://docs.python.org/3/library/logging.html) | [pdb 模块](https://docs.python.org/3/library/pdb.html)*
