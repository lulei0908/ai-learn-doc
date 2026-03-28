# 第三章：Prompts - 提示词工程

## 3.1 提示词工程概述

提示词工程（Prompt Engineering）是设计和优化输入提示（prompt）以获得更好输出的技术。它是与 LLM 交互的核心技能。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        提示词工程核心原则                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐│
│  │   清晰性    │  │   具体性    │  │   结构化    │  │   上下文    ││
│  │  Clarity   │  │ Specificity │  │ Structure  │  │  Context   ││
│  │            │  │             │  │            │  │            ││
│  │ 表达清楚    │  │ 明确需求    │  │ 组织有序    │  │ 提供背景    ││
│  │ 避免歧义    │  │ 给出示例    │  │ 使用格式    │  │ 设定角色    ││
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘│
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 3.2 PromptTemplate 基础

### 3.2.1 字符串模板

```python
from langchain_core.prompts import PromptTemplate

# 基础模板
template = "告诉我关于{topic}的{count}个有趣事实。"
prompt = PromptTemplate.from_template(template)

# 格式化
formatted = prompt.format(topic="Python", count="5")
print(formatted)
# 输出: 告诉我关于Python的5个有趣事实。

# 或使用 invoke
result = prompt.invoke({"topic": "Python", "count": "5"})
print(result.to_string())
```

### 3.2.2 多变量模板

```python
# 多变量模板
template = """
角色：{role}
任务：{task}
要求：
{requirements}
输出格式：{format}
"""

prompt = PromptTemplate.from_template(template)

formatted = prompt.format(
    role="资深Python开发者",
    task="解释装饰器的工作原理",
    requirements="- 通俗易懂\n- 包含代码示例\n- 说明使用场景",
    format="Markdown"
)
print(formatted)
```

### 3.2.3 部分变量

```python
# 部分填充模板
code_review_prompt = PromptTemplate.from_template("""
角色：{role}
代码：
```{language}
{code}
```
请进行代码审查。
""")

# 固定角色
python_reviewer = code_review_prompt.partial(role="Python专家")

# 使用时只需提供语言和代码
formatted = python_reviewer.format(
    language="python",
    code="def hello(): print('world')"
)
```

## 3.3 ChatPromptTemplate

### 3.3.1 消息模板类型

```python
from langchain_core.prompts import (
    ChatPromptTemplate,
    SystemMessagePromptTemplate,
    HumanMessagePromptTemplate,
    AIMessagePromptTemplate
)

# 系统消息模板
system_template = "你是一位专业的{domain}专家。"
system_message_prompt = SystemMessagePromptTemplate.from_template(system_template)

# 用户消息模板
human_template = "{question}"
human_message_prompt = HumanMessagePromptTemplate.from_template(human_template)

# AI 消息模板（用于 few-shot）
ai_template = "{response}"
ai_message_prompt = AIMessagePromptTemplate.from_template(ai_template)

# 组合成聊天提示
chat_prompt = ChatPromptTemplate.from_messages([
    system_message_prompt,
    human_message_prompt
])

# 使用
messages = chat_prompt.format_messages(
    domain="Python",
    question="什么是装饰器？"
)
print(messages)
```

### 3.3.2 快捷方式

```python
from langchain_core.prompts import ChatPromptTemplate

# 使用元组快捷创建
chat_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一位专业的{domain}专家。"),
    ("human", "{question}")
])

# 或使用常量消息
from langchain_core.messages import SystemMessage

chat_prompt = ChatPromptTemplate.from_messages([
    SystemMessage(content="你是一位Python专家。"),
    ("human", "{question}")
])

messages = chat_prompt.format_messages(
    question="解释列表推导式"
)
```

### 3.3.3 多轮对话模板

```python
# 多轮对话示例
few_shot_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一位翻译专家，将中文翻译成英文。"),
    ("human", "你好"),
    ("ai", "Hello"),
    ("human", "世界"),
    ("ai", "World"),
    ("human", "{input}"),
])

messages = few_shot_prompt.format_messages(input="我爱编程")
for msg in messages:
    print(f"{msg.type}: {msg.content}")
```

