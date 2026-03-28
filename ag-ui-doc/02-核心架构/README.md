# 02. AG-UI 核心架构

## 设计原则

AG-UI 被设计为**轻量级且最小化固执己见**，使其易于与各种 Agent 实现集成。协议的灵活性来自其简单要求：

1. **事件驱动通信**：Agent 在执行过程中需要发出任何 16 种标准化事件类型的事件流，客户端可以处理这些更新
2. **双向交互**：Agent 接受来自用户的输入，实现人类与 AI 协作的工作流

### 最大化兼容性的内置中间件层

协议包含一个内置的中间件层，通过两种方式最大化兼容性：

- **灵活的事件结构**：事件不需要完全匹配 AG-UI 格式，只需与 AG-UI 兼容，允许现有 Agent 框架以最小的努力调整其原生事件格式
- **传输无关**：AG-UI 不强制规定事件传递方式，支持多种传输机制，包括 Server-Sent Events（SSE）、Webhook、WebSocket 等

---

## 架构概述

AG-UI 采用客户端-服务器架构，标准化 Agent 与应用之间的通信：

```
┌─────────────────────────────────────────────────────────────────┐
│                        架构图                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    ┌──────────┐              ┌──────────┐                      │
│    │          │              │          │                      │
│    │  Frontend │              │   AI     │                      │
│    │           │              │  Agent   │                      │
│    │ Application│◄──────────►│    A     │                      │
│    │           │   AG-UI     │          │                      │
│    └────┬─────┘   Protocol   └────┬─────┘                      │
│         │                          │                            │
│         │              ┌───────────┴───────────┐                │
│         │              │                       │                │
│    ┌────▼────┐         │    ┌──────────┐     │                │
│    │  AG-UI  │         │    │  Secure  │     │                │
│    │ Client  │         │    │  Proxy   │     │                │
│    │ (Http   │         │    └────┬─────┘     │                │
│    │  Agent) │         │         │            │                │
│    └─────────┘         │    ┌────▼────┐      │                │
│                        │    │   AI    │      │                │
│                        │    │  Agent  │      │                │
│                        │    │    B    │      │                │
│                        │    └─────────┘      │                │
│                        │    ┌──────────┐     │                │
│                        │    │   AI    │     │                │
│                        │    │  Agent  │     │                │
│                        │    │    C    │     │                │
│                        │    └─────────┘     │                │
│                        └─────────────────────┘                │
│                          Backend                               │
└─────────────────────────────────────────────────────────────────┘
```

### 核心组件

| 组件 | 说明 |
|:---|:---|
| **Application（应用）** | 用户面向的应用程序（如聊天界面或任何 AI 驱动的应用） |
| **AG-UI Client** | 通用通信客户端（如 `HttpAgent`），或用于连接现有协议的专用客户端 |
| **Agents** | 处理请求并生成流式响应的后端 AI Agent |
| **Secure Proxy** | 提供额外能力并作为安全代理的后端服务 |

---

## 协议层

AG-UI 的协议层为 Agent 通信提供了灵活的基础。

### 核心抽象

协议的主要抽象使应用能够运行 Agent 并接收事件流：

```typescript
// 核心 Agent 执行接口
type RunAgent = () => Observable<BaseEvent>

class MyAgent extends AbstractAgent {
  run(input: RunAgentInput): RunAgent {
    const { threadId, runId } = input
    return () =>
      from([
        { type: EventType.RUN_STARTED, threadId, runId },
        {
          type: EventType.MESSAGES_SNAPSHOT,
          messages: [
            { id: "msg_1", role: "assistant", content: "Hello, world!" }
          ],
        },
        { type: EventType.RUN_FINISHED, threadId, runId },
      ])
  }
}
```

### 统一兼容性

- **通用兼容性**：通过实现 `run(input: RunAgentInput) -> Observable<BaseEvent>` 连接到任何协议

---

## 标准 HTTP 客户端

