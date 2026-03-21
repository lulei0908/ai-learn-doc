# 第四章：部署与运维

## 4.1 Docker 部署

```dockerfile
# Dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    volumes:
      - ./data:/app/data
```

## 4.2 API 服务

```python
from fastapi import FastAPI
from langchain_openai import ChatOpenAI

app = FastAPI()

@app.post("/chat")
async def chat(message: str):
    llm = ChatOpenAI(model="gpt-4")
    response = llm.invoke(message)
    return {"response": response.content}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## 4.3 监控与告警

```python
# 监控回调
from langchain_core.callbacks import BaseCallbackHandler

class MetricsCallback(BaseCallbackHandler):
    def __init__(self):
        self.request_count = 0
        self.error_count = 0
    
    def on_llm_start(self, *args, **kwargs):
        self.request_count += 1
    
    def on_llm_error(self, *args, **kwargs):
        self.error_count += 1
```

## 4.4 总结

部署与运维：
1. **Docker**：容器化部署
2. **API**：FastAPI 服务
3. **监控**：指标收集和告警
