# LangChain 与 LangGraph 技术文档

> 本文档档系统性地介绍 LangChain 与 LangGraph 的核心概念、组件、进阶特性及实战应用。

## 文档结构

### 第一部分：LangChain 基础

| 章节 | 标题 | 描述 |
|:---:|:---|:---|
| 01 | [简介与核心概念](./01-LangChain基础/01-简介与核心概念/README.md) | LangChain 概述、设计哲学、核心组件 |
| 02 | [Model IO - 模型交互](./01-LangChain基础/02-Model-IO-模型交互/README.md) | LLM 调用、聊天模型、模型参数配置 |
| 03 | [Prompts - 提示词工程](./01-LangChain基础/03-Prompts-提示词工程/README.md) | PromptTemplate、FewShotPromptTemplate、聊天提示 |
| 04 | [Chains - 链式调用](./01-LangChain基础/04-Chains-链式调用/README.md) | LLMChain、SequentialChain、RouterChain |
| 05 | [Memory - 记忆组件](./01-LangChain基础/05-Memory-记忆组件/README.md) | 缓冲记忆、摘要记忆、会话记忆 |
| 06 | [Tools - 工具调用](./01-LangChain基础/06-Tools-工具调用/README.md) | 工具定义、工具绑定、ReAct 模式 |
| 07 | [Indexes - 索引与检索](./01-LangChain基础/07-Indexes-索引与检索/README.md) | 文档加载器、文本分割器、向量存储 |
| 08 | [Agents - 代理](./01-LangChain基础/08-Agents-代理/README.md) | Agent 类型、Agent 执行器、Tool 使用 |
| 09 | [Callbacks - 回调机制](./01-LangChain基础/09-Callbacks-回调机制/README.md) | 事件回调、日志记录、自定义回调 |

### 第二部分：LangGraph 进阶

| 章节 | 标题 | 描述 |
|:---:|:---|:---|
| 01 | [简介与核心概念](./02-LangGraph进阶/01-简介与核心概念/README.md) | LangGraph 概述、与 LangChain 的关系 |
| 02 | [状态机与工作流](./02-LangGraph进阶/02-状态机与工作流/README.md) | StateGraph、节点状态、边定义 |
| 03 | [节点与边的定义](./02-LangGraph进阶/03-节点与边的定义/README.md) | 节点函数、条件边、并行执行 |
| 04 | [条件路由与分支](./02-LangGraph进阶/04-条件路由与分支/README.md) | 条件函数、分支逻辑、动态路由 |
| 05 | [持久化与检查点](./02-LangGraph进阶/05-持久化与检查点/README.md) | Checkpoint、内存持久化、外部存储 |
| 06 | [人机交互](./02-LangGraph进阶/06-人机交互/README.md) | 中断机制、用户确认、动态输入 |
| 07 | [多代理系统](./02-LangGraph进阶/07-多代理系统/README.md) | 多代理协作、代理间通信、任务分发 |
| 08 | [错误处理与恢复](./02-LangGraph进阶/08-错误处理与恢复/README.md) | 异常捕获、重试机制、故障恢复 |

### 第三部分：实战案例

| 章节 | 标题 | 描述 |
|:---:|:---|:---|
| 01 | [RAG 系统实战](./03-实战案例/01-RAG系统实战/README.md) | 完整 RAG 流程、向量检索、答案生成 |
| 02 | [对话代理实战](./03-实战案例/02-对话代理实战/README.md) | 会话型 Agent、上下文管理、多轮对话 |
| 03 | [自主研究 Agent](./03-实战案例/03-自主研究Agent/README.md) | 网络搜索、信息整合、报告生成 |
| 04 | [多模态应用](./03-实战案例/04-多模态应用/README.md) | 图像理解、跨模态检索、视频分析 |

### 第四部分：最佳实践

| 章节 | 标题 | 描述 |
|:---:|:---|:---|
| 01 | [性能优化](./04-最佳实践/01-性能优化/README.md) | 缓存策略、并发处理、流式输出 |
| 02 | [安全与隐私](./04-最佳实践/02-安全与隐私/README.md) | 数据脱敏、输入验证、审计日志 |
| 03 | [测试与调试](./04-最佳实践/03-测试与调试/README.md) | 单元测试、集成测试、调试技巧 |
| 04 | [部署与运维](./04-最佳实践/04-部署与运维/README.md) | Docker 部署、监控告警、扩缩容 |

## 学习路径建议

```
┌─────────────────────────────────────────────────────────────────────┐
│                        LangChain & LangGraph                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  第一阶段：LangChain 基础 (1-2周)                                     │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │
│  │ 核心概念 │→ │ Model IO │→ │ Prompts │→ │  Chains │→ │ Memory  │   │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │
│                                                                      │
│  第二阶段：LangChain 进阶 (1-2周)                                     │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐                │
│  │  Tools   │→ │ Indexes │→ │ Agents  │→ │Callbacks│                │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘                │
│                                                                      │
│  第三阶段：LangGraph (1-2周)                                          │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │
│  │ 核心概念 │→ │状态机/流│→ │ 条件路由│→ │ 持久化  │→ │ 多代理  │   │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │
│                                                                      │
│  第四阶段：实战与最佳实践 (2-3周)                                       │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐                │
│  │ RAG系统 │→ │ 对话代理 │→ │ 研究Agent│→ │ 性能优化│                │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 版本信息

- **LangChain**: v0.3.x
- **LangGraph**: v0.2.x
- **Python**: 3.10+

## 扩展阅读

- [LangChain 官方文档](https://python.langchain.com/)
- [LangGraph 官方文档](https://langchain-ai.github.io/langgraph/)
- [LangChain GitHub](https://github.com/langchain-ai/langchain)
- [LangGraph GitHub](https://github.com/langchain-ai/langgraph)

---

*本文档档将持续更新，如有疑问或建议，欢迎提交 Issue。*
