# 第八章：Agents - 代理

## 8.1 Agents 概述

Agents 是 LangChain 中让 LLM 自主决策和执行操作的组件。与固定工作流不同，Agent 能够根据输入动态决定使用哪些工具、如何处理问题。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Agent 架构                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                         Agent                                │  │
│   ├─────────────────────────────────────────────────────────────┤  │
│   │                                                              │  │
│   │   ┌──────────┐                                              │  │
│   │   │   LLM    │ ← 大脑，理解指令，决定行动                    │  │
│   │   └────┬─────┘                                              │  │
│   │        │                                                    │  │
│   │        ↓                                                    │  │
│   │   ┌──────────┐                                              │  │
│   │   │  Reasoning │ ← 思考过程，分析问题                        │  │
│   │   └────┬─────┘                                              │  │
│   │        │                                                    │  │
│   │        ↓                                                    │  │
│   │   ┌──────────┐                                              │  │
│   │   │  Planning │ ← 规划行动，选择工具                         │  │
│   │   └────┬─────┘                                              │  │
│   │        │                                                    │  │
│   │        ↓                                                    │  │
│   │   ┌──────────┐                                              │  │
│   │   │ Executor │ ← 执行工具调用                                │  │
│   │   └──────────┘                                              │  │
│   │                                                              │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                                                      │
│                         Tools                                        │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐          │  │
│   │  │ Search │  │  Code   │  │   API   │  │ Memory  │          │  │
│   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘          │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 8.2 Agent 类型

### 8.2.1 ReAct Agent

```python
from langchain.agents import AgentExecutor, create_react_agent
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain import hub

# 定义工具
@tool
def calculator(expression: str) -> str:
    """执行数学计算"""
    try:
        return str(eval(expression))
    except Exception as e:
        return f"计算错误: {e}"

@tool
def search(query: str) -> str:
    """搜索信息"""
    return f"搜索结果: {query}相关信息..."

# 创建 Agent
llm = ChatOpenAI(model="gpt-4", temperature=0)

# 从 Hub 获取 ReAct 提示词
prompt = hub.pull("hwchase17/react")

# 创建 Agent
agent = create_react_agent(llm, [calculator, search], prompt)

# 创建执行器
agent_executor = AgentExecutor.from_agent_and_tools(
    agent=agent,
    tools=[calculator, search],
    verbose=True,
    max_iterations=10
)

# 执行
result = agent_executor.invoke({"input": "搜索 LangChain 的信息，然后计算 'LangChain' 的字符长度"})
print(result["output"])
```

### 8.2.2 OpenAI Functions Agent

```python
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain_core.prompts import ChatPromptTemplate

@tool
def get_weather(city: str) -> str:
    """获取城市天气"""
    weather_data = {
        "北京": "晴天 25°C",
        "上海": "多云 22°C",
        "广州": "小雨 28°C"
    }
    return weather_data.get(city, "未找到该城市天气")

@tool
def get_time(city: str) -> str:
    """获取城市时间"""
    return f"{city} 当前时间: 10:30"

# 创建 Agent
llm = ChatOpenAI(model="gpt-4", temperature=0)
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个有帮助的助手。"),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

agent = create_openai_functions_agent(llm, [get_weather, get_time], prompt)

agent_executor = AgentExecutor(
    agent=agent,
    tools=[get_weather, get_time],
    verbose=True
)

# 执行
result = agent_executor.invoke({"input": "北京现在天气怎么样？"})
```

### 8.2.3 XML Agent (Anthropic Claude)

```python
from langchain.agents import create_xml_agent
from langchain_ad官员 import ChatAnthropic
from langchain_core.tools import tool

# 使用 Claude
llm = ChatAnthropic(model="claude-3-sonnet-20240229")

# XML Agent
agent = create_xml_agent(llm, tools, prompt)

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True
)
```

### 8.2.4 Structured Chat Agent

```python
from langchain.agents import create_structured_chat_agent
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

# 使用结构化聊天 Agent
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个有帮助的助手。"),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

agent = create_structured_chat_agent(llm, tools, prompt)
```

