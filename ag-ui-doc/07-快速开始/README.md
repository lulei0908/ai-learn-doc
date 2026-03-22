# 07. AG-UI 快速开始

## 概述

本章节将帮助你快速上手 AG-UI，包含两种主要的集成方式：

1. **服务端实现** — 直接从 Agent 或服务器发出 AG-UI 事件
2. **中间件实现** — 将现有协议转换为 AG-UI 事件

---

## 前提条件

在开始之前，确保你已安装：

- **Python** 3.12 或更高版本
- **Node.js** 18+
- **Poetry**（Python 依赖管理）
- 对应框架的 API Key（如 OpenAI）

---

## 方式一：服务端实现

### 适用场景

- 从头构建新 Agent
- 对如何以及发出什么事件有最大控制
- 将 Agent 暴露为独立 API

### 实现步骤

#### 步骤 1：克隆仓库

```bash
git clone git@github.com:ag-ui-protocol/ag-ui.git
cd ag-ui
```

#### 步骤 2：创建服务端

复制服务端模板：

```bash
cp -r integrations/server-starter integrations/my-server
```

#### 步骤 3：实现事件流

创建你的第一个 AG-UI 兼容端点：

```python
# my_server/server/python/example_server/__init__.py
from fastapi import FastAPI, Request
from fastapi.responses import StreamingResponse
from ag_ui.core import (
    RunAgentInput,
    EventType,
    RunStartedEvent,
    RunFinishedEvent,
    TextMessageStartEvent,
    TextMessageContentEvent,
    TextMessageEndEvent
)
from ag_ui.encoder import EventEncoder
import uuid

app = FastAPI(title="AG-UI Endpoint")

@app.post("/")
async def agentic_chat_endpoint(input_data: RunAgentInput, request: Request):
    """AG-UI 兼容的 Agent 端点"""
    
    # 获取 accept header 以确定编码格式
    accept_header = request.headers.get("accept")
    encoder = EventEncoder(accept=accept_header)

    async def event_generator():
        # 1. 发送运行开始事件
        yield encoder.encode(
            RunStartedEvent(
                type=EventType.RUN_STARTED,
                thread_id=input_data.thread_id,
                run_id=input_data.run_id
            )
        )

        # 2. 发送消息开始事件
        message_id = str(uuid.uuid4())
        yield encoder.encode(
            TextMessageStartEvent(
                type=EventType.TEXT_MESSAGE_START,
                message_id=message_id,
                role="assistant"
            )
        )

        # 3. 流式发送消息内容
        response_text = "Hello from AG-UI!"
        for chunk in response_text:
            yield encoder.encode(
                TextMessageContentEvent(
                    type=EventType.TEXT_MESSAGE_CONTENT,
                    message_id=message_id,
                    delta=chunk
                )
            )

        # 4. 发送消息结束事件
        yield encoder.encode(
            TextMessageEndEvent(
                type=EventType.TEXT_MESSAGE_END,
                message_id=message_id
            )
        )

        # 5. 发送运行完成事件
        yield encoder.encode(
            RunFinishedEvent(
                type=EventType.RUN_FINISHED,
                thread_id=input_data.thread_id,
                run_id=input_data.run_id
            )
        )

    return StreamingResponse(
        event_generator(),
        media_type="text/event-stream"
    )

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

#### 步骤 4：与 OpenAI 集成

使用真实的 LLM 来增强你的服务器：

```python
# my_server/server/python/openai_server.py
import os
import uuid
import uvicorn
from fastapi import FastAPI, Request
from fastapi.responses import StreamingResponse
from ag_ui.core import (
    RunAgentInput,
    EventType,
    RunStartedEvent,
    RunFinishedEvent,
    RunErrorEvent,
    TextMessageStartEvent,
    TextMessageContentEvent,
    TextMessageEndEvent
)
from ag_ui.encoder import EventEncoder
from openai import OpenAI

app = FastAPI(title="AG-UI OpenAI Server")

# 初始化 OpenAI 客户端
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

@app.post("/")
async def agentic_chat_endpoint(input_data: RunAgentInput, request: Request):
    """OpenAI AG-UI 端点"""
    accept_header = request.headers.get("accept")
    encoder = EventEncoder(accept=accept_header)

    async def event_generator():
        try:
            # 发送运行开始事件
            yield encoder.encode(
                RunStartedEvent(
                    type=EventType.RUN_STARTED,
                    thread_id=input_data.thread_id,
                    run_id=input_data.run_id
                )
            )

            # 将 AG-UI 消息转换为 OpenAI 格式
            openai_messages = []
            for msg in input_data.messages:
                if msg.role in ["user", "system", "assistant"]:
                    openai_messages.append({
                        "role": msg.role,
                        "content": msg.content or ""
                    })

            # 调用 OpenAI API（流式）
            stream = client.chat.completions.create(
                model="gpt-4o",
                stream=True,
                messages=openai_messages,
            )

            # 生成消息 ID
            message_id = str(uuid.uuid4())

            # 发送消息开始
            yield encoder.encode(
                TextMessageStartEvent(
                    type=EventType.TEXT_MESSAGE_START,
                    message_id=message_id,
                    role="assistant"
                )
            )

            # 处理流式响应
            for chunk in stream:
                if (chunk.choices and
                    len(chunk.choices) > 0 and
                    chunk.choices[0].delta and
                    hasattr(chunk.choices[0].delta, 'content') and
                    chunk.choices[0].delta.content):
                    
                    content = chunk.choices[0].delta.content
                    yield encoder.encode(
                        TextMessageContentEvent(
                            type=EventType.TEXT_MESSAGE_CONTENT,
                            message_id=message_id,
                            delta=content
                        )
                    )

            # 发送消息结束
            yield encoder.encode(
                TextMessageEndEvent(
                    type=EventType.TEXT_MESSAGE_END,
                    message_id=message_id
                )
            )

            # 发送运行完成
            yield encoder.encode(
                RunFinishedEvent(
                    type=EventType.RUN_FINISHED,
                    thread_id=input_data.thread_id,
                    run_id=input_data.run_id
                )
            )

        except Exception as e:
            # 发送错误事件
            yield encoder.encode(
                RunErrorEvent(
                    type=EventType.RUN_ERROR,
                    message=str(e)
                )
            )

    return StreamingResponse(
        event_generator(),
        media_type=encoder.get_content_type()
    )

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

