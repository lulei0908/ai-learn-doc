# 08. 异步编程

## 概述

Python 的异步编程支持让你能够编写高效的并发代码，尤其适合 I/O 密集型任务。本章介绍 asyncio 模块、async/await 语法以及异步编程的最佳实践。

---

## 1. 异步基础

### 1.1 同步 vs 异步

```python
# 同步代码 - 顺序执行，阻塞等待
import requests

def fetch_all(urls):
    results = []
    for url in urls:
        resp = requests.get(url)  # 阻塞等待
        results.append(resp.json())
    return results

# 异步代码 - 并发执行，I/O 时让出控制权
import aiohttp

async def fetch_all_async(urls):
    results = []
    async with aiohttp.ClientSession() as session:
        for url in urls:
            async with session.get(url) as resp:  # 非阻塞
                results.append(await resp.json())
    return results

# 真正并发 - 同时发起所有请求
async def fetch_all_concurrent(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [session.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        return [await r.json() for r in responses]
```

### 1.2 事件循环

```python
import asyncio

# 运行事件循环
asyncio.run(main())  # Python 3.7+

# 旧版写法
loop = asyncio.get_event_loop()
try:
    loop.run_until_complete(main())
finally:
    loop.close()

# 手动创建事件循环
loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)
try:
    loop.run_until_complete(main())
finally:
    loop.close()
```

---

## 2. async/await 语法

### 2.1 定义异步函数

```python
import asyncio

# async def 定义协程函数
async def fetch_data():
    print("Fetching data...")
    await asyncio.sleep(1)  # 模拟 I/O 操作
    return {"data": "result"}

# 调用异步函数
async def main():
    # 创建协程对象
    coro = fetch_data()
    print(type(coro))  # <class 'coroutine'>
    
    # 等待结果
    result = await coro
    print(result)  # {'data': 'result'}

# 运行
asyncio.run(main())
```

### 2.2 await 表达式

```python
async def example():
    # await 等待协程完成并获取结果
    result1 = await async_function1()
    result2 = await async_function2()
    
    # 可以并行等待多个协程
    results = await asyncio.gather(
        async_function1(),
        async_function2(),
        async_function3()
    )
    
    # asyncio.wait 也可 (不获取结果)
    await asyncio.wait([task1, task2])
```

### 2.3 任务 (Task)

```python
async def main():
    # 创建任务 - 调度协程在事件循环中运行
    task = asyncio.create_task(fetch_data())
    
    # 等待任务完成
    result = await task
    
    # 或者创建后立即做其他事
    task = asyncio.create_task(fetch_data())
    print("Doing other work...")
    result = await task
    
    # 等待多个任务
    tasks = [asyncio.create_task(i_am_slow(i)) for i in range(5)]
    results = await asyncio.gather(*tasks)
```

---

## 3. 异步原语

### 3.1 asyncio.sleep

```python
import asyncio

async def example():
    # 模拟异步操作
    await asyncio.sleep(1)  # 暂停 1 秒
    
    # 可以取消
    try:
        await asyncio.sleep(10)
    except asyncio.CancelledError:
        print("Sleep was cancelled!")
```

### 3.2 asyncio.gather

```python
async def task1():
    await asyncio.sleep(1)
    return "Task 1 done"

async def task2():
    await asyncio.sleep(2)
    return "Task 2 done"

async def main():
    # 并发执行多个协程
    results = await asyncio.gather(
        task1(),
        task2(),
        return_exceptions=True  # 捕获异常而不是抛出
    )
    print(results)  # ['Task 1 done', 'Task 2 done']

asyncio.run(main())
```

### 3.3 asyncio.wait

```python
async def main():
    # 创建任务
    task1 = asyncio.create_task(task1())
    task2 = asyncio.create_task(task2())
    
    # 等待任务完成
    # 返回 (done, pending) 元组
    done, pending = await asyncio.wait([task1, task2])
    
    for task in done:
        print(f"Completed: {task.result()}")
    
    for task in pending:
        task.cancel()  # 取消未完成的任务
```

### 3.4 asyncio.wait_for

```python
async def slow_operation():
    await asyncio.sleep(10)
    return "Done!"

async def main():
    try:
        # 设置超时
        result = await asyncio.wait_for(slow_operation(), timeout=5.0)
    except asyncio.TimeoutError:
        print("Operation timed out!")
    
    # 也可以使用 shield 保护任务不被取消
    try:
        result = await asyncio.wait_for(
            asyncio.shield(slow_operation()),
            timeout=1.0
        )
    except asyncio.TimeoutError:
        print("Shielded operation also timed out!")
```

---

## 4. 异步迭代器和生成器

### 4.1 异步迭代器

```python
class AsyncCounter:
    def __init__(self, n):
        self.n = n
        self.current = 0
    
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        if self.current >= self.n:
            raise StopAsyncIteration
        self.current += 1
        await asyncio.sleep(0.1)  # 模拟异步操作
        return self.current

async def main():
    async for i in AsyncCounter(5):
        print(i)

asyncio.run(main())
```

### 4.2 异步生成器

```python
async def async_range(start, stop):
    """异步生成器"""
    for i in range(start, stop):
        await asyncio.sleep(0.1)
        yield i

async def main():
    async for i in async_range(0, 5):
        print(i)

asyncio.run(main())
```

---

## 5. 异步上下文管理器

```python
class AsyncContextManager:
    async def __aenter__(self):
        await asyncio.sleep(0.1)  # 模拟异步初始化
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await asyncio.sleep(0.1)  # 模拟异步清理

async def main():
    async with AsyncContextManager() as cm:
        print("Inside context")

asyncio.run(main())
```

---

## 6. 异步队列

### 6.1 asyncio.Queue

