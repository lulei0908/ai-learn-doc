# 第五章：Memory - 记忆组件

## 5.1 Memory 概述

Memory 组件为 LangChain 应用提供持久化对话上下文的能力，使 AI 能够记住之前的交互内容。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Memory 组件架构                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                        Memory 类型                            │  │
│   ├─────────────────────────────────────────────────────────────┤  │
│   │  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐   │  │
│   │  │   短期记忆     │  │    长期记忆    │  │    摘要记忆    │   │  │
│   │  │ BufferMemory  │  │  VectorMemory │  │ SummaryMemory │   │  │
│   │  └───────────────┘  └───────────────┘  └───────────────┘   │  │
│   │                                                              │  │
│   │  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐   │  │
│   │  │   实体记忆    │  │   知识图谱    │  │    组合记忆    │   │  │
│   │  │ EntityMemory │  │  GraphMemory  │  │ CombinedMem  │   │  │
│   │  └───────────────┘  └───────────────┘  └───────────────┘   │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 5.2 BufferMemory

### 5.2.1 基础使用

```python
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI

# 初始化
llm = ChatOpenAI(model="gpt-4", temperature=0.7)
memory = ConversationBufferMemory(return_messages=True)

# 创建对话链
conversation = ConversationChain(
    llm=llm,
    memory=memory,
    verbose=True
)

# 第一轮对话
response = conversation.predict(input="你好，我叫张三")
print(response)

# 第二轮对话（AI 会记住你的名字）
response = conversation.predict(input="我刚才说我叫什么？")
print(response)
```

### 5.2.2 保存和加载记忆

```python
import json

# 保存记忆
memory.save_context(
    {"input": "你好"},
    {"output": "你好！有什么可以帮助你的？"}
)

# 获取记忆内容
memory_data = memory.load_memory_variables({})
print(memory_data)

# 保存到文件
with open("memory.json", "w") as f:
    json.dump(memory_data, f)

# 从文件加载
with open("memory.json", "r") as f:
    loaded_memory = json.load(f)

# 创建新记忆
new_memory = ConversationBufferMemory()
new_memory.load_memory_variables(loaded_memory)
```

### 5.2.3 自定义消息键

```python
# 自定义消息键名
memory = ConversationBufferMemory(
    human_prefix="用户",
    ai_prefix="助手",
    return_messages=True,
    output_key="response",  # 输出消息的键名
    input_key="query"       # 输入消息的键名
)
```

## 5.3 SummaryMemory

### 5.3.1 基础使用

```python
from langchain.memory import ConversationSummaryMemory
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", temperature=0.7)

# 使用摘要记忆
memory = ConversationSummaryMemory(
    llm=llm,
    return_messages=True
)

conversation = ConversationChain(
    llm=llm,
    memory=memory,
    verbose=False
)

# 多轮对话
conversation.predict(input="我叫张三，是一名软件工程师")
conversation.predict(input="我在阿里巴巴工作，主要做后端开发")
conversation.predict(input="我使用Python和Go语言")

# 查看当前摘要
print(memory.load_memory_variables({}))
```

### 5.3.2 摘要 vs 缓冲

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BufferMemory vs SummaryMemory                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   BufferMemory                    SummaryMemory                    │
│   ─────────────                   ─────────────                     │
│   • 存储完整对话                   • 动态生成摘要                     │
│   • 简单直接                       • 节省 token                      │
│   • 对话长时成本高                 • 需要额外 LLM 调用               │
│   • 适合短对话                    • 适合长对话                       │
│                                                                      │
│   消息历史：                        摘要形式：                        │
│   [                                    ┌─────────────────────┐       │
│     {"role": "user", "msg": "你好"},   │ 对话摘要：          │       │
│     {"role": "ai", "msg": "你好！"},  │ 用户介绍自己是张    │       │
│     {"role": "user", "msg": "我叫.."} │ 三，在阿里巴巴从    │       │
│   ]                                  │ 事Python后端开发。   │       │
│                                      └─────────────────────┘       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 5.4 EntityMemory

### 5.4.1 实体记忆

```python
from langchain.memory import EntityMemory
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", temperature=0.7)

# 创建实体记忆
memory = EntityMemory(
    llm=llm,
    return_messages=True
)

conversation = ConversationChain(
    llm=llm,
    memory=memory,
    verbose=False
)

# 对话并提取实体
conversation.predict(input="我和我的妻子小红住在上海")
conversation.predict(input="我们在世纪大道附近工作")
conversation.predict(input="我开的是一辆白色的特斯拉")

# 查看记住的实体
print(memory.load_memory_variables({}))
```

