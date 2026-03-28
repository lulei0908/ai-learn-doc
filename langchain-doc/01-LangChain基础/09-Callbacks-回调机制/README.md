# 第九章：Callbacks - 回调机制

## 9.1 Callbacks 概述

Callbacks 是 LangChain 中的事件监听机制，允许你在链的执行过程中捕获和响应各种事件。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Callbacks 架构                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                    Chain / Agent 执行                        │  │
│   │                                                              │  │
│   │   ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐                │  │
│   │   │Start│ →  │ LLM │ →  │ Tool │ →  │End  │                │  │
│   │   └─────┘    └─────┘    └─────┘    └─────┘                │  │
│   │       ↓        ↓         ↓         ↓                        │  │
│   │       └────────┼─────────┼─────────┘                        │  │
│   │                 ↓                                           │  │
│   │   ┌─────────────────────────────────────────────────────┐   │  │
│   │   │                   Handlers                          │   │  │
│   │   │  ┌─────────┐  ┌─────────┐  ┌─────────┐             │   │  │
│   │   │  │ Console │  │  File   │  │ Custom  │             │   │  │
│   │   │  │Handler  │  │Handler  │  │ Handler │             │   │  │
│   │   │  └─────────┘  └─────────┘  └─────────┘             │   │  │
│   │   └─────────────────────────────────────────────────────┘   │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 9.2 基础使用

### 9.2.1 Console 回调

```python
from langchain_openai import ChatOpenAI
from langchain_core.callbacks import ConsoleCallbackHandler

llm = ChatOpenAI(model="gpt-4", temperature=0, callbacks=[ConsoleCallbackHandler()])

response = llm.invoke("解释回调机制")
print(response.content)
```

### 9.2.2 链中的回调

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate
from langchain.chains import LLMChain
from langchain_core.callbacks import ConsoleCallbackHandler

llm = ChatOpenAI(model="gpt-4", temperature=0)
prompt = PromptTemplate.from_template("解释 {topic}")
chain = LLMChain(llm=llm, prompt=prompt)

# 方式1：在调用时传递回调
chain.invoke(
    {"topic": "人工智能"},
    config={"callbacks": [ConsoleCallbackHandler()]}
)

# 方式2：在初始化时设置
chain_with_callback = LLMChain(
    llm=llm,
    prompt=prompt,
    callbacks=[ConsoleCallbackHandler()]
)
```

### 9.2.3 Agent 中的回调

```python
from langchain.agents import AgentExecutor, create_react_agent
from langchain_core.callbacks import ConsoleCallbackHandler

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    callbacks=[ConsoleCallbackHandler()],
    verbose=True
)

result = agent_executor.invoke({"input": "计算 2+2"})
```

## 9.3 自定义回调处理器

### 9.3.1 继承 BaseCallbackHandler

```python
from langchain_core.callbacks import BaseCallbackHandler
from langchain_core.outputs import LLMResult
from typing import Any, Dict, List, Optional

class CustomCallbackHandler(BaseCallbackHandler):
    """自定义回调处理器"""
    
    def __init__(self):
        self.llm_starts = 0
        self.llm_ends = 0
        self.tool_starts = 0
        self.tool_ends = 0
    
    def on_llm_start(
        self,
        serialized: Dict[str, Any],
        prompts: List[str],
        **kwargs
    ) -> None:
        """LLM 开始调用时触发"""
        self.llm_starts += 1
        print(f"🔄 LLM 开始调用 #{self.llm_starts}")
    
    def on_llm_end(self, response: LLMResult, **kwargs) -> None:
        """LLM 调用结束时触发"""
        self.llm_ends += 1
        print(f"✅ LLM 调用结束 #{self.llm_ends}")
    
    def on_llm_new_token(self, token: str, **kwargs) -> None:
        """LLM 生成新 token 时触发（流式输出）"""
        print(f"📝 Token: {token}", end="", flush=True)
    
    def on_tool_start(
        self,
        serialized: Dict[str, Any],
        input_str: str,
        **kwargs
    ) -> None:
        """工具开始执行时触发"""
        self.tool_starts += 1
        tool_name = serialized.get("name", "unknown")
        print(f"\n🔧 工具开始: {tool_name}")
    
    def on_tool_end(self, output: str, **kwargs) -> None:
        """工具执行结束时触发"""
        self.tool_ends += 1
        print(f"\n✅ 工具结束 #{self.tool_ends}")
        print(f"   结果: {output[:100]}...")
    
    def on_chain_start(
        self,
        serialized: Dict[str, Any],
        inputs: Dict[str, Any],
        **kwargs
    ) -> None:
        """链开始执行时触发"""
        chain_name = serialized.get("name", "unknown")
        print(f"🔗 链开始: {chain_name}")
    
    def on_chain_end(self, outputs: Dict[str, Any], **kwargs) -> None:
        """链执行结束时触发"""
        print(f"✅ 链结束")
        print(f"   输出: {outputs}")