```python
import asyncio

async def producer(queue):
    for i in range(5):
        await asyncio.sleep(0.5)
        await queue.put(i)
        print(f"Produced: {i}")
    await queue.put(None)  # 发送结束信号

async def consumer(queue):
    while True:
        item = await queue.get()
        if item is None:
            break
        print(f"Consumed: {item}")
        await queue.task_done()

async def main():
    queue = asyncio.Queue()
    
    await asyncio.gather(
        producer(queue),
        consumer(queue)
    )

asyncio.run(main())
```

---

## 7. 锁和其他同步原语

### 7.1 asyncio.Lock

```python
import asyncio

shared_resource = 0
lock = asyncio.Lock()

async def worker(name, count):
    global shared_resource
    for _ in range(count):
        async with lock:  # 获取锁
            shared_resource += 1
            print(f"{name}: {shared_resource}")

async def main():
    await asyncio.gather(
        worker("A", 5),
        worker("B", 5)
    )

asyncio.run(main())
```

### 7.2 asyncio.Event

```python
import asyncio

event = asyncio.Event()

async def waiter(name):
    print(f"{name} waiting...")
    await event.wait()
    print(f"{name} got the event!")

async def setter():
    await asyncio.sleep(2)
    event.set()  # 触发事件

asyncio.run(asyncio.gather(waiter("W1"), waiter("W2"), setter()))
```

### 7.3 asyncio.Condition

```python
import asyncio

condition = asyncio.Condition()

async def consumer(name, queue):
    async with condition:
        while queue.empty():
            await condition.wait()
        item = queue.get()
        print(f"{name} got: {item}")

async def producer(queue):
    await asyncio.sleep(1)
    async with condition:
        await queue.put("data")
        condition.notify_all()
```

### 7.4 asyncio.Semaphore

```python
import asyncio

# 限制并发数
semaphore = asyncio.Semaphore(3)  # 最多 3 个并发

async def limited_task(n):
    async with semaphore:
        print(f"Task {n} starting")
        await asyncio.sleep(1)
        print(f"Task {n} done")

async def main():
    await asyncio.gather(*[limited_task(i) for i in range(10)])

asyncio.run(main())
```

---

## 8. 异步 HTTP 请求

### 8.1 aiohttp

```python
import aiohttp

async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        async def fetch(url):
            async with session.get(url) as resp:
                return await resp.json()
        
        return await asyncio.gather(*[fetch(url) for url in urls])

# POST 请求
async def post_data(url, data):
    async with aiohttp.ClientSession() as session:
        async with session.post(url, json=data) as resp:
            return await resp.json()

# 带参数
async def search(query):
    async with aiohttp.ClientSession() as session:
        params = {'q': query, 'format': 'json'}
        async with session.get('https://api.example.com/search', params=params) as resp:
            return await resp.json()
```

### 8.2 httpx

```python
import httpx

# httpx 也支持异步
async def fetch():
    async with httpx.AsyncClient() as client:
        resp = await client.get('https://example.com')
        return resp.text
```

---

## 9. 异步文件操作

### 9.1 aiofiles

```python
import aiofiles
import asyncio

async def async_read_write():
    # 异步读取
    async with aiofiles.open('file.txt', 'r') as f:
        content = await f.read()
    
    # 异步写入
    async with aiofiles.open('output.txt', 'w') as f:
        await f.write('Hello, async!')
    
    # 逐行读取
    async with aiofiles.open('file.txt', 'r') as f:
        async for line in f:
            print(line.strip())

asyncio.run(async_read_write())
```

---

## 10. 最佳实践

### 10.1 避免阻塞调用

```python
# ❌ 错误 - 在异步函数中使用同步阻塞调用
async def bad_example():
    import requests  # 同步库
    resp = requests.get(url)  # 阻塞!
    return resp.json()

# ✅ 正确 - 使用异步库
async def good_example():
    import aiohttp
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            return await resp.json()

# 或者在线程池中运行
async def acceptable_example():
    import asyncio
    import requests
    
    loop = asyncio.get_event_loop()
    resp = await loop.run_in_executor(None, requests.get, url)
    return resp.json()
```

### 10.2 异常处理

```python
async def safe_async_call():
    try:
        result = await risky_operation()
    except asyncio.CancelledError:
        # 任务被取消时清理资源
        await cleanup()
        raise
    except Exception as e:
        logger.error(f"Error: {e}")
        return None
    finally:
        # 清理
        await close_connections()
```

### 10.3 取消任务

```python
async def long_task():
    try:
        while True:
            await asyncio.sleep(1)
            print("Working...")
    except asyncio.CancelledError:
        print("Task cancelled, cleaning up...")
        raise

async def main():
    task = asyncio.create_task(long_task())
    await asyncio.sleep(3)
    task.cancel()  # 取消任务
    try:
        await task
    except asyncio.CancelledError:
        print("Main: Task was cancelled")
```

---

## 11. 异步 vs 多线程/多进程

| 特性 | 异步 IO | 多线程 | 多进程 |
|:---|:---|:---|:---|
| 适用场景 | I/O 密集型 | I/O 密集型 | CPU 密集型 |
| 资源消耗 | 低 | 中 | 高 |
| 并发数 | 高 | 中 | 低 |
| GIL | 不受影响 | 受影响 | 不受影响 |
| 复杂度 | 中 | 高 | 中 |
| 调试难度 | 较难 | 较难 | 易 |

---

## 下一步

- [数据库与网络](../09-数据库与网络/README.md) — 学习数据库操作和网络编程
- [性能优化](../11-性能优化/README.md) — 深入 Python 性能优化

---

*参考资料：[asyncio 模块](https://docs.python.org/3/library/asyncio.html) | [aiohttp 文档](https://docs.aiohttp.org/)*
