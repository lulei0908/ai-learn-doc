# 第七章：Indexes - 索引与检索

## 7.1 索引概述

Indexes 是构建 RAG（检索增强生成）系统的核心组件，包括文档加载、文本分割、向量存储和检索。

```
┌─────────────────────────────────────────────────────────────────────┐
│                        RAG 系统架构                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    │
│   │  文档    │ → │  分割    │ → │  向量化  │ → │  存储    │    │
│   │ Loader   │    │ Splitter │    │Embedding │    │ Vector  │    │
│   └──────────┘    └──────────┘    └──────────┘    └──────────┘    │
│        ↑                                                │          │
│        │                                                ↓          │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    │
│   │  回答    │ ← │   LLM    │ ← │  上下文  │ ← │  检索    │    │
│   │ Answer   │    │          │    │ Context │    │Retriever│    │
│   └──────────┘    └──────────┘    └──────────┘    └──────────┘    │
│                                        ↑                            │
│                                        │                            │
│                                   ┌──────────┐                      │
│                                   │  查询    │                      │
│                                   │  Query   │                      │
│                                   └──────────┘                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 7.2 文档加载器 (Document Loaders)

### 7.2.1 文本文件加载

```python
from langchain_community.document_loaders import TextLoader

# 加载文本文件
loader = TextLoader("./example.txt", encoding="utf-8")
documents = loader.load()

print(f"文档数量: {len(documents)}")
print(f"内容预览: {documents[0].page_content[:100]}")
print(f"元数据: {documents[0].metadata}")
```

### 7.2.2 PDF 文件加载

```python
from langchain_community.document_loaders import PyPDFLoader

# 加载 PDF
loader = PyPDFLoader("./document.pdf")
pages = loader.load()

print(f"页数: {len(pages)}")
for i, page in enumerate(pages):
    print(f"\n--- 第 {i+1} 页 ---")
    print(page.page_content[:200])
```

### 7.2.3 CSV 文件加载

```python
from langchain_community.document_loaders import CSVLoader

# 加载 CSV
loader = CSVLoader(
    file_path="./data.csv",
    csv_args={
        "delimiter": ",",
        "quotechar": '"'
    }
)
documents = loader.load()

# 每行作为一个文档
print(f"记录数: {len(documents)}")
```

### 7.2.4 Web 页面加载

```python
from langchain_community.document_loaders import WebBaseLoader

# 加载网页
loader = WebBaseLoader("https://python.langchain.com/docs/get_started/introduction")
documents = loader.load()

print(documents[0].page_content[:500])
```

### 7.2.5 目录加载

```python
from langchain_community.document_loaders import DirectoryLoader

# 加载目录下所有文件
loader = DirectoryLoader(
    "./documents",
    glob="**/*.txt",  # 匹配所有 txt 文件
    loader_cls=TextLoader
)
documents = loader.load()

print(f"加载文档数: {len(documents)}")
```

### 7.2.6 常用文档加载器

| 加载器 | 用途 | 依赖包 |
|:---|:---|:---|
| TextLoader | 文本文件 | 内置 |
| PyPDFLoader | PDF 文件 | pypdf |
| Docx2txtLoader | Word 文档 | docx2txt |
| UnstructuredMarkdownLoader | Markdown | unstructured |
| CSVLoader | CSV 文件 | 内置 |
| JSONLoader | JSON 文件 | 内置 |
| WebBaseLoader | 网页 | requests |
| UnstructuredHTMLLoader | HTML | unstructured |
| YoutubeLoader | YouTube 视频 | youtube-transcript-api |
| NotionDirectoryLoader | Notion 导出 | 内置 |

## 7.3 文本分割器 (Text Splitters)

### 7.3.1 字符分割器

```python
from langchain.text_splitter import CharacterTextSplitter

text = "这是一段很长的文本..." * 100

# 字符分割
splitter = CharacterTextSplitter(
    separator="\n\n",  # 分隔符
    chunk_size=100,    # 块大小
    chunk_overlap=20,  # 重叠大小
    length_function=len
)

