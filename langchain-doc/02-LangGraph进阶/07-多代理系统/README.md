# 第七章：多代理系统

## 7.1 多代理概述

LangGraph 支持构建多代理系统，多个代理协作完成复杂任务。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        多代理系统架构                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                        ┌──────────────┐                             │
│                        │   Orchestrator │                            │
│                        │   (协调器)    │                             │
│                        └──────┬───────┘                             │
│                               │                                      │
│          ┌───────────────────┼───────────────────┐                 │
│          │                   │                   │                   │
│          ▼                   ▼                   ▼                 │
│   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐           │
│   │   Researcher │   │   Writer     │   │   Critic     │           │
│   │   (研究代理)  │   │   (写作代理)  │   │   (评论代理)  │           │
│   └──────────────┘   └──────────────┘   └──────────────┘           │
│          │                   │                   │                   │
│          └───────────────────┴───────────────────┘                 │
│                               │                                      │
│                               ▼                                      │
│                        ┌──────────────┐                             │
│                        │   Output     │                             │
│                        │   (最终输出)  │                             │
│                        └──────────────┘                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 7.2 代理通信

### 7.2.1 共享状态通信

```python
from langgraph.graph import StateGraph, END, START
from typing import TypedDict, List

class MultiAgentState(TypedDict):
    messages: List[str]
    research_results: str
    writing_results: str
    feedback: str
    current_agent: str

def researcher_node(state: MultiAgentState) -> dict:
    """研究代理"""
    return {
        "research_results": f"Research on: {state.get('messages', [])}",
        "current_agent": "researcher"
    }

def writer_node(state: MultiAgentState) -> dict:
    """写作代理"""
    research = state.get("research_results", "")
    return {
        "writing_results": f"Written based on: {research}",
        "current_agent": "writer"
    }

def critic_node(state: MultiAgentState) -> dict:
    """评论代理"""
    writing = state.get("writing_results", "")
    return {
        "feedback": f"Feedback: {writing}",
        "current_agent": "critic"
    }

workflow = StateGraph(MultiAgentState)
workflow.add_node("researcher", researcher_node)
workflow.add_node("writer", writer_node)
workflow.add_node("critic", critic_node)

workflow.add_edge(START, "researcher")
workflow.add_edge("researcher", "writer")
workflow.add_edge("writer", "critic")
workflow.add_edge("critic", END)

app = workflow.compile()
```

### 7.2.2 消息传递

```python
class MessageState(TypedDict):
    messages: List[dict]  # {"from": "agent", "content": "..."}

def agent_a(state: MessageState) -> dict:
    return {
        "messages": state["messages"] + [
            {"from": "agent_a", "content": "Message from A"}
        ]
    }

def agent_b(state: MessageState) -> dict:
    # 读取来自 agent_a 的消息
    messages_from_a = [m for m in state["messages"] if m["from"] == "agent_a"]
    return {
        "messages": state["messages"] + [
            {"from": "agent_b", "content": f"Responding to {len(messages_from_a)} messages"}
        ]
    }
```

## 7.3 代理编排

### 7.3.1 协调器模式

```python
from typing import Literal

class OrchestratorState(TypedDict):
    task: str
    sub_tasks: List[dict]
    results: dict
    status: str

def orchestrator_node(state: OrchestratorState) -> dict:
    """协调器分解任务"""
    task = state["task"]
    # 分解为子任务
    sub_tasks = [
        {"id": "1", "description": f"Task 1 for {task}"},
        {"id": "2", "description": f"Task 2 for {task}"},
        {"id": "3", "description": f"Task 3 for {task}"}
    ]
    return {"sub_tasks": sub_tasks, "status": "dispatched"}

def route_to_subtasks(state: OrchestratorState) -> Literal["subtask_1", "subtask_2", "subtask_3"]:
    return "subtask_1"

def subtask_1(state: OrchestratorState) -> dict:
    results = state.get("results", {})
    results["subtask_1"] = "Result 1"
    return {"results": results}

def subtask_2(state: OrchestratorState) -> dict:
    results = state.get("results", {})
    results["subtask_2"] = "Result 2"
    return {"results": results}

def subtask_3(state: OrchestratorState) -> dict:
    results = state.get("results", {})
    results["subtask_3"] = "Result 3"
    return {"results": results}

def aggregate_results(state: OrchestratorState) -> dict:
    return {"status": "completed", "results": state.get("results", {})}
```

### 7.3.2 层级代理

```python
class HierarchicalState(TypedDict):
    root_task: str
    current_depth: int
    max_depth: int
    results: dict

def supervisor(state: HierarchicalState) -> dict:
    """主管代理"""
    if state["current_depth"] >= state["max_depth"]:
        return {"status": "max_depth_reached"}
    
    return {"current_depth": state["current_depth"] + 1}

def route_depth(state: HierarchicalState) -> Literal["continue", "stop"]:
    return "continue" if state["current_depth"] < state["max_depth"] else "stop"
```

## 7.4 完整示例

### 7.4.1 研究团队

