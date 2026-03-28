# 第六章：人机交互

## 6.1 人机交互概述

LangGraph 支持在执行过程中与用户进行交互，实现需要人类确认或输入的工作流。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        人机交互流程                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Agent 执行                    用户交互                              │
│   ────────                    ────────                             │
│                                                                      │
│   ┌─────────┐                ┌─────────────┐                       │
│   │ Node A  │                │             │                       │
│   └────┬────┘                │             │                       │
│        │                      │             │                       │
│        │  中断执行            │             │                       │
│        ▼                      │             │                       │
│   ┌─────────┐                │             │                       │
│   │interrupt│ ──────────────▶│ 等待确认    │                       │
│   │ (暂停)  │                │             │                       │
│   └─────────┘                └──────┬──────┘                       │
│                                     │                                │
│                                     │ 用户确认                       │
│                                     ▼                                │
│                              ┌─────────────┐                       │
│                              │   恢复执行   │                       │
│                              └──────┬──────┘                       │
│                                     │                                │
│                                     ▼                                │
│   ┌─────────┐                ┌─────────────┐                       │
│   │ Node B  │◀───────────────│ Resume      │                       │
│   └─────────┘                └─────────────┘                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 6.2 interrupt 中断

### 6.2.1 基本中断

```python
from langgraph.types import interrupt
from langgraph.graph import StateGraph, END, START

class State(TypedDict):
    value: str

def ask_user(state: State) -> dict:
    """询问用户"""
    result = interrupt({
        "question": "请确认操作是否继续？",
        "options": ["yes", "no"]
    })
    return {"confirmed": result.get("confirmed", False)}

def process_if_confirmed(state: State) -> dict:
    """处理确认"""
    if state.get("confirmed"):
        return {"value": "processed"}
    return {"value": "cancelled"}

workflow = StateGraph(State)
workflow.add_node("ask", ask_user)
workflow.add_node("process", process_if_confirmed)
workflow.add_edge(START, "ask")
workflow.add_edge("ask", "process")
workflow.add_edge("process", END)

app = workflow.compile(checkpointer=MemorySaver())

# 执行（会中断）
config = {"configurable": {"thread_id": "1"}}
app.invoke({"value": "start"}, config=config)

# 用户确认后恢复
app.invoke({"confirmed": True}, config=config)
```

### 6.2.2 获取中断信息

```python
# 检查当前状态
state = app.get_state(config)
print(f"中断位置: {state.next}")
print(f"当前值: {state.values}")
```

## 6.3 用户确认流程

### 6.3.1 确认节点

```python
def confirmation_node(state: State) -> dict:
    """需要确认的节点"""
    return interrupt({
        "type": "confirmation",
        "message": "确认执行以下操作？",
        "details": state.get("pending_action")
    })

def execute_action(state: State) -> dict:
    """执行操作"""
    return {"status": "executed", "result": f"executed: {state.get('pending_action')}"}

def cancel_action(state: State) -> dict:
    """取消操作"""
    return {"status": "cancelled"}
```

### 6.3.2 确认后路由

```python
def route_after_confirm(state: State) -> Literal["execute", "cancel"]:
    if state.get("user_confirmed"):
        return "execute"
    return "cancel"

workflow.add_node("confirm", confirmation_node)
workflow.add_node("execute", execute_action)
workflow.add_node("cancel", cancel_action)

workflow.add_conditional_edges(
    "confirm",
    route_after_confirm,
    {"execute": "execute", "cancel": "cancel"}
)
```

## 6.4 动态输入

### 6.4.1 请求用户输入

```python
def request_input(state: State) -> dict:
    """请求用户输入"""
    result = interrupt({
        "type": "input",
        "prompt": "请输入您的名字："
    })
    return {"user_name": result.get("value", "")}

def greet_user(state: State) -> dict:
    """问候用户"""
    name = state.get("user_name", "Guest")
    return {"greeting": f"你好，{name}！"}
```

### 6.4.2 多步骤输入

```python
def collect_info(state: State) -> dict:
    """收集信息"""
    result = interrupt({
        "type": "form",
        "fields": [
            {"name": "name", "label": "姓名", "type": "text"},
            {"name": "email", "label": "邮箱", "type": "email"},
            {"name": "phone", "label": "电话", "type": "tel"}
        ]
    })
    return {"form_data": result}
```

## 6.5 完整示例

### 6.5.1 审批工作流

