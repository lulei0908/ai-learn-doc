# 05. 工具与人机协作

## 概述

工具是 AG-UI 协议中的基础概念，使 AI Agent 能够与外部系统交互并将人类判断纳入其工作流。通过在前端定义工具并传递给 Agent，开发者可以创建复杂的人机协作体验，结合 AI 能力与人类专业知识。

---

## 什么是工具？

在 AG-UI 中，工具是 Agent 可以调用的函数，用于：

1. **请求特定信息** — 获取数据或上下文
2. **在外部系统中执行操作** — 调用 API、修改数据库等
3. **请求人类输入或确认** — Human-in-the-Loop
4. **访问专业能力** — 使用专门的 AI 模型或服务

工具弥合了 AI 推理与现实世界行动之间的差距，让 Agent 能够完成仅靠对话无法完成的任务。

---

## 工具结构

工具遵循一致的结构，定义其名称、用途和预期参数：

```typescript
interface Tool {
  name: string           // 工具的唯一标识符
  description: string    // 人类可读的工具功能说明
  parameters: {
    // 定义工具参数的 JSON Schema
    type: "object"
    properties: {
      // 工具特定参数
    }
    required: string[]   // 必需参数名称数组
  }
}
```

`parameters` 字段使用 [JSON Schema](https://json-schema.org/) 定义工具接受的参数结构。此模式被 Agent（用于生成有效的工具调用）和前端（用于验证和解析工具参数）使用。

---

## 前端定义工具

AG-UI 工具系统的一个关键方面是**工具在前端定义并传递给 Agent**：

```typescript
// 在前端定义工具
const userConfirmationTool = {
  name: "confirmAction",
  description: "Ask the user to confirm a specific action before proceeding",
  parameters: {
    type: "object",
    properties: {
      action: {
        type: "string",
        description: "The action that needs user confirmation",
      },
      importance: {
        type: "string",
        enum: ["low", "medium", "high", "critical"],
        description: "The importance level of the action",
      },
    },
    required: ["action"],
  },
};

// 在执行期间将工具传递给 Agent
agent.runAgent({
  tools: [userConfirmationTool],
  // 其他参数...
});
```

### 优势

| 优势 | 说明 |
|:---|:---|
| **前端控制** | 前端决定 Agent 可用的能力 |
| **动态能力** | 基于用户权限、上下文或应用状态添加/移除工具 |
| **关注点分离** | Agent 专注于推理，前端处理工具实现 |
| **安全** | 敏感操作由应用控制，而非 Agent |

---

## 工具调用生命周期

当 Agent 需要使用工具时，它遵循标准化的事件序列：

```
┌─────────────────────────────────────────────────────────────┐
│                    工具调用生命周期                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. TOOL_CALL_START                                         │
│     ├─ toolCallId: 唯一标识符                                │
│     ├─ toolCallName: 工具名称                                │
│     └─ parentMessageId: 可选，引用消息                        │
│                                                             │
│  2. TOOL_CALL_ARGS (流式)                                    │
│     ├─ toolCallId: 与 START 匹配                             │
│     ├─ delta: '{"act'  // 部分 JSON                          │
│     └─ delta: 'ion":"Depl'  // 更多 JSON                     │
│                                                             │
│  3. TOOL_CALL_END                                           │
│     └─ toolCallId: 与 START 匹配                             │
│                                                             │
│  4. 前端执行工具                                             │
│     └─ 执行实际逻辑                                          │
│                                                             │
│  5. TOOL_CALL_RESULT                                        │
│     ├─ id: "result-789"                                     │
│     ├─ role: "tool"                                         │
│     ├─ content: "true"                                      │
│     └─ toolCallId: "tool-123"                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 事件详解

#### ToolCallStart

```typescript
{
  type: EventType.TOOL_CALL_START,
  toolCallId: "tool-123",
  toolCallName: "confirmAction",
  parentMessageId: "msg-456" // 可选
}
```

#### ToolCallArgs

```typescript
{
  type: EventType.TOOL_CALL_ARGS,
  toolCallId: "tool-123",
  delta: '{"action":"Deploy the application to production"}'
}
```

#### ToolCallEnd

```typescript
{
  type: EventType.TOOL_CALL_END,
  toolCallId: "tool-123"
}
```

#### ToolCallResult

```typescript
{
  id: "result-789",
  role: "tool",
  content: "true", // 工具结果作为字符串
  toolCallId: "tool-123" // 引用原始工具调用
}
```

---

## Human-in-the-Loop 工作流

AG-UI 工具系统特别适用于实现人机协作工作流。通过定义请求人类输入或确认的工具，开发者可以创建 AI 体验，将自主操作与人类判断无缝融合。

### 典型流程

```
┌──────────┐      ┌──────────┐      ┌──────────┐
│  Agent   │      │  Frontend│      │   User   │
└────┬─────┘      └────┬─────┘      └────┬─────┘
     │                 │                 │
     │ 需要做出重要决策 │                 │
     │                 │                 │
     │──confirmAction─►│                 │
     │                 │                 │
     │                 │ 显示确认对话框   │
     │                 │────────────────►│
     │                 │                 │
     │                 │◄───────────────│
     │                 │ 用户输入        │
     │                 │                 │
     │◄──result────────│                 │
     │                 │                 │
     │ 继续处理         │                 │
     │                 │                 │
```

### 应用场景

| 场景 | 说明 |
|:---|:---|
| **审批工作流** | AI 建议需要人类批准的操作 |
| **数据验证** | 人类验证或纠正 AI 生成的数据 |
| **协作决策** | AI 和人类共同解决复杂问题 |
| **监督学习** | 人类反馈改进未来 AI 决策 |

---

## CopilotKit 集成

[CopilotKit](https://docs.copilotkit.ai/) 提供了一种简化的方式，在 React 应用中使用 AG-UI 工具，通过 `useCopilotAction` hook：

```tsx
import { useCopilotAction } from "@copilotkit/react-core";

function MyComponent() {
  // 定义用户确认工具
  useCopilotAction({
    name: "confirmAction",
    description: "Ask the user to confirm an action",
    parameters: {
      type: "object",
      properties: {
        action: {
          type: "string",
          description: "The action to confirm",
        },
      },
      required: ["action"],
    },
    handler: async ({ action }) => {
      // 显示确认对话框
      const confirmed = await showConfirmDialog(action);
      return confirmed ? "approved" : "rejected";
    },
  });

  return <div>...</div>;
}
```

---

## 工具示例

### 1. 用户确认

```typescript
{
  name: "confirmAction",
  description: "Ask the user to confirm an action",
  parameters: {
    type: "object",
    properties: {
      action: {
        type: "string",
        description: "The action to confirm"
      },
      importance: {
        type: "string",
        enum: ["low", "medium", "high", "critical"],
        description: "The importance level"
      }
    },
    required: ["action"]
  }
}
```

### 2. 数据检索

```typescript
{
  name: "fetchUserData",
  description: "Retrieve data about a specific user",
  parameters: {
    type: "object",
    properties: {
      userId: {
        type: "string",
        description: "ID of the user"
      },
      fields: {
        type: "array",
        items: { type: "string" },
        description: "Fields to retrieve"
      }
    },
    required: ["userId"]
  }
}
```

### 3. 用户界面控制

```typescript
{
  name: "navigateTo",
  description: "Navigate to a different page or view",
  parameters: {
    type: "object",
    properties: {
      destination: {
        type: "string",
        description: "Destination page or view"
      },
      params: {
        type: "object",
        description: "Optional parameters for the navigation"
      }
    },
    required: ["destination"]
  }
}
```

### 4. 内容生成

```typescript
{
  name: "generateImage",
  description: "Generate an image based on a description",
  parameters: {
    type: "object",
    properties: {
      prompt: {
        type: "string",
        description: "Description of the image to generate"
      },
      style: {
        type: "string",
        description: "Visual style for the image"
      },
      dimensions: {
        type: "object",
        properties: {
          width: { type: "number" },
          height: { type: "number" }
        },
        description: "Dimensions of the image"
      }
    },
    required: ["prompt"]
  }
}
```

---

## 完整示例

### Agent 端（Python）

```python
from ag_ui.core import EventType, ToolCallStartEvent, ToolCallArgsEvent, ToolCallEndEvent

async def handle_tool_call(tool_name, tool_args):
    if tool_name == "confirmAction":
        # 发送工具调用事件
        tool_call_id = str(uuid.uuid4())
        
        yield ToolCallStartEvent(
            type=EventType.TOOL_CALL_START,
            toolCallId=tool_call_id,
            toolCallName=tool_name,
        )
        
        # 流式发送参数
        args_json = json.dumps(tool_args)
        for chunk in args_json:
            yield ToolCallArgsEvent(
                type=EventType.TOOL_CALL_ARGS,
                toolCallId=tool_call_id,
                delta=chunk
            )
        
        yield ToolCallEndEvent(
            type=EventType.TOOL_CALL_END,
            toolCallId=tool_call_id
        )
        
        # 等待前端返回结果
        result = await wait_for_tool_result(tool_call_id)
        return result
```

### 前端端（React）

```tsx
import { useCopilotAction } from "@copilotkit/react-core";

function App() {
  // 定义工具
  useCopilotAction({
    name: "confirmAction",
    description: "Ask the user to confirm an action",
    parameters: {
      type: "object",
      properties: {
        action: { type: "string" },
        importance: { 
          type: "string",
          enum: ["low", "medium", "high", "critical"]
        }
      },
      required: ["action"]
    },
    handler: async ({ action, importance }) => {
      const confirmed = await showConfirmDialog(action, importance);
      return confirmed ? "approved" : "rejected";
    }
  });

  return (
    <CopilotKit url="/api/copilot">
      <ChatInterface />
    </CopilotKit>
  );
}
```

---

## 最佳实践

1. **清晰的命名** — 使用描述性的、动作导向的名称
2. **详细的描述** — 包含全面的描述，帮助 Agent 理解何时以及如何使用工具
3. **结构化的参数** — 定义精确的参数模式，使用描述性的字段名称和约束
4. **必需的字段** — 只将真正必要的参数标记为必需
5. **错误处理** — 在工具执行代码中实现健壮的错误处理
6. **用户体验** — 设计为人工决策提供适当上下文的工具 UI

---

## 总结

AG-UI 中的工具弥合了 AI 推理与现实世界行动之间的差距，实现结合 AI 和人类智能优势的工作流。通过在前端定义工具并传递给 Agent，开发者可以创建 AI 和人类高效协作的交互体验。

工具系统对于实现人机协作工作流特别强大，AI 可以建议操作，但将关键决策推迟给人类。这在自动化与人类判断之间取得平衡，创造既强大又值得信赖的 AI 体验。

---

## 下一步

- [中间件](../06-中间件/README.md) — 学习如何处理工具调用事件
- [快速开始](../07-快速开始/README.md) — 动手实践
- [SDK 参考](../08-SDK参考/README.md) — 查看完整 API

---

*参考资料：[AG-UI Tools](https://docs.ag-ui.com/concepts/tools.md)*
