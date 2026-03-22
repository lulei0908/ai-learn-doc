# 09. 数据库与网络

## 概述

Python 提供了丰富的库来连接数据库和进行网络通信。本章介绍关系型数据库、NoSQL 数据库、HTTP 客户端以及 API 开发。

---

## 1. 关系型数据库

### 1.1 sqlite3 (内置)

```python
import sqlite3

# 连接数据库 (文件不存在会自动创建)
conn = sqlite3.connect('example.db')
cursor = conn.cursor()

# 创建表
cursor.execute('''
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
''')

# 插入数据
cursor.execute(
    'INSERT INTO users (name, email) VALUES (?, ?)',
    ('Alice', 'alice@example.com')
)

# 插入多条 (executemany)
users = [
    ('Bob', 'bob@example.com'),
    ('Charlie', 'charlie@example.com')
]
cursor.executemany(
    'INSERT INTO users (name, email) VALUES (?, ?)',
    users
)

# 查询
cursor.execute('SELECT * FROM users')
rows = cursor.fetchall()
for row in rows:
    print(row)

# 带参数查询
cursor.execute('SELECT * FROM users WHERE name = ?', ('Alice',))
print(cursor.fetchone())

# 提交更改
conn.commit()

# 关闭连接
conn.close()

# 使用上下文管理器
with sqlite3.connect('example.db') as conn:
    cursor = conn.cursor()
    cursor.execute('SELECT * FROM users')
    users = cursor.fetchall()
```

### 1.2 SQLAlchemy (ORM)

```python
from sqlalchemy import create_engine, Column, Integer, String, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime

# 创建引擎
engine = create_engine('sqlite:///example.db', echo=True)

# 基类
Base = declarative_base()

# 定义模型
class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(50), nullable=False)
    email = Column(String(100), unique=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    def __repr__(self):
        return f"<User(name='{self.name}', email='{self.email}')>"

# 创建表
Base.metadata.create_all(engine)

# 创建会话
Session = sessionmaker(bind=engine)
session = Session()

# 创建
user = User(name='Alice', email='alice@example.com')
session.add(user)
session.commit()

# 查询
users = session.query(User).all()
alice = session.query(User).filter_by(name='Alice').first()
users = session.query(User).filter(User.name.like('%li%')).all()

# 更新
alice.email = 'new_email@example.com'
session.commit()

# 删除
session.delete(alice)
session.commit()

# 关系
class Post(Base):
    __tablename__ = 'posts'
    id = Column(Integer, primary_key=True)
    title = Column(String(100))
    user_id = Column(Integer, ForeignKey('users.id'))
    user = relationship('User', backref='posts')

# 关闭
session.close()
```

---

## 2. PostgreSQL (psycopg2)

```python
import psycopg2
from contextlib import contextmanager

# 连接
conn = psycopg2.connect(
    host='localhost',
    port=5432,
    database='mydb',
    user='user',
    password='password'
)

# 使用上下文管理器
@contextmanager
def get_cursor():
    conn = psycopg2.connect(...)
    try:
        cursor = conn.cursor()
        yield cursor
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        cursor.close()
        conn.close()

# 基本操作
with get_cursor() as cursor:
    cursor.execute('SELECT * FROM users')
    for row in cursor:
        print(row)
    
    # 参数化查询 (防止 SQL 注入)
    cursor.execute(
        'INSERT INTO users (name, email) VALUES (%s, %s)',
        ('Alice', 'alice@example.com')
    )
    
    # 批量插入
    data = [('Bob', 'bob@example.com'), ('Charlie', 'charlie@example.com')]
    from psycopg2.extras import execute_batch
    execute_batch(cursor,
        'INSERT INTO users (name, email) VALUES (%s, %s)',
        data
    )
```

---

## 3. Redis

### 3.1 redis-py

