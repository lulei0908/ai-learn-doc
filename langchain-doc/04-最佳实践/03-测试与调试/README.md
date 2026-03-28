# 第三章：测试与调试

## 3.1 单元测试

```python
import pytest
from langchain_openai import ChatOpenAI
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate

def test_chain():
    """测试链"""
    llm = ChatOpenAI(model="gpt-4", temperature=0)
    prompt = PromptTemplate.from_template("Say {word}")
    chain = LLMChain(llm=llm, prompt=prompt)
    
    result = chain.invoke({"word": "hello"})
    assert "hello" in result["text"].lower()

def test_agent():
    """测试 Agent"""
    # 测试 Agent 逻辑
    pass

if __name__ == "__main__":
    pytest.main([__file__])
```

## 3.2 调试技巧

```python
# 启用详细日志
import logging
logging.basicConfig(level=logging.DEBUG)

# 使用回调调试
from langchain_core.callbacks import ConsoleCallbackHandler

llm = ChatOpenAI(model="gpt-4", callbacks=[ConsoleCallbackHandler()])
response = llm.invoke("Hello")
```

## 3.3 总结

测试与调试：
1. **单元测试**：使用 pytest
2. **调试**：回调和日志
