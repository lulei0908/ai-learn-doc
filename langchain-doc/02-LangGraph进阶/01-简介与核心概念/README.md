# 第一章：LangGraph 简介与核心概念

## 1.1 LangGraph 概述

LangGraph 是 LangChain 的扩展，专门用于构建**有状态**、**多步骤**的 LLM 应用。它通过图结构来定义复杂的工作流，支持循环、条件分支、人机交互等高级流程控制。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        LangChain vs LangGraph                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   LangChain                      LangGraph                           │
│   ────────                      ──────────                          │
│                                                                      │
│   线性流程                        图结构流程                          │
│   ┌───┐    ┌───┐    ┌───┐        ┌───┐                            │
│   │ A │ →  │ B │ →  │ C │        │ A │                            │
│   └───┘    └───┘    └───┘        └─┬─┘                            │
│                                     │                               │
│                                    ▼                                │
│   单一输入/输出                     ┌───┐                            │
│                                     │ B │                            │
│   ┌─────────────┐                  └─┬─┘                            │
│   │  prompt     │                    │                              │
│   │    ↓       │                     ▼                              │
│   │    llm     │                    ┌───┐    ┌───┐                 │
│   │    ↓       │                    │ C │ ←  │ D │                 │
│   │   output   │                    └───┘    └───┘                 │
│   └─────────────┘                       ↑                            │
│                                     循环                              │
│   无状态                              │                              │
│                               支持条件/循环/状态                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 1.2 核心概念

### 1.2.1 图 (Graph)

LangGraph 使用**有向图**来表示工作流：

```python
from langgraph.graph import StateGraph, END

# 创建图
graph = StateGraph(dict)  # 定义状态类型

# 添加节点
graph.add_node("node_name", function)

# 添加边
graph.add_edge("start", "node_name")
graph.add_edge("node_name", END)

# 编译
app = graph.compile()
```

### 1.2.2 节点 (Node)

节点是图中的基本执行单元：

```python
def my_node(state):
    """节点函数"""
    # 处理状态
    return {"result": "processed"}

graph.add_node("my_node", my_node)
```

### 1.2.3 边 (Edge)

边定义节点之间的连接：

```python
# 普通边（顺序执行）
graph.add_edge("node_a", "node_b")

# 条件边（条件分支）
graph.add_conditional_edges(
    "node_a",
    routing_function,
    {
        "path_a": "node_b",
        "path_b": "node_c"
    }
)
```

### 1.2.4 状态 (State)

状态在节点之间传递：

```python
from typing import TypedDict

class AgentState(TypedDict):
    """应用状态"""
    messages: list
    next_action: str
    result: dict

# 使用状态
def node_function(state: AgentState) -> AgentState:
    # 读取状态
    messages = state["messages"]
    
    # 更新状态
    return {"result": {"data": "new"}}
```

## 1.3 与 LangChain 的关系

### 1.3.1 集成方式

```python
# LangGraph 可以使用 LangChain 的组件
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool

# 在 LangGraph 节点中使用
llm = ChatOpenAI(model="gpt-4")

def call_llm(state: AgentState):
    prompt = ChatPromptTemplate.from_template("{question}")
    chain = prompt | llm
    response = chain.invoke({"question": state["question"]})
    return {"answer": response.content}
```

### 1.3.2 选择 LangChain 还是 LangGraph？

| 场景 | 推荐 |
|:---|:---|
| 简单问答 | LangChain |
| 固定流程 | LangChain |
| 需要循环 | LangGraph |
| 复杂状态管理 | LangGraph |
| 多代理协作 | LangGraph |
| 人机交互 | LangGraph |

## 1.4 安装

```bash
pip install langgraph
```

## 1.5 第一个 LangGraph 应用

### 1.5.1 简单流程

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

# 定义状态
class FlowState(TypedDict):
    messages: list
    count: int

# 定义节点
def node_a(state: FlowState):
    return {"messages": state["messages"] + ["Node A completed"], "count": state["count"] + 1}

def node_b(state: FlowState):
    return {"messages": state["messages"] + ["Node B completed"], "count": state["count"] + 1}

def node_c(state: FlowState):
    return {"messages": state["messages"] + ["Node C completed"], "count": state["count"] + 1}

# 创建图
workflow = StateGraph(FlowState)

# 添加节点
workflow.add_node("node_a", node_a)
workflow.add_node("node_b", node_b)
workflow.add_node("node_c", node_c)

# 设置入口
workflow.set_entry_point("node_a")

# 添加边
workflow.add_edge("node_a", "node_b")
workflow.add_edge("node_b", "node_c")
workflow.add_edge("node_c", END)

# 编译
app = workflow.compile()

# 执行
result = app.invoke({
    "messages": [],
    "count": 0
})

print(result)
```

### 1.5.2 可视化

```python
# 生成图形
app.get_graph().print_ascii()
# 或使用 Mermaid
app.get_graph().to_mermaid()
```

## 1.6 核心特性

### 1.6.1 循环支持

```python
def should_continue(state: FlowState):
    """判断是否继续循环"""
    if state["count"] < 3:
        return "continue"
    return "end"

workflow.add_conditional_edges(
    "node_a",
    should_continue,
    {
        "continue": "node_a",  # 循环回到 node_a
        "end": END
    }
)
```

### 1.6.2 条件分支

```python
def route_based_on_input(state: FlowState):
    """根据输入路由"""
    message = state["messages"][-1] if state["messages"] else ""
    
    if "error" in message:
        return "error_handler"
    elif "question" in message:
        return "answer_node"
    else:
        return "default_node"

workflow.add_conditional_edges(
    "process_node",
    route_based_on_input,
    {
        "error_handler": "error_handler",
        "answer_node": "answer_node",
        "default_node": "default_node"
    }
)
```

### 1.6.3 状态管理

```python
from typing import TypedDict, Annotated
import operator

class AdvancedState(TypedDict):
    messages: list
    count: int
    data: dict
    # 使用 reducer 合并列表
    results: Annotated[list, operator.add]

def node_with_reducer(state: AdvancedState):
    # 使用 reducer 自动合并结果
    return {"results": ["new_result"]}
```

## 1.7 总结

本章介绍了 LangGraph 的基本概念：

1. **图结构**：使用有向图定义工作流
2. **节点**：基本执行单元
3. **边**：节点间的连接
4. **状态**：节点间传递的数据
5. **循环与条件**：支持复杂流程控制

LangGraph 是构建复杂 LLM 应用的核心框架，特别适合需要状态管理、多步骤工作流、人机交互的场景。

---

*下一章我们将深入学习 LangGraph 的状态机与工作流。*
