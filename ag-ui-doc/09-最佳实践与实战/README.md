# 09. AG-UI 最佳实践与实战

## 设计模式

### 1. 事件流压缩

当 Agent 生成大量流式事件时，使用 `compactEvents` 实用工具减少冗余：

```typescript
import { compactEvents } from "@ag-ui/client";

// 将 TextMessageChunk 序列压缩为标准 Start/Content/End 序列
agent.runAgent({}).pipe(
  compactEvents()
).subscribe({
  next: (event) => console.log(event.type)
});
```

**好处：**
- 减少 UI 更新频率
- 降低内存使用
- 简化事件处理逻辑

### 2. 懒加载状态

对于大型状态对象，避免一次性发送完整快照：

```python
# 好的做法：先发送核心状态，后续增量更新
yield StateSnapshotEvent(
    type=EventType.STATE_SNAPSHOT,
    snapshot={"title": "...", "status": "loading"}
)

# 后续通过 delta 发送详细信息
yield StateDeltaEvent(
    type=EventType.STATE_DELTA,
    delta=[
        {"op": "add", "path": "/details", "value": {...}}
    ]
)
```

### 3. 错误恢复策略

```typescript
agent.runAgent({}).subscribe({
  next: (event) => {
    // 处理事件
  },
  error: (err) => {
    // 实现指数退避重试
    setTimeout(() => {
      retryWithBackoff(agent, maxRetries: 3);
    }, Math.pow(2, retryCount) * 1000);
  },
  complete: () => {
    // 运行完成
  }
});
```

---

## 调试技巧

### 1. 调试中间件

使用日志中间件记录所有事件：

```typescript
const debugMiddleware: MiddlewareFunction = (input, next) => {
  console.log("── Agent Run Started ──");
  console.log("Thread ID:", input.threadId);
  console.log("Run ID:", input.runId);
  
  return next.run(input).pipe(
    tap(event => {
      const type = event.type;
      const ts = event.timestamp 
        ? new Date(event.timestamp).toISOString() 
        : Date.now();
      
      console.log(`[${ts}] Event: ${type}`);
      
      // 详细输出特定事件
      switch (type) {
        case EventType.TEXT_MESSAGE_CONTENT:
          console.log(`  delta: "${(event as any).delta}"`);
          break;
        case EventType.TOOL_CALL_START:
          console.log(`  tool: ${(event as any).toolCallName}`);
          break;
        case EventType.STATE_DELTA:
          console.log(`  patches:`, (event as any).delta);
          break;
      }
    }),
    finalize(() => {
      console.log("── Agent Run Complete ──");
    })
  );
};

agent.use(debugMiddleware);
```

### 2. 使用 Cursor IDE

