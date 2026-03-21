# 第六章：Tools - 工具调用

## 6.1 Tools 概述

Tools 是 LangChain 中让 LLM 能够执行外部操作的核心组件。通过工具，模型可以与外部世界交互，如搜索网络、执行代码、查询数据库等。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        LangChain Tools 架构                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                         LLM                                  │  │
│   └──────────────────────────┬──────────────────────────────────┘  │
│                              │                                      │
│                    ┌─────────▼─────────┐                           │
│                    │   Tool Binding    │                           │
│                    └─────────┬─────────┘                           │
│                              │                                      │
│   ┌──────────────────────────┼──────────────────────────────────┐  │
│   │                          │                                   │  │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐         │  │
│   │  │ Search  │  │Calculator│  │ Weather │  │ Database│         │  │
│   │  │  Tool   │  │  Tool   │  │  Tool   │  │  Tool   │         │  │
│   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘         │  │
│   │                                                              │  │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐         │  │
│   │  │   API   │  │  Code   │  │  File   │  │ Custom  │         │  │
│   │  │  Tool   │  │ Executor│  │  Tool   │  │  Tool   │         │  │
│   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘         │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 6.2 工具定义

### 6.2.1 使用 @tool 装饰器

```python
from langchain_core.tools import tool

@tool
def calculator(expression: str) -> str:
    """
    执行数学计算。
    
    Args:
        expression: 数学表达式，如 "2 + 3 * 4"
    
    Returns:
        计算结果
    """
    try:
        result = eval(expression)
        return str(result)
    except Exception as e:
        return f"计算错误: {e}"

@tool
def get_word_length(word: str) -> int:
    """返回单词的长度。"""
    return len(word)

# 查看工具信息
print(calculator.name)        # calculator
print(calculator.description) # 执行数学计算...
print(calculator.args)        # {'expression': {'type': 'string', ...}}
```

### 6.2.2 使用 Tool 类

```python
from langchain.tools import Tool

def search_function(query: str) -> str:
    """搜索函数"""
    # 这里可以是实际的搜索逻辑
    return f"搜索结果：{query}"

# 创建工具
search_tool = Tool(
    name="search",
    description="搜索互联网获取信息",
    func=search_function
)

# 使用
result = search_tool.invoke("Python")
print(result)
```

### 6.2.3 使用 StructuredTool

```python
from langchain.tools import StructuredTool
from pydantic import BaseModel, Field

class CalculatorInput(BaseModel):
    """计算器输入参数"""
    a: float = Field(description="第一个数字")
    b: float = Field(description="第二个数字")
    operation: str = Field(description="运算类型：add, subtract, multiply, divide")

def calculator_func(a: float, b: float, operation: str) -> str:
    """执行基本数学运算"""
    operations = {
        "add": a + b,
        "subtract": a - b,
        "multiply": a * b,
        "divide": a / b if b != 0 else "错误：除数不能为0"
    }
    return str(operations.get(operation, "未知操作"))

# 创建结构化工具
calculator_tool = StructuredTool(
    name="calculator",
    description="执行基本数学运算",
    func=calculator_func,
    args_schema=CalculatorInput
)

# 使用
result = calculator_tool.invoke({"a": 10, "b": 5, "operation": "divide"})
print(result)  # 2.0
```

## 6.3 内置工具

### 6.3.1 搜索工具

```python
from langchain_community.tools import DuckDuckGoSearchRun, GoogleSearchAPIWrapper
from langchain.tools import Tool

# DuckDuckGo 搜索（无需 API Key）
search = DuckDuckGoSearchRun()
result = search.invoke("LangChain 是什么")
print(result)

# 封装为 Tool
search_tool = Tool(
    name="web_search",
    description="搜索互联网获取信息",
    func=search.run
)
```

### 6.3.2 维基百科工具

```python
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper

# 维基百科搜索
wikipedia = WikipediaQueryRun(api_wrapper=WikipediaAPIWrapper())
result = wikipedia.invoke("Python programming language")
print(result)
```

### 6.3.3 Shell 命令工具

```python
from langchain.tools import ShellTool

shell = ShellTool()

# 执行命令
result = shell.invoke({"commands": ["echo 'Hello'", "ls -la"]})
print(result)
```

### 6.3.4 Python REPL 工具

```python
from langchain_experimental.tools import PythonREPLTool

python_repl = PythonREPLTool()

# 执行 Python 代码
code = """
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print([fibonacci(i) for i in range(10)])
"""

result = python_repl.invoke(code)
print(result)
```

### 6.3.5 常用工具列表