# 使用
handler = CustomCallbackHandler()
llm = ChatOpenAI(model="gpt-4", temperature=0, callbacks=[handler])
response = llm.invoke("写一首诗")
```

### 9.3.2 详细的事件列表

```python
from langchain_core.callbacks import BaseCallbackHandler

class DetailedCallbackHandler(BaseCallbackHandler):
    """详细事件回调"""
    
    # LLM 事件
    def on_llm_start(self, serialized, prompts, **kwargs): pass
    def on_llm_created_at(self, run_id, parent_run_id, **kwargs): pass
    def on_llm_new_token(self, token, **kwargs): pass
    def on_llm_error(self, error, **kwargs): pass
    def on_llm_end(self, response, **kwargs): pass
    
    # 链事件
    def on_chain_start(self, serialized, inputs, **kwargs): pass
    def on_chain_error(self, error, **kwargs): pass
    def on_chain_end(self, outputs, **kwargs): pass
    
    # 工具事件
    def on_tool_start(self, serialized, input_str, **kwargs): pass
    def on_tool_error(self, error, **kwargs): pass
    def on_tool_end(self, output, **kwargs): pass
    
    # 文本事件
    def on_text(self, text, **kwargs): pass
    
    # Agent 事件
    def on_agent_action(self, action, **kwargs): pass
    def on_agent_finish(self, output, **kwargs): pass
    
    # 聊天模型事件
    def on_chat_model_start(self, serialized, messages, **kwargs): pass
```

## 9.4 常用回调处理器

### 9.4.1 文件日志回调

```python
from langchain_core.callbacks import FileCallbackHandler
from pathlib import Path

# 创建日志文件
log_file = Path("./logs/chain_execution.log")
log_file.parent.mkdir(exist_ok=True)

# 文件回调
file_handler = FileCallbackHandler(log_file)

chain = LLMChain(
    llm=llm,
    prompt=prompt,
    callbacks=[file_handler]
)

chain.invoke({"topic": "AI"})
```

### 9.4.2 标准输出回调

```python
from langchain_core.callbacks import StdOutCallbackHandler

# 标准输出回调（详细模式）
stdout_handler = StdOutCallbackHandler()
```

### 9.4.3 LangSmith 回调

```python
from langchain_community.callbacks import LangChainCallbackHandler

# LangSmith 追踪
langsmith_handler = LangChainCallbackHandler(
    project_name="my-project",
    api_key="your-langsmith-api-key"
)

chain = LLMChain(
    llm=llm,
    prompt=prompt,
    callbacks=[langsmith_handler]
)
```

### 9.4.4 Tracability 回调

```python
from langchain_core.callbacks import get_openai_callback
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")

# 使用上下文管理器
with get_openai_callback() as cb:
    llm.invoke("解释什么是机器学习")
    llm.invoke("解释什么是深度学习")
    
    print(f"总 Token: {cb.total_tokens}")
    print(f"提示 Token: {cb.prompt_tokens}")
    print(f"完成 Token: {cb.completion_tokens}")
    print(f"总费用: ${cb.total_cost}")
```

## 9.5 异步回调

### 9.5.1 异步回调处理器

```python
from langchain_core.callbacks import AsyncCallbackHandler

class AsyncLoggingHandler(AsyncCallbackHandler):
    """异步日志回调"""
    
    async def on_llm_start(self, serialized, prompts, **kwargs):
        print("LLM 开始（异步）")
    
    async def on_llm_end(self, response, **kwargs):
        print("LLM 结束（异步）")
    
    async def on_chain_start(self, serialized, inputs, **kwargs):
        print(f"链开始: {inputs}")
    
    async def on_chain_end(self, outputs, **kwargs):
        print(f"链结束: {outputs}")