AG- 提供了一个标准的 HTTP 客户端 `HttpAgent`，可用于连接到任何接受 POST 请求（请求体类型为 `RunAgentInput`）并发送 `BaseEvent` 流 的端点。

### 支持的传输方式

1. **HTTP SSE (Server-Sent Events)**
   - 文本流式传输，兼容性广泛
   - 易于阅读和调试

2. **HTTP Binary Protocol**
   - 高性能、空间效率高
   - 生产环境的健壮二进制序列化

---

## 消息类型

AG-UI 定义了多个事件类别，用于 Agent 通信的不同方面：

### 生命周期事件

- `RUN_STARTED` — Agent 运行开始
- `RUN_FINISHED` — Agent 运行成功完成
- `RUN_ERROR` — Agent 运行出错
- `STEP_STARTED` — 步骤开始
- `STEP_FINISHED` — 步骤完成

### 文本消息事件

- `TEXT_MESSAGE_START` — 文本消息开始
- `TEXT_MESSAGE_CONTENT` — 文本消息内容（流式）
- `TEXT_MESSAGE_END` — 文本消息结束
- `TEXT_MESSAGE_CHUNK` — 便捷事件，自动展开为 Start → Content → End

### 工具调用事件

- `TOOL_CALL_START` — 工具调用开始
- `TOOL_CALL_ARGS` — 工具调用参数（流式）
- `TOOL_CALL_END` — 工具调用结束

### 状态管理事件

- `STATE_SNAPSHOT` — 完整状态快照
- `STATE_DELTA` — 增量状态更新（JSON Patch 格式）
- `MESSAGES_SNAPSHOT` — 完整对话历史

### 特殊事件

- `RAW` — 原始事件
- `CUSTOM` — 自定义事件

---

## 运行 Agent

要运行 Agent，创建客户端实例并执行：

```typescript
// 创建 HTTP Agent 客户端
const agent = new HttpAgent({
  url: "https://your-agent-endpoint.com/agent",
  agentId: "unique-agent-id",
  threadId: "conversation-thread"
});

// 启动 Agent 并处理事件
agent.runAgent({
  tools: [...],
  context: [...]
}).subscribe({
  next: (event) => {
    // 处理不同的事件类型
    switch(event.type) {
      case EventType.TEXT_MESSAGE_CONTENT:
        // 用新内容更新 UI
        break;
      // 处理其他事件类型
    }
  },
  error: (error) => console.error("Agent error:", error),
  complete: () => console.log("Agent run complete")
});
```

---

## 状态管理

AG-UI 通过专门的事件提供高效的状态管理：

| 事件 | 用途 |
|:---|:---|
| `STATE_SNAPSHOT` | 某时刻的完整状态表示 |
| `STATE_DELTA` | 使用 JSON Patch 格式（RFC 6902）的增量状态变化 |
| `MESSAGES_SNAPSHOT` | 完整对话历史 |

这些事件使客户端状态管理高效，最小化数据传输。

---

## 工具和交接

AG-UI 通过标准化事件支持 Agent 间交接和工具使用：

- 工具定义在 `runAgent` 参数中传递
- 工具调用作为事件序列流式传输：`TOOL_CALL_START` → `TOOL_CALL_ARGS` → `TOOL_CALL_END`
- Agent 可以交接给其他 Agent，保持上下文连续性

---

## 事件基础

AG-UI 中的所有通信都基于类型化的事件。每个事件都继承自 `BaseEvent`：

```typescript
interface BaseEvent {
  type: EventType
  timestamp?: number
  rawEvent?: any
}
```

事件是严格类型化和验证的，确保组件之间的可靠通信。

---

## 下一步

- [事件系统](../03-事件系统/README.md) — 深入了解 16 种事件类型
- [状态管理](../04-状态管理/README.md) — 学习状态同步机制
- [快速开始](../07-快速开始/README.md) — 动手实践

---

*参考资料：[AG-UI Core Architecture](https://docs.ag-ui.com/concepts/architecture.md)*
