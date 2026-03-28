# 04. AG-UI 状态管理

## 概述

状态管理是 AG-UI 协议的核心功能，支持 Agent 与前端应用之间的实时同步。通过提供高效的状态共享和更新机制，AG-UI 为人类与 AI Agent 协作体验奠定了基础。

---

## 共享状态架构

在 AG-UI 中，状态是一个结构化数据对象，具备以下特性：

1. **跨交互持久化** — 在与 Agent 的交互过程中持久存在
2. **双向访问** — Agent 和前端都可以访问
3. **实时更新** — 交互过程中实时变化
4. **决策上下文** — 为双方决策提供上下文

### 双向通信

```
┌──────────────┐                    ┌──────────────┐
│              │  STATE_SNAPSHOT    │              │
│   Frontend   │◄──────────────────│     Agent    │
│   (UI)       │                    │   (Backend)  │
│              │  STATE_DELTA       │              │
│              │───────────────────►│              │
└──────────────┘                    └──────────────┘
     │                                        │
     │   Agent 可访问应用的当前状态            │
     │   来做出明智的决策                      │
     │                                        │
     │   前端可观察和响应 Agent               │
     │   内部状态的变化                       │
     │                                        │
     │   双方都可以修改状态                    │
     │   创建协作工作流                        │
     └────────────────────────────────────────┘
```

---

## 状态同步方法

AG-UI 提供了两种互补的状态同步方法。

### State Snapshots（状态快照）

`STATE_SNAPSHOT` 事件交付 Agent 当前状态的完整表示：

```typescript
interface StateSnapshotEvent {
  type: EventType.STATE_SNAPSHOT
  snapshot: any // 完整状态对象
}
```

**使用场景：**

- 交互开始时建立初始状态
- 连接中断后确保同步
- 发生需要完全刷新的重大状态变化时
- 为未来的增量更新建立新基线

**使用方式：** 前端接收到 `STATE_SNAPSHOT` 事件时，应完全替换现有的状态模型。

---

### State Deltas（状态增量）

`STATE_DELTA` 事件使用 JSON Patch 格式（RFC 6902）交付状态的增量更新：

```typescript
interface StateDeltaEvent {
  type: EventType.STATE_DELTA
  delta: JsonPatchOperation[] // JSON Patch 操作数组
}
```

**优势：**

- **带宽高效** — 只发送变化的部分，而非整个状态
- **适合频繁的小更新**
- **适合大部分属性不变的大状态对象**
- **适合高频更新场景**

---

## JSON Patch 格式

AG-UI 使用 JSON Patch 格式（RFC 6902）来表达 JSON 文档的变化。

### 操作类型

```typescript
interface JsonPatchOperation {
  op: "add" | "remove" | "replace" | "move" | "copy" | "test"
  path: string // JSON Pointer (RFC 6901) 到目标位置的路径
  value?: any  // 要应用的值（用于 add, replace）
  from?: string // 源路径（用于 move, copy）
}
```

### 常用操作示例

#### 1. add — 添加值

```json
{ "op": "add", "path": "/user/preferences", "value": { "theme": "dark" } }
```

#### 2. replace — 替换值

```json
{ "op": "replace", "path": "/conversation_state", "value": "paused" }
```

#### 3. remove — 移除值

```json
{ "op": "remove", "path": "/temporary_data" }
```

#### 4. move — 移动值

```json
{ "op": "move", "path": "/completed_items", "from": "/pending_items/0" }
```

#### 5. test — 测试值（验证）

```json
{ "op": "test", "path": "/version", "value": 2 }
```

#### 6. copy — 复制值

```json
{ "op": "copy", "path": "/backup/user", "from": "/user" }
```

---

## 状态处理实现

在 AG-UI 实现中，状态增量使用 `fast-json-patch` 库应用：

```typescript
case EventType.STATE_DELTA: {
  const { delta } = event as StateDeltaEvent;

  try {
    // 应用 JSON Patch 操作，不修改原始状态
    const result = applyPatch(state, delta, true, false);
    state = result.newDocument;
    return emitUpdate({ state });
  } catch (error: unknown) {
    console.warn(
      `Failed to apply state patch:\n` +
      `Current state: ${JSON.stringify(state, null, 2)}\n` +
      `Patch operations: ${JSON.stringify(delta, null, 2)}\n` +
      `Error: ${errorMessage}`
    );
    return emitNoUpdate();
  }
}
```

**实现要点：**

- 补丁以原子方式应用（全有或全无）
- 应用过程中不会修改原始状态
- 错误被捕获并优雅处理

