# 第三章：节点与边的定义

## 3.1 节点基础

### 3.1.1 节点函数

节点是 LangGraph 中的基本执行单元，是一个接收状态并返回状态更新的函数：

```python
from typing import TypedDict
from langgraph.graph import StateGraph, END, START

class State(TypedDict):
    value: str
    count: int

def node_function(state: State) -> dict:
    """节点函数"""
    # 读取当前状态
    current_value = state["value"]
    
    # 返回状态更新（部分更新）
    return {
        "value": current_value + "_processed",
        "count": state["count"] + 1
    }

# 创建图并添加节点
workflow = StateGraph(State)
workflow.add_node("my_node", node_function)
workflow.set_entry_point("my_node")
workflow.add_edge("my_node", END)
```

### 3.1.2 节点名称

```python
# 节点名称可以自定义
workflow.add_node("custom_name", node_function)

# 查看节点列表
print(workflow.nodes)
```

### 3.1.3 节点返回值

```python
def returns_dict(state: State) -> dict:
    """返回字典 - 部分更新"""
    return {"key": "value"}

def returns_state(state: State) -> State:
    """返回完整状态 - 完全替换"""
    return {"value": "new", "count": 0}

def returns_none(state: State) -> None:
    """不返回 - 保持状态不变"""
    print("执行了一些操作")
```

## 3.2 节点类型

### 3.2.1 普通节点

```python
def simple_node(state: State) -> dict:
    """普通处理节点"""
    return {"result": "processed"}
```

### 3.2.2 LLM 节点

```python
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage

llm = ChatOpenAI(model="gpt-4")

class ChatState(TypedDict):
    messages: list

def llm_node(state: ChatState) -> dict:
    """LLM 调用节点"""
    response = llm.invoke(state["messages"])
    return {"messages": [response]}
```

### 3.2.3 工具节点

```python
from langchain_core.tools import tool

@tool
def calculate(expression: str) -> str:
    """计算工具"""
    return str(eval(expression))

def tool_node(state: State) -> dict:
    """工具调用节点"""
    result = calculate.invoke({"expression": "2 + 2"})
    return {"result": result}
```

### 3.2.4 条件节点

```python
def condition_node(state: State) -> dict:
    """条件判断节点"""
    if state["count"] > 5:
        return {"action": "continue"}
    return {"action": "stop"}
```

### 3.2.5 聚合节点

```python
def aggregator_node(state: State) -> dict:
    """聚合节点 - 收集多个子节点的结果"""
    # 假设通过共享状态获取其他节点的结果
    results = state.get("partial_results", [])
    aggregated = sum(results) if results else 0
    return {"aggregated": aggregated}
```

## 3.3 边的定义

### 3.3.1 普通边

```python
# 从节点 A 到节点 B 的单向边
workflow.add_edge("node_a", "node_b")

# 设置入口点
workflow.set_entry_point("start_node")

# 设置终点
workflow.add_edge("end_node", END)
```

### 3.3.2 START 和 END

```python
from langgraph.graph import START, END

# START 是特殊的虚拟节点，表示图的入口
workflow.add_edge(START, "first_node")

# END 是特殊的虚拟节点，表示图的出口
workflow.add_edge("last_node", END)
```

### 3.3.3 条件边

```python
from typing import Literal

class ConditionalState(TypedDict):
    value: int
    next_step: str

def should_continue(state: ConditionalState) -> Literal["high", "low", "end"]:
    """返回目标节点名称"""
    if state["value"] > 10:
        return "high"
    elif state["value"] > 5:
        return "low"
    return "end"

# 添加条件边
workflow.add_conditional_edges(
    "process_node",
    should_continue,
    {
        "high": "high_value_handler",
        "low": "low_value_handler",
        "end": END
    }
)
```

### 3.3.4 条件边映射

```python
def route_function(state: State) -> str:
    """简单路由 - 直接返回节点名称"""
    if state["value"] > 10:
        return "big"
    return "small"

# 简化写法：直接返回节点名称
workflow.add_conditional_edges("process", route_function)
```

