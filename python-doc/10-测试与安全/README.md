# 10. 测试与安全

## 概述

测试是确保代码质量的关键环节，安全编码则是保护应用的基础。本章介绍 Python 的测试框架和安全最佳实践。

---

## 1. 单元测试

### 1.1 unittest

```python
import unittest

class TestMathOperations(unittest.TestCase):
    def setUp(self):
        """每个测试前执行"""
        self.data = [1, 2, 3, 4, 5]
    
    def tearDown(self):
        """每个测试后执行"""
        pass
    
    def test_addition(self):
        self.assertEqual(1 + 1, 2)
        self.assertTrue(1 + 1 == 2)
    
    def test_list_operations(self):
        self.assertEqual(sum(self.data), 15)
        self.assertEqual(len(self.data), 5)
        self.assertIn(3, self.data)
    
    def test_raises(self):
        with self.assertRaises(ZeroDivisionError):
            1 / 0
    
    def test_almost_equal(self):
        self.assertAlmostEqual(0.1 + 0.2, 0.3, places=5)
    
    @unittest.skip("跳过此测试")
    def test_skipped(self):
        pass
    
    @unittest.expectedFailure
    def test_expected_fail(self):
        self.assertEqual(1, 2)

if __name__ == '__main__':
    unittest.main()
```

### 1.2 pytest

```bash
pip install pytest pytest-cov pytest-mock
```

```python
import pytest

# 基本测试
def test_basic():
    assert 1 + 1 == 2

# 使用 fixture
@pytest.fixture
def sample_data():
    return {"name": "test", "value": 42}

def test_with_fixture(sample_data):
    assert sample_data["value"] == 42

# 参数化测试
@pytest.mark.parametrize("input,expected", [
    (1, 1),
    (2, 4),
    (3, 9),
    (4, 16)
])
def test_square(input, expected):
    assert input ** 2 == expected

# 跳过和预期失败
@pytest.mark.skip(reason="Not implemented")
def test_not_implemented():
    pass

@pytest.mark.xfail
def test_expected_fail():
    assert 1 == 2

# 异常测试
def test_raises():
    with pytest.raises(ZeroDivisionError):
        1 / 0
    
    with pytest.raises(ValueError, match="must be positive"):
        raise ValueError("value must be positive")

# Mock
from unittest.mock import Mock, patch, MagicMock

def test_mock():
    mock = Mock()
    mock.method.return_value = 42
    assert mock.method() == 42

@patch('module.function')
def test_patch(mock_func):
    mock_func.return_value = "mocked"
    # 调用被 patch 的函数
    result = function()
    assert result == "mocked"
```

### 1.3 pytest 配置

```ini
# pytest.ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = -v --tb=short --strict-markers
markers =
    slow: marks tests as slow
    integration: marks tests as integration tests

# .coveragerc
[run]
source = src
omit = tests/*,venv/*
```

---

## 2. 测试覆盖率

```bash
pip install coverage
coverage run -m pytest
coverage report
coverage html
```

```python
# .coveragerc
[run]
source = mypackage
omit =
    */tests/*
    */venv/*

[report]
exclude_lines =
    pragma: no cover
    def __repr__
    raise NotImplementedError
    if __name__ == .__main__.:
```

---

## 3. 集成测试

### 3.1 HTTP 测试

```python
import pytest
from fastapi.testclient import TestClient

# FastAPI 测试
from main import app

client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}

def test_create_user():
    response = client.post(
        "/users",
        json={"name": "Alice", "email": "alice@example.com"}
    )
    assert response.status_code == 201
    assert response.json()["name"] == "Alice"
```

### 3.2 数据库测试

```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from base import Base

@pytest.fixture(scope="function")
def test_db():
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)
    TestSession = sessionmaker(bind=engine)
    session = TestSession()
    yield session
    session.close()
```

---

## 4. 安全编码

### 4.1 SQL 注入防护