```python
from langgraph.graph import StateGraph, END, START
from typing import TypedDict, List
from langgraph.checkpoint.memory import MemorySaver

class ResearchTeamState(TypedDict):
    topic: str
    search_queries: List[str]
    search_results: List[str]
    analysis: str
    report: str
    review: str
    current_phase: str

def plan_research(state: ResearchTeamState) -> dict:
    """制定研究计划"""
    topic = state["topic"]
    queries = [
        f"{topic} 定义",
        f"{topic} 应用场景",
        f"{topic} 最新进展"
    ]
    return {"search_queries": queries, "current_phase": "search"}

def web_search(state: ResearchTeamState) -> dict:
    """网络搜索"""
    queries = state.get("search_queries", [])
    results = [f"Result for: {q}" for q in queries]
    return {"search_results": results, "current_phase": "analyze"}

def analyze_results(state: ResearchTeamState) -> dict:
    """分析结果"""
    results = state.get("search_results", [])
    analysis = f"Analysis of {len(results)} sources"
    return {"analysis": analysis, "current_phase": "write"}

def write_report(state: ResearchTeamState) -> dict:
    """撰写报告"""
    analysis = state.get("analysis", "")
    report = f"Report based on: {analysis}"
    return {"report": report, "current_phase": "review"}

def review_report(state: ResearchTeamState) -> dict:
    """审查报告"""
    report = state.get("report", "")
    review = f"Review: {report[:100]}..."
    return {"review": review, "current_phase": "done"}

def route_phase(state: ResearchTeamState) -> str:
    phase = state.get("current_phase", "plan")
    routes = {
        "plan": "search",
        "search": "analyze",
        "analyze": "write",
        "write": "review",
        "review": "done"
    }
    return routes.get(phase, "done")

workflow = StateGraph(ResearchTeamState)
workflow.add_node("plan", plan_research)
workflow.add_node("search", web_search)
workflow.add_node("analyze", analyze_results)
workflow.add_node("write", write_report)
workflow.add_node("review", review_report)

workflow.add_edge(START, "plan")
workflow.add_edge("plan", "search")
workflow.add_edge("search", "analyze")
workflow.add_edge("analyze", "write")
workflow.add_edge("write", "review")
workflow.add_edge("review", END)

app = workflow.compile(checkpointer=MemorySaver())

result = app.invoke({
    "topic": "人工智能",
    "search_queries": [],
    "search_results": [],
    "analysis": "",
    "report": "",
    "review": "",
    "current_phase": "init"
})

print(f"报告: {result['report']}")
print(f"审查: {result['review']}")
```

### 7.4.2 辩论系统

```python
from typing import Literal

class DebateState(TypedDict):
    topic: str
    affirmative_args: List[str]
    negative_args: List[str]
    current_round: int
    max_rounds: int
    winner: str

def affirmative_opening(state: DebateState) -> dict:
    """正方开场"""
    topic = state["topic"]
    arg = f"支持{topic}的第一个论点"
    return {"affirmative_args": [arg], "current_round": 1}

def negative_opening(state: DebateState) -> dict:
    """反方开场"""
    topic = state["topic"]
    arg = f"反对{topic}的第一个论点"
    return {"negative_args": [arg]}

def affirmative_rebuttal(state: DebateState) -> dict:
    """正方反驳"""
    negative = state.get("negative_args", [])
    affirmative = state.get("affirmative_args", [])
    new_arg = f"正方反驳反方观点：{negative[-1] if negative else ''}"
    return {"affirmative_args": affirmative + [new_arg]}

def negative_rebuttal(state: DebateState) -> dict:
    """反方反驳"""
    affirmative = state.get("affirmative_args", [])
    negative = state.get("negative_args", [])
    new_arg = f"反方反驳正方观点：{affirmative[-1] if affirmative else ''}"
    return {"negative_args": negative + [new_arg]}

def judge_decide(state: DebateState) -> dict:
    """裁判判决"""
    affirmative = state.get("affirmative_args", [])
    negative = state.get("negative_args", [])
    
    if len(affirmative) > len(negative):
        winner = "affirmative"
    elif len(negative) > len(affirmative):
        winner = "negative"
    else:
        winner = "tie"
    
    return {"winner": winner}

def should_continue(state: DebateState) -> Literal["continue", "judge"]:
    return "continue" if state["current_round"] < state["max_rounds"] else "judge"

workflow = StateGraph(DebateState)
workflow.add_node("aff_open", affirmative_opening)
workflow.add_node("neg_open", negative_opening)
workflow.add_node("aff_rebuttal", affirmative_rebuttal)
workflow.add_node("neg_rebuttal", negative_rebuttal)
workflow.add_node("judge", judge_decide)

workflow.add_edge(START, "aff_open")
workflow.add_edge("aff_open", "neg_open")
workflow.add_edge("neg_open", "aff_rebuttal")
workflow.add_edge("aff_rebuttal", "neg_rebuttal")

workflow.add_conditional_edges(
    "neg_rebuttal",
    should_continue,
    {"continue": "aff_rebuttal", "judge": "judge"}
)

workflow.add_edge("judge", END)

app = workflow.compile()

result = app.invoke({
    "topic": "AI是否应该取代人类工作",
    "affirmative_args": [],
    "negative_args": [],
    "current_round": 0,
    "max_rounds": 3,
    "winner": ""
})

print(f"正方论点: {len(result['affirmative_args'])}")
print(f"反方论点: {len(result['negative_args'])}")
print(f"获胜方: {result['winner']}")
```

## 7.5 总结

本章介绍了多代理系统的构建：

1. **代理通信**：通过共享状态传递信息
2. **协调器模式**：主代理协调多个子代理
3. **层级代理**：多层级代理结构
4. **协作示例**：研究团队、辩论系统

---

*下一章我们将学习错误处理与恢复。*