## 3.4 节点组合

### 3.4.1 多个节点到同一目标

```python
workflow.add_edge("node_a", "merge")
workflow.add_edge("node_b", "merge")
workflow.add_edge("node_c", "merge")
```

### 3.4.2 一个节点到多个目标

```python
workflow.add_conditional_edges(
    "branch_point",
    lambda state: state["choice"],
    {
        "path_a": "node_a",
        "path_b": "node_b",
        "path_c": "node_c"
    }
)
```

### 3.4.3 并行执行后聚合

```python
# 使用条件边实现并行聚合
workflow.add_edge(START, "parallel_1")
workflow.add_edge(START, "parallel_2")
workflow.add_edge(START, "parallel_3")

# 等待所有并行任务完成
workflow.add_edge("parallel_1", "aggregate")
workflow.add_edge("parallel_2", "aggregate")
workflow.add_edge("parallel_3", "aggregate")
workflow.add_edge("aggregate", END)
```

## 3.5 节点间通信

### 3.5.1 通过状态传递

```python
class SharedState(TypedDict):
    data: dict
    results: list

def node_a(state: SharedState) -> dict:
    """节点 A 处理数据"""
    processed = process_data(state["data"])
    return {"results": [processed]}

def node_b(state: SharedState) -> dict:
    """节点 B 使用节点 A 的结果"""
    results = state.get("results", [])
    if results:
        return {"data": {"final": results[-1]}}
    return {"data": {"final": None}}
```

### 3.5.2 状态隔离

```python
class IsolatedState(TypedDict):
    node_a_data: dict
    node_b_data: dict

def isolated_node_a(state: IsolatedState) -> dict:
    """节点 A 只能访问和修改 node_a_data"""
    return {"node_a_data": {"value": "processed by A"}}

def isolated_node_b(state: IsolatedState) -> dict:
    """节点 B 只能访问和修改 node_b_data"""
    return {"node_b_data": {"value": "processed by B"}}
```

## 3.6 高级节点特性

### 3.6.1 异步节点

```python
import asyncio

async def async_node(state: State) -> dict:
    """异步节点"""
    await asyncio.sleep(0.1)
    return {"async_result": "done"}

# 在图中使用
workflow.add_node("async", async_node)
```

### 3.6.2 节点装饰器

```python
from functools import wraps

def log_node(func):
    """日志装饰器"""
    @wraps(func)
    def wrapper(state):
        print(f"Entering {func.__name__}")
        result = func(state)
        print(f"Exiting {func.__name__}")
        return result
    return wrapper

@log_node
def decorated_node(state: State) -> dict:
    return {"value": "decorated"}
```

### 3.6.3 节点重试

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=2, max=10))
def resilient_node(state: State) -> dict:
    """带重试的节点"""
    # 可能失败的逻辑
    return {"value": "success"}
```

## 3.7 完整示例

### 3.7.1 多阶段处理管道

```python
from langgraph.graph import StateGraph, END, START
from typing import TypedDict, List, Optional

class PipelineState(TypedDict):
    input_data: str
    stage_1_result: Optional[str]
    stage_2_result: Optional[str]
    stage_3_result: Optional[str]
    errors: List[str]
    completed: bool

def extract(state: PipelineState) -> dict:
    """阶段1: 提取"""
    try:
        # 模拟提取
        extracted = f"extracted_from({state['input_data']})"
        return {"stage_1_result": extracted}
    except Exception as e:
        return {"errors": [f"Extract error: {str(e)}"]}

def transform(state: PipelineState) -> dict:
    """阶段2: 转换"""
    if state.get("errors"):
        return {}
    
    try:
        # 模拟转换
        transformed = f"transformed({state['stage_1_result']})"
        return {"stage_2_result": transformed}
    except Exception as e:
        return {"errors": state["errors"] + [f"Transform error: {str(e)}"]}