chunks = splitter.split_text(text)
print(f"分割后块数: {len(chunks)}")
print(f"第一块长度: {len(chunks[0])}")
```

### 7.3.2 递归字符分割器

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

text = """# 标题

这是第一段内容。

## 子标题

这是第二段内容。

```python
print("Hello")
```
"""

# 递归分割（推荐）
splitter = RecursiveCharacterTextSplitter(
    chunk_size=200,
    chunk_overlap=50,
    separators=["\n\n", "\n", "。", "！", "？", " ", ""]
)

chunks = splitter.split_text(text)
for i, chunk in enumerate(chunks):
    print(f"\n--- 块 {i+1} ---")
    print(chunk)
```

### 7.3.3 代码分割器

```python
from langchain.text_splitter import PythonCodeTextSplitter, RecursiveCharacterTextSplitter

code = """
def function_one():
    '''第一个函数'''
    print("Function 1")
    return 1

def function_two():
    '''第二个函数'''
    print("Function 2")
    return 2

class MyClass:
    def __init__(self):
        self.value = 0
    
    def increment(self):
        self.value += 1
        return self.value
"""

# Python 代码分割
splitter = RecursiveCharacterTextSplitter.from_language(
    language="python",
    chunk_size=100,
    chunk_overlap=20
)

chunks = splitter.split_text(code)
for chunk in chunks:
    print(chunk)
    print("---")
```

### 7.3.4 Markdown 分割器

```python
from langchain.text_splitter import MarkdownHeaderTextSplitter

markdown_text = """
# 主标题

这是主标题下的内容。

## 二级标题 A

这是二级标题 A 的内容。

### 三级标题

三级标题的内容。

## 二级标题 B

二级标题 B 的内容。
"""

# 按 Markdown 标题分割
headers_to_split_on = [
    ("#", "header1"),
    ("##", "header2"),
    ("###", "header3")
]

splitter = MarkdownHeaderTextSplitter(headers_to_split_on=headers_to_split_on)
chunks = splitter.split_text(markdown_text)

for chunk in chunks:
    print(f"元数据: {chunk.metadata}")
    print(f"内容: {chunk.page_content}")
    print("---")
```

### 7.3.5 按 Token 分割

```python
from langchain.text_splitter import TokenTextSplitter

text = "这是一段很长的文本..." * 1000

# 按 Token 数量分割
splitter = TokenTextSplitter(
    encoding_name="cl100k_base",  # GPT-4 编码
    chunk_size=100,               # 每 100 个 token 一块
    chunk_overlap=20
)

chunks = splitter.split_text(text)
print(f"分割后块数: {len(chunks)}")
```

### 7.3.6 文档分割

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import TextLoader

# 加载文档
loader = TextLoader("./document.txt")
documents = loader.load()

# 分割文档
splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)

splits = splitter.split_documents(documents)
print(f"原始文档数: {len(documents)}")
print(f"分割后块数: {len(splits)}")

# 查看分割后的文档
for i, split in enumerate(splits[:3]):
    print(f"\n--- 块 {i+1} ---")
    print(f"内容长度: {len(split.page_content)}")
    print(f"元数据: {split.metadata}")
```

## 7.4 向量存储 (Vector Stores)

### 7.4.1 使用 Chroma

```python
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter

# 准备文档
texts = [
    "LangChain 是一个用于构建 LLM 应用的框架",
    "LangGraph 是 LangChain 的扩展，支持状态机工作流",
    "RAG 是检索增强生成的缩写",
    "向量数据库用于存储文本的向量表示"
]

# 创建向量存储
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_texts(
    texts=texts,
    embedding=embeddings,
    persist_directory="./chroma_db"  # 持久化目录
)

# 相似性搜索
results = vectorstore.similarity_search("什么是 LangChain", k=2)
for doc in results:
    print(doc.page_content)
```

### 7.4.2 使用 FAISS

```python
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings

# 创建 FAISS 向量存储
texts = ["文本1", "文本2", "文本3"]
embeddings = OpenAIEmbeddings()

vectorstore = FAISS.from_texts(texts, embeddings)

# 保存到本地
vectorstore.save_local("./faiss_index")

# 从本地加载
loaded_vectorstore = FAISS.load_local(
    "./faiss_index",
    embeddings,
    allow_dangerous_deserialization=True
)

# 搜索
results = loaded_vectorstore.similarity_search("查询", k=1)
```

### 7.4.3 使用 Pinecone

```python
from langchain_community.vectorstores import Pinecone
from langchain_openai import OpenAIEmbeddings
import pinecone

# 初始化 Pinecone
pinecone.init(
    api_key="your-api-key",
    environment="us-west1-gcp"
)

