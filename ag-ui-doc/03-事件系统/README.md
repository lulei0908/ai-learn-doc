# 03. AG-UI 事件系统

## 概述

AG-UI 采用**流式事件架构**。事件是 Agent 与前端之间通信的基本单元，支持实时、结构化的交互。

## 事件类型概览

AG-UI 中的事件按用途分为以下类别：

| 类别 | 说明 |
|:---|:---|
| **生命周期事件** | 监控 Agent 运行的进程 |
| **文本消息事件** | 处理流式文本内容 |
| **工具调用事件** | 管理 Agent 的工具执行 |
| **状态管理事件** | 在 Agent 和 UI 之间同步状态 |
| **活动事件** | 表示进行中的活动进度 |
| **特殊事件** | 支持自定义功能 |
| **草稿事件** | 正在开发中的提案事件 |

---

## 事件基础属性

所有事件共享一组公共属性：

| 属性 | 说明 |
|:---|:---|
| `type` | 特定事件类型标识符 |
| `timestamp` | 可选，事件创建时的时间戳 |
| `rawEvent` | 可选，如果转换了，包含原始事件数据 |

---

## 生命周期事件

生命周期事件代表 Agent 运行的生命周期。典型的 Agent 运行遵循可预测的模式：以 `RunStarted` 事件开始，可能包含多个可选的 `StepStarted`/`StepFinished` 对，以 `RunFinished` 事件（成功）或 `RunError` 事件（失败）结束。

### 事件流程

```
┌──────────┐     ┌──────────┐
│  Agent   │     │  Client  │
└────┬─────┘     └────┬─────┘
     │                │
     │──RunStarted──►│   ← 运行开始
     │                │
     │──StepStarted─►│   ← 步骤开始（可选）
     │                │
     │──StepFinished►│   ← 步骤完成
     │                │
     │──RunFinished─►│   ← 运行完成
     │                │
     └────────────────┘
```

### RunStarted

**信号：Agent 运行开始**

`RunStarted` 事件是 Agent 开始处理请求时发出的第一个事件。它建立由唯一 `runId` 标识的新执行上下文。此事件作为前端初始化 UI 元素（如进度指示器或加载状态）的标记。

| 属性 | 说明 |
|:---|:---|
| `threadId` | 对话线程 ID |
| `runId` | Agent 运行 ID |
| `parentRunId` | 可选，血缘指针，用于分支/时间回溯 |
| `input` | 可选，发送到 Agent 的精确输入载荷 |

### RunFinished

**信号：Agent 运行成功完成**

`RunFinished` 事件表示 Agent 已成功完成当前运行的所有工作。

| 属性 | 说明 |
|:---|:---|
| `threadId` | 对话线程 ID |
| `runId` | Agent 运行 ID |
| `result` | 可选，运行产生的结果数据 |

### RunError

**信号：Agent 运行期间发生错误**

`RunError` 事件表示 Agent 遇到无法恢复的错误，导致运行提前终止。

| 属性 | 说明 |
|:---|:---|
| `message` | 错误消息 |
| `code` | 可选，错误代码 |

### StepStarted

**信号：Agent 运行中步骤开始**

`StepStarted` 事件表示 Agent 开始执行特定的子任务或处理阶段。

| 属性 | 说明 |
|:---|:---|
| `stepName` | 步骤名称 |

### StepFinished

**信号：Agent 运行中步骤完成**

`StepFinished` 事件表示 Agent 已完成特定的子任务或阶段。

| 属性 | 说明 |
|:---|:---|
| `stepName` | 步骤名称 |

---

## 文本消息事件

文本消息事件表示对话中文本消息的生命周期。文本消息事件遵循流式模式，内容以增量方式传递。

### 事件流程

```
┌──────────┐     ┌──────────┐
│  Agent   │     │  Client  │
└────┬─────┘     └────┬─────┘
     │                │
     │──TextMessageStart──►│   ← 消息开始
     │                │
     │──TextMessageContent►│   ← 内容流 1
     │──TextMessageContent►│   ← 内容流 2
     │──TextMessageContent►│   ← 内容流 N
     │                │
     │──TextMessageEnd────►│   ← 消息结束
     └────────────────┘
```

### TextMessageStart

**信号：文本消息开始**

`TextMessageStart` 事件在对话中初始化新的文本消息。

| 属性 | 说明 |
|:---|:---|
| `messageId` | 消息的唯一标识符 |
| `role` | 发送者角色（"developer", "system", "assistant", "user", "tool"） |

### TextMessageContent

**表示：流式文本消息中的内容块**

`TextMessageContent` 事件在文本可用时传递消息文本的增量部分。

| 属性 | 说明 |
|:---|:---|
| `messageId` | 与 `TextMessageStart` 的 ID 匹配 |
| `delta` | 文本内容块（非空） |

### TextMessageEnd

**信号：文本消息结束**

`TextMessageEnd` 事件标记流式文本消息的完成。

| 属性 | 说明 |
|:---|:---|
| `messageId` | 与 `TextMessageStart` 的 ID 匹配 |

### TextMessageChunk

**便捷事件：自动展开为 Start → Content → End**

`TextMessageChunk` 事件让你省略显式的 `TextMessageStart` 和 `TextMessageEnd` 事件。客户端流转换器自动将块展开为标准三元组：

