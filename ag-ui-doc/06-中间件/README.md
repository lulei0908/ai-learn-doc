# 06. 中间件

## 概述

中间件是 AG-UI 中强大的机制，用于转换、过滤和增强流经 Agent 的事件流。它使你能够添加横切关注点（如日志记录、身份验证、速率限制和事件过滤），而无需修改核心 Agent 逻辑。

---

## 什么是中间件？

中间件位于 Agent 执行和事件消费者之间，允许你：

1. **转换事件** — 在事件流经管道时修改或增强事件
2. **过滤事件** — 选择性地允许或阻止某些事件
3. **添加元数据** — 注入额外的上下文或追踪信息
4. **处理错误** — 实现自定义错误恢复策略
5. **监控执行** — 添加日志、指标或调试能力

---

## 中间件如何工作

中间件形成链式结构，每个中间件包装下一个，创建功能层。当 Agent 运行时，事件流按顺序流经每个中间件：

```
┌─────────────────────────────────────────────────────────────┐
│                    中间件链执行流程                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   agent.use(logging, auth, filter)                          │
│                                                             │
│   ┌─────────┐                                              │
│   │ Middleware 1 │ ← 首先接收原始输入                        │
│   │   (日志)    │                                          │
│   └──┬──────┘                                              │
│      │                                                     │
│      ▼                                                     │
│   ┌─────────┐                                              │
│   │ Middleware 2 │ ← 可以修改输入后再传递                   │
│   │   (认证)    │                                          │
│   └──┬──────┘                                              │
│      │                                                     │
│      ▼                                                     │
│   ┌─────────┐                                              │
│   │ Middleware 3 │ ← 最内层中间件                           │
│   │   (过滤)    │                                          │
│   └──┬──────┘                                              │
│      │                                                     │
│      ▼                                                     │
│   ┌─────────┐                                              │
│   │  Agent  │ ← 实际 Agent 执行                            │
│   └─────────┘                                              │
│                                                             │
│   事件沿相反方向流回：Agent → Middleware 3 → Middleware 2   │
│                                   → Middleware 1 → 客户端   │
└─────────────────────────────────────────────────────────────┘
```

---

## 函数式中间件

对于简单的转换，可以使用函数式中间件。这是最简洁的添加中间件方式：

```typescript
import { MiddlewareFunction } from "@ag-ui/client";
import { EventType } from "@ag-ui/core";
import { map } from "rxjs/operators";

// 为所有文本消息添加前缀
const prefixMiddleware: MiddlewareFunction = (input, next) => {
  return next.run(input).pipe(
    map(event => {
      if (
        event.type === EventType.TEXT_MESSAGE_CHUNK ||
        event.type === EventType.TEXT_MESSAGE_CONTENT
      ) {
        return {
          ...event,
          delta: `[AI]: ${event.delta}`
        }
      }
      return event
    })
  )
}

agent.use(prefixMiddleware)
```

### 更多函数式中间件示例

#### 日志中间件

```typescript
const loggingMiddleware: MiddlewareFunction = (input, next) => {
  console.log("Agent starting with input:", input);
  
  return next.run(input).pipe(
    tap(event => {
      console.log("Event received:", event.type);
    }),
    finalize(() => {
      console.log("Agent run completed");
    })
  );
};
```

#### 事件统计中间件

```typescript
const metricsMiddleware: MiddlewareFunction = (input, next) => {
  const startTime = Date.now();
  let eventCount = 0;

  return next.run(input).pipe(
    tap(() => eventCount++),
    finalize(() => {
      const duration = Date.now() - startTime;
      console.log(`Agent ran for ${duration}ms, received ${eventCount} events`);
    })
  );
};
```

---

## 类式中间件

对于需要状态或配置更复杂场景，使用类式中间件：

```typescript
import { Middleware, AbstractAgent } from "@ag-ui/client";
import { Observable } from "rxjs";
import { tap, finalize } from "rxjs/operators";

class MetricsMiddleware extends Middleware {
  private eventCount = 0;

  constructor(private metricsService: MetricsService) {
    super();
  }

  run(input: RunAgentInput, next: AbstractAgent): Observable<BaseEvent> {
    const startTime = Date.now();

    return this.runNext(input, next).pipe(
      tap(event => {
        this.eventCount++;
        this.metricsService.recordEvent(event.type);
      }),
      finalize(() => {
        const duration = Date.now() - startTime;
        this.metricsService.recordDuration(duration);
        this.metricsService.recordEventCount(this.eventCount);
      })
    );
  }
}

// 使用
agent.use(new MetricsMiddleware(metricsService));
```

### 类式中间件辅助方法

