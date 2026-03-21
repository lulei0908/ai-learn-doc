# 第四章：Chains - 链式调用

## 4.1 Chains 概述

Chains 是 LangChain 的核心概念之一，它允许我们将多个组件（提示词模板、模型、输出解析器等）串联起来，形成一个完整的工作流。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        LangChain 链式调用                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐        │
│   │  Input  │ → │ Prompt  │ → │   LLM   │ → │ Parser  │ → Output│
│   │         │    │ Template│    │         │    │         │        │
│   └─────────┘    └─────────┘    └─────────┘    └─────────┘        │
│                                                                      │
│                          LCEL 语法                                   │
│                                                                      │
│        chain = prompt | llm | output_parser                          │
│                     ↓                                                │
│              Runnable 协议支持                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 4.2 LCEL (LangChain Expression Language)

### 4.2.1 基础语法

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 基础链
llm = ChatOpenAI(model="gpt-4")
prompt = PromptTemplate.from_template("解释{topic}，用一句话概括")
parser = StrOutputParser()

# 使用 | 操作符组合链
chain = prompt | llm | parser

# 执行
result = chain.invoke({"topic": "人工智能"})
print(result)
```

### 4.2.2 Runnable 协议

LangChain 中的所有组件都实现了 `Runnable` 协议，具有以下方法：

```python
# Runnable 接口方法
chain.invoke(input)        # 同步调用
chain.ainvoke(input)       # 异步调用
chain.batch(inputs)        # 批量同步调用
chain.abatch(inputs)       # 批量异步调用
chain.stream(input)        # 流式调用
chain.get_graph()          # 获取计算图
```

### 4.2.3 链的检查和调试

```python
# 查看链的结构
print(chain.get_graph().print_ascii())

# 使用 LangSmith 追踪（需要配置）
from langchain.callbacks import LangChainCallbackHandler

chain = prompt | llm | parser
chain.invoke(
    {"topic": "机器学习"},
    config={"callbacks": [LangChainCallbackHandler()]}
)
```

## 4.3 LLMChain

### 4.3.1 基础 LLMChain

```python
from langchain_openai import OpenAI
from langchain_core.prompts import PromptTemplate
from langchain.chains import LLMChain

# 创建 LLMChain
llm = OpenAI(temperature=0.9)
prompt = PromptTemplate.from_template("为{product}写一句广告词")

chain = LLMChain(llm=llm, prompt=prompt)

# 执行
result = chain.run(product="智能手机")
print(result)
```

### 4.3.2 多输入多输出

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")
prompt = PromptTemplate.from_template("""
为以下产品撰写营销文案：

产品名称：{product_name}
目标用户：{target_audience}
核心卖点：{key_benefits}

请生成：
1. 一句话广告语
2. 产品描述（100字以内）
3. 行动号召语
""")

chain = LLMChain(
    llm=llm,
    prompt=prompt,
    output_key="marketing_content"  # 指定输出键名
)

result = chain.invoke({
    "product_name": "智能手表",
    "target_audience": "年轻职场人士",
    "key_benefits": "健康监测、高效办公、时尚设计"
})

print(result["marketing_content"])
```

### 4.3.3 指定输出变量

```python
# 指定多个输出变量
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import JsonOutputParser

llm = ChatOpenAI(model="gpt-4", temperature=0)
prompt = PromptTemplate.from_template("""
分析以下公司并返回JSON格式：
公司名：{company}
行业：{industry}

返回格式：
{{
    "strengths": ["优势1", "优势2"],
    "weaknesses": ["劣势1", "劣势2"],
    "opportunities": ["机会1", "机会2"],
    "threats": ["威胁1", "威胁2"]
}}
""")

chain = LLMChain(llm=llm, prompt=prompt, output_key="swot_analysis")

result = chain.invoke({
    "company": "特斯拉",
    "industry": "新能源汽车"
})

print(result["swot_analysis"])
```

