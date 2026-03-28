# 第八章：错误处理与恢复

## 8.1 错误处理概述

LangGraph 提供了完善的错误处理机制，包括异常捕获、重试策略和故障恢复。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        错误处理机制                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    错误处理层级                              │   │
│   ├─────────────────────────────────────────────────────────────┤   │
│   │                                                              │   │
│   │   节点级处理           图级处理           全局处理           │   │
│   │   ┌─────────┐         ┌─────────┐       ┌─────────┐       │   │
│   │   │ try/catch│         │ 条件边   │      │ Middleware│       │   │
│   │   └─────────┘         └─────────┘       └─────────┘       │   │
│   │         │                  │                  │             │   │
│   │         └──────────────────┼──────────────────┘             │   │
│   │                            │                                │   │
│   │                            ▼                                │   │
│   │                    ┌─────────────────┐                      │   │
│   │                    │   Retry Logic   │                      │   │
│   │                    │   重试机制       │                      │   │
│   │                    └─────────────────┘                      │   │
│   │                                                              │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 8.2 节点级错误处理

### 8.2.1 Try/Catch

```python
from langgraph.graph import StateGraph, END, START
from typing import TypedDict

class ErrorState(TypedDict):
    value: str
    error: str
    retry_count: int

def risky_operation(state: ErrorState) -> dict:
    """可能失败的操作"""
    try:
        # 模拟可能失败的操作
        if "error" in state.get("value", ""):
            raise ValueError("Simulated error")
        return {"value": f"Success: {state['value']}"}
    except Exception as e:
        return {"error": str(e)}

workflow = StateGraph(ErrorState)
workflow.add_node("process", risky_operation)
workflow.add_edge(START, "process")
workflow.add_edge("process", END)
```

### 8.2.2 错误状态

```python
def process_with_error_tracking(state: ErrorState) -> dict:
    """带错误跟踪的处理"""
    try:
        result = state["value"] * 2
        return {"value": result, "error": ""}
    except Exception as e:
        return {"error": str(e), "value": ""}

def error_handler(state: ErrorState) -> dict:
    """错误处理节点"""
    return {"error": f"Handled: {state.get('error', 'unknown')}"}

def has_error(state: ErrorState) -> bool:
    return bool(state.get("error"))

workflow = StateGraph(ErrorState)
workflow.add_node("process", process_with_error_tracking)
workflow.add_node("handle_error", error_handler)

workflow.add_edge(START, "process")

# 条件边判断是否有错误
workflow.add_conditional_edges(
    "process",
    lambda s: "error" if s.get("error") else "success",
    {"error": "handle_error", "success": END}
)
```

## 8.3 重试机制

### 8.3.1 使用 tenacity

```python
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10),
    retry=retry_if_exception_type(Exception),
    reraise=True
)
def unreliable_function(state):
    """不可靠的函数"""
    if state.get("fail"):
        raise Exception("Temporary failure")
    return {"result": "success"}

def node_with_retry(state):
    return unreliable_function(state)
```

### 8.3.2 重试状态

```python
class RetryState(TypedDict):
    attempt: int
    max_attempts: int
    result: str
    error: str

def operation_with_retry(state: RetryState) -> dict:
    """带重试的操作"""
    attempt = state.get("attempt", 0) + 1
    
    if attempt >= state.get("max_attempts", 3):
        return {"error": "Max attempts reached", "attempt": attempt}
    
    try:
        # 模拟操作
        return {"result": "Success", "attempt": attempt}
    except Exception as e:
        return {"error": str(e), "attempt": attempt}

def should_retry(state: RetryState) -> Literal["retry", "give_up"]:
    if state.get("error") and state.get("attempt", 0) < state.get("max_attempts", 3):
        return "retry"
    return "give_up"
```

## 8.4 条件边错误处理

### 8.4.1 错误路由

```python
def check_error(state: ErrorState) -> Literal["success", "error"]:
    if state.get("error"):
        return "error"
    return "success"

workflow.add_conditional_edges(
    "process",
    check_error,
    {
        "success": "next_success_node",
        "error": "error_recovery_node"
    }
)
```

### 8.4.2 多重错误处理

```python
def classify_error(state: ErrorState) -> Literal["retry", "fallback", "fatal"]:
    error = state.get("error", "")
    
    if "timeout" in error.lower():
        return "retry"
    elif "fallback" in error.lower():
        return "fallback"
    return "fatal"

workflow.add_conditional_edges(
    "error_handler",
    classify_error,
    {
        "retry": "retry_node",
        "fallback": "fallback_node",
        "fatal": "fatal_error_node"
    }
)
```

## 8.5 完整示例

### 8.5.1 可靠的数据处理管道

```python
from langgraph.graph import StateGraph, END, START
from typing import TypedDict, List
from tenacity import retry, stop_after_attempt, wait_exponential

class PipelineState(TypedDict):
    data: List
    processed: List
    errors: List
    stage: str

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=1, max=5))
def fetch_data(state: PipelineState) -> dict:
    """获取数据"""
    if not state.get("data"):
        raise Exception("No data available")
    return {"stage": "fetched"}

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=1, max=5))
def process_data(state: PipelineState) -> dict:
    """处理数据"""
    data = state.get("data", [])
    processed = [str(x) for x in data]  # 简单处理
    return {"processed": processed, "stage": "processed"}

def validate(state: PipelineState) -> dict:
    """验证"""
    processed = state.get("processed", [])
    if len(processed) == 0:
        return {"errors": ["No processed data"]}
    return {"stage": "validated"}

def error_recovery(state: PipelineState) -> dict:
    """错误恢复"""
    return {
        "errors": state.get("errors", []) + ["Recovered"],
        "stage": "recovered"
    }

def route_after_error(state: PipelineState) -> Literal["retry", "continue"]:
    errors = state.get("errors", [])
    if errors:
        return "retry"
    return "continue"

workflow = StateGraph(PipelineState)
workflow.add_node("fetch", fetch_data)
workflow.add_node("process", process_data)
workflow.add_node("validate", validate)
workflow.add_node("error_recovery", error_recovery)

workflow.add_edge(START, "fetch")
workflow.add_edge("fetch", "process")
workflow.add_edge("process", "validate")

workflow.add_conditional_edges(
    "validate",
    route_after_error,
    {"retry": "error_recovery", "continue": END}
)

app = workflow.compile()

result = app.invoke({
    "data": [1, 2, 3, 4, 5],
    "processed": [],
    "errors": [],
    "stage": "init"
})
```

## 8.6 总结

本章介绍了错误处理与恢复：

1. **节点级错误处理**：使用 try/catch
2. **重试机制**：使用 tenacity 实现重试
3. **条件边错误路由**：根据错误类型路由
4. **恢复策略**：错误后恢复执行

---

*LangGraph 进阶部分到此结束。接下来进入实战案例部分。*
