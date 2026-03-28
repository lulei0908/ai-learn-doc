# 第二章：状态机与工作流

## 2.1 状态机基础

LangGraph 的核心是**状态机**（State Machine），它定义了应用在不同状态之间的转换规则。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        状态机示例                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                         ┌─────────┐                                 │
│                         │  START  │                                 │
│                         └────┬────┘                                 │
│                              │                                      │
│                              ▼                                      │
│                        ┌─────────┐                                  │
│                   ┌───▶│PROCESSING│◀───┐                           │
│                   │    └────┬────┘    │                           │
│                   │         │         │                           │
│              重试 │         │成功     │ 需要更多信息              │
│                   │         │         │                           │
│                   │         ▼         │                           │
│                   │    ┌─────────┐    │                           │
│                   └────│  ERROR  │────┘                           │
│                        └────┬────┘                                │
│                             │                                      │
│                             ▼                                      │
│                        ┌─────────┐                                  │
│                        │   END   │                                  │
│                        └─────────┘                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 2.2 状态定义

### 2.2.1 基础状态

```python
from typing import TypedDict, List, Optional

class BasicState(TypedDict):
    """基础状态定义"""
    messages: List[str]
    current_step: str
    result: Optional[dict]
```

### 2.2.2 带注释的状态

```python
from typing import Annotated
import operator

class AnnotatedState(TypedDict):
    """带注释的状态"""
    # 使用 operator.add 作为 reducer
    # 每次节点返回的值会与现有值相加（对于列表是追加）
    messages: Annotated[List[str], operator.add]
    
    # 常规字段，会被覆盖
    current_step: str
    
    # 默认值
    count: int
```

### 2.2.3 状态 Reducer

```python
from typing import TypedDict, Annotated
import operator
from typing import List

def custom_reducer(existing: List[str], new: List[str]) -> List[str]:
    """自定义 reducer：去重追加"""
    result = existing.copy()
    for item in new:
        if item not in result:
            result.append(item)
    return result

class StateWithReducer(TypedDict):
    """带自定义 reducer 的状态"""
    # 使用默认追加
    messages: Annotated[List[str], operator.add]
    
    # 使用自定义 reducer
    unique_items: Annotated[List[str], custom_reducer]
    
    # 覆盖更新
    current_value: str
```

### 2.2.4 嵌套状态

```python
class NestedState(TypedDict):
    """嵌套状态"""
    user_info: dict
    session: dict
    data: dict

# 使用
initial_state = {
    "user_info": {"name": "张三", "id": "user_001"},
    "session": {"id": "session_001", "start_time": "2024-01-01"},
    "data": {"items": [], "results": {}}
}
```

## 2.3 图的构建

### 2.3.1 创建图

```python
from langgraph.graph import StateGraph, END

# 创建图
workflow = StateGraph(BasicState)
```

### 2.3.2 添加节点

```python
def process_input(state: BasicState) -> dict:
    """处理输入节点"""
    # 返回状态更新
    return {
        "messages": ["处理输入完成"],
        "current_step": "processing"
    }

def validate_data(state: BasicState) -> dict:
    """验证数据节点"""
    if len(state["messages"]) > 0:
        return {"current_step": "validated"}
    return {"current_step": "error"}

def generate_result(state: BasicState) -> dict:
    """生成结果节点"""
    return {
        "result": {"status": "success", "data": state["messages"]},
        "current_step": "completed"
    }

# 添加节点到图
workflow.add_node("process_input", process_input)
workflow.add_node("validate_data", validate_data)
workflow.add_node("generate_result", generate_result)
```

### 2.3.3 设置入口点

```python
# 方式1：设置单个入口点
workflow.set_entry_point("process_input")

# 方式2：使用 START 常量
from langgraph.graph import START
workflow.add_edge(START, "process_input")
```

### 2.3.4 添加边

```python
# 普通边
workflow.add_edge("process_input", "validate_data")

# 条件边
def route_based_on_validation(state: BasicState):
    if state["current_step"] == "validated":
        return "success"
    return "error"

workflow.add_conditional_edges(
    "validate_data",
    route_based_on_validation,
    {
        "success": "generate_result",
        "error": END
    }
)

# 结束边
workflow.add_edge("generate_result", END)
```

### 2.3.5 编译和执行

```python
# 编译图
app = workflow.compile()

# 执行
result = app.invoke({
    "messages": [],
    "current_step": "init",
    "result": None
})

print(result)
```

## 2.4 工作流模式

### 2.4.1 顺序工作流