## 8.3 Tool 绑定

### 8.3.1 自动工具选择

```python
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool

@tool
def search_news(topic: str) -> str:
    """搜索新闻"""
    return f"{topic}相关新闻..."

@tool
def get_stock(symbol: str) -> str:
    """获取股票价格"""
    return f"{symbol} 价格: $100"

@tool
def send_email(to: str, content: str) -> str:
    """发送邮件"""
    return f"邮件已发送给 {to}"

@tool
def create_reminder(time: str, content: str) -> str:
    """创建提醒"""
    return f"已在 {time} 创建提醒: {content}"

# 绑定多个工具
llm = ChatOpenAI(model="gpt-4", temperature=0)
llm_with_tools = llm.bind_tools([search_news, get_stock, send_email, create_reminder])

# Agent 会根据问题自动选择合适的工具
response = llm_with_tools.invoke("帮我查一下苹果公司的股票价格")
print(response.tool_calls)
```

### 8.3.2 强制使用特定工具

```python
# 强制使用工具（适用于需要特定工具的场景）
llm_forced = llm.bind_tools(
    [get_stock],
    tool_choice="get_stock"  # 强制使用 get_stock
)
```

## 8.4 AgentExecutor 配置

### 8.4.1 基础配置

```python
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,                    # 打印详细输出
    max_iterations=10,              # 最大迭代次数
    max_execution_time=60,          # 最大执行时间（秒）
    handle_parsing_errors=True,     # 处理解析错误
    early_stopping_method="force"   # 强制停止方法
)
```

### 8.4.2 错误处理配置

```python
# 自定义错误处理
def handle_error(error):
    return f"执行出错: {str(error)}"

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    handle_parsing_errors=handle_error,
    max_iterations=5
)
```

### 8.4.3 返回中间步骤

```python
# 返回所有中间步骤
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    return_intermediate_steps=True
)

result = agent_executor.invoke({"input": "计算 2+2"})

print("最终答案:", result["output"])
print("\n中间步骤:")
for step in result["intermediate_steps"]:
    print(f"  行动: {step[0]}")
    print(f"  观察: {step[1]}")
```

## 8.5 对话式 Agent

### 8.5.1 带记忆的 Agent

```python
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain.memory import ConversationBufferMemory
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

# 创建记忆
memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

# 创建 Agent
llm = ChatOpenAI(model="gpt-4", temperature=0)

prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个有帮助的助手。"),
    ("placeholder", "{chat_history}"),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

agent = create_openai_functions_agent(llm, tools, prompt)

# 创建带记忆的执行器
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    memory=memory,
    verbose=True
)

# 多轮对话
agent_executor.invoke({"input": "我叫张三"})
agent_executor.invoke({"input": "我叫什么名字？"})  # 应该记住
```

### 8.5.2 状态管理

```python
from langchain_core.runnables import RunnableConfig

# 带状态的 Agent
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    config=RunnableConfig(
        configurable={
            "user_id": "user_123",
            "session_id": "session_456"
        }
    )
)

# 状态会传递给工具
result = agent_executor.invoke({"input": "记住我的偏好"})
```

## 8.6 自定义 Agent

### 8.6.1 自定义 Agent 类

```python
from langchain_core.agents import Agent, AgentAction, AgentFinish
from langchain_core.callbacks import CallbackManagerForChainRun
from typing import List, Union, Optional

class CustomAgent(Agent):
    """自定义 Agent"""
    
    @property
    def observation_prefix(self) -> str:
        return "观察: "
    
    @property
    def llm_prefix(self) -> str:
        return "思考: "
    
    def plan(
        self,
        intermediate_steps: List[tuple],
        callbacks: Optional[CallbackManagerForChainRun] = None,
        **kwargs
    ) -> Union[AgentAction, AgentFinish]:
        """实现 Agent 的计划方法"""
        # 简单的实现
        if intermediate_steps:
            return AgentFinish(
                return_values={"output": "任务完成"},
                log=""
            )
        
        # 调用 LLM 决定下一步
        ...
    
    async def aplan(self, *args, **kwargs) -> Union[AgentAction, AgentFinish]:
        """异步版本"""
        return self.plan(*args, **kwargs)
```

