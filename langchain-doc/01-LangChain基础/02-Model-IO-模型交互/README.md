# 第二章：Model I/O - 模型交互

## 2.1 模型类型概述

LangChain 支持多种类型的语言模型，主要分为两大类：

```
┌─────────────────────────────────────────────────────────────────────┐
│                        LangChain 模型类型                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                     LLM (文本补全模型)                        │  │
│   │  • 输入：文本字符串                                            │  │
│   │  • 输出：文本字符串                                            │  │
│   │  • 示例：GPT-3, text-davinci-003                              │  │
│   │  • 使用场景：文本生成、补全                                     │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                  Chat Model (聊天模型)                        │  │
│   │  • 输入：消息列表 (HumanMessage, AIMessage, SystemMessage)    │  │
│   │  • 输出：消息对象                                              │  │
│   │  • 示例：GPT-4, Claude, Gemini                                │  │
│   │  • 使用场景：对话、多轮交互                                     │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 2.2 LLM (文本补全模型)

### 2.2.1 基础使用

```python
from langchain_openai import OpenAI

# 初始化 LLM
llm = OpenAI(
    model="gpt-3.5-turbo-instruct",
    temperature=0.7,
    max_tokens=256
)

# 调用
response = llm.invoke("解释什么是人工智能")
print(response)
```

### 2.2.2 批量调用

```python
# 批量调用
inputs = [
    "什么是机器学习？",
    "什么是深度学习？",
    "什么是神经网络？"
]

responses = llm.batch(inputs)
for i, response in enumerate(responses):
    print(f"问题 {i+1}: {inputs[i]}")
    print(f"回答: {response}\n")
```

### 2.2.3 流式输出

```python
# 流式输出
for chunk in llm.stream("写一首关于春天的诗"):
    print(chunk, end="", flush=True)
```

## 2.3 Chat Model (聊天模型)

### 2.3.1 消息类型

```python
from langchain.schema import (
    HumanMessage,      # 用户消息
    AIMessage,         # AI 回复
    SystemMessage,     # 系统提示
    FunctionMessage,   # 函数调用结果
    ToolMessage        # 工具调用结果
)

# 系统消息：设置 AI 的角色和行为
system_msg = SystemMessage(content="你是一位专业的 Python 开发者")

# 用户消息：用户的输入
human_msg = HumanMessage(content="如何优化 Python 代码性能？")

# AI 消息：模型的回复
ai_msg = AIMessage(content="以下是优化 Python 代码性能的方法...")
```

### 2.3.2 基础对话

```python
from langchain_openai import ChatOpenAI

# 初始化聊天模型
chat = ChatOpenAI(
    model="gpt-4",
    temperature=0.7,
    max_tokens=1024
)

# 单轮对话
messages = [
    SystemMessage(content="你是一位专业的数据科学家"),
    HumanMessage(content="解释什么是过拟合")
]

response = chat.invoke(messages)
print(response.content)
```

### 2.3.3 多轮对话

```python
# 多轮对话
messages = [
    SystemMessage(content="你是一位友好的助手"),
    HumanMessage(content="我叫张三"),
    AIMessage(content="你好张三，很高兴认识你！有什么我可以帮助你的吗？"),
    HumanMessage(content="我喜欢什么颜色？")
]

# 注意：这里 AI 无法知道用户喜欢什么颜色
# 需要使用 Memory 组件来保存上下文
response = chat.invoke(messages)
print(response.content)
```

### 2.3.4 批量处理

```python
# 批量处理多组对话
batch_messages = [
    [SystemMessage(content="你是翻译助手"), HumanMessage(content="Hello")],
    [SystemMessage(content="你是翻译助手"), HumanMessage(content="World")],
    [SystemMessage(content="你是翻译助手"), HumanMessage(content="Python")]
]

responses = chat.batch(batch_messages)
for response in responses:
    print(response.content)
```

## 2.4 模型参数详解

### 2.4.1 核心参数

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    # 模型名称
    model="gpt-4",
    
    # 温度参数：控制输出的随机性 (0-2)
    # 0: 确定性输出
    # 1: 平衡
    # 2: 高度随机
    temperature=0.7,
    
    # 最大生成 token 数
    max_tokens=2048,
    
    # Top-p 采样：控制输出多样性
    top_p=1.0,
    
    # 频率惩罚：降低重复词的概率 (-2 到 2)
    frequency_penalty=0.0,
    
    # 存在惩罚：鼓励使用新词 (-2 到 2)
    presence_penalty=0.0,
    
    # 停止序列：遇到这些序列停止生成
    stop=None,
    
    # 种子：用于可重复输出
    seed=None,
    
    # 超时时间
    timeout=30,
    
    # 最大重试次数
    max_retries=2,
)
```

### 2.4.2 参数选择指南