```python
from langgraph.graph import StateGraph, END, START

class SequentialState(TypedDict):
    step: int
    results: List[str]

def step_1(state: SequentialState):
    return {"step": 1, "results": ["Step 1 done"]}

def step_2(state: SequentialState):
    return {"step": 2, "results": ["Step 2 done"]}

def step_3(state: SequentialState):
    return {"step": 3, "results": ["Step 3 done"]}

workflow = StateGraph(SequentialState)

# 添加节点
workflow.add_node("step_1", step_1)
workflow.add_node("step_2", step_2)
workflow.add_node("step_3", step_3)

# 顺序边
workflow.add_edge(START, "step_1")
workflow.add_edge("step_1", "step_2")
workflow.add_edge("step_2", "step_3")
workflow.add_edge("step_3", END)

app = workflow.compile()
result = app.invoke({"step": 0, "results": []})
```

### 2.4.2 循环工作流

```python
class LoopState(TypedDict):
    counter: int
    max_iterations: int
    results: List[str]

def process_item(state: LoopState):
    return {
        "counter": state["counter"] + 1,
        "results": [f"Iteration {state['counter']}"]
    }

def should_continue(state: LoopState):
    if state["counter"] < state["max_iterations"]:
        return "continue"
    return "end"

workflow = StateGraph(LoopState)
workflow.add_node("process_item", process_item)
workflow.set_entry_point("process_item")

# 添加循环边
workflow.add_conditional_edges(
    "process_item",
    should_continue,
    {
        "continue": "process_item",  # 回到自身
        "end": END
    }
)

app = workflow.compile()
result = app.invoke({"counter": 0, "max_iterations": 3, "results": []})
```

### 2.4.3 分支工作流

```python
class BranchState(TypedDict):
    input_type: str
    result: str

def classify_input(state: BranchState):
    # 分类逻辑
    if state["input_type"].startswith("query"):
        return {"input_type": "query"}
    elif state["input_type"].startswith("command"):
        return {"input_type": "command"}
    return {"input_type": "unknown"}

def handle_query(state: BranchState):
    return {"result": "Query handled"}

def handle_command(state: BranchState):
    return {"result": "Command handled"}

def handle_unknown(state: BranchState):
    return {"result": "Unknown input"}

def route_input(state: BranchState):
    return state["input_type"]

workflow = StateGraph(BranchState)
workflow.add_node("classify", classify_input)
workflow.add_node("query_handler", handle_query)
workflow.add_node("command_handler", handle_command)
workflow.add_node("unknown_handler", handle_unknown)

workflow.set_entry_point("classify")

workflow.add_conditional_edges(
    "classify",
    route_input,
    {
        "query": "query_handler",
        "command": "command_handler",
        "unknown": "unknown_handler"
    }
)

workflow.add_edge("query_handler", END)
workflow.add_edge("command_handler", END)
workflow.add_edge("unknown_handler", END)

app = workflow.compile()
```

### 2.4.4 并行工作流

```python
from typing import TypedDict, List
from langgraph.graph import StateGraph, END, START

class ParallelState(TypedDict):
    input: str
    result_a: str
    result_b: str
    result_c: str
    final: str

def process_a(state: ParallelState):
    return {"result_a": f"A: {state['input']}"}

def process_b(state: ParallelState):
    return {"result_b": f"B: {state['input']}"}

def process_c(state: ParallelState):
    return {"result_c": f"C: {state['input']}"}

def combine_results(state: ParallelState):
    return {
        "final": f"{state['result_a']} | {state['result_b']} | {state['result_c']}"
    }

workflow = StateGraph(ParallelState)

# 添加节点
workflow.add_node("process_a", process_a)
workflow.add_node("process_b", process_b)
workflow.add_node("process_c", process_c)
workflow.add_node("combine", combine_results)

# 并行分支
workflow.add_edge(START, "process_a")
workflow.add_edge(START, "process_b")
workflow.add_edge(START, "process_c")

# 汇合
workflow.add_edge("process_a", "combine")
workflow.add_edge("process_b", "combine")
workflow.add_edge("process_c", "combine")
workflow.add_edge("combine", END)

app = workflow.compile()
result = app.invoke({"input": "test"})
```

## 2.5 完整示例

### 2.5.1 文档处理工作流

