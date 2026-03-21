# 第五章：持久化与检查点

## 5.1 持久化概述

LangGraph 支持状态的持久化，允许在任意节点保存和恢复执行状态，实现暂停、恢复和中断功能。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        持久化架构                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   执行流程：                                                         │
│                                                                      │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐       │
│   │  Node A │ → │  Node B │ → │  Node C │ → │  Node D │       │
│   └────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘       │
│        │              │              │              │             │
│        │              │              │              │             │
│        ▼              ▼              ▼              ▼             │
│   ┌─────────────────────────────────────────────────────────┐      │
│   │                    Checkpointer                         │      │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  │      │
│   │  │ State A │  │ State B │  │ State C │  │ State D │  │      │
│   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  │      │
│   └─────────────────────────────────────────────────────────┘      │
│                                    │                                │
│                                    ▼                                │
│                         ┌──────────────────┐                       │
│                         │   恢复执行        │                       │
│                         │   Resume         │                       │
│                         └──────────────────┘                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 5.2 检查点基础

### 5.2.1 内存检查点

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, END, START

# 创建内存检查点
checkpointer = MemorySaver()

# 创建图并编译时指定检查点
workflow = StateGraph(dict)
workflow.add_node("node_a", lambda s: {"result": "a"})
workflow.add_edge(START, "node_a")
workflow.add_edge("node_a", END)

# 编译时添加检查点
app = workflow.compile(checkpointer=checkpointer)

# 初始执行
config = {"configurable": {"thread_id": "thread_1"}}
app.invoke({"value": "initial"}, config=config)

# 查看保存的状态
print(app.get_state(config))
```

### 5.2.2 配置参数

```python
# 执行配置
config = {
    "configurable": {
        "thread_id": "user_123_session",  # 线程ID
        "checkpoint_id": "checkpoint_abc"   # 特定检查点ID
    },
    "recursion_limit": 50,  # 最大递归深度
    "tags": ["production"]  # 标签
}
```

### 5.2.3 持久化检查点

```python
from langgraph.checkpoint.sqlite import SqliteSaver
import sqlite3

# SQLite 检查点
conn = sqlite3.connect("checkpoints.db")
checkpointer = SqliteSaver(conn)

# 编译图
app = workflow.compile(checkpointer=checkpointer)

# 执行
config = {"configurable": {"thread_id": "session_1"}}
app.invoke({"value": "test"}, config=config)

# 列出所有检查点
checkpoints = list(app.get_list(config))
for cp in checkpoints:
    print(f"Checkpoint ID: {cp['id']}")
```

## 5.3 状态管理

### 5.3.1 获取状态

```python
# 获取当前状态
current_state = app.get_state(config)
print(current_state.values)
print(current_state.next)

# 获取所有检查点
all_checkpoints = app.get_list(config)
for cp in all_checkpoints:
    print(f"ID: {cp.id}, Next: {cp.next}")
```

### 5.3.2 更新状态

```python
# 直接更新状态
app.update_state(
    config,
    {"value": "updated_value", "count": 10}
)
```

### 5.3.3 恢复执行

```python
# 从检查点恢复并继续执行
# 先获取之前的配置
config = {"configurable": {"thread_id": "session_1"}}
app.invoke(None, config=config)  # None 表示使用保存的状态继续
```

## 5.4 中断与恢复

### 5.4.1 使用 interrupt

```python
from langgraph.types import interrupt

def node_with_interrupt(state):
    """带中断的节点"""
    # 中断执行，等待用户输入
    user_input = interrupt({"prompt": "请确认操作"})
    
    if user_input.get("confirmed"):
        return {"status": "confirmed"}
    return {"status": "cancelled"}

workflow = StateGraph(dict)
workflow.add_node("process", node_with_interrupt)
workflow.add_edge(START, "process")
workflow.add_edge("process", END)

app = workflow.compile(checkpointer=MemorySaver())

# 执行（会中断）
config = {"configurable": {"thread_id": "test"}}
app.invoke({"action": "process"}, config=config)

# 用户确认后恢复
app.invoke(
    {"confirmed": True},  # resume 输入
    config=config
)
```

### 5.4.2 处理中断

```python
# 检查是否有中断
state = app.get_state(config)
if state.next:
    print(f"在节点 {state.next} 处中断")
    
    # 查看中断信息
    # interrupt 会暂停在这里
```

## 5.5 持久化存储

### 5.5.1 PostgreSQL

```python
from langgraph.checkpoint.postgres import PostgresSaver
import psycopg2