| 工具名称 | 功能描述 | 安装依赖 |
|:---|:---|:---|
| DuckDuckGoSearchRun | 网络搜索 | `duckduckgo-search` |
| WikipediaQueryRun | 维基百科查询 | `wikipedia` |
| PythonREPLTool | 执行 Python 代码 | `langchain-experimental` |
| ShellTool | 执行 Shell 命令 | 内置 |
| HumanInputRun | 人工输入 | 内置 |
| SerperDevTool | Google 搜索 API | `google-serp-api` |
| BingSearchRun | Bing 搜索 | `bing-search` |

## 6.4 工具绑定

### 6.4.1 绑定工具到模型

```python
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool

@tool
def get_weather(city: str) -> str:
    """获取指定城市的天气信息"""
    # 模拟天气 API
    weather_data = {
        "北京": "晴天，25°C",
        "上海": "多云，22°C",
        "广州": "小雨，28°C"
    }
    return weather_data.get(city, f"未找到{city}的天气信息")

@tool
def get_stock_price(symbol: str) -> str:
    """获取股票价格"""
    # 模拟股票价格
    return f"{symbol} 当前价格: $100.00"

# 初始化模型并绑定工具
llm = ChatOpenAI(model="gpt-4", temperature=0)
llm_with_tools = llm.bind_tools([get_weather, get_stock_price])

# 调用
response = llm_with_tools.invoke("北京今天天气怎么样？")

# 检查工具调用
if response.tool_calls:
    for tool_call in response.tool_calls:
        print(f"工具名称: {tool_call['name']}")
        print(f"工具参数: {tool_call['args']}")
```

### 6.4.2 处理工具调用结果

```python
from langchain_core.messages import HumanMessage, ToolMessage

# 完整的工具调用流程
messages = [HumanMessage(content="北京和上海今天天气怎么样？")]

# 第一步：模型决定调用工具
response = llm_with_tools.invoke(messages)
messages.append(response)

# 第二步：执行工具调用
for tool_call in response.tool_calls:
    # 选择工具
    if tool_call["name"] == "get_weather":
        tool_output = get_weather.invoke(tool_call["args"])
    elif tool_call["name"] == "get_stock_price":
        tool_output = get_stock_price.invoke(tool_call["args"])
    
    # 添加工具结果到消息
    messages.append(ToolMessage(
        content=tool_output,
        tool_call_id=tool_call["id"]
    ))

# 第三步：模型生成最终回答
final_response = llm_with_tools.invoke(messages)
print(final_response.content)
```

## 6.5 自定义工具

### 6.5.1 API 调用工具

```python
from langchain_core.tools import tool
import requests

@tool
def get_github_user_info(username: str) -> str:
    """
    获取 GitHub 用户信息。
    
    Args:
        username: GitHub 用户名
    
    Returns:
        用户信息 JSON 字符串
    """
    url = f"https://api.github.com/users/{username}"
    response = requests.get(url)
    
    if response.status_code == 200:
        data = response.json()
        return f"用户: {data['login']}\n姓名: {data.get('name', 'N/A')}\n公司: {data.get('company', 'N/A')}\n简介: {data.get('bio', 'N/A')}"
    else:
        return f"获取用户信息失败: {response.status_code}"

@tool
def get_crypto_price(symbol: str) -> str:
    """
    获取加密货币价格。
    
    Args:
        symbol: 加密货币符号，如 BTC, ETH
    """
    url = f"https://api.coingecko.com/api/v3/simple/price?ids={symbol.lower()}&vs_currencies=usd"
    response = requests.get(url)
    
    if response.status_code == 200:
        data = response.json()
        if symbol.lower() in data:
            return f"{symbol} 当前价格: ${data[symbol.lower()]['usd']}"
    return "无法获取价格信息"
```

### 6.5.2 数据库查询工具

```python
from langchain_core.tools import tool
from langchain_community.utilities import SQLDatabase
from pydantic import BaseModel, Field

class DatabaseQueryInput(BaseModel):
    """数据库查询输入"""
    query: str = Field(description="SQL 查询语句")

def create_database_tool(connection_string: str):
    """创建数据库查询工具"""
    db = SQLDatabase.from_uri(connection_string)
    
    @tool("database_query", args_schema=DatabaseQueryInput)
    def query_database(query: str) -> str:
        """
        执行 SQL 查询并返回结果。
        只支持 SELECT 查询，不允许修改数据。
        """
        # 安全检查
        if not query.strip().upper().startswith("SELECT"):
            return "只允许执行 SELECT 查询"
        
        try:
            result = db.run(query)
            return str(result)
        except Exception as e:
            return f"查询错误: {e}"
    
    return query_database

# 使用
# db_tool = create_database_tool("sqlite:///mydb.db")
# result = db_tool.invoke({"query": "SELECT * FROM users LIMIT 5"})
```

