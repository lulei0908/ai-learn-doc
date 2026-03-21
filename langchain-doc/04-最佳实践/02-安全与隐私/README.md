# 第二章：安全与隐私

## 2.1 输入验证

```python
from langchain_core.prompts import PromptTemplate
import re

def sanitize_input(text: str) -> str:
    """清理输入"""
    # 移除危险字符
    text = re.sub(r'[<>"\']', '', text)
    # 限制长度
    return text[:1000]

# 在提示词中使用
safe_prompt = PromptTemplate.from_template(
    "基于以下内容回答：{input}",
    input_variables=["input"],
    partial_variables={"input": sanitize_input(user_input)}
)
```

## 2.2 数据脱敏

```python
import re

def redact_pii(text: str) -> str:
    """脱敏个人信息"""
    # 手机号
    text = re.sub(r'1[3-9]\d{9}', '****', text)
    # 邮箱
    text = re.sub(r'\b[\w.-]+@[\w.-]+\.\w+\b', '****@***.com', text)
    # 身份证
    text = re.sub(r'\d{17}[\dXx]', '****', text)
    return text
```

## 2.3 权限控制

```python
# 工具权限控制
RESTRICTED_TOOLS = ["delete", "execute", "modify"]

def check_permission(user: str, tool_name: str) -> bool:
    if tool_name in RESTRICTED_TOOLS:
        return user in ["admin"]
    return True
```

## 2.4 总结

安全最佳实践：
1. **输入验证**：清理和限制输入
2. **数据脱敏**：保护敏感信息
3. **权限控制**：限制危险操作