# 创建向量存储
embeddings = OpenAIEmbeddings()
vectorstore = Pinecone.from_texts(
    texts=["文本1", "文本2"],
    embedding=embeddings,
    index_name="my-index"
)

# 搜索
results = vectorstore.similarity_search("查询", k=3)
```

### 7.4.4 常用向量存储

| 向量存储 | 特点 | 适用场景 |
|:---|:---|:---|
| Chroma | 开源、本地、易用 | 开发、小型项目 |
| FAISS | Meta 开源、高性能 | 本地、大规模 |
| Pinecone | 云托管、可扩展 | 生产环境 |
| Weaviate | 开源、功能丰富 | 企业级应用 |
| Milvus | 开源、高性能 | 大规模生产 |
| Qdrant | 开源、过滤功能强 | 需要元数据过滤 |

## 7.5 Embedding 模型

### 7.5.1 OpenAI Embeddings

```python
from langchain_openai import OpenAIEmbeddings

# 初始化
embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small"  # 或 text-embedding-3-large
)

# 嵌入文本
text = "这是一段测试文本"
embedding = embeddings.embed_query(text)
print(f"向量维度: {len(embedding)}")
print(f"前10个值: {embedding[:10]}")

# 批量嵌入
texts = ["文本1", "文本2", "文本3"]
embeddings_list = embeddings.embed_documents(texts)
print(f"嵌入数量: {len(embeddings_list)}")
```

### 7.5.2 HuggingFace Embeddings

```python
from langchain_community.embeddings import HuggingFaceEmbeddings

# 使用本地模型
embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2",
    model_kwargs={'device': 'cpu'},
    encode_kwargs={'normalize_embeddings': True}
)

# 嵌入
embedding = embeddings.embed_query("测试文本")
print(f"向量维度: {len(embedding)}")
```

### 7.5.3 其他 Embedding 选项

```python
# Cohere
from langchain_community.embeddings import CohereEmbeddings
embeddings = CohereEmbeddings(model="embed-english-v3.0")

# Google
from langchain_community.embeddings import GooglePalmEmbeddings
embeddings = GooglePalmEmbeddings()

# Azure OpenAI
from langchain_openai import AzureOpenAIEmbeddings
embeddings = AzureOpenAIEmbeddings(
    azure_deployment="text-embedding-ada-002"
)
```

## 7.6 检索器 (Retrievers)

### 7.6.1 基础检索器

```python
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings

# 创建向量存储
texts = ["文档1内容", "文档2内容", "文档3内容"]
vectorstore = Chroma.from_texts(texts, OpenAIEmbeddings())

# 获取检索器
retriever = vectorstore.as_retriever(
    search_type="similarity",  # 相似性搜索
    search_kwargs={"k": 3}     # 返回3个结果
)

# 检索
docs = retriever.invoke("查询内容")
for doc in docs:
    print(doc.page_content)
```

### 7.6.2 MMR 检索器

```python
# 最大边际相关性检索（增加多样性）
retriever = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 3,           # 返回3个结果
        "fetch_k": 10,    # 候选集大小
        "lambda_mult": 0.5  # 多样性权重
    }
)

docs = retriever.invoke("查询内容")
```

### 7.6.3 相似度阈值检索器

```python
# 设置相似度阈值
retriever = vectorstore.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={
        "k": 5,
        "score_threshold": 0.8  # 只返回相似度 > 0.8 的结果
    }
)

docs = retriever.invoke("查询内容")
```

### 7.6.4 多查询检索器

```python
from langchain.retrievers.multi_query import MultiQueryRetriever
from langchain_openai import ChatOpenAI

# 创建多查询检索器
llm = ChatOpenAI(model="gpt-4", temperature=0)
retriever = MultiQueryRetriever.from_llm(
    retriever=vectorstore.as_retriever(),
    llm=llm
)

# 执行检索（自动生成多个查询变体）
docs = retriever.invoke("LangChain 是什么")
```

### 7.6.5 上下文压缩检索器

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import LLMChainExtractor
from langchain_openai import ChatOpenAI

# 创建压缩器
llm = ChatOpenAI(model="gpt-4", temperature=0)
compressor = LLMChainExtractor.from_llm(llm)

# 创建压缩检索器
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=vectorstore.as_retriever()
)

# 检索（会自动压缩文档）
docs = compression_retriever.invoke("查询内容")
```

