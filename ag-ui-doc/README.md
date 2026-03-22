# AG-UI 技术文档

> **AG-UI（Agent-User Interaction Protocol）** — 将 AI Agent 接入前端应用的开放协议

## 📖 文档简介

AG-UI 是一个**开放、轻量、基于事件**的协议，标准化了 AI Agent 与用户界面应用之间的连接方式。它是 AI 协议生态中专注于"Agent ↔ 用户交互"层的核心标准。

本文档系统性地介绍 AG-UI 协议的核心概念、架构设计、事件系统、状态管理、工具调用、SDK 使用及最佳实践。

---

## 📚 文档目录

| 章节 | 内容 |
|:---|:---|
| [01. 概述与协议介绍](./01-概述与协议介绍/README.md) | AG-UI 是什么、设计目标、与 MCP/A2A 的关系 |
| [02. 核心架构](./02-核心架构/README.md) | 架构设计原则、组件说明、客户端-服务端模型 |
| [03. 事件系统](./03-事件系统/README.md) | 16 种标准事件类型、事件流、生命周期 |
| [04. 状态管理](./04-状态管理/README.md) | 共享状态、STATE_SNAPSHOT、STATE_DELTA、JSON Patch |
| [05. 工具与人机协作](./05-工具与人机协作/README.md) | 工具定义、工具调用生命周期、Human-in-the-Loop |
| [06. 中间件](./06-中间件/README.md) | 中间件机制、函数式/类式中间件、内置中间件 |
| [07. 快速开始](./07-快速开始/README.md) | 服务端实现、中间件集成、客户端构建 |
| [08. SDK 参考](./08-SDK参考/README.md) | TypeScript SDK、Python SDK、核心类型 |
| [09. 最佳实践与实战](./09-最佳实践与实战/README.md) | 设计模式、调试技巧、框架集成案例 |

---

## 🚀 快速导航

### 初学者路径
```
1. 概述与协议介绍 → 了解 AG-UI 是什么
2. 核心架构       → 理解整体设计
3. 事件系统       → 掌握通信基础
4. 快速开始       → 动手实践
```

### 进阶路径
```
1. 状态管理       → 双向状态同步
2. 工具与人机协作 → Human-in-the-Loop
3. 中间件         → 扩展与定制
4. SDK 参考       → 深入 API
5. 最佳实践与实战 → 生产级应用
```

---

## 🌐 相关资源

- [AG-UI 官方文档](https://docs.ag-ui.com)
- [AG-UI GitHub 仓库](https://github.com/ag-ui-protocol/ag-ui)
- [CopilotKit（AG-UI 主要实现框架）](https://docs.copilotkit.ai)
- [AG-UI TypeScript SDK](https://www.npmjs.com/package/@ag-ui/client)
- [AG-UI Python SDK](https://pypi.org/project/ag-ui-protocol/)

---

## 📦 版本信息

- **协议版本**: AG-UI v1.x
- **TypeScript SDK**: `@ag-ui/client`, `@ag-ui/core`
- **Python SDK**: `ag-ui-protocol`
- **文档更新**: 2026-03

---

*本文档基于 AG-UI 官方文档整理，持续更新中。*