- 消息的第一个块必须包含 `messageId`，将发出 `TextMessageStart`（当未提供时角色默认为 `assistant`）
- 每个带有 `delta` 的块为当前 `messageId` 发出 `TextMessageContent`
- 当流切换到新消息 ID 或流完成时，自动发出 `TextMessageEnd`

| 属性 | 说明 |
|:---|:---|
| `messageId` | 可选，消息的唯一标识符（消息的第一个块必需） |
| `role` | 可选，发送者角色 |
| `delta` | 可选，消息的文本内容 |

---

## 工具调用事件

工具调用事件表示 Agent 进行的工具调用的生命周期。工具调用遵循与文本消息类似的流式模式。

### 事件流程

```
┌──────────┐     ┌──────────┐
│  Agent   │     │  Client  │
└────┬─────┘     └────┬─────┘
     │                │
     │──ToolCallStart──►│   ← 工具调用开始
     │                │
     │──ToolCallArgs──►│   ← 参数流 1
     │──ToolCallArgs──►│   ← 参数流 2
     │──ToolCallArgs──►│   ← 参数流 N
     │                │
     │──ToolCallEnd───►│   ← 工具调用结束
     │                │
     │──ToolCallResult►│   ← 工具执行结果
     └────────────────┘
```

### ToolCallStart

**信号：工具调用开始**

`ToolCallStart` 事件表示 Agent 正在调用工具来执行特定功能。

| 属性 | 说明 |
|:---|:---|
| `toolCallId` | 工具调用的唯一标识符 |
| `toolCallName` | 被调用工具的名称 |
| `parentMessageId` | 可选，父消息的 ID |

### ToolCallArgs

**表示：工具调用的参数数据块**

`ToolCallArgs` 事件传递工具参数的增量部分。

| 属性 | 说明 |
|:---|:---|
| `toolCallId` | 与 `ToolCallStart` 的 ID 匹配 |
| `delta` | 参数内容块 |

### ToolCallEnd

**信号：工具调用结束**

`ToolCallEnd` 事件标记工具调用的完成。

| 属性 | 说明 |
|:---|:---|
| `toolCallId` | 与 `ToolCallStart` 的 ID 匹配 |

---

## 状态管理事件

状态管理事件用于 Agent 与前端之间的状态同步。

| 事件 | 说明 |
|:---|:---|
| `STATE_SNAPSHOT` | 完整状态快照 |
| `STATE_DELTA` | 增量状态更新（JSON Patch） |
| `MESSAGES_SNAPSHOT` | 完整对话历史快照 |

详细说明请参阅 [状态管理](../04-状态管理/README.md) 章节。

---

## 特殊事件

### RAW

保留用于传递原始事件数据，不经过 AG-UI 转换。

### CUSTOM

用于自定义应用特定的事件扩展。

---

## 事件代码示例

### 基本消息流

```typescript
import { EventType, RunStartedEvent, TextMessageStartEvent, 
         TextMessageContentEvent, TextMessageEndEvent, RunFinishedEvent }

// Agent 端发送事件
async function* eventGenerator(): AsyncGenerator<BaseEvent> {
  yield new RunStartedEvent({
    type: EventType.RUN_STARTED,
    threadId: "thread-123",
    runId: "run-456"
  });
  
  const messageId = "msg-789";
  yield new TextMessageStartEvent({
    type: EventType.TEXT_MESSAGE_START,
    messageId,
    role: "assistant"
  });
  
  const response = "Hello, how can I help you?";
  for (const chunk of response) {
    yield new TextMessageContentEvent({
      type: EventType.TEXT_MESSAGE_CONTENT,
      messageId,
      delta: chunk
    });
  }
  
  yield new TextMessageEndEvent({
    type: EventType.TEXT_MESSAGE_END,
    messageId
  });
  
  yield new RunFinishedEvent({
    type: EventType.RUN_FINISHED,
    threadId: "thread-123",
    runId: "run-456"
  });
}
```

### 客户端处理事件

```typescript
agent.runAgent({}).subscribe({
  next: (event) => {
    switch (event.type) {
      case EventType.RUN_STARTED:
        console.log("Run started:", event.runId);
        break;
        
      case EventType.TEXT_MESSAGE_START:
        currentMessageId = event.messageId;
        console.log("Message started, role:", event.role);
        break;
        
      case EventType.TEXT_MESSAGE_CONTENT:
        // 追加内容到 UI
        appendToChat(event.delta);
        break;
        
      case EventType.TEXT_MESSAGE_END:
        console.log("Message ended");
        break;
        
      case EventType.TOOL_CALL_START:
        console.log("Tool call:", event.toolCallName);
        break;
        
      case EventType.RUN_FINISHED:
        console.log("Run finished");
        break;
        
      case EventType.RUN_ERROR:
        console.error("Error:", event.message);
        break;
    }
  }
});
```

---

## 事件验证

事件是严格类型化的。AG-UI SDK 提供了完整的事件类型定义和运行时验证：

```typescript
import { BaseEvent, EventType, validateEvent } from "@ag-ui/core";

// 验证事件
function handleEvent(event: BaseEvent) {
  if (!validateEvent(event)) {
    throw new Error(`Invalid event: ${event.type}`);
  }
  // 处理事件
}
```

---

## 下一步

- [状态管理](../04-状态管理/README.md) — 深入了解状态同步
- [工具与人机协作](../05-工具与人机协作/README.md) — 学习工具调用
- [SDK 参考](../08-SDK参考/README.md) — 查看完整 API

---

*参考资料：[AG-UI Events](https://docs.ag-ui.com/concepts/events.md)*
