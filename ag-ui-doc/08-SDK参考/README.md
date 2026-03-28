# 08. AG-UI SDK 参考

## 概述

AG-UI 提供官方 SDK，支持 TypeScript/JavaScript 和 Python 两个主要平台。本章节详细介绍各 SDK 的安装、核心 API 和使用示例。

---

## TypeScript SDK

### 安装

```bash
npm install @ag-ui/client @ag-ui/core
# 或
yarn add @ag-ui/client @ag-ui/core
# 或
pnpm add @ag-ui/client @ag-ui/core
```

### 包结构

| 包 | 说明 |
|:---|:---|
| `@ag-ui/core` | 核心类型、事件定义和事件类型常量 |
| `@ag-ui/client` | 客户端实现（HttpAgent、AbstractAgent、中间件） |
| `@ag-ui/encoder` | 事件编码器（SSE、Binary） |
| `@ag-ui/proto` | Protocol Buffers 支持 |

---

## 核心类型

### BaseEvent

所有 AG-UI 事件的基类：

```typescript
import { BaseEvent, EventType } from "@ag-ui/core";

interface BaseEvent {
  type: EventType;        // 事件类型
  timestamp?: number;      // 可选时间戳
  rawEvent?: any;         // 原始事件（转换时保留）
}
```

### RunAgentInput

启动 Agent 运行时的输入参数：

```typescript
interface RunAgentInput {
  threadId: string;           // 线程 ID
  runId?: string;             // 运行 ID（可选，自动生成）
  messages?: Message[];        // 消息历史
  tools?: Tool[];              // 工具定义
  state?: any;                 // 初始状态
  forwardedProps?: Record<string, any>;  // 转发属性
  parentRunId?: string;        // 父运行 ID（用于分支）
}
```

### Tool

工具定义：

```typescript
interface Tool {
  name: string;                // 工具名称
  description: string;         // 工具描述
  parameters: {                // JSON Schema 参数定义
    type: "object";
    properties: Record<string, any>;
    required?: string[];
  };
}
```

---

## 事件类型

### EventType 枚举

```typescript
import { EventType } from "@ag-ui/core";

// 生命周期事件
EventType.RUN_STARTED
EventType.RUN_FINISHED
EventType.RUN_ERROR
EventType.STEP_STARTED
EventType.STEP_FINISHED

// 文本消息事件
EventType.TEXT_MESSAGE_START
EventType.TEXT_MESSAGE_CONTENT
EventType.TEXT_MESSAGE_END
EventType.TEXT_MESSAGE_CHUNK

// 工具调用事件
EventType.TOOL_CALL_START
EventType.TOOL_CALL_ARGS
EventType.TOOL_CALL_END
EventType.TOOL_CALL_RESULT

// 状态管理事件
EventType.STATE_SNAPSHOT
EventType.STATE_DELTA
EventType.MESSAGES_SNAPSHOT

// 特殊事件
EventType.RAW
EventType.CUSTOM
```

### 事件类

```typescript
import {
  // 生命周期
  RunStartedEvent,
  RunFinishedEvent,
  RunErrorEvent,
  StepStartedEvent,
  StepFinishedEvent,
  
  // 文本消息
  TextMessageStartEvent,
  TextMessageContentEvent,
  TextMessageEndEvent,
  TextMessageChunkEvent,
  
  // 工具调用
  ToolCallStartEvent,
  ToolCallArgsEvent,
  ToolCallEndEvent,
  ToolCallResultEvent,
  
  // 状态管理
  StateSnapshotEvent,
  StateDeltaEvent,
  MessagesSnapshotEvent,
} from "@ag-ui/core";
```

### 使用示例

```typescript
// 创建事件
const runStarted = new RunStartedEvent({
  type: EventType.RUN_STARTED,
  threadId: "thread-123",
  runId: "run-456"
});

const textMessageStart = new TextMessageStartEvent({
  type: EventType.TEXT_MESSAGE_START,
  messageId: "msg-789",
  role: "assistant"
});

const textContent = new TextMessageContentEvent({
  type: EventType.TEXT_MESSAGE_CONTENT,
  messageId: "msg-789",
  delta: "Hello "
});
```