## 4.4 SequentialChain

### 4.4.1 简单顺序链

```python
from langchain.chains import SequentialChain, LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", temperature=0)

# 第一个链：生成故事
first_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate.from_template("""
写一个关于{theme}的短篇故事，控制在200字以内。
主题：{theme}
"""),
    output_key="story"
)

# 第二个链：生成标题
title_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate.from_template("""
根据以下故事起一个吸引人的标题：

{story}

标题：
"""),
    output_key="title"
)

# 第三个链：生成摘要
summary_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate.from_template("""
为以下故事写一个50字的摘要：

{story}

摘要：
"""),
    output_key="summary"
)

# 组合顺序链
overall_chain = SequentialChain(
    chains=[first_chain, title_chain, summary_chain],
    input_variables=["theme"],
    output_variables=["story", "title", "summary"],
    verbose=True
)

# 执行
result = overall_chain.invoke({"theme": "人工智能"})
print("标题:", result["title"])
print("故事:", result["story"])
print("摘要:", result["summary"])
```

### 4.4.2 带内存的顺序链

```python
from langchain.chains import SequentialChain, LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI
from langchain.memory import ConversationBufferMemory

llm = ChatOpenAI(model="gpt-4", temperature=0)

# 第一个链：分析用户需求
analyze_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate.from_template("""
分析用户需求：
用户描述：{user_input}

输出结构化的需求分析。
"""),
    output_key="requirements"
)

# 第二个链：生成解决方案
solution_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate.from_template("""
基于以下需求，提供解决方案：

{requirements}

解决方案：
"""),
    output_key="solution"
)

# 第三个链：评估方案
eval_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate.from_template("""
评估以下解决方案的优缺点：

{solution}

评估：
"""),
    output_key="evaluation"
)

# 组合
overall_chain = SequentialChain(
    chains=[analyze_chain, solution_chain, eval_chain],
    input_variables=["user_input"],
    output_variables=["requirements", "solution", "evaluation"],
    verbose=True
)

result = overall_chain.invoke({
    "user_input": "我需要一个网站来展示我的摄影作品"
})
```

## 4.5 RouterChain

### 4.5.1 基础路由链

```python
from langchain.chains import LLMChain, RouterChain
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", temperature=0)

# 定义多个目的地链
math_prompt = PromptTemplate.from_template("""
计算并解释：{problem}
""")

history_prompt = PromptTemplate.from_template("""
解释历史事件：{topic}
""")

science_prompt = PromptTemplate.from_template("""
用通俗易懂的方式解释科学概念：{concept}
""")

# 创建各个链
math_chain = LLMChain(llm=llm, prompt=math_prompt)
history_chain = LLMChain(llm=llm, prompt=history_prompt)
science_chain = LLMChain(llm=llm, prompt=science_prompt)

# 路由链
router_template = """
根据用户问题，选择最合适的方向：
- math：数学计算问题
- history：历史相关问题
- science：科学概念问题

用户问题：{input}

只返回方向名称，不要其他内容。
"""

router_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate.from_template(router_template),
    output_key="destination"
)

# 定义路由映射
chain_map = {
    "math": math_chain,
    "history": history_chain,
    "science": science_chain
}

def routing_function(inputs):
    # 使用路由链确定目的地
    result = router_chain.invoke({"input": inputs["input"]})
    destination = result["destination"].strip().lower()
    return chain_map.get(destination, math_chain)

# 使用 RouterChain
router_chain_instance = RouterChain(
    router_chain=router_chain,
    destination_chains=chain_map,
    default_chain=math_chain,
    routing_function=routing_function
)

result = router_chain_instance.invoke({"input": "计算圆的面积公式"})
print(result)
```

### 4.5.2 使用 LCEL 实现路由

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnableBranch

llm = ChatOpenAI(model="gpt-4", temperature=0)