```

### 9.5.2 异步调用

```python
import asyncio
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", callbacks=[AsyncLoggingHandler()])

async def main():
    response = await llm.ainvoke("写一首诗")
    print(response.content)

asyncio.run(main())
```

## 9.6 回调上下文传递

### 9.6.1 在链之间传递上下文

```python
from langchain_core.callbacks import CallbackManager
from contextvars import ContextVar

# 定义上下文变量
request_id: ContextVar[str] = ContextVar("request_id")

class ContextAwareHandler(BaseCallbackHandler):
    def on_chain_start(self, serialized, inputs, **kwargs):
        req_id = request_id.get("unknown")
        print(f"Request ID: {req_id}")
        print(f"链输入: {inputs}")

# 使用上下文
def process_request(request_id_value: str, input_data: dict):
    request_id.set(request_id_value)
    
    chain = LLMChain(
        llm=llm,
        prompt=prompt,
        callbacks=[ContextAwareHandler()]
    )
    
    return chain.invoke(input_data)

# 执行
process_request("req-123", {"topic": "AI"})
process_request("req-456", {"topic": "ML"})
```

### 9.6.2 子链回调

```python
# 父链使用回调管理器
callback_manager = CallbackManager([parent_handler])

parent_chain = LLMChain(
    llm=llm,
    prompt=parent_prompt,
    callbacks=callback_manager
)

# 子链继承父链的回调
child_chain = LLMChain(
    llm=llm,
    prompt=child_prompt,
    callbacks=callback_manager.inheritable_handlers  # 继承给子链
)
```

## 9.7 完整示例

### 9.7.1 性能监控回调

```python
import time
from langchain_core.callbacks import BaseCallbackHandler
from typing import Dict, Any, List
from dataclasses import dataclass, field

@dataclass
class ExecutionMetrics:
    """执行指标"""
    llm_calls: int = 0
    tool_calls: int = 0
    chain_calls: int = 0
    total_tokens: int = 0
    start_time: float = field(default_factory=time.time)
    end_time: float = 0
    errors: List[str] = field(default_factory=list)
    
    @property
    def duration(self) -> float:
        return self.end_time - self.start_time
    
    def summary(self) -> Dict[str, Any]:
        return {
            "LLM 调用次数": self.llm_calls,
            "工具调用次数": self.tool_calls,
            "链调用次数": self.chain_calls,
            "总 Token 数": self.total_tokens,
            "执行时长": f"{self.duration:.2f}s",
            "错误数": len(self.errors)
        }

class MetricsCallbackHandler(BaseCallbackHandler):
    """性能监控回调"""
    
    def __init__(self):
        self.metrics = ExecutionMetrics()
    
    def on_llm_start(self, serialized, prompts, **kwargs):
        self.metrics.llm_calls += 1
    
    def on_llm_end(self, response, **kwargs):
        # 统计 token
        if hasattr(response, 'llm_output'):
            usage = response.llm_output.get('token_usage', {})
            self.metrics.total_tokens += usage.get('total_tokens', 0)
    
    def on_tool_start(self, serialized, input_str, **kwargs):
        self.metrics.tool_calls += 1
    
    def on_chain_start(self, serialized, inputs, **kwargs):
        self.metrics.chain_calls += 1
    
    def on_chain_end(self, outputs, **kwargs):
        self.metrics.end_time = time.time()
    
    def on_chain_error(self, error, **kwargs):
        self.metrics.errors.append(str(error))
        self.metrics.end_time = time.time()

# 使用
metrics_handler = MetricsCallbackHandler()
chain = LLMChain(llm=llm, prompt=prompt, callbacks=[metrics_handler])

result = chain.invoke({"topic": "AI"})
print(metrics_handler.metrics.summary())
```

### 9.7.2 详细日志记录

```python
import json
from datetime import datetime
from pathlib import Path
from langchain_core.callbacks import BaseCallbackHandler