---

## HttpAgent

标准的 HTTP 客户端，用于连接 AG-UI 兼容服务器：

```typescript
import { HttpAgent } from "@ag-ui/client";

const agent = new HttpAgent({
  url: "https://your-agent-endpoint.com/agent",
  agentId: "unique-agent-id",
  threadId: "conversation-thread",
  transport?: "sse" | "binary",  // 可选传输方式
  headers?: Record<string, string>  // 自定义请求头
});
```

### 方法

#### runAgent(input)

启动 Agent 运行并返回事件流：

```typescript
import { runAgent } from "@ag-ui/client";

const observable = agent.runAgent({
  messages: [
    { role: "user", content: "Hello!" }
  ],
  tools: [
    {
      name: "search",
      description: "Search for information",
      parameters: {
        type: "object",
        properties: {
          query: { type: "string" }
        },
        required: ["query"]
      }
    }
  ],
  state: { context: "initial" },
  forwardedProps: {
    userId: "user-123"
  }
});

observable.subscribe({
  next: (event) => console.log("Event:", event),
  error: (err) => console.error("Error:", err),
  complete: () => console.log("Complete!")
});
```

### 订阅者系统

```typescript
import { AgentSubscriber } from "@ag-ui/client";

const subscriber = new AgentSubscriber({
  onRunStarted: (event) => { /* ... */ },
  onTextMessageStart: (event) => { /* ... */ },
  onTextMessageContent: (event) => { /* ... */ },
  onTextMessageEnd: (event) => { /* ... */ },
  onToolCallStart: (event) => { /* ... */ },
  onToolCallArgs: (event) => { /* ... */ },
  onToolCallEnd: (event) => { /* ... */ },
  onStateSnapshot: (event) => { /* ... */ },
  onStateDelta: (event) => { /* ... */ },
  onRunFinished: (event) => { /* ... */ },
  onRunError: (event) => { /* ... */ }
});

agent.runAgent({}).subscribe(subscriber);
```

---

## AbstractAgent

用于实现自定义 Agent 的基类：

```typescript
import { AbstractAgent } from "@ag-ui/client";

class MyAgent extends AbstractAgent {
  run(input: RunAgentInput) {
    const { threadId, runId, messages } = input;
    
    return from([/* events */]);
  }
}

// 使用
const agent = new MyAgent();
agent.runAgent({ messages: [] }).subscribe({
  next: (event) => console.log(event)
});
```

---

## 中间件 API

### MiddlewareFunction

函数式中间件类型：

```typescript
import { MiddlewareFunction } from "@ag-ui/client";

const myMiddleware: MiddlewareFunction = (input, next) => {
  return next.run(input).pipe(
    map(event => /* transform */),
    catchError(err => /* handle */)
  );
};

agent.use(myMiddleware);
```

### Middleware

类式中间件基类：

```typescript
import { Middleware } from "@ag-ui/client";

class MyMiddleware extends Middleware {
  run(input: RunAgentInput, next: AbstractAgent) {
    return this.runNext(input, next).pipe(/* ... */);
  }
}

agent.use(new MyMiddleware());
```

### FilterToolCallsMiddleware

工具调用过滤器：

```typescript
import { FilterToolCallsMiddleware } from "@ag-ui/client";

// 只允许特定工具
agent.use(new FilterToolCallsMiddleware({
  allowedToolCalls: ["search", "calculate"]
}));

// 或禁止特定工具
agent.use(new FilterToolCallsMiddleware({
  disallowedToolCalls: ["delete", "sendEmail"]
}));
```

---

## 事件编码器

### EventEncoder

用于编码和解码 AG-UI 事件：

