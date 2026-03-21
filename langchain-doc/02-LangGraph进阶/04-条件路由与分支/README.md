# 第四章：条件路由与分支

## 4.1 条件路由基础

条件路由允许根据状态动态决定下一步执行哪个节点，实现复杂的流程控制。

```python
from typing import TypedDict, Literal
from langgraph.graph import StateGraph, END, START

class RouteState(TypedDict):
    value: int
    route: str

def determine_route(state: RouteState) -> Literal["low", "medium", "high"]:
    """根据值决定路由"""
    if state["value"] < 10:
        return "low"
    elif state["value"] < 20:
        return "medium"
    return "high"

# 创建图
workflow = StateGraph(RouteState)

# 添加节点
workflow.add_node("process_low", lambda s: {"route": "processed_low"})
workflow.add_node("process_medium", lambda s: {"route": "processed_medium"})
workflow.add_node("process_high", lambda s: {"route": "processed_high"})

# 设置入口
workflow.add_edge(START, "determine")

# 添加条件边
workflow.add_conditional_edges(
    "determine",
    determine_route,
    {
        "low": "process_low",
        "medium": "process_medium",
        "high": "process_high"
    }
)

# 添加结束边
workflow.add_edge("process_low", END)
workflow.add_edge("process_medium", END)
workflow.add_edge("process_high", END)
```

## 4.2 路由函数

### 4.2.1 简单路由

```python
def simple_router(state: RouteState) -> str:
    """直接返回目标节点名称"""
    return "node_b" if state["value"] > 5 else "node_a"
```

### 4.2.2 枚举路由

```python
from typing import Literal

def enum_router(state: RouteState) -> Literal["a", "b", "c"]:
    """枚举类型返回"""
    if state["value"] < 5:
        return "a"
    elif state["value"] < 15:
        return "b"
    return "c"
```

### 4.2.3 多条件路由

```python
def multi_condition_router(state: RouteState) -> str:
    """多条件路由"""
    value = state.get("value", 0)
    category = state.get("category", "default")
    
    if category == "error":
        return "error_handler"
    if value < 0:
        return "negative_handler"
    if value == 0:
        return "zero_handler"
    if value > 100:
        return "overflow_handler"
    return "normal_handler"
```

## 4.3 分支模式

### 4.3.1 二元分支

```python
def should_continue(state: RouteState) -> Literal["yes", "no"]:
    return "yes" if state["value"] < 10 else "no"

workflow.add_conditional_edges(
    "check",
    should_continue,
    {"yes": "continue_node", "no": "stop_node"}
)
```

### 4.3.2 多元分支

```python
def classify(state: RouteState) -> Literal["a", "b", "c", "d"]:
    categories = ["a", "b", "c", "d"]
    index = state["value"] % 4
    return categories[index]

workflow.add_conditional_edges(
    "classify",
    classify,
    {"a": "node_a", "b": "node_b", "c": "node_c", "d": "node_d"}
)
```

### 4.3.3 动态分支

```python
def dynamic_router(state: RouteState) -> list:
    """返回多个目标节点"""
    if state["value"] > 5:
        return ["validate", "log"]
    return ["log"]

# LangGraph 不直接支持多目标，使用并行节点处理
```

## 4.4 嵌套条件

### 4.4.1 多层条件

```python
workflow.add_conditional_edges(
    "level1_node",
    level1_router,
    {
        "path_a": "level2a_node",
        "path_b": "level2b_node"
    }
)

workflow.add_conditional_edges(
    "level2a_node",
    level2_router,
    {
        "path_1": "final_node_1",
        "path_2": "final_node_2"
    }
)
```

### 4.4.2 条件中的状态更新

```python
def router_with_update(state: RouteState) -> Literal["a", "b"]:
    # 在路由的同时更新状态
    new_value = state["value"] * 2
    
    # 返回值会作为 next_route 键
    return {"next_route": "a" if new_value < 10 else "b"}
```

## 4.5 完整示例

### 4.5.1 订单处理流程