AG-UI 官方推荐使用 [Cursor](https://cursor.com) 来加速 AG-UI 的开发。Cursor 能够理解 AG-UI 的协议规范和 SDK API，提供智能代码补全和错误检测。

### 3. AG-UI Dojo

AG-UI 提供了 Dojo（道场）——一个交互式的测试环境：

```bash
# 克隆仓库
git clone git@github.com:ag-ui-protocol/ag-ui.git
cd ag-ui

# 安装依赖并启动
pnpm install
pnpm dev

# 访问 http://localhost:3000
```

Dojo 允许你：
- 测试不同的 Agent 集成
- 可视化事件流
- 调试事件序列
- 测试前端与后端的交互

---

## 框架集成实战

### 集成 LangGraph + CopilotKit

#### 后端（Python + LangGraph）

```python
from typing import Annotated
from typing_extensions import TypedDict
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langgraph.checkpoint.memory import MemorySaver
from copilotkit import CopilotKitContext, copilotkit_emit_message, copilotkit_emit_state

class State(TypedDict):
    messages: Annotated[list, add_messages]

# 定义工具
async def search_tool(state: State, config: RunnableConfig):
    """搜索工具"""
    ck_context = CopilotKitContext.from_config(config)
    await ck_context.emit_state({
        "is_searching": True,
        "search_query": state["messages"][-1].content
    })
    
    # 执行搜索
    results = await search(state["messages"][-1].content)
    
    await ck_context.emit_state({
        "is_searching": False,
        "results": results
    })
    
    return {"messages": [{"role": "assistant", "content": str(results)}]}

# 构建图
workflow = StateGraph(State)
workflow.add_node("agent", model)
workflow.add_node("search", search_tool)
workflow.add_edge(START, "agent")
workflow.add_conditional_edges("agent", should_search, {"search": "search", "end": END})
workflow.add_edge("search", "agent")

# 创建应用
checkpointer = MemorySaver()
app = workflow.compile(checkpointer=checkpointer)
```

#### 前端（React + CopilotKit）

```tsx
import { CopilotKit, CopilotSidebar, useCopilotAction, useCoAgent } from "@copilotkit/react-core";
import "@copilotkit/react-ui/styles.css";

function ResearchApp() {
  return (
    <CopilotKit url="/api/copilot">
      <ResearchInterface />
      <CopilotSidebar />
    </CopilotKit>
  );
}

function ResearchInterface() {
  const { state: agentState } = useCoAgent({
    name: "research_agent",
    initialState: { is_searching: false, results: null }
  });

  // 定义前端工具
  useCopilotAction({
    name: "showResults",
    description: "Display search results to the user",
    parameters: {
      type: "object",
      properties: {
        title: { type: "string" },
        content: { type: "string" }
      },
      required: ["title", "content"]
    },
    handler: async ({ title, content }) => {
      displayResults(title, content);
    }
  });

  return (
    <div>
      {agentState.is_searching && <Spinner message="Searching..." />}
      {agentState.results && <ResultsList results={agentState.results} />}
    </div>
  );
}
```

---

## 性能优化

### 1. 事件批处理

对于高频事件（如 STATE_DELTA），合并多个操作：

```python
# 不好的做法：每次变化都发送
for item in items:
    yield StateDeltaEvent(
        type=EventType.STATE_DELTA,
        delta=[{"op": "add", "path": f"/items/{i}", "value": item}]
    )

# 好的做法：批量合并
all_patches = []
for i, item in enumerate(items):
    all_patches.append({"op": "add", "path": f"/items/{i}", "value": item})

yield StateDeltaEvent(
    type=EventType.STATE_DELTA,
    delta=all_patches
)
```

### 2. 二进制传输

在生产环境使用二进制传输以获得更好的性能：

```typescript
const agent = new HttpAgent({
  url: "https://agent.example.com",
  transport: "binary",  // 使用二进制传输
});
```

### 3. 选择性事件订阅

如果不需要所有事件，使用中间件过滤：

```typescript
import { map, filter } from "rxjs/operators";

agent.runAgent({}).pipe(
  filter(event => {
    // 只处理需要的 events
    return [
      EventType.TEXT_MESSAGE_CONTENT,
      EventType.RUN_FINISHED,
      EventType.RUN_ERROR
    ].includes(event.type);
  })
).subscribe({
  next: (event) => handleEvent(event)
});
```

---

## 安全考虑

### 1. 认证与授权

```typescript
const authMiddleware: MiddlewareFunction = (input, next) => {
  const token = input.forwardedProps?.authToken;
  
  if (!token) {
    return throwError(() => new Error("Authentication required"));
  }
  
  return next.run(input).pipe(
    // 确保敏感信息不被泄露
    map(event => {
      if (event.type === EventType.STATE_DELTA) {
        return filterSensitiveData(event);
      }
      return event;
    })
  );
};
```

### 2. 输入验证

```python
from pydantic import BaseModel, validator

class AgentInput(BaseModel):
    thread_id: str
    run_id: str
    messages: list[Message]
    
    @validator("messages")
    def validate_messages(cls, v):
        if len(v) > 100:
            raise ValueError("Too many messages")
        return v
```

### 3. 工具访问控制

```typescript
// 前端根据用户权限动态传递工具
const userTools = getUserTools(currentUser.permissions);
agent.runAgent({
  tools: userTools  // 只传递用户有权使用的工具
});
```

---

## 测试策略

### 1. 事件流测试

```typescript
import { of } from "rxjs";
import { toArray } from "rxjs/operators";

async function testAgentEvents() {
  const agent = new MyAgent();
  const events = await agent.runAgent({
    messages: [{ role: "user", content: "test" }]
  }).pipe(toArray()).toPromise();
  
  // 验证事件序列
  expect(events[0].type).toBe(EventType.RUN_STARTED);
  expect(events[events.length - 1].type).toBe(EventType.RUN_FINISHED);
  
  // 验证消息内容
  const contentEvents = events.filter(e => e.type === EventType.TEXT_MESSAGE_CONTENT);
  const fullMessage = contentEvents.map(e => (e as any).delta).join("");
  expect(fullMessage).toBeTruthy();
}
```

### 2. 中间件测试

```typescript
async function testMiddleware() {
  const agent = new MyAgent();
  agent.use(myMiddleware);
  
  const events = await agent.runAgent({}).pipe(toArray()).toPromise();
  
  // 验证中间件转换
  const textEvents = events.filter(e => e.type === EventType.TEXT_MESSAGE_CONTENT);
  textEvents.forEach(event => {
    expect((event as any).delta).toMatch(/^\[AI\]:/);  // 验证前缀中间件
  });
}
```

### 3. 端到端测试

```typescript
import request from "supertest";

async function testEndpoint() {
  const response = await request(app)
    .post("/agent")
    .send({
      thread_id: "test-thread",
      run_id: "test-run",
      messages: [{ role: "user", content: "hello" }]
    })
    .set("Accept", "text/event-stream")
    .expect(200);
  
  // 解析 SSE 事件
  const events = parseSSEEvents(response.text);
  expect(events[0].type).toBe("RUN_STARTED");
}
```

---

## 部署建议

### 1. 生产环境配置

```python
# 生产环境 FastAPI 配置
from fastapi import FastAPI
from fastapi.middleware.gzip import GZipMiddleware

app = FastAPI(
    title="AG-UI Agent",
    docs_url=None,       # 禁用 docs
    redoc_url=None,       # 禁用 redoc
)

# 启用压缩
app.add_middleware(GZipMiddleware, minimum_size=1000)

# CORS 配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourapp.com"],
    allow_credentials=True,
    allow_methods=["POST"],
    allow_headers=["Content-Type", "Accept"],
)
```

### 2. 监控

```typescript
// 使用中间件添加监控
class MonitoringMiddleware extends Middleware {
  run(input: RunAgentInput, next: AbstractAgent) {
    const startTime = Date.now();
    const eventId = generateTraceId();
    
    return this.runNext(input, next).pipe(
      tap(event => {
        trackEvent(eventId, event.type, Date.now() - startTime);
      }),
      catchError(err => {
        trackError(eventId, err);
        return throwError(() => err);
      })
    );
  }
}
```

---

## 学习资源

- [AG-UI 官方文档](https://docs.ag-ui.com) — 完整的官方文档
- [AG-UI GitHub](https://github.com/ag-ui-protocol/ag-ui) — 源代码和示例
- [CopilotKit 文档](https://docs.copilotkit.ai) — 最流行的 AG-UI 实现框架
- [AG-UI Dojo](http://localhost:3000) — 交互式测试环境
- [AG-UI OpenAPI](https://docs.ag-ui.com/api-reference/openapi.json) — OpenAPI 规范

---

*参考资料：[AG-UI Debugging](https://docs.ag-ui.com/tutorials/debugging.md) | [Cursor Guide](https://docs.ag-ui.com/tutorials/cursor.md) | [Roadmap](https://docs.ag-ui.com/development/roadmap.md)*