---

## Human-in-the-Loop 协作

共享状态系统是 AG-UI 中 Human-in-the-Loop 工作流的基础。它实现：

1. **实时可见性** — 用户可以观察 Agent 的思维过程和当前状态
2. **上下文感知** — Agent 可以访问用户操作、偏好和应用状态
3. **协作决策** — 人类和 AI 都可以为演进中的状态做出贡献
4. **反馈循环** — 人类可以通过修改状态属性来纠正或引导 Agent

### 协作示例

Agent 可能在状态中更新一个提议：

```json
{
  "proposal": {
    "action": "send_email",
    "recipient": "client@example.com",
    "content": "Draft email content..."
  }
}
```

前端可以向用户显示此提议，用户可以：
- **批准** — 同意执行
- **拒绝** — 取消操作
- **修改** — 调整后执行

---

## CopilotKit 实现

[CopilotKit](https://docs.copilotkit.ai) 是一个流行的 AI 助手构建框架，通过其"共享状态"功能利用 AG-UI 的状态管理系统。

### 前端使用

```tsx
// 在 React 应用中使用
import { useCoAgent } from "@copilotkit/react-core";

function MyComponent() {
  const { state: agentState, setState: setAgentState } = useCoAgent({
    name: "research_agent",
    initialState: { 
      someProperty: "initialValue" 
    },
  });

  return (
    <div>
      <p>Agent state: {JSON.stringify(agentState)}</p>
      <button onClick={() => setAgentState({...agentState, newField: "value"})}>
        Update State
      </button>
    </div>
  );
}
```

### 后端更新状态

```python
# 在 LangGraph Agent 中
from copilotkit import copilotkit_emit_state

async def tool_node(self, state: ResearchState, config: RunnableConfig):
    # 用新信息更新状态
    tool_state = {
        "title": new_state.get("title", ""),
        "outline": new_state.get("outline", {}),
        "sections": new_state.get("sections", []),
        # 其他状态属性...
    }

    # 向前端发送更新的状态
    await copilotkit_emit_state(config, tool_state)

    return tool_state
```

---

## 完整示例：聊天 + 状态

### Agent 端

```python
from ag_ui.core import (
    EventType, StateSnapshotEvent, StateDeltaEvent
)
from fast_json_patch import compare, apply_patch

current_state = {"messages": [], "context": {}}

async def emit_state_delta(new_state):
    delta = compare(current_state, new_state)
    if delta:
        yield {
            "type": EventType.STATE_DELTA,
            "delta": delta
        }
        current_state = apply_patch(current_state, delta).new_document

async def emit_state_snapshot(state):
    yield {
        "type": EventType.STATE_SNAPSHOT,
        "snapshot": state
    }
    current_state = state
```

### 前端端

```typescript
import { useCoAgent } from "@copilotkit/react-core";

function ChatInterface() {
  const { state: agentState } = useCoAgent({
    name: "chat_agent",
    initialState: { messages: [], context: {} }
  });

  // 响应状态变化
  useEffect(() => {
    console.log("State updated:", agentState);
  }, [agentState]);

  return (
    <div>
      {agentState.messages.map((msg, i) => (
        <Message key={i} {...msg} />
      ))}
    </div>
  );
}
```

---

## 最佳实践

1. **明智地使用快照** — 只在必要时发送完整快照来建立基线
2. **优先使用增量** — 小状态更新使用增量以最小化数据传输
3. **深思熟虑地设计状态** — 设计支持部分更新且补丁复杂度最小的状态对象
4. **处理状态冲突** — 实现解决 Agent 和前端冲突更新的策略
5. **包含错误恢复** — 提供在检测到不一致时重新同步状态的机制
6. **考虑安全影响** — 避免在共享状态中存储敏感信息

---

## 总结

AG-UI 的状态管理系统为构建人类与 AI Agent 协作应用提供了强大的基础。通过状态快照和 JSON Patch 增量在前后端之间高效同步状态，AG-UI 支持复杂的人机协作工作流，结合了人类直觉和 AI 能力的优势。

CopilotKit 等框架的实现展示了这种共享状态方法如何创建比完全自主系统或传统用户界面更有效的协作体验。

---

## 下一步

- [工具与人机协作](../05-工具与人机协作/README.md) — 深入了解工具调用
- [中间件](../06-中间件/README.md) — 学习如何处理和转换状态事件
- [快速开始](../07-快速开始/README.md) — 动手实践

---

*参考资料：[AG-UI State Management](https://docs.ag-ui.com/concepts/state.md)*
