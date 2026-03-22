# 01. Python 基础

## 概述

Python 是一种高级、解释型、通用的编程语言，以其简洁的语法和强大的生态系统闻名。本章介绍 Python 的基础知识和环境配置。

---

## 1. Python 简介

### 1.1 语言特点

| 特点 | 说明 |
|:---|:---|
| **简洁易读** | 语法简洁，代码可读性强 |
| **跨平台** | 支持 Windows、Linux、macOS |
| **解释执行** | 无需编译，逐行解释执行 |
| **动态类型** | 变量类型在运行时确定 |
| **自动内存管理** | 垃圾自动回收 |
| **丰富的库** | 拥有庞大的第三方库生态 |

### 1.2 Python 版本

目前主流使用 **Python 3.x**，推荐使用 Python 3.10+ 版本。

```python
# 查看版本
import sys
print(sys.version)  # 3.11.x
```

---

## 2. 环境搭建

### 2.1 安装 Python

#### macOS (Homebrew)
```bash
brew install python@3.11
```

#### Ubuntu/Debian
```bash
sudo apt update
sudo apt install python3.11 python3.11-venv python3.11-dev
```

#### Windows
从 [python.org](https://www.python.org/downloads/) 下载安装包安装。

### 2.2 验证安装

```bash
python --version
# Python 3.11.x

python -c "print('Hello, Python!')"
# Hello, Python!
```

---

## 3. 虚拟环境

### 3.1 venv (内置)

```bash
# 创建虚拟环境
python -m venv .venv

# 激活虚拟环境
source .venv/bin/activate  # macOS/Linux
.venv\Scripts\activate     # Windows

# 退出虚拟环境
deactivate
```

### 3.2 pip 包管理

```bash
# 安装包
pip install requests

# 安装指定版本
pip install requests==2.28.0

# 从 requirements.txt 安装
pip install -r requirements.txt

# 查看已安装包
pip list

# 导出依赖
pip freeze > requirements.txt

# 卸载包
pip uninstall requests
```

### 3.3 Poetry (现代化工具)

```bash
# 安装 Poetry
curl -sSL https://install.python-poetry.org | python3 -

# 创建项目
poetry new myproject
cd myproject

# 初始化现有项目
poetry init

# 安装依赖
poetry install

# 添加依赖
poetry add requests
poetry add pytest --group dev

# 激活虚拟环境
poetry shell
```

---

## 4. 运行 Python 代码

### 4.1 交互式解释器 (REPL)

```bash
python
>>> print("Hello, World!")
Hello, World!
>>> 2 + 3
5
>>> exit()  # 退出
```

### 4.2 脚本文件

```python
# hello.py
def greet(name):
    return f"Hello, {name}!"

if __name__ == "__main__":
    print(greet("Python"))
```

```bash
python hello.py
# Hello, Python!
```

### 4.3 Jupyter Notebook

```bash
pip install jupyter
jupyter notebook
```

---

## 5. 基本语法

### 5.1 注释

```python
# 单行注释

"""
多行注释
可以使用三引号字符串
"""

'''
也可以使用单引号
多行注释
'''
```

### 5.2 变量

```python
# 变量无需声明类型
name = "Python"
version = 3.11
is_popular = True

# 多变量赋值
a, b, c = 1, 2, 3

# 交换变量
a, b = b, a

# 变量命名规范
# - 使用小写字母和下划线
# - 不能以数字开头
# - 避免使用关键字
my_variable = 10
_private = 20
```

### 5.3 输入输出

```python
# 输出
print("Hello, World!")
print("Hello", "Python", sep=", ", end="!\n")
print(f"Version: {version}")  # f-string 格式化

# 输入
name = input("Enter your name: ")
age = int(input("Enter your age: "))  # 需要转换类型
```

### 5.4 代码缩进

Python 使用缩进表示代码块，通常使用 4 个空格：

```python
if True:
    print("Indentation matters!")  # 正确
    if True:
        print("Nested block")      # 嵌套块
print("Back to main block")
```

---

## 6. 运算符

### 6.1 算术运算符

```python
+   # 加法: 2 + 3 = 5
-   # 减法: 5 - 2 = 3
*   # 乘法: 2 * 3 = 6
/   # 除法: 6 / 2 = 3.0 (浮点)
//  # 整除: 7 // 2 = 3
%   # 取模: 7 % 2 = 1
**  # 幂运算: 2 ** 3 = 8
```

### 6.2 比较运算符

```python
==  # 等于
!=  # 不等于
<   # 小于
>   # 大于
<=  # 小于等于
>=  # 大于等于
is  # 身份比较 (同一对象)
is not  # 非同一对象
```

### 6.3 逻辑运算符

```python
and  # 与: True and False = False
or   # 或: True or False = True
not  # 非: not True = False
```

### 6.4 位运算符

```python
&   # 按位与
|   # 按位或
^   # 按位异或
~   # 按位取反
<<  # 左移
>>  # 右移
```

### 6.5 赋值运算符

```python
=    # 赋值
+=   # a += b 等于 a = a + b
-=   # a -= b 等于 a = a - b
*=   # a *= b 等于 a = a * b
/=   # a /= b 等于 a = a / b
//=  # 整除赋值
%=   # 取模赋值
**=  # 幂赋值
```

---

## 7. 控制流

### 7.1 条件语句

```python
# if-elif-else
score = 85

if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
else:
    grade = "F"

# 三元表达式
status = "pass" if score >= 60 else "fail"

# 链式比较
if 80 <= score < 90:
    print("Good!")
```

### 7.2 循环语句

```python
# for 循环
for i in range(5):
    print(i)  # 0, 1, 2, 3, 4

for i in range(1, 10, 2):  # 开始, 结束, 步长
    print(i)  # 1, 3, 5, 7, 9

# 遍历序列
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

# 带索引遍历
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")

# while 循环
count = 0
while count < 5:
    print(count)
    count += 1

# break 和 continue
for i in range(10):
    if i == 3:
        continue  # 跳过本次迭代
    if i == 7:
        break     # 退出循环
    print(i)

# for-else / while-else
for i in range(5):
    if i == 10:
        break
else:
    print("Loop completed without break")
```

---

## 8. 类型系统基础

### 8.1 类型检查

```python
# type() 函数
print(type(42))        # <class 'int'>
print(type(3.14))      # <class 'float'>
print(type("hello"))   # <class 'str'>
print(type([1, 2]))    # <class 'list'>

# isinstance() 函数
print(isinstance(42, int))       # True
print(isinstance(42, (int, str)) # True
```

### 8.2 类型转换

```python
# 数值转换
int("42")       # 42
float("3.14")   # 3.14
str(42)         # "42"

# 布尔转换
bool(0)         # False
bool("")        # False
bool([])        # False
bool(1)         # True
bool("hello")   # True
```

---

## 9. 最佳实践

### 9.1 编码规范 (PEP 8)

```python
# 使用 4 空格缩进
# 使用 UTF-8 编码
# 行长度不超过 79 字符

# 变量命名
my_variable = 1           # 小写 + 下划线 (变量)
MyClass = type            # 大驼峰 (类)
MY_CONSTANT = 100         # 全大写 (常量)
_private_variable = 10    # 单下划线开头 (私有)

# 函数命名
def calculate_sum(a, b):
    return a + b

# 类命名
class MyClassName:
    pass

# 导入顺序
import os           # 标准库
import sys

import numpy as np  # 第三方库
import requests

import mymodule     # 本地模块
```

### 9.2 Main 函数入口

```python
def main():
    """主函数"""
    print("Running main function...")

if __name__ == "__main__":
    main()
```

---

## 下一步

- [数据类型与结构](../02-数据类型与结构/README.md) — 深入学习 Python 的数据类型

---

*参考资料：[Python 官方文档](https://docs.python.org/3/) | [PEP 8 风格指南](https://peps.python.org/pep-0008/)*