| 场景 | Temperature | Top-p | 说明 |
|:---:|:---:|:---:|:---|
| 代码生成 | 0.0-0.3 | 0.9 | 确定性输出，减少错误 |
| 数据分析 | 0.0-0.2 | 0.9 | 精确回答，避免幻觉 |
| 创意写作 | 0.7-1.0 | 1.0 | 增加多样性 |
| 对话助手 | 0.5-0.7 | 0.95 | 平衡自然度和准确性 |
| 翻译任务 | 0.0-0.2 | 0.9 | 准确翻译 |
| 头脑风暴 | 0.9-1.2 | 1.0 | 最大化创意 |

## 2.5 多模型支持

### 2.5.1 OpenAI 系列

```python
from langchain_openai import ChatOpenAI, OpenAI

# GPT-4
chat_gpt4 = ChatOpenAI(model="gpt-4")

# GPT-4 Turbo
chat_gpt4_turbo = ChatOpenAI(model="gpt-4-turbo-preview")

# GPT-3.5 Turbo
chat_gpt35 = ChatOpenAI(model="gpt-3.5-turbo")

# GPT-3.5 Instruct (文本补全)
llm_gpt35 = OpenAI(model="gpt-3.5-turbo-instruct")
```

### 2.5.2 Anthropic Claude

```python
from langchain_anthropic import ChatAnthropic

# Claude 3 Opus (最强)
claude_opus = ChatAnthropic(model="claude-3-opus-20240229")

# Claude 3 Sonnet (平衡)
claude_sonnet = ChatAnthropic(model="claude-3-sonnet-20240229")

# Claude 3 Haiku (最快)
claude_haiku = ChatAnthropic(model="claude-3-haiku-20240307")
```

### 2.5.3 Google Gemini

```python
from langchain_google_genai import ChatGoogleGenerativeAI

# Gemini Pro
gemini = ChatGoogleGenerativeAI(model="gemini-pro")

# Gemini Pro Vision (多模态)
gemini_vision = ChatGoogleGenerativeAI(model="gemini-pro-vision")
```

### 2.5.4 本地模型 (Ollama)

```python
from langchain_community.chat_models import ChatOllama

# 本地 Llama 模型
llama = ChatOllama(model="llama2")

# 本地 Mistral 模型
mistral = ChatOllama(model="mistral")

# 本地 Code Llama 模型
code_llama = ChatOllama(model="codellama")
```

### 2.5.5 Azure OpenAI

```python
from langchain_openai import AzureChatOpenAI

azure_chat = AzureChatOpenAI(
    azure_deployment="gpt-4",
    azure_endpoint="https://your-resource.openai.azure.com/",
    api_version="2024-02-01"
)
```

## 2.6 输出解析器

### 2.6.1 字符串输出

```python
from langchain_core.output_parsers import StrOutputParser

parser = StrOutputParser()

# 将模型输出转换为字符串
response = chat.invoke([HumanMessage(content="你好")])
text = parser.invoke(response)
print(text)  # 纯文本输出
```

### 2.6.2 JSON 输出

```python
from langchain_core.output_parsers import JsonOutputParser
from langchain_core.pydantic_v1 import BaseModel, Field

# 定义输出结构
class Person(BaseModel):
    name: str = Field(description="人物姓名")
    age: int = Field(description="年龄")
    occupation: str = Field(description="职业")

parser = JsonOutputParser(pydantic_object=Person)

# 在提示词中说明输出格式
prompt = f"""
从以下描述中提取人物信息：
张三，35岁，是一名软件工程师。

{parser.get_format_instructions()}
"""

response = chat.invoke([HumanMessage(content=prompt)])
person = parser.invoke(response)
print(person)  # {'name': '张三', 'age': 35, 'occupation': '软件工程师'}
```

### 2.6.3 列表输出

```python
from langchain_core.output_parsers import CommaSeparatedListOutputParser

parser = CommaSeparatedListOutputParser()

prompt = """
列出5种编程语言，用逗号分隔。
"""

response = chat.invoke([HumanMessage(content=prompt)])
languages = parser.invoke(response)
print(languages)  # ['Python', 'JavaScript', 'Java', 'C++', 'Go']
```

### 2.6.4 结构化输出

```python
from langchain.output_parsers import StructuredOutputParser, ResponseSchema

# 定义响应结构
response_schemas = [
    ResponseSchema(name="answer", description="问题的答案"),
    ResponseSchema(name="confidence", description="置信度，0-1之间"),
    ResponseSchema(name="sources", description="信息来源列表")
]

parser = StructuredOutputParser.from_response_schemas(response_schemas)

prompt = f"""
回答以下问题并提供结构化输出：
什么是机器学习？

{parser.get_format_instructions()}
"""

response = chat.invoke([HumanMessage(content=prompt)])
result = parser.parse(response.content)
print(result)
```

