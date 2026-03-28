# 06. 模块与包管理

## 概述

Python 的模块系统是代码组织和复用的基础。本章介绍模块导入机制、包结构、虚拟环境管理以及包管理工具。

---

## 1. 模块基础

### 1.1 模块导入

```python
# 导入整个模块
import math
print(math.sqrt(16))  # 4.0

# 导入特定函数
from math import sqrt, pi
print(sqrt(16))  # 4.0
print(pi)        # 3.14159...

# 导入并重命名
import numpy as np
from math import sqrt as square_root

# 导入所有 (不推荐)
from math import *

# 相对导入 (在包内使用)
from . import sibling_module
from .. import parent_module
from .module import function
```

### 1.2 模块搜索路径

```python
import sys

# 查看模块搜索路径
print(sys.path)
# ['', '/usr/lib/python3.11', ...]

# 添加搜索路径
sys.path.append('/path/to/modules')

# 模块位置
import math
print(math.__file__)  # 模块文件路径
```

### 1.3 创建模块

```python
# mymodule.py
"""我的模块文档字符串"""

# 模块级变量
MY_CONSTANT = 42

# 函数
def greet(name):
    return f"Hello, {name}!"

# 类
class MyClass:
    pass

# 私有成员 (约定)
_private_function = lambda x: x * 2

# 模块测试代码
if __name__ == "__main__":
    print(greet("World"))
```

---

## 2. 包结构

### 2.1 包目录结构

```
mypackage/
├── __init__.py          # 包初始化文件 (可以为空)
├── __main__.py          # python -m mypackage 运行
├── module1.py
├── module2.py
├── subpackage/
│   ├── __init__.py
│   └── module3.py
└── data/
    └── resource.txt
```

### 2.2 __init__.py

```python
# mypackage/__init__.py
"""包的文档字符串"""

# 包级变量
__version__ = "1.0.0"
__author__ = "Developer"

# 暴露的公共 API
from .module1 import function1, Class1
from .module2 import function2

__all__ = ["function1", "Class1", "function2"]
```

### 2.3 相对导入 vs 绝对导入

```python
# 在 mypackage/subpackage/module3.py 中

# 绝对导入
from mypackage.module1 import function1

# 相对导入
from ..module1 import function1    # 向上一级
from . import sibling_module       # 当前包
from ..subpackage import module3   # 跨子包
```

---

## 3. 包管理工具

### 3.1 pip

```bash
# 安装包
pip install requests
pip install requests==2.28.0
pip install requests>=2.28.0,<3.0.0

# 从 PyPI 安装
pip install package-name

# 从本地安装
pip install ./package
pip install -e ./package  # 可编辑模式

# 从 Git 安装
pip install git+https://github.com/user/repo.git
pip install git+https://github.com/user/repo.git@v1.0.0

# 卸载包
pip uninstall requests

# 查看已安装包
pip list
pip show requests

# 冻结依赖
pip freeze > requirements.txt

# 安装依赖
pip install -r requirements.txt

# 升级包
pip install --upgrade requests
```

### 3.2 requirements.txt

```text
# requirements.txt
requests>=2.28.0
numpy>=1.20.0
pandas>=1.5.0

# 开发依赖 (通常分开)
pytest>=7.0.0
black>=22.0.0
mypy>=0.990
```

### 3.3 setup.py / pyproject.toml

```python
# setup.py (传统方式)
from setuptools import setup, find_packages

setup(
    name="mypackage",
    version="1.0.0",
    description="My Python Package",
    author="Developer",
    packages=find_packages(),
    install_requires=[
        "requests>=2.28.0",
        "numpy>=1.20.0",
    ],
    extras_require={
        "dev": ["pytest>=7.0.0", "black>=22.0.0"],
    },
)
```

```toml
# pyproject.toml (现代方式)
[build-system]
requires = ["setuptools>=45", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "mypackage"
version = "1.0.0"
description = "My Python Package"
authors = [
    {name = "Developer", email = "dev@example.com"}
]
requires-python = ">=3.10"
dependencies = [
    "requests>=2.28.0",
    "numpy>=1.20.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "black>=22.0.0",
    "mypy>=0.990",
]

[project.scripts]
my-cli = "mypackage.cli:main"
```