## 3.4 Few-Shot Prompting

### 3.4.1 基础 Few-Shot

```python
from langchain_core.prompts import FewShotPromptTemplate, PromptTemplate

# 示例
examples = [
    {
        "input": "快乐",
        "output": "喜悦、愉快、开心、高兴、欢欣"
    },
    {
        "input": "悲伤",
        "output": "难过、哀伤、悲痛、忧郁、沮丧"
    },
    {
        "input": "愤怒",
        "output": "生气、恼怒、气愤、暴怒、发火"
    }
]

# 示例模板
example_template = """
输入：{input}
输出：{output}
"""

example_prompt = PromptTemplate.from_template(example_template)

# Few-Shot 提示
few_shot_prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    suffix="输入：{input}\n输出：",
    input_variables=["input"]
)

# 使用
result = few_shot_prompt.format(input="美丽")
print(result)
```

### 3.4.2 动态选择示例

```python
from langchain_core.prompts import FewShotPromptTemplate
from langchain_core.example_selectors import SemanticSimilarityExampleSelector
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma

# 示例库
examples = [
    {"input": "什么是机器学习？", "output": "机器学习是人工智能的一个分支..."},
    {"input": "什么是深度学习？", "output": "深度学习是机器学习的一个子集..."},
    {"input": "什么是神经网络？", "output": "神经网络是一种模拟人脑结构的算法..."},
    {"input": "什么是监督学习？", "output": "监督学习是一种机器学习方法..."},
]

# 创建示例选择器
example_selector = SemanticSimilarityExampleSelector.from_examples(
    examples,
    OpenAIEmbeddings(),
    Chroma,
    k=2  # 选择最相似的2个示例
)

# 创建 Few-Shot 提示
few_shot_prompt = FewShotPromptTemplate(
    example_selector=example_selector,
    example_prompt=PromptTemplate.from_template("问题：{input}\n回答：{output}\n"),
    suffix="问题：{input}\n回答：",
    input_variables=["input"]
)

# 使用
result = few_shot_prompt.format(input="什么是强化学习？")
print(result)
```

### 3.4.3 长度限制选择

```python
from langchain_core.example_selectors import LengthBasedExampleSelector

# 基于长度的示例选择器
example_selector = LengthBasedExampleSelector(
    examples=examples,
    example_prompt=example_prompt,
    max_length=1000  # 最大长度限制
)

few_shot_prompt = FewShotPromptTemplate(
    example_selector=example_selector,
    example_prompt=example_prompt,
    suffix="输入：{input}\n输出：",
    input_variables=["input"]
)
```

## 3.5 提示词工程技巧

### 3.5.1 角色设定 (Role Prompting)

```python
from langchain_core.prompts import ChatPromptTemplate

# 角色设定模板
role_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一位{role}，具有以下特点：
- {trait1}
- {trait2}
- {trait3}

请以专业、友好的态度回答用户问题。"""),
    ("human", "{question}")
])

# 使用
messages = role_prompt.format_messages(
    role="资深Python开发者",
    trait1="10年Python开发经验",
    trait2="熟悉各种Python框架",
    trait3="善于用通俗语言解释复杂概念",
    question="什么是GIL？"
)
```

### 3.5.2 思维链 (Chain-of-Thought)

```python
# 思维链提示
cot_prompt = PromptTemplate.from_template("""
问题：{question}

请按照以下步骤思考：
1. 理解问题的核心要点
2. 分析可能的解决方案
3. 选择最佳方案
4. 给出详细解答

思考过程：
""")

# 使用
result = cot_prompt.format(question="一个水箱有两个进水管，A管单独注满需要6小时，B管单独注满需要4小时。如果两个管子同时打开，需要多长时间注满？")
```

### 3.5.3 自洽性 (Self-Consistency)

```python
# 生成多个答案并选择最一致的
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(temperature=0.7)

# 多次采样
answers = []
for _ in range(5):
    response = llm.invoke([HumanMessage(content="问题：{question}")])
    answers.append(response.content)

# 投票选择最常见的答案
from collections import Counter
most_common = Counter(answers).most_common(1)[0][0]
```

