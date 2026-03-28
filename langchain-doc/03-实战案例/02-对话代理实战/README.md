# 第二章：对话代理实战

## 2.1 对话代理概述

构建一个完整的对话代理，支持多轮对话、上下文记忆、工具调用等功能。

```python
from langgraph.graph import StateGraph, END, START
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict, List
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage

class ConversationState(TypedDict):
    messages: List
    current_intent: str
    context: dict

def process_message(state: ConversationState) -> dict:
    """处理用户消息"""
    messages = state["messages"]
    
    # 判断意图
    last_message = messages[-1].content if messages else ""
    
    if "查询" in last_message:
        intent = "query"
    elif "预订" in last_message:
        intent = "booking"
    else:
        intent = "general"
    
    return {"current_intent": intent}

def handle_query(state: ConversationState) -> dict:
    """处理查询"""
    return {"context": {"type": "query_result"}}

def handle_booking(state: ConversationState) -> dict:
    """处理预订"""
    return {"context": {"type": "booking_confirmed"}}

def generate_response(state: ConversationState) -> dict:
    """生成回复"""
    intent = state.get("current_intent", "general")
    messages = state["messages"]
    
    response = f"这是对 {intent} 的回复"
    return {"messages": messages + [AIMessage(content=response)]}

workflow = StateGraph(ConversationState)
workflow.add_node("process", process_message)
workflow.add_node("handle_query", handle_query)
workflow.add_node("handle_booking", handle_booking)
workflow.add_node("generate", generate_response)

workflow.add_edge(START, "process")
workflow.add_edge("process", "handle_query")
workflow.add_edge("handle_query", "generate")
workflow.add_edge("handle_booking", "generate")
workflow.add_edge("generate", END)

checkpointer = MemorySaver()
app = workflow.compile(checkpointer=checkpointer)
```

## 2.2 总结

本章介绍了对话代理的实现：
- 多轮对话管理
- 意图识别
- 上下文记忆