## 5.5 VectorStore 记忆

### 5.5.1 使用向量存储记忆

```python
from langchain.memory import VectorStoreMemory
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_community.vectorstores import Chroma

# 创建向量存储
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_texts(
    texts=["用户喜欢蓝色，住在上海"],
    embedding=embeddings,
    collection_name="user_preferences"
)

# 创建向量记忆
memory = VectorStoreMemory(
    vectorstore=vectorstore,
    return_messages=True,
    input_key="input",
    memory_key="chat_history",
    k=3  # 检索最近3条相关记忆
)

conversation = ConversationChain(
    llm=ChatOpenAI(model="gpt-4", temperature=0.7),
    memory=memory,
    verbose=True
)

# 使用记忆
response = conversation.predict(input="我喜欢什么颜色？")
print(response)  # 应该能回答"蓝色"
```

## 5.6 组合记忆

### 5.6.1 CombinedMemory

```python
from langchain.memory import ConversationBufferMemory, CombinedMemory
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", temperature=0.7)

# 组合多种记忆
buffer_memory = ConversationBufferMemory(
    return_messages=True,
    input_key="input"
)

summary_memory = ConversationSummaryMemory(
    llm=llm,
    return_messages=True,
    input_key="input"
)

# 合并记忆
memory = CombinedMemory(memories=[buffer_memory, summary_memory])

conversation = ConversationChain(
    llm=llm,
    memory=memory,
    verbose=False
)
```

## 5.7 记忆的高级用法

### 5.7.1 在链中使用记忆

```python
from langchain.chains import LLMChain
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain.memory import ConversationBufferMemory

llm = ChatOpenAI(model="gpt-4", temperature=0.7)

# 带记忆的 LLMChain
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个有帮助的助手。以下是对话历史：{history}"),
    ("human", "{input}")
])

memory = ConversationBufferMemory(
    memory_key="history",
    return_messages=True,
    input_key="input"
)

chain = LLMChain(
    llm=llm,
    prompt=prompt,
    memory=memory
)

# 执行
result = chain.predict(input="你好")
result = chain.predict(input="我正在学习Python")
result = chain.predict(input="我在学什么？")

print(result)
```

### 5.7.2 自定义记忆

```python
from langchain.memory import Memory
from langchain.schema import get_buffer_string

class CustomMemory(Memory):
    """自定义记忆实现"""
    
    def load_memory_variables(self, inputs: dict) -> dict:
        # 实现记忆加载
        return {"history": self.chat_memory.messages}
    
    def save_context(self, inputs: dict, outputs: dict) -> None:
        # 实现记忆保存
        self.chat_memory.add_user_message(inputs["input"])
        self.chat_memory.add_ai_message(outputs["output"])
    
    def clear(self) -> None:
        # 实现记忆清除
        self.chat_memory.clear()
```

### 5.7.3 记忆持久化

```python
import json
from langchain.memory import ConversationBufferMemory
from pathlib import Path

# 带持久化的记忆
class PersistentMemory(ConversationBufferMemory):
    def __init__(self, *args, **kwargs):
        self.storage_path = kwargs.pop("storage_path", "./memory.json")
        super().__init__(*args, **kwargs)
        self._load()
    
    def _load(self):
        if Path(self.storage_path).exists():
            with open(self.storage_path, "r") as f:
                data = json.load(f)
                for msg in data:
                    if msg["type"] == "human":
                        self.chat_memory.add_user_message(msg["content"])
                    else:
                        self.chat_memory.add_ai_message(msg["content"])
    
    def save_context(self, inputs: dict, outputs: dict) -> None:
        super().save_context(inputs, outputs)
        self._persist()
    
    def _persist(self):
        messages = []
        for msg in self.chat_memory.messages:
            messages.append({
                "type": msg.type,
                "content": msg.content
            })
        with open(self.storage_path, "w") as f:
            json.dump(messages, f)
```

## 5.8 完整示例

### 5.8.1 多用户记忆隔离

```python
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI

class UserSessionManager:
    """用户会话管理器"""
    
    def __init__(self):
        self.sessions = {}
        self.llm = ChatOpenAI(model="gpt-4", temperature=0.7)
    
    def get_session(self, user_id: str) -> ConversationChain:
        """获取或创建用户会话"""
        if user_id not in self.sessions:
            memory = ConversationBufferMemory(
                return_messages=True,
                memory_key=f"chat_history_{user_id}"
            )
            self.sessions[user_id] = ConversationChain(
                llm=self.llm,
                memory=memory,
                verbose=False
            )
        return self.sessions[user_id]
    
    def chat(self, user_id: str, message: str) -> str:
        """发送消息并获取回复"""
        session = self.get_session(user_id)
        return session.predict(input=message)
    
    def clear_session(self, user_id: str):
        """清除用户会话"""
        if user_id in self.sessions:
            self.sessions[user_id].memory.clear()
            del self.sessions[user_id]

# 使用
manager = UserSessionManager()

# 用户 A 对话
print(manager.chat("user_A", "我叫张三"))
print(manager.chat("user_A", "我是谁？"))

# 用户 B 对话（独立记忆）
print(manager.chat("user_B", "我叫李四"))
print(manager.chat("user_B", "我是谁？"))
```