```python
# ❌ 危险 - SQL 注入
query = f"SELECT * FROM users WHERE name = '{name}'"

# ✅ 安全 - 参数化查询
cursor.execute(
    "SELECT * FROM users WHERE name = %s",
    (name,)  # 参数元组
)

# SQLAlchemy ORM (自动防护)
user = session.query(User).filter(User.name == name).first()
```

### 4.2 XSS 防护

```python
# ❌ 危险
html = f"<div>{user_input}</div>"

# ✅ 安全 - 转义
import html
safe_html = html.escape(user_input)
html = f"<div>{safe_html}</div>"

# 在模板中 (Jinja2 自动转义)
# {{ user_input }}  # 自动转义
```

### 4.3 CSRF 防护

```python
# Flask-WTF
from flask_wtf import FlaskForm
from wtforms import StringField
from wtforms.validators import DataRequired

class MyForm(FlaskForm):
    name = StringField('Name', validators=[DataRequired()])

# 模板中使用
# <form method="POST">
#   {{ form.hidden_tag() }}
#   {{ form.name() }}
# </form>
```

### 4.4 密码存储

```python
import bcrypt

# 哈希密码
def hash_password(password: str) -> str:
    salt = bcrypt.gensalt()
    return bcrypt.hashpw(password.encode(), salt).decode()

# 验证密码
def verify_password(password: str, hashed: str) -> bool:
    return bcrypt.checkpw(
        password.encode(),
        hashed.encode()
    )

# 使用
hashed = hash_password("secret123")
assert verify_password("secret123", hashed)
assert not verify_password("wrong", hashed)
```

### 4.5 JWT 认证

```python
import jwt
from datetime import datetime, timedelta

SECRET_KEY = "your-secret-key"

def create_token(user_id: int) -> str:
    payload = {
        "user_id": user_id,
        "exp": datetime.utcnow() + timedelta(hours=24)
    }
    return jwt.encode(payload, SECRET_KEY, algorithm="HS256")

def verify_token(token: str) -> dict:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return payload
    except jwt.ExpiredSignatureError:
        raise ValueError("Token expired")
    except jwt.InvalidTokenError:
        raise ValueError("Invalid token")
```

---

## 5. 输入验证

### 5.1 Pydantic 验证

```python
from pydantic import BaseModel, validator, Field, EmailStr
from typing import Optional

class User(BaseModel):
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    age: Optional[int] = Field(None, ge=0, le=150)
    password: str = Field(..., min_length=8)
    
    @validator('username')
    def username_alphanumeric(cls, v):
        assert v.isalnum(), 'must be alphanumeric'
        return v
    
    @validator('password')
    def password_strength(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('must contain uppercase')
        if not any(c.islower() for c in v):
            raise ValueError('must contain lowercase')
        return v

# 使用
user = User(
    username="alice123",
    email="alice@example.com",
    age=25,
    password="SecurePass123"
)
```

---

## 6. 加密

### 6.1 hashlib

```python
import hashlib

# SHA-256 (推荐)
h = hashlib.sha256(b"data")
h.hexdigest()

# 带盐值
salt = b"random_salt"
h = hashlib.pbkdf2_hmac('sha256', b'password', salt, 100000)

# 文件哈希
with open("file.txt", "rb") as f:
    h = hashlib.file_digest(f, 'sha256')
```

### 6.2 cryptography 库

```python
from cryptography.fernet import Fernet

# 生成密钥
key = Fernet.generate_key()
cipher = Fernet(key)

# 加密
encrypted = cipher.encrypt(b"secret message")

# 解密
decrypted = cipher.decrypt(encrypted)
```

---

## 7. 安全工具

### 7.1 bandit (代码安全扫描)

```bash
pip install bandit
bandit -r myproject/
bandit -r myproject/ -f json -o report.json
```

### 7.2 safety (依赖安全检查)

```bash
pip install safety
safety check
safety check --json
```

---

## 下一步

- [性能优化](../11-性能优化/README.md) — 深入 Python 性能优化
- [最佳实践与设计模式](../12-最佳实践与设计模式/README.md) — 学习设计模式

---

*参考资料：[pytest 文档](https://docs.pytest.org/) | [OWASP](https://owasp.org/)*