## 2.7 高级特性

### 2.7.1 函数调用 (Function Calling)

```python
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain_core.messages import HumanMessage

# 定义工具
@tool
def get_weather(location: str) -> str:
    """获取指定城市的天气"""
    return f"{location}今天晴天，25°C"

@tool
def calculate(expression: str) -> str:
    """执行数学计算"""
    return str(eval(expression))

# 绑定工具
chat = ChatOpenAI(model="gpt-4")
chat_with_tools = chat.bind_tools([get_weather, calculate])

# 调用
messages = [HumanMessage(content="北京今天天气怎么样？")]
response = chat_with_tools.invoke(messages)

# 检查是否调用工具
if response.tool_calls:
    for tool_call in response.tool_calls:
        print(f"调用工具: {tool_call['name']}")
        print(f"参数: {tool_call['args']}")
```

### 2.7.2 异步调用

```python
import asyncio
from langchain_openai import ChatOpenAI

chat = ChatOpenAI(model="gpt-4")

async def async_chat():
    # 异步调用
    response = await chat.ainvoke([HumanMessage(content="你好")])
    print(response.content)

async def async_batch():
    # 异步批量调用
    messages_list = [
        [HumanMessage(content="问题1")],
        [HumanMessage(content="问题2")],
        [HumanMessage(content="问题3")]
    ]
    responses = await chat.abatch(messages_list)
    for r in responses:
        print(r.content)

# 运行
asyncio.run(async_chat())
```

### 2.7.3 流式输出

```python
from langchain_openai import ChatOpenAI
from langchain.callbacks import StreamingStdOutCallbackHandler

# 方式1：使用回调
chat = ChatOpenAI(
    model="gpt-4",
    streaming=True,
    callbacks=[StreamingStdOutCallbackHandler()]
)

response = chat.invoke([HumanMessage(content="写一个故事")])

# 方式2：手动处理流
chat = ChatOpenAI(model="gpt-4")

for chunk in chat.stream([HumanMessage(content="写一个故事")]):
    print(chunk.content, end="", flush=True)
```

### 2.7.4 重试与错误处理

```python
from langchain_openai import ChatOpenAI
from tenacity import retry, stop_after_attempt, wait_exponential

# 配置重试
chat = ChatOpenAI(
    model="gpt-4",
    max_retries=3,
    timeout=30
)

# 或使用 tenacity 自定义重试逻辑
@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10)
)
def robust_chat(messages):
    return chat.invoke(messages)

try:
    response = robust_chat([HumanMessage(content="你好")])
except Exception as e:
    print(f"调用失败: {e}")
```

## 2.8 完整示例

### 2.8.1 智能问答系统

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 创建问答链
def create_qa_chain():
    # 提示词模板
    prompt = ChatPromptTemplate.from_messages([
        ("system", "你是一位专业的{domain}专家，请用简洁清晰的语言回答问题。"),
        ("human", "{question}")
    ])
    
    # 模型
    model = ChatOpenAI(model="gpt-4", temperature=0.3)
    
    # 输出解析
    parser = StrOutputParser()
    
    # 组合链
    chain = prompt | model | parser
    
    return chain

# 使用
qa_chain = create_qa_chain()

# 技术问答
response = qa_chain.invoke({
    "domain": "Python",
    "question": "解释装饰器的工作原理"
})
print(response)

# 医学问答
response = qa_chain.invoke({
    "domain": "医学",
    "question": "什么是高血压？"
})
print(response)
```

### 2.8.2 代码生成器

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 代码生成链
code_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一位资深程序员，请根据需求生成高质量的代码。
要求：
1. 代码要有清晰的注释
2. 包含错误处理
3. 使用最佳实践
4. 提供使用示例"""),
    ("human", """编程语言：{language}
需求：{requirement}""")
])

code_model = ChatOpenAI(model="gpt-4", temperature=0.2)
code_chain = code_prompt | code_model | StrOutputParser()

# 生成代码
result = code_chain.invoke({
    "language": "Python",
    "requirement": "实现一个带缓存的斐波那契数列计算函数"
})
print(result)
```

## 2.9 总结

本章详细介绍了 LangChain 中 Model I/O 的各种用法：

1. **模型类型**：LLM (文本补全) vs Chat Model (聊天模型)
2. **消息类型**：SystemMessage、HumanMessage、AIMessage 等
3. **模型参数**：temperature、max_tokens、top_p 等
4. **多模型支持**：OpenAI、Claude、Gemini、Ollama 等
5. **输出解析**：字符串、JSON、列表、结构化输出
6. **高级特性**：函数调用、异步调用、流式输出

---

*下一章我们将学习 Prompts - 提示词工程，这是与 LLM 交互的核心技能。*