class DetailedLoggingHandler(BaseCallbackHandler):
    """详细日志记录"""
    
    def __init__(self, log_dir: str = "./logs"):
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(exist_ok=True)
        self.session_id = datetime.now().strftime("%Y%m%d_%H%M%S")
        self.events = []
    
    def _log_event(self, event_type: str, data: Dict):
        """记录事件"""
        event = {
            "timestamp": datetime.now().isoformat(),
            "type": event_type,
            "data": data
        }
        self.events.append(event)
        
        # 同时打印到控制台
        print(f"[{event['timestamp']}] {event_type}")
        print(json.dumps(data, ensure_ascii=False, indent=2))
        print()
    
    def on_llm_start(self, serialized, prompts, **kwargs):
        self._log_event("LLM_START", {
            "model": serialized.get("name", "unknown"),
            "prompts": prompts
        })
    
    def on_llm_end(self, response, **kwargs):
        # 提取关键信息
        usage = {}
        if hasattr(response, 'llm_output'):
            usage = response.llm_output.get('token_usage', {})
        
        self._log_event("LLM_END", {"usage": usage})
    
    def on_tool_start(self, serialized, input_str, **kwargs):
        self._log_event("TOOL_START", {
            "tool": serialized.get("name", "unknown"),
            "input": input_str
        })
    
    def on_tool_end(self, output, **kwargs):
        self._log_event("TOOL_END", {"output": output[:500]})
    
    def on_chain_start(self, serialized, inputs, **kwargs):
        self._log_event("CHAIN_START", {
            "chain": serialized.get("name", "unknown"),
            "inputs": inputs
        })
    
    def on_chain_end(self, outputs, **kwargs):
        self._log_event("CHAIN_END", {"outputs": outputs})
    
    def save_logs(self):
        """保存日志到文件"""
        log_file = self.log_dir / f"session_{self.session_id}.json"
        with open(log_file, "w", encoding="utf-8") as f:
            json.dump(self.events, f, ensure_ascii=False, indent=2)
        return str(log_file)

# 使用
logging_handler = DetailedLoggingHandler()
chain = LLMChain(llm=llm, prompt=prompt, callbacks=[logging_handler])

result = chain.invoke({"topic": "AI"})
log_file = logging_handler.save_logs()
print(f"\n日志已保存到: {log_file}")
```

### 9.7.3 进度显示回调

```python
import sys
from langchain_core.callbacks import BaseCallbackHandler

class ProgressCallbackHandler(BaseCallbackHandler):
    """进度显示回调"""
    
    def __init__(self, total_steps: int = 10):
        self.total_steps = total_steps
        self.current_step = 0
        self.bar_length = 30
    
    def _draw_progress(self, message: str = ""):
        """绘制进度条"""
        self.current_step = min(self.current_step + 1, self.total_steps)
        filled = int(self.bar_length * self.current_step / self.total_steps)
        bar = "█" * filled + "░" * (self.bar_length - filled)
        percent = int(100 * self.current_step / self.total_steps)
        
        sys.stdout.write(f"\r[{bar}] {percent}% {message}")
        sys.stdout.flush()
        
        if self.current_step >= self.total_steps:
            sys.stdout.write("\n")
    
    def on_llm_start(self, serialized, prompts, **kwargs):
        self._draw_progress("LLM 调用中...")
    
    def on_tool_start(self, serialized, input_str, **kwargs):
        tool_name = serialized.get("name", "tool")
        self._draw_progress(f"执行 {tool_name}...")
    
    def on_chain_start(self, serialized, inputs, **kwargs):
        self._draw_progress("处理中...")
    
    def on_chain_end(self, outputs, **kwargs):
        self._draw_progress("完成!")
    
    def on_text(self, text: str, **kwargs):
        """显示文本输出"""
        sys.stdout.write(f"\r{' ' * 50}\r")
        print(text)

# 使用
progress_handler = ProgressCallbackHandler(total_steps=5)
chain = LLMChain(llm=llm, prompt=prompt, callbacks=[progress_handler])
result = chain.invoke({"topic": "AI"})
```

## 9.8 总结

本章介绍了 LangChain 中 Callbacks 的各种用法：

1. **基础回调**: ConsoleCallbackHandler、StdOutCallbackHandler
2. **自定义回调**: 继承 BaseCallbackHandler 实现事件处理
3. **常用回调**: FileCallbackHandler、LangSmithCallback
4. **异步回调**: AsyncCallbackHandler
5. **回调应用**: 性能监控、日志记录、进度显示

**最佳实践**：
- 使用回调进行执行监控
- 自定义回调实现业务需求
- 注意回调中的异常处理
- 使用 ContextVar 进行上下文传递
- 避免在回调中进行耗时操作

---

*第一部分「LangChain 基础」到此结束。接下来我们将学习 LangGraph 进阶内容。*