#### 步骤 5：启动服务器

```bash
# 设置 API Key
export OPENAI_API_KEY=your-api-key-here

# 安装依赖
cd integrations/my-server/server/python
poetry install

# 启动服务器
poetry run dev
```

---

## 方式二：中间件实现

### 适用场景

- 将现有协议或 API 转换为通用协议
- 在现有系统或框架内工作
- 无法直接控制 Agent 框架时

### 示例：将 LangGraph 适配到 AG-UI

```python
# middleware/langgraph_to_agui.py
from ag_ui.core import (
    EventType,
    RunStartedEvent,
    RunFinishedEvent,
    TextMessageStartEvent,
    TextMessageContentEvent,
    TextMessageEndEvent
)
from ag_ui.encoder import EventEncoder
from langgraph.graph import StateGraph
import uuid

class LangGraphToAGUIMiddleware:
    """将 LangGraph 事件转换为 AG-UI 事件"""
    
    def __init__(self, app, encoder: EventEncoder):
        self.app = app
        self.encoder = encoder
    
    async def run(self, input_data):
        thread_id = input_data.get("thread_id", str(uuid.uuid4()))
        run_id = str(uuid.uuid4())
        
        # 发送开始事件
        yield self.encoder.encode(
            RunStartedEvent(
                type=EventType.RUN_STARTED,
                thread_id=thread_id,
                run_id=run_id
            )
        )
        
        # 运行 LangGraph
        async for event in self.app.astream_events(input_data):
            # 将 LangGraph 事件转换为 AG-UI 事件
            if event["event"] == "on_chat_model_stream":
                message_id = str(uuid.uuid4())
                content = event["data"]["chunk"].content
                
                yield self.encoder.encode(
                    TextMessageStartEvent(
                        type=EventType.TEXT_MESSAGE_START,
                        message_id=message_id,
                        role="assistant"
                    )
                )
                yield self.encoder.encode(
                    TextMessageContentEvent(
                        type=EventType.TEXT_MESSAGE_CONTENT,
                        message_id=message_id,
                        delta=content
                    )
                )
                yield self.encoder.encode(
                    TextMessageEndEvent(
                        type=EventType.TEXT_MESSAGE_END,
                        message_id=message_id
                    )
                )
        
        # 发送完成事件
        yield self.encoder.encode(
            RunFinishedEvent(
                type=EventType.RUN_FINISHED,
                thread_id=thread_id,
                run_id=run_id
            )
        )
```

---

## 前端集成

### 使用 TypeScript SDK

```typescript
import { HttpAgent, EventType } from "@ag-ui/client";

const agent = new HttpAgent({
  url: "http://localhost:8000",
  agentId: "my-agent",
  threadId: "thread-123"
});

agent.runAgent({
  messages: [
    { role: "user", content: "Hello, how can you help me?" }
  ]
}).subscribe({
  next: (event) => {
    switch (event.type) {
      case EventType.TEXT_MESSAGE_START:
        console.log("Message started");
        break;
        
      case EventType.TEXT_MESSAGE_CONTENT:
        // 追加到聊天界面
        appendMessage(event.delta);
        break;
        
      case EventType.TEXT_MESSAGE_END:
        console.log("Message ended");
        break;
        
      case EventType.RUN_FINISHED:
        console.log("Run completed");
        break;
        
      case EventType.RUN_ERROR:
        console.error("Error:", event.message);
        break;
    }
  }
});
```

### 使用 CopilotKit

```tsx
import { CopilotKit } from "@copilotkit/react-core";
import { CopilotSidebar } from "@copilotkit/react-ui";
import "@copilotkit/react-ui/styles.css";

function App() {
  return (
    <CopilotKit url="/api/copilot">
      <YourApp />
      <CopilotSidebar />
    </CopilotKit>
  );
}
```

---

## 常见问题排查

### 1. 连接失败

```bash
# 检查服务器是否运行
curl http://localhost:8000/health

# 检查 CORS 配置
# 在 FastAPI 中添加：
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### 2. 事件未接收

```typescript
// 确保正确订阅事件流
agent.runAgent({}).subscribe({
  next: (event) => console.log("Received:", event.type),
  error: (err) => console.error("Error:", err),
  complete: () => console.log("Stream complete")
});
```

### 3. 工具调用未触发

```typescript
// 确保传递工具定义
agent.runAgent({
  tools: [
    {
      name: "myTool",
      description: "A test tool",
      parameters: {
        type: "object",
        properties: {},
        required: []
      }
    }
  ]
});
```

---

## 下一步

- [SDK 参考](../08-SDK参考/README.md) — 完整的 API 文档
- [最佳实践与实战](../09-最佳实践与实战/README.md) — 生产级应用指南
- [官方示例](https://github.com/ag-ui-protocol/ag-ui) — 更多代码示例

---

*参考资料：[AG-UI Quickstart](https://docs.ag-ui.com/quickstart/introduction.md) | [Server Implementation](https://docs.ag-ui.com/quickstart/server.md)*