def load(state: PipelineState) -> dict:
    """阶段3: 加载"""
    if state.get("errors"):
        return {"completed": False}
    
    try:
        # 模拟加载
        loaded = f"loaded({state['stage_2_result']})"
        return {"stage_3_result": loaded, "completed": True}
    except Exception as e:
        return {
            "errors": state["errors"] + [f"Load error: {str(e)}"],
            "completed": False
        }

def should_continue(state: PipelineState):
    """决定是否继续"""
    if state.get("errors"):
        return "error_handler"
    return "continue"

# 构建管道
workflow = StateGraph(PipelineState)

workflow.add_node("extract", extract)
workflow.add_node("transform", transform)
workflow.add_node("load", load)

workflow.add_edge(START, "extract")
workflow.add_edge("extract", "transform")
workflow.add_edge("transform", "load")
workflow.add_edge("load", END)

# 执行
app = workflow.compile()
result = app.invoke({
    "input_data": "raw_data",
    "stage_1_result": None,
    "stage_2_result": None,
    "stage_3_result": None,
    "errors": [],
    "completed": False
})

print(result)
```

### 3.7.2 任务调度器

```python
from langgraph.graph import StateGraph, END, START
from typing import TypedDict, List
from datetime import datetime

class TaskState(TypedDict):
    tasks: List[dict]
    current_task_index: int
    completed_tasks: List[dict]
    failed_tasks: List[dict]
    results: dict

def submit_tasks(state: TaskState) -> dict:
    """提交任务"""
    return {"current_task_index": 0}

def execute_task(state: TaskState) -> dict:
    """执行当前任务"""
    if state["current_task_index"] >= len(state["tasks"]):
        return {"current_task_index": -1}
    
    current_task = state["tasks"][state["current_task_index"]]
    
    # 模拟执行
    success = current_task.get("id", "") != "fail"
    
    if success:
        return {
            "completed_tasks": [current_task],
            "results": {current_task["id"]: "success"}
        }
    return {
        "failed_tasks": [current_task],
        "results": {current_task["id"]: "failed"}
    }

def should_continue(state: TaskState) -> Literal["execute", "finish"]:
    """检查是否继续"""
    if state["current_task_index"] == -1:
        return "finish"
    return "execute"

def next_task(state: TaskState) -> dict:
    """移动到下一个任务"""
    return {"current_task_index": state["current_task_index"] + 1}

def summarize(state: TaskState) -> dict:
    """汇总结果"""
    total = len(state["completed_tasks"]) + len(state["failed_tasks"])
    return {
        "summary": f"完成: {len(state['completed_tasks'])}/{total}"
    }

# 构建调度器
workflow = StateGraph(TaskState)

workflow.add_node("submit", submit_tasks)
workflow.add_node("execute", execute_task)
workflow.add_node("next", next_task)
workflow.add_node("summarize", summarize)

workflow.add_edge(START, "submit")
workflow.add_edge("submit", "execute")

workflow.add_conditional_edges(
    "execute",
    should_continue,
    {
        "execute": "next",
        "finish": "summarize"
    }
)

workflow.add_edge("next", "execute")
workflow.add_edge("summarize", END)

app = workflow.compile()

# 执行
result = app.invoke({
    "tasks": [
        {"id": "task_1", "name": "任务1"},
        {"id": "task_2", "name": "任务2"},
        {"id": "fail", "name": "失败任务"},
        {"id": "task_4", "name": "任务4"}
    ],
    "current_task_index": 0,
    "completed_tasks": [],
    "failed_tasks": [],
    "results": {}
})

print(f"完成: {len(result['completed_tasks'])}, 失败: {len(result['failed_tasks'])}")
```

## 3.8 总结

本章深入介绍了 LangGraph 中节点与边的定义：

1. **节点函数**：接收状态、返回状态更新
2. **节点类型**：普通节点、LLM 节点、工具节点、条件节点
3. **边的类型**：普通边、条件边
4. **节点组合**：并行、聚合、路由
5. **高级特性**：异步节点、装饰器、重试

---

*下一章我们将学习条件路由与分支逻辑。*