```typescript
import { EventEncoder } from "@ag-ui/encoder";
import { EventType } from "@ag-ui/core";

// SSE 编码
const sseEncoder = new EventEncoder({ accept: "text/event-stream" });

// Binary 编码
const binaryEncoder = new EventEncoder({ accept: "application/x-ag-ui-binary" });

// 编码事件
const encoded = sseEncoder.encode(event);
const contentType = sseEncoder.getContentType();
```

---

## Python SDK

### 安装

```bash
pip install ag-ui-protocol
# 或
poetry add ag-ui-protocol
```

### 核心模块

```python
from ag_ui.core import (
    # 事件类型
    EventType,
    
    # 事件类
    RunStartedEvent,
    RunFinishedEvent,
    RunErrorEvent,
    StepStartedEvent,
    StepFinishedEvent,
    TextMessageStartEvent,
    TextMessageContentEvent,
    TextMessageEndEvent,
    ToolCallStartEvent,
    ToolCallArgsEvent,
    ToolCallEndEvent,
    StateSnapshotEvent,
    StateDeltaEvent,
    MessagesSnapshotEvent,
    
    # 输入类型
    RunAgentInput,
    Message,
    Tool,
)
from ag_ui.encoder import EventEncoder
```

### 使用示例

```python
import uuid
from ag_ui.core import (
    EventType,
    RunStartedEvent,
    RunFinishedEvent,
    TextMessageStartEvent,
    TextMessageContentEvent,
    TextMessageEndEvent
)
from ag_ui.encoder import EventEncoder
from fastapi import FastAPI, Request
from fastapi.responses import StreamingResponse

app = FastAPI()
encoder = EventEncoder(accept="text/event-stream")

@app.post("/agent")
async def agent_endpoint(input_data: RunAgentInput, request: Request):
    async def event_generator():
        # 发送运行开始
        yield encoder.encode(
            RunStartedEvent(
                type=EventType.RUN_STARTED,
                thread_id=input_data.thread_id,
                run_id=input_data.run_id
            )
        )
        
        # 处理并发送消息
        message_id = str(uuid.uuid4())
        yield encoder.encode(
            TextMessageStartEvent(
                type=EventType.TEXT_MESSAGE_START,
                message_id=message_id,
                role="assistant"
            )
        )
        
        response = "Hello from Python AG-UI!"
        for chunk in response:
            yield encoder.encode(
                TextMessageContentEvent(
                    type=EventType.TEXT_MESSAGE_CONTENT,
                    message_id=message_id,
                    delta=chunk
                )
            )
        
        yield encoder.encode(
            TextMessageEndEvent(
                type=EventType.TEXT_MESSAGE_END,
                message_id=message_id
            )
        )
        
        yield encoder.encode(
            RunFinishedEvent(
                type=EventType.RUN_FINISHED,
                thread_id=input_data.thread_id,
                run_id=input_data.run_id
            )
        )
    
    return StreamingResponse(
        event_generator(),
        media_type=encoder.get_content_type()
    )
```

---

## 类型对比

| TypeScript | Python | 说明 |
|:---|:---|:---|
| `BaseEvent` | `BaseEvent` | 所有事件的基类 |
| `RunAgentInput` | `RunAgentInput` | 运行输入参数 |
| `EventType` | `EventType` | 事件类型枚举 |
| `HttpAgent` | 需自行实现 | HTTP 客户端 |
| `AbstractAgent` | 需自行实现 | Agent 基类 |
| `Middleware` | 需自行实现 | 中间件基类 |

---

## 下一步

- [最佳实践与实战](../09-最佳实践与实战/README.md) — 生产级应用指南
- [官方 GitHub](https://github.com/ag-ui-protocol/ag-ui) — 更多示例

---

*参考资料：[AG-UI TypeScript SDK](https://docs.ag-ui.com/sdk/js/core/overview) | [Python SDK](https://docs.ag-ui.com/sdk/python/core/overview)*