编写类中间件时，优先使用辅助方法：

| 方法 | 说明 |
|:---|:---|
| `runNext(input, next)` | 标准化块事件为完整的 `TEXT_MESSAGE_*`/`TOOL_CALL_*` 序列 |
| `runNextWithState(input, next)` | 还提供每个事件后的累积 `messages` 和 `state` |

---

## 内置中间件

AG-UI 提供了几个内置中间件组件，用于常见场景。

### FilterToolCallsMiddleware

根据允许或禁止列表过滤工具调用：

```typescript
import { FilterToolCallsMiddleware } from "@ag-ui/client";

// 只允许特定工具
const allowedFilter = new FilterToolCallsMiddleware({
  allowedToolCalls: ["search", "calculate"]
});

// 或阻止特定工具
const blockedFilter = new FilterToolCallsMiddleware({
  disallowedToolCalls: ["delete", "modify", "send_email"]
});

agent.use(allowedFilter);
```

> **注意**：`FilterToolCallsMiddleware` 过滤发出的 `TOOL_CALL_*` 事件。它不会阻止上游模型/运行时的工具执行。

---

## 中间件模式

### 认证中间件

```typescript
const authMiddleware: MiddlewareFunction = (input, next) => {
  // 检查认证
  const token = input.forwardedProps?.authToken;
  
  if (!token) {
    throw new Error("Unauthorized");
  }
  
  // 验证令牌
  const user = validateToken(token);
  if (!user) {
    throw new Error("Invalid token");
  }
  
  // 传递带有用户信息的输入
  return next.run({
    ...input,
    forwardedProps: {
      ...input.forwardedProps,
      user
    }
  });
};
```

### 速率限制中间件

```typescript
import { timer, windowTime, switchMap, tap } from "rxjs/operators";

const rateLimitMiddleware: MiddlewareFunction = (input, next) => {
  return next.run(input).pipe(
    // 每秒最多 10 个事件
    windowTime(1000),
    switchMap(window => window.pipe(
      scan((count, event) => {
        if (count >= 10) {
          throw new Error("Rate limit exceeded");
        }
        return count + 1;
      }, 0)
    ))
  );
};
```

### 调试中间件

```typescript
const debugMiddleware: MiddlewareFunction = (input, next) => {
  if (input.forwardedProps?.debug === true) {
    return next.run(input).pipe(
      tap(event => {
        console.debug(`[DEBUG] Event: ${event.type}`, {
          event,
          timestamp: new Date().toISOString()
        });
      })
    );
  }
  return next.run(input);
};
```

---

## 组合中间件

可以组合多个中间件创建复杂的处理管道：

```typescript
// 多个中间件组合
const logMiddleware: MiddlewareFunction = (input, next) => next.run(input);
const metricsMiddleware = new MetricsMiddleware(metricsService);
const filterMiddleware = new FilterToolCallsMiddleware({ allowedToolCalls: ["search"] });
const authMiddleware: MiddlewareFunction = (input, next) => next.run(input);

// 按顺序应用
agent.use(logMiddleware, metricsMiddleware, filterMiddleware, authMiddleware);
```

---

## 执行顺序

中间件按添加顺序执行，每个中间件包装下一个：

```typescript
agent.use(middleware1, middleware2, middleware3)

// 执行流程：
// → middleware1
//   → middleware2
//     → middleware3
//       → agent.run()
//     ← 事件流经 middleware3 返回
//   ← 事件流经 middleware2 返回
// ← 事件流经 middleware1 返回
```

---

## 中间件与 `connectAgent`

- 使用 `agent.use(...)` 添加的中间件在 `runAgent()` 中应用
- `connect()` 当前直接调用 `connect()`，不运行中间件

---

## 最佳实践

1. **保持中间件专注** — 每个中间件应只有一个职责
2. **优雅处理错误** — 使用 RxJS 错误处理操作符
3. **避免阻塞操作** — 使用 I/O 操作的异步模式
4. **记录副作用** — 明确说明中间件是否修改状态
5. **独立测试中间件** — 为每个中间件编写单元测试
6. **考虑性能** — 注意事件流中的处理开销

---

## 总结

中间件提供了一种灵活而强大的方式扩展 AG-UI Agent，而无需修改其核心逻辑。无论是简单的事件转换还是复杂的有状态处理，中间件系统都提供了构建健壮、可维护的 Agent 应用的工具。

---

## 下一步

- [快速开始](../07-快速开始/README.md) — 在实践中使用中间件
- [SDK 参考](../08-SDK参考/README.md) — 查看完整中间件 API

---

*参考资料：[AG-UI Middleware](https://docs.ag-ui.com/concepts/middleware.md)*