```python
from langgraph.graph import StateGraph, END, START
from langgraph.types import interrupt
from typing import TypedDict, List
from langgraph.checkpoint.memory import MemorySaver

class ApprovalState(TypedDict):
    request_id: str
    request_type: str
    amount: float
    requester: str
    approver: str
    status: str
    comments: str
    approved: bool

def submit_request(state: ApprovalState) -> dict:
    """提交请求"""
    return {"status": "pending_approval"}

def request_approval(state: ApprovalState) -> dict:
    """请求审批"""
    result = interrupt({
        "type": "approval",
        "title": "审批请求",
        "details": {
            "请求ID": state["request_id"],
            "类型": state["request_type"],
            "金额": state["amount"],
            "申请人": state["requester"]
        },
        "actions": ["approve", "reject"]
    })
    return {
        "approved": result.get("action") == "approve",
        "comments": result.get("comments", "")
    }

def approve_request(state: ApprovalState) -> dict:
    """批准请求"""
    return {"status": "approved", "approver": "Manager"}

def reject_request(state: ApprovalState) -> dict:
    """拒绝请求"""
    return {"status": "rejected", "approver": "Manager"}

def route_approval(state: ApprovalState) -> Literal["approve", "reject"]:
    return "approve" if state.get("approved") else "reject"

workflow = StateGraph(ApprovalState)
workflow.add_node("submit", submit_request)
workflow.add_node("request_approval", request_approval)
workflow.add_node("approve", approve_request)
workflow.add_node("reject", reject_request)

workflow.add_edge(START, "submit")
workflow.add_edge("submit", "request_approval")
workflow.add_conditional_edges(
    "request_approval",
    route_approval,
    {"approve": "approve", "reject": "reject"}
)
workflow.add_edge("approve", END)
workflow.add_edge("reject", END)

checkpointer = MemorySaver()
app = workflow.compile(checkpointer=checkpointer)

# 执行
config = {"configurable": {"thread_id": "approval_1"}}
app.invoke({
    "request_id": "REQ_001",
    "request_type": "采购",
    "amount": 5000.0,
    "requester": "张三",
    "approver": "",
    "status": "draft",
    "comments": "",
    "approved": False
}, config=config)

# 模拟审批
app.invoke({
    "approved": True,
    "comments": "同意采购"
}, config=config)
```

### 6.5.2 客服对话

```python
from langgraph.graph import StateGraph, END, START
from langgraph.types import interrupt
from typing import TypedDict, List
from langgraph.checkpoint.memory import MemorySaver

class SupportState(TypedDict):
    issue: str
    category: str
    solution: str
    user_confirmed: bool
    rating: int
    feedback: str

def collect_issue(state: SupportState) -> dict:
    """收集问题"""
    result = interrupt({
        "type": "input",
        "prompt": "请描述您遇到的问题："
    })
    return {"issue": result.get("value", "")}

def analyze_issue(state: SupportState) -> dict:
    """分析问题"""
    issue = state.get("issue", "").lower()
    
    if "网络" in issue or "wifi" in issue:
        category = "network"
    elif "账号" in issue or "登录" in issue:
        category = "account"
    elif "支付" in issue or "退款" in issue:
        category = "payment"
    else:
        category = "other"
    
    return {"category": category}

def provide_solution(state: SupportState) -> dict:
    """提供解决方案"""
    category = state.get("category")
    
    solutions = {
        "network": "请检查您的网络连接，重启路由器后重试。",
        "account": "请尝试找回密码，或联系客服。",
        "payment": "请提供订单号，我们会尽快处理。",
        "other": "我们的客服人员会尽快与您联系。"
    }
    
    return {"solution": solutions.get(category, solutions["other"])}

def confirm_solution(state: SupportState) -> dict:
    """确认解决方案"""
    result = interrupt({
        "type": "confirmation",
        "message": f"解决方案：{state.get('solution')}",
        "question": "这个方案是否解决了您的问题？"
    })
    return {"user_confirmed": result.get("confirmed", False)}

def collect_feedback(state: SupportState) -> dict:
    """收集反馈"""
    result = interrupt({
        "type": "rating",
        "title": "满意度调查",
        "question": "请对本次服务评分（1-5分）："
    })
    return {"rating": result.get("value", 0)}

def close_ticket(state: SupportState) -> dict:
    """关闭工单"""
    return {"status": "closed"}

workflow = StateGraph(SupportState)
workflow.add_node("collect", collect_issue)
workflow.add_node("analyze", analyze_issue)
workflow.add_node("solve", provide_solution)
workflow.add_node("confirm", confirm_solution)
workflow.add_node("feedback", collect_feedback)
workflow.add_node("close", close_ticket)

workflow.add_edge(START, "collect")
workflow.add_edge("collect", "analyze")
workflow.add_edge("analyze", "solve")
workflow.add_edge("solve", "confirm")
workflow.add_conditional_edges(
    "confirm",
    lambda s: "feedback" if not s.get("user_confirmed") else "close",
    {"feedback": "feedback", "close": "close"}
)
workflow.add_edge("feedback", "solve")  # 循环回到解决方案
workflow.add_edge("close", END)

checkpointer = MemorySaver()
app = workflow.compile(checkpointer=checkpointer)
```

## 6.6 总结

本章介绍了人机交互的实现：

1. **interrupt**：中断执行等待用户输入
2. **确认流程**：请求用户确认后继续
3. **动态输入**：收集用户输入信息
4. **多步骤交互**：复杂的人机对话流程

---

*下一章我们将学习多代理系统。*
