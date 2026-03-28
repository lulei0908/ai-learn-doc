# 第四章：多模态应用

## 4.1 多模态概述

LangChain 支持多种模态的处理，包括图像、视频、音频等。

```python
from langchain_openai import ChatOpenAI

# 支持多模态的模型
llm = ChatOpenAI(model="gpt-4-vision-preview")

# 图像理解
from langchain_core.messages import HumanMessage
import base64

def encode_image(image_path):
    with open(image_path, "rb") as f:
        return base64.b64encode(f.read()).decode()

image_message = {
    "type": "image_url",
    "image_url": {
        "url": f"data:image/jpeg;base64,{encode_image('image.jpg')}"
    }
}

messages = [
    HumanMessage(content=[
        {"type": "text", "text": "描述这张图片"},
        image_message
    ])
]

response = llm.invoke(messages)
print(response.content)
```

## 4.2 总结

本章介绍了多模态应用：
- 图像理解
- 多模态输入处理