### 6.5.3 文件操作工具

```python
from langchain_core.tools import tool
from pathlib import Path
from typing import Optional

@tool
def read_file(file_path: str) -> str:
    """
    读取文件内容。
    
    Args:
        file_path: 文件路径
    """
    try:
        path = Path(file_path)
        if not path.exists():
            return f"文件不存在: {file_path}"
        return path.read_text(encoding="utf-8")
    except Exception as e:
        return f"读取错误: {e}"

@tool
def write_file(file_path: str, content: str) -> str:
    """
    写入文件内容。
    
    Args:
        file_path: 文件路径
        content: 文件内容
    """
    try:
        path = Path(file_path)
        path.parent.mkdir(parents=True, exist_ok=True)
        path.write_text(content, encoding="utf-8")
        return f"成功写入: {file_path}"
    except Exception as e:
        return f"写入错误: {e}"

@tool
def list_directory(directory: str) -> str:
    """
    列出目录内容。
    
    Args:
        directory: 目录路径
    """
    try:
        path = Path(directory)
        if not path.is_dir():
            return f"不是目录: {directory}"
        
        items = []
        for item in path.iterdir():
            item_type = "📁" if item.is_dir() else "📄"
            items.append(f"{item_type} {item.name}")
        
        return "\n".join(items)
    except Exception as e:
        return f"错误: {e}"
```

## 6.6 多工具组合

### 6.6.1 工具集合

```python
from langchain.tools import Tool

# 创建工具集合
tools = [
    Tool(
        name="calculator",
        description="执行数学计算。输入数学表达式，返回计算结果。",
        func=lambda x: str(eval(x))
    ),
    Tool(
        name="search",
        description="搜索网络获取信息。输入搜索关键词，返回搜索结果。",
        func=lambda x: f"搜索结果: {x}"
    ),
    Tool(
        name="weather",
        description="查询天气。输入城市名称，返回天气信息。",
        func=lambda x: f"{x}天气: 晴天，25°C"
    )
]

print(f"可用工具数量: {len(tools)}")
for tool in tools:
    print(f"- {tool.name}: {tool.description}")
```

### 6.6.2 工具选择器

```python
from langchain_core.tools import ToolException
from typing import Dict, Any

class ToolRegistry:
    """工具注册中心"""
    
    def __init__(self):
        self._tools: Dict[str, Any] = {}
    
    def register(self, tool):
        """注册工具"""
        self._tools[tool.name] = tool
    
    def get(self, name: str):
        """获取工具"""
        return self._tools.get(name)
    
    def list_tools(self):
        """列出所有工具"""
        return list(self._tools.keys())
    
    def execute(self, name: str, *args, **kwargs):
        """执行工具"""
        tool = self.get(name)
        if tool is None:
            raise ToolException(f"工具不存在: {name}")
        return tool.invoke(*args, **kwargs)

# 使用
registry = ToolRegistry()
registry.register(calculator)
registry.register(search_tool)

print(registry.list_tools())
```

## 6.7 工具调用模式

### 6.7.1 ReAct 模式

```python
from langchain.agents import AgentExecutor, create_react_agent
from langchain_openai import ChatOpenAI
from langchain import hub

# 工具列表
tools = [calculator, search_tool]

# ReAct 提示词模板
prompt = hub.pull("hwchase17/react")

# 创建 Agent
llm = ChatOpenAI(model="gpt-4", temperature=0)
agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# 执行
result = agent_executor.invoke({
    "input": "搜索 LangChain 的信息，然后计算其名字的字符长度"
})
```

### 6.7.2 OpenAI Functions 模式

```python
from langchain.agents import AgentExecutor, create_openai_functions_agent

# OpenAI Functions Agent
llm = ChatOpenAI(model="gpt-4", temperature=0)
prompt = hub.pull("hwchase17/openai-functions-agent")

agent = create_openai_functions_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    max_iterations=5
)

result = agent_executor.invoke({"input": "计算 123 * 456"})
```

### 6.7.3 自定义工具调用流程

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough

def execute_tool(tool_name: str, tool_args: dict) -> str:
    """执行工具并返回结果"""
    tool_map = {
        "calculator": calculator,
        "get_weather": get_weather,
        "search": search_tool
    }
    
    tool = tool_map.get(tool_name)
    if tool:
        return tool.invoke(tool_args)
    return f"未知工具: {tool_name}"