### 8.6.2 自定义输出解析器

```python
from langchain.agents import AgentOutputParser
from langchain_core.agents import AgentAction, AgentFinish
from typing import Union

class CustomOutputParser(AgentOutputParser):
    """自定义输出解析器"""
    
    def parse(self, text: str) -> Union[AgentAction, AgentFinish]:
        # 解析 LLM 输出
        if "最终答案" in text:
            return AgentFinish(
                return_values={"output": text},
                log=text
            )
        
        # 解析工具调用
        if "行动:" in text:
            # 解析行动
            return AgentAction(
                tool="tool_name",
                tool_input={"arg": "value"},
                log=text
            )
        
        return AgentFinish(
            return_values={"output": "无法解析"},
            log=text
        )
```

## 8.7 工具选择策略

### 8.7.1 单工具选择

```python
from langchain.agents import Tool, AgentExecutor, ZeroShotAgent
from langchain.prompts import PromptTemplate

# 工具列表
tools = [
    Tool(name="Search", func=search_func, description="搜索信息"),
    Tool(name="Calculator", func=calc_func, description="执行计算"),
    Tool(name="Weather", func=weather_func, description="查天气")
]

# 单工具选择（一次只选择一个工具）
prompt = PromptTemplate.from_template("""
你可以使用以下工具：
{tools}

用户问题: {input}
{agent_scratchpad}
""")
```

### 8.7.2 多工具选择

```python
# OpenAI Functions Agent 支持一次调用多个工具
llm_with_multiple_tools = llm.bind_tools(
    [tool1, tool2, tool3, tool4],
    tool_choice="auto"  # 自动选择，可以是多个
)
```

## 8.8 完整示例

### 8.8.1 自主研究 Agent

```python
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain_core.prompts import ChatPromptTemplate
from langchain.memory import ConversationBufferMemory

# 研究工具集
@tool
def search_arxiv(query: str) -> str:
    """搜索 arXiv 论文"""
    return f"arXiv 论文: {query}相关研究..."

@tool
def get_paper_details(paper_id: str) -> str:
    """获取论文详情"""
    return f"论文 {paper_id} 详情..."

@tool
def download_pdf(url: str) -> str:
    """下载 PDF"""
    return f"已下载 PDF: {url}"

@tool
def summarize_text(text: str) -> str:
    """总结文本"""
    return f"总结: {text[:100]}..."

@tool
def translate_text(text: str, target_lang: str) -> str:
    """翻译文本"""
    return f"翻译 ({target_lang}): {text}"

# 创建 Agent
llm = ChatOpenAI(model="gpt-4", temperature=0)

prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个研究助手，可以帮助用户：
1. 搜索学术论文
2. 获取论文详情
3. 下载论文
4. 总结内容
5. 翻译文字

请系统地进行研究，确保信息准确。"""),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

research_tools = [search_arxiv, get_paper_details, download_pdf, summarize_text, translate_text]
agent = create_openai_functions_agent(llm, research_tools, prompt)

agent_executor = AgentExecutor(
    agent=agent,
    tools=research_tools,
    verbose=True,
    max_iterations=15
)

# 执行研究
result = agent_executor.invoke({
    "input": "帮我研究一下 RAG 系统的最新进展，找到相关论文并总结"
})
```

### 8.8.2 智能助手 Agent

