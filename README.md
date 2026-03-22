# AI Learn Doc

> AI 学习文档仓库，包含 LangChain、LangGraph、AG-UI 等框架的技术文档。

## 📚 文档目录

### [LangChain 与 LangGraph 技术文档](./langchain-doc/README.md)

系统性地介绍 LangChain 与 LangGraph 的核心概念、组件、进阶特性及实战应用。

**文档结构：**

| 部分 | 章节数 | 描述 |
|:---|:---:|:---|
| [LangChain 基础](./langchain-doc/01-LangChain基础/) | 9 | 核心概念、Model I/O、Prompts、Chains、Memory、Tools、Indexes、Agents、Callbacks |
| [LangGraph 进阶](./langchain-doc/02-LangGraph进阶/) | 8 | 状态机、工作流、条件路由、持久化、人机交互、多代理系统 |
| [实战案例](./langchain-doc/03-实战案例/) | 4 | RAG 系统、对话代理、自主研究 Agent、多模态应用 |
| [最佳实践](./langchain-doc/04-最佳实践/) | 4 | 性能优化、安全隐私、测试调试、部署运维 |

**总计：25 个章节，涵盖从入门到实战的完整学习路径**

---

### [AG-UI 技术文档](./ag-ui-doc/README.md)

**AG-UI（Agent-User Interaction Protocol）** — 将 AI Agent 接入前端应用的开放协议

**文档结构：**

| 章节 | 内容 | 说明 |
|:---|:---|:---|
| [概述与协议介绍](./ag-ui-doc/01-概述与协议介绍/) | AG-UI 是什么、设计目标、与 MCP/A2A 的关系 | 了解 AG-UI 在 AI 协议栈中的定位 |
| [核心架构](./ag-ui-doc/02-核心架构/) | 架构设计原则、组件说明、客户端-服务端模型 | 深入理解整体设计 |
| [事件系统](./ag-ui-doc/03-事件系统/) | 16 种标准事件类型、事件流、生命周期 | 掌握通信基础 |
| [状态管理](./ag-ui-doc/04-状态管理/) | 共享状态、STATE_SNAPSHOT、STATE_DELTA、JSON Patch | 双向状态同步 |
| [工具与人机协作](./ag-ui-doc/05-工具与人机协作/) | 工具定义、工具调用生命周期、Human-in-the-Loop | Human-in-the-Loop |
| [中间件](./ag-ui-doc/06-中间件/) | 中间件机制、函数式/类式中间件、内置中间件 | 扩展与定制 |
| [快速开始](./ag-ui-doc/07-快速开始/) | 服务端实现、中间件集成、客户端构建 | 动手实践 |
| [SDK 参考](./ag-ui-doc/08-SDK参考/) | TypeScript SDK、Python SDK、核心类型 | 深入 API |
| [最佳实践与实战](./ag-ui-doc/09-最佳实践与实战/) | 设计模式、调试技巧、框架集成案例 | 生产级应用 |

**总计：9 个章节，涵盖从入门到实战的完整学习路径**

---

## 📖 快速导航

### LangChain 与 LangGraph
- [01. 简介与核心概念](./langchain-doc/01-LangChain基础/01-简介与核心概念/README.md)
- [02. Model I/O - 模型交互](./langchain-doc/01-LangChain基础/02-Model-IO-模型交互/README.md)
- [03. Prompts - 提示词工程](./langchain-doc/01-LangChain基础/03-Prompts-提示词工程/README.md)

### AG-UI 入门
- [01. 概述与协议介绍](./ag-ui-doc/01-概述与协议介绍/README.md)
- [02. 核心架构](./ag-ui-doc/02-核心架构/README.md)
- [03. 事件系统](./ag-ui-doc/03-事件系统/README.md)
- [07. 快速开始](./ag-ui-doc/07-快速开始/README.md)

---

## 🎯 学习路径建议

### 阶段一：LangChain 基础 (1-2周)
```
├── 简介与核心概念
├── Model I/O - 模型交互
├── Prompts - 提示词工程
├── Chains - 链式调用
├── Memory - 记忆组件
├── Tools - 工具调用
├── Indexes - 索引与检索
├── Agents - 代理
└── Callbacks - 回调机制
```

### 阶段二：LangGraph 进阶 (1-2周)
```
├── 简介与核心概念
├── 状态机与工作流
├── 节点与边的定义
├── 条件路由与分支
├── 持久化与检查点
├── 人机交互
├── 多代理系统
└── 错误处理与恢复
```

### 阶段三：AG-UI 前端集成 (1周)
```
├── 概述与协议介绍
├── 核心架构
├── 事件系统
├── 状态管理
├── 工具与人机协作
├── 中间件
├── 快速开始
└── SDK 参考
```

### 阶段四：实战与最佳实践 (2-3周)
```
├── RAG 系统实战
├── 对话代理实战
├── 自主研究 Agent
├── 多模态应用
├── AG-UI 框架集成
├── 性能优化
├── 安全与隐私
├── 测试与调试
└── 部署与运维
```

---

## 🌐 相关资源

### LangChain & LangGraph
- [LangChain 官方文档](https://python.langchain.com/)
- [LangGraph 官方文档](https://langchain-ai.github.io/langgraph/)
- [LangChain GitHub](https://github.com/langchain-ai/langchain)
- [LangGraph GitHub](https://github.com/langchain-ai/langgraph)

### AG-UI
- [AG-UI 官方文档](https://docs.ag-ui.com)
- [AG-UI GitHub 仓库](https://github.com/ag-ui-protocol/ag-ui)
- [CopilotKit（AG-UI 主要实现框架）](https://docs.copilotkit.ai)
- [AG-UI TypeScript SDK (npm)](https://www.npmjs.com/package/@ag-ui/client)
- [AG-UI Python SDK (PyPI)](https://pypi.org/project/ag-ui-protocol/)

### AI 协议生态
- [MCP (Model Context Protocol)](https://modelcontextprotocol.io/)
- [A2A (Agent to Agent)](https://google.github.io/A2A/)
- [AG-UI](https://docs.ag-ui.com)

---

## 📦 版本信息

### LangChain & LangGraph
- **LangChain**: v0.3.x
- **LangGraph**: v0.2.x
- **Python**: 3.10+

### AG-UI
- **协议版本**: AG-UI v1.x
- **TypeScript SDK**: `@ag-ui/client`, `@ag-ui/core`
- **Python SDK**: `ag-ui-protocol`

---

## 📄 许可证

MIT License

---

*本文档持续更新中，如有疑问或建议，欢迎提交 Issue。*