```python
from langgraph.graph import StateGraph, END, START
from typing import TypedDict, List, Optional
from datetime import datetime

class OrderState(TypedDict):
    order_id: str
    amount: float
    status: str
    verification_result: Optional[dict]
    approval_result: Optional[dict]
    fulfillment_result: Optional[dict]
    errors: List[str]

def validate_order(state: OrderState) -> dict:
    """验证订单"""
    if state["amount"] <= 0:
        return {"status": "invalid", "errors": ["金额必须大于0"]}
    return {"status": "validated", "verification_result": {"valid": True}}

def verify_risk(state: OrderState) -> dict:
    """风险评估"""
    if state["amount"] > 10000:
        return {"verification_result": {"risk": "high", "requires_approval": True}}
    return {"verification_result": {"risk": "low", "requires_approval": False}}

def route_after_verification(state: OrderState) -> Literal["approve", "reject", "fulfill"]:
    """验证后路由"""
    if state.get("errors"):
        return "reject"
    if state["verification_result"].get("requires_approval"):
        return "approve"
    return "fulfill"

def approve_order(state: OrderState) -> dict:
    """审批订单"""
    if state["amount"] > 50000:
        return {"status": "rejected", "errors": ["金额超限"]}
    return {"status": "approved", "approval_result": {"approved": True}}

def fulfill_order(state: OrderState) -> dict:
    """履行订单"""
    return {"status": "fulfilled", "fulfillment_result": {"shipped": True}}

def handle_error(state: OrderState) -> dict:
    """错误处理"""
    return {"status": "failed"}

# 构建工作流
workflow = StateGraph(OrderState)

workflow.add_node("validate", validate_order)
workflow.add_node("verify_risk", verify_risk)
workflow.add_node("approve", approve_order)
workflow.add_node("fulfill", fulfill_order)
workflow.add_node("error", handle_error)

workflow.add_edge(START, "validate")
workflow.add_edge("validate", "verify_risk")

workflow.add_conditional_edges(
    "verify_risk",
    route_after_verification,
    {
        "approve": "approve",
        "fulfill": "fulfill",
        "reject": "error"
    }
)

workflow.add_edge("approve", END)
workflow.add_edge("fulfill", END)
workflow.add_edge("error", END)

app = workflow.compile()

# 测试
for amount in [100, 5000, 20000, 60000]:
    result = app.invoke({
        "order_id": f"ORD_{amount}",
        "amount": float(amount),
        "status": "pending",
        "verification_result": None,
        "approval_result": None,
        "fulfillment_result": None,
        "errors": []
    })
    print(f"金额: {amount} -> 状态: {result['status']}")
```

### 4.5.2 智能客服路由

```python
from langgraph.graph import StateGraph, END, START
from typing import TypedDict, List

class CustomerServiceState(TypedDict):
    query: str
    intent: Optional[str]
    sentiment: Optional[str]
    handler: Optional[str]
    response: Optional[str]
   满意度: Optional[int]

def classify_intent(state: CustomerServiceState) -> dict:
    """意图分类"""
    query = state["query"].lower()
    
    if "订单" in query or "shipping" in query:
        intent = "order"
    elif "退款" in query or "refund" in query:
        intent = "refund"
    elif "产品" in query or "product" in query:
        intent = "product"
    elif "技术" in query or "technical" in query:
        intent = "technical"
    else:
        intent = "general"
    
    return {"intent": intent}

def analyze_sentiment(state: CustomerServiceState) -> dict:
    """情感分析"""
    # 简化示例
    if "!" in state["query"] or "!!" in state["query"]:
        sentiment = "urgent"
    else:
        sentiment = "normal"
    return {"sentiment": sentiment}

def route_intent(state: CustomerServiceState) -> Literal["order", "refund", "product", "technical", "general"]:
    return state["intent"]

def handle_order(state: CustomerServiceState) -> dict:
    return {"response": "订单相关问题", "handler": "order_team"}

def handle_refund(state: CustomerServiceState) -> dict:
    return {"response": "退款相关问题", "handler": "refund_team"}

def handle_product(state: CustomerServiceState) -> dict:
    return {"response": "产品相关问题", "handler": "product_team"}

def handle_technical(state: CustomerServiceState) -> dict:
    return {"response": "技术相关问题", "handler": "technical_team"}

def handle_general(state: CustomerServiceState) -> dict:
    return {"response": "一般问题", "handler": "general_team"}

def urgent_handler(state: CustomerServiceState) -> dict:
    """紧急情况处理"""
    return {"response": "您的请求已加急处理", "handler": "priority"}

def route_sentiment(state: CustomerServiceState) -> Literal["urgent", "normal"]:
    return state.get("sentiment", "normal")

# 构建工作流
workflow = StateGraph(CustomerServiceState)

workflow.add_node("classify", classify_intent)
workflow.add_node("analyze_sentiment", analyze_sentiment)
workflow.add_node("order", handle_order)
workflow.add_node("refund", handle_refund)
workflow.add_node("product", handle_product)
workflow.add_node("technical", handle_technical)
workflow.add_node("general", handle_general)
workflow.add_node("urgent", urgent_handler)

workflow.add_edge(START, "classify")
workflow.add_edge("classify", "analyze_sentiment")

workflow.add_conditional_edges(
    "analyze_sentiment",
    route_sentiment,
    {"urgent": "urgent", "normal": "route_intent"}
)

workflow.add_conditional_edges(
    "route_intent",
    route_intent,
    {
        "order": "order",
        "refund": "refund",
        "product": "product",
        "technical": "technical",
        "general": "general"
    }
)

workflow.add_edge("order", END)
workflow.add_edge("refund", END)
workflow.add_edge("product", END)
workflow.add_edge("technical", END)
workflow.add_edge("general", END)
workflow.add_edge("urgent", END)

app = workflow.compile()

# 测试
result = app.invoke({
    "query": "我的订单什么时候发货？？",
    "intent": None,
    "sentiment": None,
    "handler": None,
    "response": None,
    "满意度": None
})
print(f"意图: {result['intent']}, 处理: {result['handler']}")
```

## 4.6 总结

本章介绍了条件路由与分支的用法：

1. **条件路由函数**：根据状态返回目标节点
2. **路由类型**：简单路由、枚举路由、多条件路由
3. **分支模式**：二元分支、多元分支、动态分支
4. **嵌套条件**：多层条件嵌套

---

*下一章我们将学习持久化与检查点。*