# 定义路由条件
def route(topic: str) -> str:
    if "数学" in topic or "计算" in topic:
        return "math"
    elif "历史" in topic:
        return "history"
    elif "科学" in topic:
        return "science"
    else:
        return "general"

# 定义各分支链
math_chain = PromptTemplate.from_template("数学问题：{topic}") | llm
history_chain = PromptTemplate.from_template("历史问题：{topic}") | llm
science_chain = PromptTemplate.from_template("科学问题：{topic}") | llm
general_chain = PromptTemplate.from_template("一般问题：{topic}") | llm

# 创建分支
branch_chain = RunnableBranch(
    (lambda x: "数学" in x["topic"] or "计算" in x["topic"], math_chain),
    (lambda x: "历史" in x["topic"], history_chain),
    (lambda x: "科学" in x["topic"], science_chain),
    general_chain
)

# 主链
main_chain = (
    {"topic": lambda x: x["topic"]}
    | branch_chain
)

result = main_chain.invoke({"topic": "勾股定理是什么"})
```

## 4.6 TransformChain

### 4.6.1 数据转换

```python
from langchain.chains import TransformChain, LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.runnables import RunnableLambda

# 自定义转换函数
def transform_func(inputs):
    """清理和标准化文本"""
    text = inputs["text"]
    # 简单的文本清理
    cleaned = text.strip().replace("\n\n", "\n")
    return {"cleaned_text": cleaned}

# 创建转换链
transform_chain = TransformChain(
    input_variables=["text"],
    output_variables=["cleaned_text"],
    transform=transform_func
)

# 与 LLMChain 组合
llm = ChatOpenAI(model="gpt-4")
prompt = PromptTemplate.from_template("总结以下文本：{cleaned_text}")
summary_chain = LLMChain(llm=llm, prompt=prompt)

# 组合完整链
full_chain = transform_chain | summary_chain

result = full_chain.invoke({
    "text": """
    
    这是一段需要清理的文本。
    
    
    包含多余的空行。
    
    """
})

print(result["text"])
```

### 4.6.2 使用 RunnableLambda

```python
from langchain_core.runnables import RunnableLambda

# 提取文档摘要
def extract_summary(documents: list) -> str:
    """提取文档摘要"""
    return "\n".join([doc.page_content[:100] for doc in documents])

summary_runnable = RunnableLambda(extract_summary)

# 组合
chain = summary_runnable | llm
```

## 4.7 RetrievalChain

### 4.7.1 基础检索链

```python
from langchain.chains import RetrievalQA
from langchain_openai import OpenAI, OpenAIEmbeddings
from langchain_community.vectorstores import Chroma

# 假设已有向量存储
vectorstore = Chroma(...)

# 创建检索链
qa_chain = RetrievalQA.from_chain_type(
    llm=OpenAI(temperature=0),
    chain_type="stuff",  # stuff, map_reduce, refine
    retriever=vectorstore.as_retriever(),
    return_source_documents=True
)

# 执行问答
result = qa_chain.invoke({"query": "LangChain 是什么？"})
print(result["result"])
print(result["source_documents"])
```

### 4.7.2 自定义检索链

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", temperature=0)

# 检索并生成
prompt = PromptTemplate.from_template("""
基于以下上下文回答问题。如果上下文中没有相关信息，请说明不知道。

上下文：
{context}

问题：{question}

回答：
""")

# 假设有一个检索函数
def retrieve_docs(query: str) -> str:
    # 实际应用中这里会进行向量检索
    return "检索到的相关文档内容..."

retrieval_chain = (
    {"context": retrieve_docs, "question": lambda x: x["question"]}
    | prompt
    | llm
)

result = retrieval_chain.invoke({"question": "LangChain 的核心组件有哪些？"})
```

## 4.8 对话链

### 4.8.1 带记忆的对话链

