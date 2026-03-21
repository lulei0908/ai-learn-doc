# 第一章：性能优化

## 1.1 缓存策略

### 1.1.1 LLM 响应缓存

```python
from langchain.cache import InMemoryCache
from langchain_openai import ChatOpenAI
import langchain

# 启用缓存
langchain.llm_cache = InMemoryCache()

llm = ChatOpenAI(model="gpt-4", temperature=0)

# 相同输入会被缓存
response1 = llm.invoke("What is AI?")
response2 = llm.invoke("What is AI?")  # 从缓存获取
```

### 1.1.2 嵌入缓存

```python
from langchain.embeddings import CacheBackedEmbeddings
from langchain_community.vectorstores import FAISS

# 创建带缓存的嵌入
cache_embeddings = CacheBackedEmbeddings.from_bytes_store(
    underlying_embeddings=embeddings,
    document_embedding_cache=InMemoryCache(),
    namespace=embeddings.model
)

# 使用缓存的嵌入创建向量存储
vectorstore = FAISS.from_documents(
    documents, 
    cache_embeddings
)
```

## 1.2 并发处理

```python
import asyncio
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", temperature=0)

async def process_batch(queries: list):
    """批量异步处理"""
    tasks = [llm.ainvoke(q) for q in queries]
    results = await asyncio.gather(*tasks)
    return results

# 使用
results = asyncio.run(process_batch(["Query 1", "Query 2", "Query 3"]))
```

## 1.3 流式输出

```python
# 流式输出减少等待时间
for chunk in llm.stream("Write a long story"):
    print(chunk.content, end="", flush=True)
```

## 1.4 总结

性能优化策略：
1. **缓存**：使用 LLM 缓存和嵌入缓存
2. **并发**：批量异步处理
3. **流式**：使用流式输出提升体验