conn = psycopg2.connect(
    host="localhost",
    database="langgraph",
    user="user",
    password="password"
)
checkpointer = PostgresSaver(conn)
checkpointer.setup()  # 创建表
```

### 5.5.2 Redis

```python
from langgraph.checkpoint.redis import RedisSaver
import redis

r = redis.Redis(host='localhost', port=6379, db=0)
checkpointer = RedisSaver(r)
```

### 5.5.3 自定义存储

```python
from langgraph.checkpoint.base import BaseCheckpointSaver
from typing import Any, Optional, Iterator

class CustomCheckpointSaver(BaseCheckpointSaver):
    """自定义检查点存储"""
    
    def __init__(self):
        self.checkpoints = {}
    
    def get(self, config: dict) -> Optional[Any]:
        """获取检查点"""
        key = config.get("configurable", {}).get("checkpoint_id")
        return self.checkpoints.get(key)
    
    def put(self, config: dict, checkpoint: Any, metadata: dict) -> None:
        """保存检查点"""
        key = config.get("configurable", {}).get("checkpoint_id")
        self.checkpoints[key] = (checkpoint, metadata)
    
    def list(self, config: dict, **kwargs) -> Iterator[Any]:
        """列出检查点"""
        return iter(self.checkpoints.values())
```

## 5.6 完整示例

### 5.6.1 可恢复的对话

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, END, START
from typing import TypedDict

class ChatState(TypedDict):
    messages: list
    user_input: str

def process_message(state: ChatState) -> dict:
    """处理消息"""
    new_messages = state["messages"] + [f"User: {state['user_input']}"]
    return {"messages": new_messages}

def generate_response(state: ChatState) -> dict:
    """生成回复（模拟）"""
    response = f"Response to: {state['user_input']}"
    return {"messages": state["messages"] + [f"Bot: {response}"]}

workflow = StateGraph(ChatState)
workflow.add_node("process", process_message)
workflow.add_node("respond", generate_response)
workflow.add_edge(START, "process")
workflow.add_edge("process", "respond")
workflow.add_edge("respond", END)

checkpointer = MemorySaver()
app = workflow.compile(checkpointer=checkpointer)

# 第一次对话
config = {"configurable": {"thread_id": "user_123"}}
app.invoke({"messages": [], "user_input": "Hello"}, config=config)

# 模拟用户断开后重新连接
# 获取之前的对话历史
state = app.get_state(config)
print("对话历史:", state.values["messages"])

# 继续对话
app.invoke({"user_input": "How are you?"}, config=config)
```

### 5.6.2 长流程任务

```python
from langgraph.checkpoint.sqlite import SqliteSaver
from langgraph.graph import StateGraph, END, START
from typing import TypedDict, List
import uuid

class TaskState(TypedDict):
    task_id: str
    steps: List[str]
    current_step: int
    results: dict

def step_1(state: TaskState) -> dict:
    """步骤1"""
    return {"steps": ["step_1_done"]}

def step_2(state: TaskState) -> dict:
    """步骤2"""
    return {"steps": ["step_2_done"]}

def step_3(state: TaskState) -> dict:
    """步骤3"""
    return {"steps": ["step_3_done"]}

workflow = StateGraph(TaskState)
workflow.add_node("step_1", step_1)
workflow.add_node("step_2", step_2)
workflow.add_node("step_3", step_3)
workflow.add_edge(START, "step_1")
workflow.add_edge("step_1", "step_2")
workflow.add_edge("step_2", "step_3")
workflow.add_edge("step_3", END)

# SQLite 持久化
checkpointer = SqliteSaver.from_conn_string("tasks.db")
app = workflow.compile(checkpointer=checkpointer)

# 启动任务
task_id = str(uuid.uuid4())
config = {"configurable": {"thread_id": task_id}}
app.invoke({
    "task_id": task_id,
    "steps": [],
    "current_step": 0,
    "results": {}
}, config=config)

# 检查进度
state = app.get_state(config)
print(f"当前步骤: {state.values['current_step']}")
print(f"步骤记录: {state.values['steps']}")

# 恢复继续执行
app.invoke(None, config=config)
```

## 5.7 总结

本章介绍了持久化与检查点的用法：

1. **MemorySaver**：内存检查点
2. **SqliteSaver**：SQLite 持久化
3. **状态获取与更新**：get_state、update_state
4. **中断与恢复**：interrupt 实现人机交互
5. **自定义存储**：实现 BaseCheckpointSaver

---

*下一章我们将学习人机交互。*