# 创建工具调用链
tool_chain = (
    ChatPromptTemplate.from_messages([
        ("system", "你是一个有帮助的助手，可以使用工具。"),
        ("human", "{input}")
    ])
    | llm_with_tools
)

# 执行
response = tool_chain.invoke({"input": "北京天气怎么样？"})

# 处理工具调用
if response.tool_calls:
    for tc in response.tool_calls:
        result = execute_tool(tc["name"], tc["args"])
        print(f"工具 {tc['name']} 结果: {result}")
```

## 6.8 完整示例

### 6.8.1 智能助手

```python
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain.memory import ConversationBufferMemory

# 定义工具
@tool
def calculate(expression: str) -> str:
    """执行数学计算"""
    try:
        return str(eval(expression))
    except Exception as e:
        return f"计算错误: {e}"

@tool
def search_web(query: str) -> str:
    """搜索网络获取信息"""
    # 实际应用中接入真实搜索 API
    return f"关于 '{query}' 的搜索结果..."

@tool
def get_current_time() -> str:
    """获取当前时间"""
    from datetime import datetime
    return datetime.now().strftime("%Y-%m-%d %H:%M:%S")

@tool
def translate(text: str, target_lang: str) -> str:
    """翻译文本"""
    # 实际应用中接入翻译 API
    return f"翻译结果 ({target_lang}): {text}"

# 创建 Agent
llm = ChatOpenAI(model="gpt-4", temperature=0)
tools = [calculate, search_web, get_current_time, translate]

prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个智能助手，可以使用多种工具帮助用户。"),
    MessagesPlaceholder(variable_name="chat_history"),
    ("human", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad")
])

memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

agent = create_openai_functions_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    memory=memory,
    verbose=True
)

# 使用
response = agent_executor.invoke({"input": "现在几点了？"})
print(response["output"])

response = agent_executor.invoke({"input": "帮我计算 25 * 4 + 100"})
print(response["output"])
```

### 6.8.2 数据分析助手

```python
from langchain_core.tools import tool
from langchain_experimental.tools import PythonREPLTool
import pandas as pd
import json

@tool
def analyze_csv(file_path: str, operation: str) -> str:
    """
    分析 CSV 文件。
    
    Args:
        file_path: CSV 文件路径
        operation: 操作类型 - summary, head, columns, describe
    """
    try:
        df = pd.read_csv(file_path)
        
        if operation == "summary":
            return f"行数: {len(df)}, 列数: {len(df.columns)}"
        elif operation == "head":
            return df.head().to_string()
        elif operation == "columns":
            return json.dumps(df.columns.tolist())
        elif operation == "describe":
            return df.describe().to_string()
        else:
            return f"未知操作: {operation}"
    except Exception as e:
        return f"分析错误: {e}"

@tool
def create_chart(data: str, chart_type: str, title: str) -> str:
    """
    创建图表。
    
    Args:
        data: JSON 格式的数据
        chart_type: 图表类型 - bar, line, pie
        title: 图表标题
    """
    # 实际应用中实现图表生成
    return f"已创建 {chart_type} 图表: {title}"

# 组合工具
analysis_tools = [analyze_csv, create_chart, PythonREPLTool()]
```

## 6.9 工具调试与错误处理

### 6.9.1 错误处理

```python
from langchain_core.tools import ToolException
from langchain.tools import Tool

def safe_tool_invoke(tool, *args, **kwargs):
    """安全的工具调用"""
    try:
        result = tool.invoke(*args, **kwargs)
        return {"success": True, "result": result}
    except ToolException as e:
        return {"success": False, "error": str(e)}
    except Exception as e:
        return {"success": False, "error": f"未知错误: {e}"}

# 使用
result = safe_tool_invoke(calculator, {"expression": "1/0"})
if not result["success"]:
    print(f"工具调用失败: {result['error']}")
```

### 6.9.2 工具调试

```python
from langchain.callbacks import StdOutCallbackHandler

# 使用回调查看工具执行过程
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    callbacks=[StdOutCallbackHandler()],
    verbose=True
)
```

## 6.10 总结

本章介绍了 LangChain 中 Tools 的各种用法：

1. **工具定义**: @tool 装饰器、Tool 类、StructuredTool
2. **内置工具**: 搜索、维基百科、Shell、Python REPL
3. **自定义工具**: API 调用、数据库查询、文件操作
4. **工具绑定**: bind_tools、处理工具调用结果
5. **工具模式**: ReAct、OpenAI Functions

**最佳实践**：
- 工具描述要清晰准确
- 使用结构化输入验证参数
- 处理工具调用异常
- 限制危险操作权限

---

*下一章我们将学习 Indexes - 索引与检索，构建 RAG 系统的核心组件。*
