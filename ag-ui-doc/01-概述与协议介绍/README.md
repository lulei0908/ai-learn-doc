# 01. AG-UI 概述与协议介绍

## 什么是 AG-UI？

**AG-UI（Agent-User Interaction Protocol）** 是一个**开放、轻量、基于事件**的协议，专门用于标准化 AI Agent 与用户界面应用之间的连接方式。

AG-UI 被设计为通用的、双向的连接层，连接用户界面应用与任意 Agentic 后端。它标准化了 Agent 状态、UI 意图和用户交互在模型/Agent 运行时与前端应用之间的流转方式，让应用开发者能够快速构建可靠、可调试、用户友好的 Agentic 功能，同时专注于应用需求，避免复杂的临时接线。

---

## 核心特性

| 特性 | 说明 |
|:---|:---|
| **开放标准** | 完全开源，任何人都可以实现和扩展 |
| **轻量级** | 最小化协议开销，易于集成 |
| **事件驱动** | 基于流式事件，支持实时交互 |
| **双向通信** | Agent 与前端可以互相发送数据 |
| **传输无关** | 支持 SSE、WebSocket、Webhook 等多种传输方式 |
| **框架无关** | 可与任何 Agent 框架（LangGraph、CrewAI、Mastra 等）集成 |

---

## AG-UI 能做什么？

### 当前支持的核心能力

1. **流式聊天（Streaming Chat）**
   - 实时 token 和事件流，支持响应式多轮会话
   - 支持取消和恢复

2. **多模态（Multimodality）**
   - 类型化附件和实时媒体（文件、图片、音频、转录）
   - 支持语音、预览、注释、来源追踪

3. **静态生成式 UI（Generative UI, Static）**
   - 将模型输出渲染为稳定的、类型化的组件，由应用控制

4. **声明式生成式 UI（Generative UI, Declarative）**
   - 用于受约束但开放的 Agent UI 的小型声明式语言
   - Agent 提出树形结构和约束，应用验证并挂载

5. **共享状态（Shared State）**
   - 只读和读写模式
   - Agent 与应用之间的类型化共享存储
   - 通过流式事件溯源差异和冲突解决实现快速协作

---

## AI 协议生态：三大协议

AG-UI 是 AI 协议生态中三大核心协议之一，各自负责不同的交互层：

```
┌─────────────────────────────────────────────────────────────┐
│                    AI 协议栈                                  │
├─────────────────┬───────────────────┬───────────────────────┤
│     层级         │      协议          │       职责             │
├─────────────────┼───────────────────┼───────────────────────┤
│ Agent ↔ 用户    │ AG-UI             │ 连接 Agent 与用户界面  │
│ Agent ↔ 工具    │ MCP               │ 连接 Agent 与工具/数据 │
│ Agent ↔ Agent   │ A2A               │ Agent 间协调与协作     │
└─────────────────┴───────────────────┴───────────────────────┘
```

### AG-UI vs MCP vs A2A

| 协议 | 全称 | 起源 | 核心职责 |
|:---|:---|:---|:---|
| **AG-UI** | Agent-User Interaction Protocol | CopilotKit 社区 | 连接 Agent 与用户界面应用，实现实时、多模态、交互式体验 |
| **MCP** | Model Context Protocol | Anthropic | 让 Agent 安全连接外部系统——工具、工作流和数据源 |
| **A2A** | Agent to Agent | Google | 定义 Agent 如何在分布式系统中协调和共享工作 |

> **重要提示**：这三个协议是互补的，一个 Agent 通常会同时使用全部三个协议。

### AG-UI 与 MCP/A2A 的握手

AG-UI 贡献者最近添加了握手机制，允许 AG-UI 通过 MCP 和 A2A 协议"代理"Agent，使 AG-UI 客户端应用和库能够无缝使用支持 MCP 和 A2A 的 Agent。

---

## AG-UI 与 A2UI 的区别

> **容易混淆**：尽管名称相似，AG-UI 和 A2UI 是完全不同的东西，但可以很好地配合使用。

| | AG-UI | A2UI |
|:---|:---|:---|
| **定位** | Agent ↔ 用户交互协议 | 生成式 UI 规范 |
| **作用** | 连接 Agentic 前端与任意 Agentic 后端 | 允许 Agent 传递 UI 组件/小部件 |
| **关系** | 传输层/协议层 | 内容/渲染层 |

---

## 为什么需要 AG-UI？

### 传统方式的问题

在没有 AG-UI 之前，开发者需要为每个 Agent 框架构建自定义 API：

```
前端应用 ──自定义API──> LangGraph Agent
前端应用 ──自定义API──> CrewAI Agent  
前端应用 ──自定义API──> 自定义 Agent
```

每次集成都需要：
- 重新设计通信协议
- 处理不同的事件格式
- 实现各自的流式传输逻辑
- 维护多套集成代码

### AG-UI 的解决方案

```
前端应用 ──AG-UI──> LangGraph Agent
前端应用 ──AG-UI──> CrewAI Agent  
前端应用 ──AG-UI──> 任意 Agent
```

**一次实现，处处可用**。AG-UI 就像给 Agent 添加了一个通用翻译器。

---

## 支持的 Agent 框架

AG-UI 已与以下主流框架集成：

- **LangGraph** — LangChain 的图形化 Agent 框架
- **CrewAI** — 多 Agent 协作框架
- **Mastra** — TypeScript Agent 框架
- **Agno** — 高性能 Agent 框架
- **Vercel AI SDK** — Vercel 的 AI 工具包
- **自定义 Agent** — 通过服务端实现或中间件适配

---

## 下一步

- [核心架构](../02-核心架构/README.md) — 深入了解 AG-UI 的架构设计
- [事件系统](../03-事件系统/README.md) — 学习 AG-UI 的通信基础
- [快速开始](../07-快速开始/README.md) — 立即开始构建

---

*参考资料：[AG-UI 官方文档](https://docs.ag-ui.com/introduction.md) | [Agentic Protocols](https://docs.ag-ui.com/agentic-protocols.md)*