```python
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI
from langchain.memory import ConversationBufferMemory

llm = ChatOpenAI(model="gpt-4", temperature=0.7)
memory = ConversationBufferMemory(return_messages=True)

conversation = ConversationChain(
    llm=llm,
    memory=memory,
    verbose=True
)

# 对话
conversation.invoke("我叫张三")
conversation.invoke("我最喜欢的颜色是蓝色")
conversation.invoke("我叫什么名字？")  # 应该回答"张三"

# 查看记忆
print(memory.buffer)
```

### 4.8.2 对话摘要链

```python
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI
from langchain.memory import ConversationSummaryMemory

llm = ChatOpenAI(model="gpt-4", temperature=0.7)
memory = ConversationSummaryMemory(llm=llm, return_messages=True)

conversation = ConversationChain(
    llm=llm,
    memory=memory,
    verbose=False
)

# 长时间对话后查看摘要
conversation.invoke("我叫张三，是一名软件工程师")
conversation.invoke("我目前在阿里巴巴工作")
conversation.invoke("我正在学习机器学习")

# 查看摘要
print(memory.load_memory_variables({}))
```

## 4.9 自定义 Chain

### 4.9.1 继承 BaseChain

```python
from langchain.chains import BaseChain
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI

class CustomTextChain(BaseChain):
    """自定义文本处理链"""
    
    def __init__(self):
        super().__init__()
        self.llm = ChatOpenAI(model="gpt-4", temperature=0.7)
        self.prompt = PromptTemplate.from_template(
            "将以下文本改写成{tone}风格：\n\n{text}"
        )
    
    @property
    def input_keys(self) -> list:
        return ["text", "tone"]
    
    @property
    def output_keys(self) -> list:
        return ["rewritten_text"]
    
    def _call(self, inputs: dict) -> dict:
        text = inputs["text"]
        tone = inputs["tone"]
        
        prompt_text = self.prompt.format(text=text, tone=tone)
        response = self.llm.invoke(prompt_text)
        
        return {"rewritten_text": response.content}

# 使用自定义链
chain = CustomTextChain()
result = chain.invoke({
    "text": "今天的天气很好",
    "tone": "正式"
})
print(result["rewritten_text"])
```

### 4.9.2 使用 RunnableLambda

```python
from langchain_core.runnables import RunnableLambda, RunnablePassthrough

# 自定义处理函数
def extract_entities(text: str) -> dict:
    """提取命名实体"""
    # 这里可以是复杂的 NER 处理
    return {
        "persons": ["张三", "李四"],
        "organizations": ["阿里巴巴"],
        "locations": ["北京"]
    }

def generate_summary(info: dict) -> str:
    """基于提取的信息生成摘要"""
    return f"提到了{len(info['persons'])}个人，{len(info['organizations'])}个组织"

# 组合链
entity_chain = RunnableLambda(extract_entities)
summary_chain = RunnableLambda(generate_summary)

# 完整链
full_chain = entity_chain | summary_chain

result = full_chain.invoke("张三和李四在阿里巴巴工作，他们住在上海。")
```

## 4.10 链的复用和组合

### 4.10.1 链的复用

```python
from langchain_core.runnables import Runnable

# 定义一个可复用的链
base_qa_chain = (
    {"context": ..., "question": RunnablePassthrough()}
    | prompt
    | llm
    | parser
)

# 在不同场景中复用
medical_qa = base_qa_chain.with_config(configurable={"medical": True})
legal_qa = base_qa_chain.with_config(configurable={"legal": True})
```

### 4.10.2 链的并行执行

```python
from langchain_core.runnables import RunnableParallel

# 并行执行多个链
parallel_chain = RunnableParallel(
    summary=summary_chain,
    sentiment=sentiment_chain,
    keywords=keyword_chain
)

result = parallel_chain.invoke({"text": "长文本..."})

# 并行结果
print(result["summary"])
print(result["sentiment"])
print(result["keywords"])
```

### 4.10.3 链的条件分支