### 3.5.4 ReAct 模式

```python
# ReAct (Reasoning + Acting) 提示
react_prompt = PromptTemplate.from_template("""
解决以下问题，你可以使用以下工具：
{tools}

请按照以下格式回答：
问题：你要解决的问题
思考：思考如何解决问题
行动：采取的行动（必须是以下之一：{tool_names}）
行动输入：行动的输入
观察：行动的结果
...（这个思考/行动/行动输入/观察可以重复多次）
思考：我现在知道最终答案
最终答案：问题的最终答案

开始！

问题：{input}
思考：{agent_scratchpad}
""")
```

## 3.6 输出格式控制

### 3.6.1 JSON 格式

```python
from langchain_core.prompts import PromptTemplate

json_prompt = PromptTemplate.from_template("""
从以下文本中提取信息，以JSON格式返回：

文本：{text}

要求输出格式：
{{
    "name": "姓名",
    "age": 年龄（数字）,
    "skills": ["技能1", "技能2"],
    "experience": [
        {{
            "company": "公司名",
            "position": "职位",
            "years": 工作年限
        }}
    ]
}}

请确保输出是有效的JSON格式。
""")

# 使用
result = json_prompt.format(text="""
张三，30岁，精通Python和JavaScript。
曾在阿里巴巴担任高级工程师3年，在腾讯担任技术专家2年。
""")
```

### 3.6.2 Markdown 格式

```python
markdown_prompt = PromptTemplate.from_template("""
请将以下内容整理成Markdown格式的文档：

内容：{content}

要求：
- 使用适当的标题层级
- 使用列表和表格
- 添加代码块（如有代码）
- 使用粗体和斜体强调重点
""")
```

### 3.6.3 XML 格式

```python
xml_prompt = PromptTemplate.from_template("""
请将以下数据转换为XML格式：

数据：{data}

要求：
- 使用有意义的标签名
- 包含适当的属性
- 格式正确，可解析
""")
```

## 3.7 提示词优化技巧

### 3.7.1 分隔符使用

```python
delimiter_prompt = PromptTemplate.from_template("""
请翻译以下文本：

```
{text}
```

要求：
1. 保持原文意思
2. 语言流畅自然
3. 专业术语准确
""")
```

### 3.7.2 条件提示

```python
conditional_prompt = PromptTemplate.from_template("""
{context}

根据以上背景信息，{task}

{if_detail}
""")

# 条件渲染
detailed_instruction = """请提供：
- 详细的步骤说明
- 代码示例
- 常见错误及解决方案
- 最佳实践建议""" if need_detail else "请简要回答。"

result = conditional_prompt.format(
    context="背景信息...",
    task="解释如何实现用户认证",
    if_detail=detailed_instruction
)
```

### 3.7.3 提示词组合

```python
from langchain_core.prompts import PipelinePromptTemplate

# 定义各个部分
intro_prompt = PromptTemplate.from_template("你是一位{role}。")
task_prompt = PromptTemplate.from_template("你的任务是{task}。")
format_prompt = PromptTemplate.from_template("请以{format}格式输出。")

# 组合
full_prompt = PromptTemplate.from_template("""
{intro}
{task}
{format}

{input}
""")

pipeline_prompt = PipelinePromptTemplate(
    final_prompt=full_prompt,
    pipeline_prompts=[
        ("intro", intro_prompt),
        ("task", task_prompt),
        ("format", format_prompt)
    ]
)

# 使用
result = pipeline_prompt.format(
    role="Python专家",
    task="解释装饰器",
    format="Markdown",
    input="什么是装饰器？"
)
```

## 3.8 完整示例

### 3.8.1 代码审查助手

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import JsonOutputParser

code_review_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一位资深代码审查专家。请对提供的代码进行全面审查。

审查维度：
1. 代码风格 - 是否符合PEP8规范
2. 可读性 - 命名、注释、结构
3. 性能 - 算法复杂度、资源使用
4. 安全性 - 潜在的安全漏洞
5. 可维护性 - 模块化、耦合度

