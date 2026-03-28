# 第一章：RAG 系统实战

## 1.1 RAG 概述

RAG（Retrieval-Augmented Generation）即检索增强生成，结合了检索系统和 LLM 的能力，通过检索相关文档来增强生成质量。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        RAG 系统架构                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐       │
│   │  Query  │ → │ Retrieve │ → │  Augment │ → │ Generate│       │
│   │  查询    │    │  检索    │    │   增强   │    │   生成  │       │
│   └─────────┘    └─────────┘    └─────────┘    └─────────┘       │
│                                                                      │
│   1. 用户输入 → 2. 检索相关文档 → 3. 构建提示 → 4. 生成回答          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 1.2 基础 RAG 实现

```python
from langchain_community.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain.chains import RetrievalQA
from langchain_core.prompts import PromptTemplate

# 1. 加载文档
loader = TextLoader("./documents/article.txt")
documents = loader.load()

# 2. 分割文档
splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
chunks = splitter.split_documents(documents)

# 3. 创建向量存储
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(chunks, embeddings, persist_directory="./chroma_db")

# 4. 创建检索器
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

# 5. 创建 QA 链
llm = ChatOpenAI(model="gpt-4", temperature=0)
qa_chain = RetrievalQA.from_chain_type(llm=llm, retriever=retriever)

# 6. 查询
result = qa_chain.invoke({"query": "文档主要讲了什么？"})
print(result["result"])
```

## 1.3 LangGraph RAG

```python
from langgraph.graph import StateGraph, END, START
from typing import TypedDict, List
from langgraph.checkpoint.memory import MemorySaver

class RAGState(TypedDict):
    question: str
    retrieved_docs: List
    context: str
    answer: str

def retrieve(state: RAGState) -> dict:
    """检索相关文档"""
    docs = retriever.get_relevant_documents(state["question"])
    return {"retrieved_docs": docs}

def generate(state: RAGState) -> dict:
    """生成回答"""
    docs = state["retrieved_docs"]
    context = "\n".join([doc.page_content for doc in docs])
    
    prompt = f"""基于以下上下文回答问题：
    
    上下文：
    {context}
    
    问题：{state['question']}
    
    回答："""
    
    response = llm.invoke(prompt)
    return {"context": context, "answer": response.content}

workflow = StateGraph(RAGState)
workflow.add_node("retrieve", retrieve)
workflow.add_node("generate", generate)
workflow.add_edge(START, "retrieve")
workflow.add_edge("retrieve", "generate")
workflow.add_edge("generate", END)

app = workflow.compile(checkpointer=MemorySaver())

result = app.invoke({"question": "LangChain 是什么？", "retrieved_docs": [], "context": "", "answer": ""})
print(result["answer"])
```

## 1.4 高级 RAG 特性

### 1.4.1 混合检索

```python
from langchain.retrievers import EnsembleRetriever
from langchain_community.retrievers import BM25Retriever

# 向量检索
vector_retriever = vectorstore.as_retriever(k=3)

# BM25 检索
bm25_retriever = BM25Retriever.from_documents(chunks)

# 混合检索
hybrid_retriever = EnsembleRetriever(
    retrievers=[vector_retriever, bm25_retriever],
    weights=[0.5, 0.5]
)
```

### 1.4.2 查询改写

```python
def rewrite_query(state: RAGState) -> dict:
    """查询改写"""
    original = state["question"]
    prompt = f"将以下问题改写得更清晰准确：\n{original}"
    rewritten = llm.invoke(prompt).content
    return {"question": rewritten}
```

## 1.5 总结

本章介绍了 RAG 系统的实现：

1. **基础 RAG**：文档加载、分割、检索、生成
2. **LangGraph RAG**：使用状态图实现
3. **高级特性**：混合检索、查询改写