```python
import redis

# 连接
r = redis.Redis(host='localhost', port=6379, db=0)

# 字符串操作
r.set('name', 'Alice')
r.setex('token', 3600, 'abc123')  # 过期时间 1 小时
r.get('name')  # b'Alice'

# 批量操作
r.mset({'key1': 'value1', 'key2': 'value2'})
r.mget(['key1', 'key2'])  # [b'value1', b'value2']

# 计数器
r.set('counter', 0)
r.incr('counter')  # 1
r.incrby('counter', 5)  # 6
r.decr('counter')  # 5

# 哈希
r.hset('user:1', mapping={'name': 'Alice', 'email': 'alice@example.com'})
r.hget('user:1', 'name')  # b'Alice'
r.hgetall('user:1')  # {b'name': b'Alice', b'email': b'alice@example.com'}

# 列表
r.lpush('queue', 'task1', 'task2')  # [b'task2', b'task1']
r.rpush('queue', 'task3')
r.lrange('queue', 0, -1)  # [b'task2', b'task1', b'task3']
r.lpop('queue')  # b'task2'

# 集合
r.sadd('tags', 'python', 'redis', 'database')
r.smembers('tags')
r.sismember('tags', 'python')

# 有序集合
r.zadd('leaderboard', {'Alice': 100, 'Bob': 90})
r.zrevrange('leaderboard', 0, -1, withscores=True)

# 过期
r.expire('name', 60)  # 60 秒后过期
r.ttl('name')  # 剩余 TTL

# 事务
pipe = r.pipeline()
pipe.set('a', 1)
pipe.incr('a')
pipe.execute()

# 发布/订阅
pubsub = r.pubsub()
pubsub.subscribe('channel')
for message in pubsub.listen():
    if message['type'] == 'message':
        print(message['data'])
```

---

## 4. MongoDB

### 4.1 pymongo

```python
from pymongo import MongoClient

# 连接
client = MongoClient('mongodb://localhost:27017/')
db = client['mydb']

# 插入文档
result = db.users.insert_one({
    'name': 'Alice',
    'email': 'alice@example.com',
    'age': 30
})
print(result.inserted_id)

# 插入多条
users = [
    {'name': 'Bob', 'email': 'bob@example.com'},
    {'name': 'Charlie', 'email': 'charlie@example.com'}
]
result = db.users.insert_many(users)

# 查询
user = db.users.find_one({'name': 'Alice'})
all_users = db.users.find({'age': {'$gte': 18}})

# 更新
db.users.update_one(
    {'name': 'Alice'},
    {'$set': {'email': 'new_email@example.com'}}
)
db.users.update_many(
    {'age': {'$lt': 18}},
    {'$set': {'status': 'minor'}}
)

# 删除
db.users.delete_one({'name': 'Bob'})
db.users.delete_many({'status': 'inactive'})

# 聚合
pipeline = [
    {'$group': {'_id': '$department', 'count': {'$sum': 1}}}
]
result = db.users.aggregate(pipeline)

# 索引
db.users.create_index([('email', 1)], unique=True)
```

---

## 5. HTTP 客户端

### 5.1 requests

```python
import requests

# GET 请求
response = requests.get('https://api.example.com/data')
print(response.status_code)
print(response.json())
print(response.text)

# 带参数
params = {'key': 'value', 'page': 1}
response = requests.get('https://api.example.com/data', params=params)

# POST 请求
data = {'username': 'alice', 'password': 'secret'}
response = requests.post('https://api.example.com/login', json=data)

# Headers
headers = {'Authorization': 'Bearer token123'}
response = requests.get('https://api.example.com/protected', headers=headers)

# 上传文件
files = {'file': open('document.pdf', 'rb')}
response = requests.post('https://api.example.com/upload', files=files)

# 设置超时
response = requests.get('https://api.example.com', timeout=10)

# Session (保持 Cookie)
session = requests.Session()
session.headers.update({'Authorization': 'Bearer token'})
response = session.get('https://api.example.com/me')

# 错误处理
try:
    response = requests.get('https://api.example.com', timeout=5)
    response.raise_for_status()
except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e}")
except requests.exceptions.Timeout:
    print("Request timed out")
except requests.exceptions.ConnectionError:
    print("Connection error")
```

### 5.2 httpx (推荐)

```python
import httpx

# 同步
response = httpx.get('https://example.com')
print(response.json())

# 异步
import httpx

async def fetch_all(urls):
    async with httpx.AsyncClient() as client:
        responses = await client.get(urls)
        return responses

# 带超时
client = httpx.Client(timeout=10.0)
response = client.get('https://example.com')

# 认证
client = httpx.Client(auth=('user', 'pass'))

# 重试
from httpx import HTTPTransport
transport = HTTPTransport(retries=3)
client = httpx.Client(transport=transport)
```