### 7.6.6 自定义检索器

```python
from langchain_core.retrievers import BaseRetriever
from langchain_core.documents import Document
from typing import List

class CustomRetriever(BaseRetriever):
    """自定义检索器"""
    
    def __init__(self, documents: List[Document]):
        super().__init__()
        self._documents = documents
    
    def _get_relevant_documents(self, query: str) -> List[Document]:
        """实现检索逻辑"""
        # 简单的关键词匹配
        relevant = []
        for doc in self._documents:
            if query.lower() in doc.page_content.lower():
                relevant.append(doc)
        return relevant

# 使用
docs = [Document(page_content="LangChain 教程"), Document(page_content="Python 编程")]
retriever = CustomRetriever(docs)
results = retriever.invoke("LangChain")
```

## 7.7 完整 RAG 示例

### 7.7.1 文档问答系统

```python
from langchain_community.document_loaders import TextLoader, PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain.chains import RetrievalQA
from langchain_core.prompts import PromptTemplate

# 1. 加载文档
loader = PyPDFLoader("./document.pdf")
documents = loader.load()

# 2. 分割文本
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    separators=["\n\n", "\n", "。", " "]
)
splits = text_splitter.split_documents(documents)

# 3. 创建向量存储
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(
    documents=splits,
    embedding=embeddings,
    persist_directory="./chroma_db"
)

# 4. 创建检索器
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 4}
)

# 5. 创建问答链
llm = ChatOpenAI(model="gpt-4", temperature=0)

prompt_template = """基于以下上下文回答问题。如果上下文中没有相关信息，请说明不知道。

上下文：
{context}

问题：{question}

回答："""

PROMPT = PromptTemplate(
    template=prompt_template,
    input_variables=["context", "question"]
)

qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=retriever,
    return_source_documents=True,
    chain_type_kwargs={"prompt": PROMPT}
)

# 6. 问答
result = qa_chain.invoke({"query": "文档中提到了哪些主要内容？"})
print(f"回答: {result['result']}")
print(f"\n来源文档:")
for doc in result["source_documents"]:
    print(f"- {doc.metadata}")
```

### 7.7.2 对话式 RAG

```python
from langchain.chains import ConversationalRetrievalChain
from langchain.memory import ConversationBufferMemory
from langchain_openai import ChatOpenAI

# 创建对话 RAG
llm = ChatOpenAI(model="gpt-4", temperature=0)
memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True,
    output_key="answer"
)

qa = ConversationalRetrievalChain.from_llm(
    llm=llm,
    retriever=retriever,
    memory=memory,
    return_source_documents=True,
    verbose=True
)

# 对话
response1 = qa.invoke({"question": "文档主要内容是什么？"})
print(response1["answer"])

response2 = qa.invoke({"question": "能详细解释第一点吗？"})
print(response2["answer"])
```

### 7.7.3 混合检索

```python
from langchain.retrievers import EnsembleRetriever
from langchain_community.retrievers import BM25Retriever
from langchain_community.vectorstores import Chroma

# 准备文档
documents = [Document(page_content="内容1"), Document(page_content="内容2")]

# 向量检索器
vector_retriever = Chroma.from_documents(
    documents, OpenAIEmbeddings()
).as_retriever(k=3)

# 关键词检索器 (BM25)
bm25_retriever = BM25Retriever.from_documents(documents)
bm25_retriever.k = 3

# 混合检索器
ensemble_retriever = EnsembleRetriever(
    retrievers=[vector_retriever, bm25_retriever],
    weights=[0.5, 0.5]  # 权重分配
)

# 检索
docs = ensemble_retriever.invoke("查询内容")
```

## 7.8 总结

本章介绍了 LangChain 中 Indexes 的各种组件：

1. **文档加载器**: 从各种来源加载文档
2. **文本分割器**: 将长文本分割成小块
3. **向量存储**: 存储和检索向量
4. **Embedding 模型**: 将文本转换为向量
5. **检索器**: 从向量存储中检索相关文档

**最佳实践**：
- 选择合适的 chunk_size 和 chunk_overlap
- 使用适合语言的分割器
- 根据规模选择向量存储
- 考虑使用混合检索提高准确性

---

*下一章我们将学习 Agents - 代理，让 AI 自主决策和执行任务。*
