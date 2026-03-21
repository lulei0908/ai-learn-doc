# 第三章：自主研究 Agent

## 3.1 研究 Agent 概述

构建一个能够自主进行网络搜索、信息整合、报告生成的研究 Agent。

```python
from langgraph.graph import StateGraph, END, START
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict, List

class ResearchState(TypedDict):
    topic: str
    search_queries: List[str]
    search_results: List[str]
    analysis: str
    report: str
    current_step: str

def plan_research(state: ResearchState) -> dict:
    """制定研究计划"""
    topic = state["topic"]
    queries = [
        f"{topic} 定义",
        f"{topic} 应用场景",
        f"{topic} 最新进展"
    ]
    return {"search_queries": queries, "current_step": "search"}

def web_search(state: ResearchState) -> dict:
    """执行搜索"""
    queries = state["search_queries"]
    results = [f"Result for: {q}" for q in queries]
    return {"search_results": results, "current_step": "analyze"}

def analyze(state: ResearchState) -> dict:
    """分析结果"""
    results = state["search_results"]
    analysis = f"Analysis of {len(results)} sources"
    return {"analysis": analysis, "current_step": "write"}

def write_report(state: ResearchState) -> dict:
    """撰写报告"""
    analysis = state["analysis"]
    report = f"Report: {analysis}"
    return {"report": report, "current_step": "done"}

workflow = StateGraph(ResearchState)
workflow.add_node("plan", plan_research)
workflow.add_node("search", web_search)
workflow.add_node("analyze", analyze)
workflow.add_node("write", write_report)

workflow.add_edge(START, "plan")
workflow.add_edge("plan", "search")
workflow.add_edge("search", "analyze")
workflow.add_edge("analyze", "write")
workflow.add_edge("write", END)

app = workflow.compile(checkpointer=MemorySaver())

result = app.invoke({
    "topic": "人工智能",
    "search_queries": [],
    "search_results": [],
    "analysis": "",
    "report": "",
    "current_step": "init"
})

print(result["report"])
```

## 3.2 总结

本章介绍了自主研究 Agent 的实现：
- 研究计划制定
- 网络搜索
- 信息分析
- 报告生成