```python
from langgraph.graph import StateGraph, END, START
from typing import TypedDict, List, Optional

class DocumentState(TypedDict):
    document: str
    chunks: List[str]
    processed_chunks: List[str]
    summary: Optional[str]
    status: str
    error: Optional[str]

def split_document(state: DocumentState):
    """分割文档"""
    doc = state["document"]
    # 简单分割
    chunks = [doc[i:i+100] for i in range(0, len(doc), 100)]
    return {"chunks": chunks, "status": "split"}

def process_chunks(state: DocumentState):
    """处理每个块"""
    processed = []
    for chunk in state["chunks"]:
        processed.append(chunk.upper())  # 模拟处理
    return {"processed_chunks": processed, "status": "processed"}

def generate_summary(state: DocumentState):
    """生成摘要"""
    total = len(state["processed_chunks"])
    return {
        "summary": f"Processed {total} chunks",
        "status": "completed"
    }

def validate_state(state: DocumentState):
    """验证状态"""
    if not state["document"]:
        return {"status": "error", "error": "No document provided"}
    return {"status": "valid"}

def route_after_validation(state: DocumentState):
    if state["status"] == "error":
        return "error"
    return "continue"

# 构建工作流
workflow = StateGraph(DocumentState)

# 添加节点
workflow.add_node("validate", validate_state)
workflow.add_node("split", split_document)
workflow.add_node("process", process_chunks)
workflow.add_node("summarize", generate_summary)

# 设置流程
workflow.add_edge(START, "validate")

workflow.add_conditional_edges(
    "validate",
    route_after_validation,
    {
        "error": END,
        "continue": "split"
    }
)

workflow.add_edge("split", "process")
workflow.add_edge("process", "summarize")
workflow.add_edge("summarize", END)

# 编译
app = workflow.compile()

# 执行
result = app.invoke({
    "document": "This is a long document that needs to be processed..." * 10,
    "chunks": [],
    "processed_chunks": [],
    "summary": None,
    "status": "init",
    "error": None
})

print(result["summary"])
```

### 2.5.2 多阶段审批工作流

```python
from langgraph.graph import StateGraph, END, START
from typing import TypedDict, List

class ApprovalState(TypedDict):
    request_id: str
    request_type: str
    amount: float
    approvals: List[str]
    current_stage: str
    status: str

def initial_review(state: ApprovalState):
    """初审"""
    if state["amount"] < 1000:
        return {
            "current_stage": "approved",
            "status": "auto_approved",
            "approvals": ["auto_approval"]
        }
    return {"current_stage": "manager_review"}

def manager_review(state: ApprovalState):
    """经理审批"""
    # 模拟审批
    approved = state["amount"] < 10000
    if approved:
        return {
            "current_stage": "approved",
            "approvals": ["manager"]
        }
    return {"current_stage": "director_review"}

def director_review(state: ApprovalState):
    """总监审批"""
    approved = state["amount"] < 50000
    if approved:
        return {
            "current_stage": "approved",
            "approvals": ["director"]
        }
    return {
        "current_stage": "rejected",
        "status": "amount_exceeds_limit"
    }

def finalize(state: ApprovalState):
    """最终处理"""
    if state["current_stage"] == "approved":
        return {"status": "approved"}
    return {"status": "rejected"}

def route_request(state: ApprovalState):
    if state["current_stage"] == "approved":
        return "approved"
    elif state["current_stage"] == "rejected":
        return "rejected"
    return state["current_stage"]

# 构建工作流
workflow = StateGraph(ApprovalState)

workflow.add_node("initial_review", initial_review)
workflow.add_node("manager_review", manager_review)
workflow.add_node("director_review", director_review)
workflow.add_node("finalize", finalize)

workflow.add_edge(START, "initial_review")

workflow.add_conditional_edges(
    "initial_review",
    route_request,
    {
        "approved": "finalize",
        "manager_review": "manager_review"
    }
)

workflow.add_conditional_edges(
    "manager_review",
    route_request,
    {
        "approved": "finalize",
        "director_review": "director_review"
    }
)

workflow.add_conditional_edges(
    "director_review",
    route_request,
    {
        "approved": "finalize",
        "rejected": "finalize"
    }
)

workflow.add_edge("finalize", END)

app = workflow.compile()

# 测试不同金额
for amount in [500, 5000, 15000, 60000]:
    result = app.invoke({
        "request_id": f"REQ_{amount}",
        "request_type": "expense",
        "amount": float(amount),
        "approvals": [],
        "current_stage": "init",
        "status": "pending"
    })
    print(f"Amount: {amount} -> Status: {result['status']}, Approvals: {result['approvals']}")
```

## 2.6 可视化

### 2.6.1 ASCII 图

```python
app.get_graph().print_ascii()
```

### 2.6.2 Mermaid 图

```python
print(app.get_graph().to_mermaid())
```

### 2.6.3 生成图片

```python
from langgraph.graph import StateGraph

# 生成图片
img_data = app.get_graph().draw_mermaid_png()
with open("graph.png", "wb") as f:
    f.write(img_data)
```

## 2.7 总结

本章介绍了 LangGraph 中的状态机和工作流：

1. **状态定义**：使用 TypedDict 定义状态结构
2. **状态 Reducer**：使用 Annotated 定义状态更新策略
3. **图构建**：创建 StateGraph、添加节点和边
4. **工作流模式**：顺序、循环、分支、并行

**最佳实践**：
- 清晰定义状态结构
- 使用条件边处理复杂流程
- 合理使用状态 Reducer
- 使用可视化工具调试

---

*下一章我们将深入学习节点与边的定义。*