### 5.8.2 带知识库的记忆

```python
from langchain.memory import ConversationBufferMemory
from langchain.chains import RetrievalQA
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_community.vectorstores import Chroma

# 初始化
llm = ChatOpenAI(model="gpt-4", temperature=0.7)
embeddings = OpenAIEmbeddings()

# 创建知识库向量存储
vectorstore = Chroma.from_texts(
    texts=[
        "用户喜欢喝咖啡，不喝茶",
        "用户的公司是字节跳动",
        "用户住在朝阳区"
    ],
    embedding=embeddings,
    collection_name="user_knowledge"
)

# 创建记忆
memory = ConversationBufferMemory(
    return_messages=True,
    output_key="answer",
    input_key="question"
)

# 创建问答链
qa = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vectorstore.as_retriever(),
    memory=memory
)

# 问答
qa.invoke({"query": "用户喜欢喝什么？"})
qa.invoke({"query": "用户住在哪里？"})
qa.invoke({"query": "根据之前的对话，用户在哪里工作？"})

# 查看记忆
print(memory.load_memory_variables({}))
```

### 5.8.3 长期记忆系统

```python
from langchain.memory import ConversationBufferMemory, VectorStoreMemory
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_community.vectorstores import Chroma
from datetime import datetime

class LongTermMemorySystem:
    """长期记忆系统"""
    
    def __init__(self, user_id: str):
        self.user_id = user_id
        self.llm = ChatOpenAI(model="gpt-4", temperature=0.7)
        
        # 短期记忆：当前会话
        self.short_term = ConversationBufferMemory(
            return_messages=True
        )
        
        # 长期记忆：重要信息向量存储
        self.long_term = Chroma(
            collection_name=f"memory_{user_id}",
            embedding_function=OpenAIEmbeddings()
        )
        
        # 当前对话链
        self.conversation = ConversationChain(
            llm=self.llm,
            memory=self.short_term,
            verbose=False
        )
    
    def add_memory(self, content: str, memory_type: str = "fact"):
        """添加长期记忆"""
        metadata = {
            "type": memory_type,
            "timestamp": datetime.now().isoformat(),
            "user_id": self.user_id
        }
        self.long_term.add_texts([content], [{"source": metadata}])
    
    def recall(self, query: str, k: int = 3) -> list:
        """检索相关记忆"""
        return self.long_term.similarity_search(query, k=k)
    
    def chat(self, message: str) -> str:
        """带记忆的对话"""
        # 检索相关记忆
        relevant_memories = self.recall(message)
        
        # 构建上下文
        context = "\n".join([f"- {doc.page_content}" for doc in relevant_memories])
        
        # 带记忆的提示
        if context:
            enhanced_prompt = f"""根据以下记忆回答：
记忆：{context}

当前问题：{message}

回答："""
        else:
            enhanced_prompt = message
        
        # 对话
        response = self.conversation.predict(input=enhanced_prompt)
        
        return response

# 使用
memory_system = LongTermMemorySystem("user_001")

# 添加重要记忆
memory_system.add_memory("用户对猫过敏，不能养猫")
memory_system.add_memory("用户最喜欢的电影是《盗梦空间》")

# 对话
response = memory_system.chat("我可以送只猫给他吗？")
print(response)  # 应该提到过敏
```

## 5.9 总结

本章介绍了 LangChain 中 Memory 的各种用法：

1. **BufferMemory**: 存储完整对话历史
2. **SummaryMemory**: 生成对话摘要，节省 token
3. **EntityMemory**: 记忆实体信息
4. **VectorStoreMemory**: 基于向量检索的记忆
5. **CombinedMemory**: 组合多种记忆
6. **自定义记忆**: 可扩展的记忆系统

**最佳实践**：
- 短对话使用 BufferMemory
- 长对话使用 SummaryMemory
- 需要检索时使用 VectorStoreMemory
- 多用户场景注意记忆隔离

---

*下一章我们将学习 Tools - 工具调用，让 AI 能够执行外部操作。*
