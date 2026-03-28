# 第一章：LangChain 简介与核心概念

## 1.1 LangChain 概述

### 1.1.1 什么是 LangChain？

LangChain 是一个**框架**，旨在帮助开发者更便捷地构建基于大型语言模型（LLM）的应用程序。它由 [Harrison Chase](https://github.com/hwchase17) 于 2023 年创建，至今已成为 LLM 应用开发领域最受欢迎的框架之一。

LangChain 的核心理念是：
> **将 LLM 与外部数据源、计算工具进行连接，构建智能化的应用工作流**

```python
# LangChain 简单示例
from langchain_openai import ChatOpenAI
from langchain.schema import HumanMessage

# 初始化聊天模型
llm = ChatOpenAI(model="gpt-4", temperature=0.7)

# 发起对话
response = llm.invoke([HumanMessage(content="解释一下什么是LangChain")])
print(response.content)
```

### 1.1.2 为什么选择 LangChain？

| 特性 | 描述 |
|:---|:---|
| **模块化设计** | 各组件解耦，可独立使用或组合 |
| **丰富的集成** | 预置 100+ 种模型、数据源、工具的集成 |
| **抽象层级** | 简化 LLM 应用开发的复杂度 |
| **生产就绪** | 支持流式输出、回调、异步等生产特性 |
| **活跃社区** | 庞大的用户群体和丰富的学习资源 |

### 1.1.3 LangChain 能做什么？

```
┌─────────────────────────────────────────────────────────────────┐
│                        LangChain 应用场景                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │  聊天机器人   │  │   RAG 系统   │  │   自主代理   │             │
│  │  Chatbot    │  │  Retrieval  │  │   Agent     │             │
│  │             │  │  Augmented   │  │             │             │
│  │             │  │  Generation │  │             │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │  文档问答    │  │  代码生成    │  │  数据分析    │             │
│  │  QA System  │  │ Code Gen    │  │  Analysis  │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │  内容生成    │  │  任务自动化  │  │  多模态应用  │             │
│  │Content Gen │  │ Automation  │  │Multi-modal │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 1.2 核心组件架构

### 1.2.1 组件总览

LangChain 的核心组件可以分为以下几个层次：

```
┌─────────────────────────────────────────────────────────────────────┐
│                        LangChain 架构                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                      Application Layer                       │  │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐          │  │
│   │  │ Agents  │  │  Chains │  │ Memory │  │ Tools  │          │  │
│   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘          │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                    │                                 │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                      Component Layer                         │  │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐          │  │
│   │  │Prompts │  │Indexes  │  │  LLMs   │  │Output  │          │  │
│   │  │        │  │        │  │        │  │Parsers │          │  │
│   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘          │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                    │                                 │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                    Integration Layer                        │  │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐          │  │
│   │  │OpenAI   │  │  AWS    │  │Google   │  │  Meta  │          │  │
│   │  │Anthropic│  │ Azure   │  │  etc.   │  │        │          │  │
│   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘          │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2.2 核心组件详解

#### 1. Model I/O - 模型交互

负责与各种 LLM 的交互，包括：
- **LLM/ChatModel**: 不同类型的语言模型
- **Prompt**: 提示词模板
- **Output Parser**: 输出解析

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import CommaSeparatedListOutputParser

# 模型
llm = ChatOpenAI(model="gpt-4")

# 提示词模板
prompt = PromptTemplate.from_template(
    "列出{topic}的{count}个优点，用逗号分隔"
)

# 输出解析器
parser = CommaSeparatedListOutputParser()

# 构建链
chain = prompt | llm | parser

# 执行
result = chain.invoke({"topic": "学习编程", "count": "5"})
print(result)  # ['提高逻辑思维能力', '增加就业机会', ...]
```

#### 2. Data Connection - 数据连接

用于连接外部数据：
- **Document Loaders**: 文档加载器
- **Text Splitters**: 文本分割器
- **Vector Stores**: 向量存储
- **Retrievers**: 检索器

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings

# 加载 PDF
loader = PyPDFLoader("document.pdf")
docs = loader.load()

# 文本分割
splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
splits = splitter.split_documents(docs)

# 向量存储
vectorstore = Chroma.from_documents(
    documents=splits,
    embedding=OpenAIEmbeddings()
)

# 检索
retriever = vectorstore.as_retriever()
docs = retriever.invoke("查找相关内容")
```

#### 3. Chains - 链式调用

将多个组件串联成工作流：
- **LLMChain**: 基础链
- **SequentialChain**: 顺序链
- **RouterChain**: 路由链

```python
from langchain.chains import LLMChain, SequentialChain
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate

llm = ChatOpenAI(model="gpt-4")

# 第一个链：生成标题
title_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate.from_template("为以下文章生成一个标题：{content}"),
    output_key="title"
)

# 第二个链：生成摘要
summary_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate.from_template("为以下内容生成摘要：{content}"),
    output_key="summary"
)

# 组合成顺序链
overall_chain = SequentialChain(
    chains=[title_chain, summary_chain],
    input_variables=["content"],
    output_variables=["title", "summary"]
)

result = overall_chain.invoke({"content": "LangChain是一个强大的框架..."})
```

#### 4. Memory - 记忆组件

为应用添加持久化对话上下文：
- **ConversationBufferMemory**: 缓冲记忆
- **ConversationSummaryMemory**: 摘要记忆
- **EntityMemory**: 实体记忆

```python
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")

# 创建带记忆的对话链
memory = ConversationBufferMemory(return_messages=True)
conversation = ConversationChain(llm=llm, memory=memory)

# 对话
conversation.invoke("我叫张三")
conversation.invoke("我喜欢的颜色是蓝色")
print(memory.buffer)  # 包含完整的对话历史
```

#### 5. Agents - 代理

让 LLM 自主决策和执行操作：
- **Agent Types**: 代理类型
- **Tools**: 工具定义
- **Tool Calling**: 工具调用

```python
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain import hub

# 定义工具
@tool
def calculator(expression: str) -> str:
    """执行数学计算"""
    return str(eval(expression))

@tool
def search(query: str) -> str:
    """搜索信息"""
    return f"搜索结果：{query}的相关信息"

llm = ChatOpenAI(model="gpt-4", temperature=0)

# 获取提示词
prompt = hub.pull("hwchase17/openai-functions-agent")

# 创建代理
agent = create_openai_functions_agent(llm, [calculator, search], prompt)
agent_executor = AgentExecutor(agent=agent, tools=[calculator, search], verbose=True)

# 执行
result = agent_executor.invoke({"input": "计算 123 * 456 然后搜索结果"})
```

#### 6. Callbacks - 回调机制

用于监控和记录执行过程：
- **LoggingCallbackHandler**: 日志记录
- **TracingCallbackHandler**: 链路追踪

```python
from langchain.callbacks import ConsoleCallbackHandler
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", callbacks=[ConsoleCallbackHandler()])
response = llm.invoke("你好")
```

## 1.3 安装与环境配置

### 1.3.1 安装 LangChain

```bash
# 基础安装
pip install langchain

# 完整安装
pip install langchain[all]

# 常用依赖
pip install langchain-openai      # OpenAI 模型
pip install langchain-anthropic   # Anthropic 模型
pip install langchain-community   # 社区集成
```

### 1.3.2 环境变量配置

```bash
# .env 文件
OPENAI_API_KEY=sk-xxxxx
ANTHROPIC_API_KEY=sk-ant-xxxxx
AZURE_OPENAI_API_KEY=xxxxx
AZURE_OPENAI_ENDPOINT=https://xxxxx.openai.azure.com/

# 在代码中加载
from dotenv import load_dotenv
load_dotenv()
```

### 1.3.3 版本选择

| 版本 | 特点 | 适用场景 |
|:---:|:---|:---|
| LangChain v0.1.x | 稳定版本 | 生产环境 |
| LangChain v0.2.x | 新特性 | 开发新功能 |
| LangChain v0.3.x | 全面升级 | 新项目推荐 |

```python
# 检查版本
import langchain
print(langchain.__version__)  # 0.3.x
```

## 1.4 设计哲学

### 1.4.1 组合优于继承

LangChain 采用**组合式设计**，通过管道操作符 `|` 将各个组件串联：

```python
# 组合式设计示例
chain = prompt_template | llm | output_parser
#           ↑          ↑      ↑
#         输入      处理      输出
```

### 1.4.2 接口抽象

LangChain 为每个组件提供抽象接口，便于替换底层实现：

```python
# 抽象接口
from langchain.schema import BaseRetriever

# 可以替换不同的向量存储实现
class MyVectorStore(BaseRetriever):
    # 实现抽象方法
    ...

# 向上转型为 Retriever 使用
retriever = MyVectorStore(...)
docs = retriever.invoke("query")
```

### 1.4.3 流式处理

支持实时流式输出，提升用户体验：

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", stream=True)

for chunk in llm.stream("写一首诗"):
    print(chunk.content, end="", flush=True)
```

## 1.5 与 LangGraph 的关系

```
┌─────────────────────────────────────────────────────────────────────┐
│                     LangChain vs LangGraph                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────┐        ┌─────────────────────┐           │
│   │     LangChain       │        │      LangGraph      │           │
│   ├─────────────────────┤        ├─────────────────────┤           │
│   │ • 线性工作流         │        │ • 循环工作流         │           │
│   │ • 顺序执行           │        │ • 条件分支           │           │
│   │ • 简单场景           │        │ • 复杂状态机         │           │
│   │ • Chain 接口        │        │ • StateGraph 接口    │           │
│   └─────────────────────┘        └─────────────────────┘           │
│              │                             │                        │
│              └──────────┬──────────────────┘                        │
│                         ↓                                           │
│              ┌─────────────────────┐                                 │
│              │  LangChain v0.2+    │                                 │
│              │  内置 LangGraph     │                                 │
│              └─────────────────────┘                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 1.6 总结

本章介绍了 LangChain 的基本概念、核心组件和设计哲学。LangChain 通过模块化的组件设计，让开发者能够快速构建基于 LLM 的应用程序。

**核心要点**：
1. LangChain 是 LLM 应用开发框架
2. 核心组件：Model I/O、Data Connection、Chains、Memory、Agents、Callbacks
3. 组合式设计，通过 `|` 操作符串联组件
4. 支持流式输出、异步处理等生产特性
5. LangGraph 是 LangChain 的进阶版本，支持更复杂的工作流

---

*下一章我们将深入学习 Model I/O - 模型交互的详细用法。*