---

## 4. 虚拟环境

### 4.1 venv (内置)

```bash
# 创建虚拟环境
python -m venv .venv
python -m venv .venv --python=python3.11

# 激活虚拟环境
source .venv/bin/activate  # macOS/Linux
.venv\Scripts\activate     # Windows

# 退出虚拟环境
deactivate

# 从虚拟环境运行
.venv/bin/python script.py
```

### 4.2 virtualenv

```bash
# 安装
pip install virtualenv

# 创建虚拟环境
virtualenv .venv
virtualenv .venv --python=python3.11

# 激活和退出同 venv
```

### 4.3 conda

```bash
# 创建环境
conda create -n myenv python=3.11
conda create -n myenv python=3.11 numpy pandas

# 激活环境
conda activate myenv

# 退出环境
conda deactivate

# 列出所有环境
conda env list

# 删除环境
conda env remove -n myenv

# 导出环境
conda env export > environment.yml

# 从文件创建环境
conda env create -f environment.yml
```

### 4.4 Poetry

```bash
# 安装
curl -sSL https://install.python-poetry.org | python3 -

# 创建新项目
poetry new myproject
cd myproject

# 初始化现有项目
poetry init

# 安装依赖
poetry install

# 添加依赖
poetry add requests
poetry add pytest --group dev

# 移除依赖
poetry remove requests

# 激活虚拟环境
poetry shell

# 运行命令
poetry run python script.py
poetry run pytest

# 构建和发布
poetry build
poetry publish
```

---

## 5. 项目结构最佳实践

### 5.1 推荐结构

```
myproject/
├── pyproject.toml
├── README.md
├── LICENSE
├── .gitignore
├── .python-version       # pyenv 版本文件
├── src/
│   └── mypackage/
│       ├── __init__.py
│       ├── core.py
│       ├── utils.py
│       └── cli.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   └── test_core.py
└── docs/
    └── index.md
```

### 5.2 pyproject.toml 配置

```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "mypackage"
version = "1.0.0"
description = "My Python Package"
readme = "README.md"
license = {text = "MIT"}
requires-python = ">=3.10"
dependencies = [
    "requests>=2.28.0",
]

[project.optional-dependencies]
dev = ["pytest>=7.0", "black>=22.0", "mypy>=0.990"]
docs = ["sphinx>=5.0", "sphinx-rtd-theme"]

[project.urls]
Homepage = "https://github.com/user/mypackage"
Documentation = "https://mypackage.readthedocs.io"

[project.scripts]
my-cli = "mypackage.cli:main"

[tool.setuptools.packages.find]
where = ["src"]

[tool.pytest.ini_options]
testpaths = ["tests"]

[tool.black]
line-length = 88
target-version = ["py310"]

[tool.mypy]
python_version = "3.10"
strict = true

[tool.ruff]
line-length = 88
select = ["E", "F", "W"]
```

---

## 6. 私有仓库

### 6.1 使用私有 PyPI

```bash
# 配置 ~/.pypirc
[distutils]
index-servers =
    pypi
    private

[pypi]
username = __token__
password = pypi-...

[private]
repository = https://private.pypi.org/simple/
username = __token__
password = private-token

# 上传到私有仓库
python -m twine upload --repository private dist/*

# 从私有仓库安装
pip install --index-url https://private.pypi.org/simple/ package
```

### 6.2 使用 .netrc

```text
# ~/.netrc
machine private.pypi.org
login __token__
password your-token
```

---

## 下一步

- [常用标准库](../07-常用标准库/README.md) — 学习 Python 内置库的使用
- [异步编程](../08-异步编程/README.md) — 深入 Python 异步机制

---

*参考资料：[Python 模块](https://docs.python.org/3/tutorial/modules.html) | [pip 文档](https://pip.pypa.io/) | [Poetry 文档](https://python-poetry.org/)*