输出要求：
- 给出整体评分（1-10分）
- 列出发现的问题（按严重程度排序）
- 提供改进建议
- 给出优化后的代码示例"""),
    ("human", """编程语言：{language}
代码：
```{language}
{code}
```""")
])

# 使用
review_chain = code_review_prompt | ChatOpenAI(model="gpt-4", temperature=0.3)

code = """
def calculate_sum(n):
    result = 0
    for i in range(n):
        result = result + i
    return result
"""

response = review_chain.invoke({
    "language": "python",
    "code": code
})
print(response.content)
```

### 3.8.2 智能客服提示

```python
from langchain_core.prompts import ChatPromptTemplate

support_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是{company}的智能客服助手，名叫{name}。

基本信息：
- 公司主营：{business}
- 服务时间：{service_hours}
- 联系方式：{contact}

回答原则：
1. 保持友好、专业的语气
2. 先理解用户问题，再给出解答
3. 如果问题超出知识范围，引导用户联系人工客服
4. 涉及账户安全的问题，必须验证身份
5. 复杂问题分步骤说明

禁止事项：
- 不要泄露其他用户的隐私信息
- 不要承诺无法保证的事项
- 不要提供可能有害的建议"""),
    ("human", "{question}")
])

# 使用
messages = support_prompt.format_messages(
    company="TechShop",
    name="小助手",
    business="电子产品销售与售后服务",
    service_hours="9:00-21:00",
    contact="客服热线：400-123-4567",
    question="我的订单什么时候能到？"
)
```

### 3.8.3 结构化数据提取

```python
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import JsonOutputParser
from langchain_core.pydantic_v1 import BaseModel, Field
from typing import List, Optional

# 定义输出结构
class Person(BaseModel):
    name: str = Field(description="姓名")
    age: Optional[int] = Field(description="年龄")
    email: Optional[str] = Field(description="邮箱")
    phone: Optional[str] = Field(description="电话")
    address: Optional[str] = Field(description="地址")
    skills: List[str] = Field(description="技能列表", default=[])

parser = JsonOutputParser(pydantic_object=Person)

extract_prompt = PromptTemplate.from_template("""
从以下文本中提取人物信息：

文本：
{text}

{format_instructions}

注意：
- 如果信息不存在，使用 null
- 技能请以列表形式返回
- 确保JSON格式正确
""")

extract_prompt = extract_prompt.partial(format_instructions=parser.get_format_instructions())

# 使用
text = """
联系人：李明，今年32岁，是一名全栈开发者。
邮箱：liming@example.com，电话：138-1234-5678。
住在北京市朝阳区。
精通Python、JavaScript、React和Docker。
"""

from langchain_openai import ChatOpenAI
extract_chain = extract_prompt | ChatOpenAI(model="gpt-4", temperature=0) | parser

result = extract_chain.invoke({"text": text})
print(result)
```

## 3.9 提示词模板管理

### 3.9.1 从文件加载

```python
from langchain_core.prompts import load_prompt

# 从 YAML 文件加载
prompt = load_prompt("prompts/qa_prompt.yaml")

# YAML 格式示例：
# _type: prompt
# input_variables:
#   - question
# template: |
#   请回答以下问题：
#   {question}
```

### 3.9.2 保存提示词

```python
# 保存为文件
prompt.save("prompts/my_prompt.json")

# JSON 格式
{
    "input_variables": ["topic"],
    "template": "告诉我关于{topic}的信息。",
    "template_format": "f-string",
    "validate_template": true
}
```

## 3.10 总结

本章详细介绍了 LangChain 中的提示词工程技术：

1. **基础模板**：PromptTemplate、ChatPromptTemplate
2. **Few-Shot**：通过示例引导模型输出
3. **高级技巧**：角色设定、思维链、ReAct
4. **输出控制**：JSON、Markdown、XML 格式
5. **模板管理**：加载、保存、组合

**最佳实践**：
- 提示词要清晰、具体、结构化
- 使用 Few-Shot 提高输出质量
- 合理使用分隔符和格式说明
- 根据场景选择合适的温度参数

---

*下一章我们将学习 Chains - 链式调用，将多个组件组合成工作流。*