```python
from langchain_core.runnables import RunnableBranch

branch_chain = RunnableBranch(
    (lambda x: x["type"] == "positive", positive_chain),
    (lambda x: x["type"] == "negative", negative_chain),
    neutral_chain  # 默认
)

result = branch_chain.invoke({"text": "...", "type": "positive"})
```

## 4.11 完整示例

### 4.11.1 文档分析流水线

```python
from langchain.chains import SequentialChain, LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import JsonOutputParser

llm = ChatOpenAI(model="gpt-4", temperature=0)

# Step 1: 提取关键信息
extract_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate.from_template("""
从以下文本中提取关键信息和实体：

{text}

提取：
- 主要主题
- 关键人物
- 重要日期
- 核心观点
"""),
    output_key="extracted_info"
)

# Step 2: 生成摘要
summary_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate.from_template("""
基于以下信息生成简洁摘要：

{extracted_info}

摘要（100字以内）：
"""),
    output_key="summary"
)

# Step 3: 问答生成
qa_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate.from_template("""
基于以下内容，生成3个常见问题和答案：

{text}

Q1:
A1:
Q2:
A2:
Q3:
A3:
"""),
    output_key="qa_pairs"
)

# Step 4: 标签生成
tag_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate.from_template("""
为以下内容生成标签（3-5个）：

{text}

标签：
"""),
    output_key="tags"
)

# 组合完整流水线
document_pipeline = SequentialChain(
    chains=[extract_chain, summary_chain, qa_chain, tag_chain],
    input_variables=["text"],
    output_variables=["extracted_info", "summary", "qa_pairs", "tags"],
    verbose=True
)

# 执行
text = """
2024年3月15日，阿里巴巴集团发布了2023年度财报。财报显示，
公司全年营收达到8686.87亿元，同比增长7.3%。云计算业务
表现强劲，营收同比增长29.7%。CEO张勇表示，公司将继续
加大在AI和云计算领域的投入。
"""

result = document_pipeline.invoke({"text": text})

print("摘要:", result["summary"])
print("标签:", result["tags"])
```

### 4.11.2 多语言翻译流水线

```python
from langchain.chains import LLMChain, ParallelChainRunner
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.runnables import RunnableParallel

llm = ChatOpenAI(model="gpt-4", temperature=0.3)

# 定义翻译链模板
def create_translation_chain(target_lang: str):
    return LLMChain(
        llm=llm,
        prompt=PromptTemplate.from_template("""
将以下中文文本翻译成{lang}：

{text}
""".format(lang=target_lang)),
        output_key=target_lang
    )

# 创建多个语言的翻译链
chains = {
    "english": create_translation_chain("English"),
    "japanese": create_translation_chain("Japanese"),
    "korean": create_translation_chain("Korean"),
    "french": create_translation_chain("French"),
}

# 并行翻译
from langchain_core.runnables import RunnableParallel

parallel_translation = RunnableParallel(**chains)

# 执行
result = parallel_translation.invoke({
    "text": "LangChain是一个强大的AI应用开发框架。"
})

print("English:", result["english"]["text"])
print("Japanese:", result["japanese"]["text"])
print("Korean:", result["korean"]["text"])
print("French:", result["french"]["text"])
```

## 4.12 总结

本章介绍了 LangChain 中 Chains 的各种用法：

1. **LCEL 语法**：使用 `|` 操作符组合链
2. **LLMChain**：最基本的链式调用
3. **SequentialChain**：顺序执行多个链
4. **RouterChain**：根据条件路由到不同链
5. **TransformChain**：数据转换
6. **RetrievalChain**：检索增强生成
7. **ConversationChain**：带记忆的对话

**核心要点**：
- 所有 LangChain 组件都实现 Runnable 接口
- 使用 `|` 操作符组合组件
- 支持同步、异步、流式调用
- 可以自定义 Chain 和处理函数

---

*下一章我们将学习 Memory - 记忆组件，为应用添加持久化的上下文。*