---

## 6. API 开发

### 6.1 Flask

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

# GET 路由
@app.route('/')
def index():
    return jsonify({'message': 'Hello, World!'})

@app.route('/users/<int:user_id>')
def get_user(user_id):
    user = {'id': user_id, 'name': 'Alice'}
    return jsonify(user)

# POST 路由
@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    return jsonify({'id': 1, **data}), 201

# 查询参数
@app.route('/search')
def search():
    query = request.args.get('q', '')
    page = request.args.get('page', 1, type=int)
    return jsonify({'query': query, 'page': page})

# 错误处理
@app.errorhandler(404)
def not_found(e):
    return jsonify({'error': 'Not found'}), 404

# 运行
if __name__ == '__main__':
    app.run(debug=True, port=5000)
```

### 6.2 FastAPI (推荐)

```python
from fastapi import FastAPI, HTTPException, Query, Body
from pydantic import BaseModel, EmailStr
from typing import Optional, List

app = FastAPI()

# Pydantic 模型
class User(BaseModel):
    name: str
    email: EmailStr
    age: Optional[int] = None

class UserResponse(BaseModel):
    id: int
    name: str
    email: str

# GET
@app.get('/')
def read_root():
    return {'message': 'Hello, World!'}

@app.get('/users/{user_id}')
def get_user(user_id: int):
    if user_id not in users:
        raise HTTPException(status_code=404, detail='User not found')
    return users[user_id]

# 查询参数
@app.get('/search')
def search(q: str = Query(..., min_length=1), limit: int = 10):
    return {'query': q, 'limit': limit}

# POST
@app.post('/users', response_model=UserResponse, status_code=201)
def create_user(user: User):
    new_user = {'id': len(users) + 1, **user.dict()}
    users[new_user['id']] = new_user
    return new_user

# 启动服务器
# uvicorn main:app --reload --port 8000
```

---

## 7. WebSocket

### 7.1 websockets

```python
# 服务端
import asyncio
import websockets

async def echo(websocket, path):
    async for message in websocket:
        await websocket.send(message)

start_server = websockets.serve(echo, 'localhost', 8765)
asyncio.get_event_loop().run_until_complete(start_server)
asyncio.get_event_loop().run_forever()

# 客户端
import asyncio
import websockets

async def client():
    async with websockets.connect('ws://localhost:8765') as websocket:
        await websocket.send('Hello!')
        response = await websocket.recv()
        print(response)

asyncio.get_event_loop().run_until_complete(client())
```

### 7.2 FastAPI WebSocket

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

app = FastAPI()

class ConnectionManager:
    def __init__(self):
        self.active_connections = []
    
    async def connect(self, websocket):
        await websocket.accept()
        self.active_connections.append(websocket)
    
    async def disconnect(self, websocket):
        self.active_connections.remove(websocket)
    
    async def broadcast(self, message):
        for connection in self.active_connections:
            await connection.send_text(message)

manager = ConnectionManager()

@app.websocket('/ws')
async def websocket_endpoint(websocket: WebSocket):
    await manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            await manager.broadcast(data)
    except WebSocketDisconnect:
        await manager.disconnect(websocket)
```

---

## 8. 数据库连接池

### 8.1 SQLAlchemy 连接池

```python
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

engine = create_engine(
    'postgresql://user:pass@localhost/mydb',
    poolclass=QueuePool,
    pool_size=5,           # 连接池大小
    max_overflow=10,       # 最大溢出
    pool_pre_ping=True,    # 连接前测试
    pool_recycle=3600      # 回收时间
)

# 使用
with engine.connect() as conn:
    result = conn.execute("SELECT * FROM users")
```

---

## 下一步

- [测试与安全](../10-测试与安全/README.md) — 学习测试和安全编码
- [性能优化](../11-性能优化/README.md) — 深入性能优化

---

*参考资料：[SQLAlchemy 文档](https://docs.sqlalchemy.org/) | [FastAPI 文档](https://fastapi.tiangolo.com/) | [Redis 文档](https://redis-py.readthedocs.io/)*