```python
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain_core.prompts import ChatPromptTemplate
from langchain.memory import ConversationBufferMemory

# 助手工具
@tool
def web_search(query: str) -> str:
    """搜索网络"""
    return f"搜索结果: {query}..."

@tool
def calculator(expression: str) -> str:
    """计算器"""
    try:
        return str(eval(expression))
    except:
        return "计算错误"

@tool
def get_weather(city: str) -> str:
    """查天气"""
    return f"{city}天气: 晴 25°C"

@tool
def send_message(to: str, message: str) -> str:
    """发送消息"""
    return f"消息已发送给 {to}"

@tool
def create_event(title: str, time: str, description: str) -> str:
    """创建日程"""
    return f"已创建日程: {title} 在 {time}"

@tool
def set_reminder(time: str, content: str) -> str:
    """设置提醒"""
    return f"已设置提醒: {content} 在 {time}"

assistant_tools = [
    web_search, calculator, get_weather,
    send_message, create_event, set_reminder
]

# 创建助手
llm = ChatOpenAI(model="gpt-4", temperature=0.7)

prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个智能助手，名叫小助手。
你可以帮助用户：
- 搜索信息
- 执行计算
- 查询天气
- 发送消息
- 管理日程
- 设置提醒

请友好、耐心地帮助用户。"""),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

agent = create_openai_functions_agent(llm, assistant_tools, prompt)

# 带记忆的助手
memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

assistant_executor = AgentExecutor(
    agent=agent,
    tools=assistant_tools,
    memory=memory,
    verbose=True
)

# 对话
assistant_executor.invoke({"input": "我叫张三"})
assistant_executor.invoke({"input": "帮我查一下北京天气"})
assistant_executor.invoke({"input": "设置一个明天上午10点的会议提醒"})
assistant_executor.invoke({"input": "我叫什么名字？"})
```

### 8.8.3 数据分析 Agent

```python
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain_core.prompts import ChatPromptTemplate
from langchain_experimental.tools import PythonREPLTool

# 数据分析工具
@tool
def load_data(file_path: str) -> str:
    """加载数据文件"""
    return f"已加载数据: {file_path}"

@tool
def analyze_data(operation: str, params: str) -> str:
    """数据分析操作"""
    return f"分析结果: {operation} ({params})"

@tool
def create_visualization(chart_type: str, data: str) -> str:
    """创建可视化"""
    return f"已创建 {chart_type} 图表"

@tool
def export_report(format: str, content: str) -> str:
    """导出报告"""
    return f"已导出 {format} 报告"

python_repl = PythonREPLTool()

data_tools = [
    load_data, analyze_data, create_visualization,
    export_report, python_repl
]

# 创建数据分析 Agent
llm = ChatOpenAI(model="gpt-4", temperature=0)

prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个数据分析专家。
可以执行：
- 加载和查看数据
- 数据清洗和转换
- 统计分析
- 创建可视化图表
- 生成分析报告
- 执行 Python 代码"""),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

agent = create_openai_functions_agent(llm, data_tools, prompt)

data_agent_executor = AgentExecutor(
    agent=agent,
    tools=data_tools,
    verbose=True,
    max_iterations=20
)

# 执行分析
result = data_agent_executor.invoke({
    "input": "加载 sales.csv，分析销售趋势并创建可视化报告"
})
```

## 8.9 调试和监控

### 8.9.1 使用回调

```python
from langchain.callbacks import StdOutCallbackHandler

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    callbacks=[StdOutCallbackHandler()]
)

result = agent_executor.invoke({"input": "..."})
```

### 8.9.2 查看中间步骤

```python
# 启用详细输出
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    return_intermediate_steps=True
)

result = agent_executor.invoke({"input": "..."})

# 打印所有中间步骤
for i, (action, observation) in enumerate(result["intermediate_steps"]):
    print(f"\n=== 步骤 {i+1} ===")
    print(f"行动: {action}")
    print(f"观察: {observation}")
```

## 8.10 总结

本章介绍了 LangChain 中 Agents 的各种用法：

1. **Agent 类型**: ReAct、OpenAI Functions、XML、Structured Chat
2. **Tool 绑定**: 自动工具选择、强制使用特定工具
3. **AgentExecutor**: 配置、错误处理、中间步骤
4. **对话式 Agent**: 带记忆的 Agent
5. **自定义 Agent**: 自定义 Agent 类和输出解析器

**最佳实践**：
- 根据模型选择合适的 Agent 类型
- 提供清晰、准确的工具描述
- 设置合理的 max_iterations 防止无限循环
- 使用记忆组件实现多轮对话
- 添加适当的错误处理

---

*下一章我们将学习 Callbacks - 回调机制，实现执行过程的监控和日志记录。*
