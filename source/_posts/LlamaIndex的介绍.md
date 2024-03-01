---
title: LlamaIndex的介绍
date: 2024-03-01 12:04:54
tags:
categories: Python
---

LlamaIndex 是一个基于 LLM 应用程序的数据框架，受益于上下文增强。它提供了必要的抽象，可以更轻松地摄取、构建和访问私有或特定领域的数据，以便将这些数据安全可靠地注入 LLM 中，以实现更准确的文本生成。

<!--more-->

LlamaIndex 可以在 Python 和 Typescript 中使用。

为什么要进行上下文增强呢？
LLM 在人类和数据之间提供自然语言接口。广泛可用的模型是根据大量公开数据（如维基百科、邮件列表、教科书、源代码等）进行预训练的。
然而，虽然 LLM 接受了大量数据的培训，但他们并没有接受你的数据的培训，这些数据可能是私有的或特定于您试图解决的问题。它位于 API 后面、SQL 数据库中，或者隐藏在 PDF 和幻灯片中。
您可以选择用您的数据微调 LLM，但是： 

1. 训练 LLM 的费用很高，
2. 由于培训成本的原因，很难用最新信息更新 LLM，
3. 缺乏可观察性。当你向 LLM 提出问题时，LLM 是如何得出答案的并不明显。

我们可以使用一种称为检索增强生成 (RAG) 的上下文增强模式来代替微调，以获得与您的特定数据相关的更准确的文本生成。 RAG 涉及以下高级步骤：

1. 首先从数据源检索信息
2. 将其作为上下文添加到您的问题中，并且
3. 请 LLM 根据丰富的提示进行回答

通过这样做，RAG 克服了微调方法的三个弱点：

1. 不需要训练 LLM，所以很便宜
2. 仅当您请求数据时才会获取数据，因此数据始终是最新的
3. LlamaIndex可以向您显示检索到的文档，因此更值得信赖

为什么使用 LlamaIndex 进行上下文增强？
首先，LlamaIndex 对您如何使用 LLM 没有任何限制。您仍然可以将 LLM 用作自动完成、聊天机器人、半自主代理等。它只会让 LLM 与您更相关。
LlamaIndex 提供以下工具来帮助您快速建立可用于生产的 RAG 系统：

1. Data Connector 从其本机源和格式获取现有数据。这些可以是 API、PDF、SQL 等等。
2. Data Index 以中间表示形式构建数据，这些中间表示形式对于 LLM 来说既简单又高效
3. Engines 提供对数据的自然语言访问，例如
查询引擎是用于知识增强输出的强大检索接口。
聊天引擎是用于与数据进行多消息、“来回”交互的对话界面。
6. Data Agents 是由 LLM 提供支持的知识工作者，并通过工具进行增强，从简单的辅助函数到 API 集成等。
7. Application integrations 将 LlamaIndex 重新融入生态系统的其余部分。这可以是 LangChain、Flask、Docker、ChatGPT，或者……其他任何东西！

LlamaIndex 适合谁？
1. LlamaIndex 为初学者、高级用户以及介于两者之间的每个人提供工具。 
2. 我们的高级 API 允许初学者使用 LlamaIndex 通过 5 行代码获取和查询他们的数据。
3. 对于更复杂的应用程序，我们的较低级别 API 允许高级用户自定义和扩展任何模块（数据连接器、索引、检索器、查询引擎、重新排名模块）以满足他们的需求。

# LlamaIndex的安装

```python
pip install llama-index
```

这个包中包含了以下包：

```txt
llama-index-core
llama-index-legacy  # temporarily included
llama-index-llms-openai
llama-index-embeddings-openai
llama-index-program-openai
llama-index-question-gen-openai
llama-index-agent-openai
llama-index-readers-file
llama-index-multi-modal-llms-openai
```

LlamaIndex 可以下载并存储各种软件包的本地文件（NLTK、HuggingFace 等）。使用环境变量“LLAMA_INDEX_CACHE_DIR”来控制这些文件的保存位置。

默认情况下，llamaIndex 使用 OpenAI gpt-3.5-turbo 模型进行文本生成，使用 text-embedding-ada-002 进行检索和嵌入。为了使用它，必须将 OPENAI_API_KEY 设置为环境变量。

# LlamaIndex的使用

下面是一个在 LlamaIndex 中使用 Ollama 的例子，需要首先安装 llama-index-llms-ollama：pip install llama-index-llms-ollama。

```python
from llama_index.core import SummaryIndex, SimpleDirectoryReader
from llama_index.core import Settings
from llama_index.llms.ollama import Ollama

# 设置 llm
Settings.llm = Ollama(model="llama2", request_timeout=60.0)

# 加载数据
documents = SimpleDirectoryReader("./data").load_data()
# 根据数据创建index
index = SummaryIndex.from_documents(documents)
# 创建查询引擎
query_engine = index.as_query_engine()
# 查询数据
response = query_engine.query("What did the author do growing up?")
print(response)
```

从上面的代码中可以看出，使用 LlamaIndex 开发的大致步骤如下：
1. 创建一个 LLM 对象
2. 加载数据
3. 根据数据创建索引
4. 创建查询引擎
5. 查询引擎调用接口获取接口

下面具体介绍每个步骤如何执行。

## 如何加载数据

### 从目录中加载数据

最容易使用的 reader 是我们的 SimpleDirectoryReader，它可以根据给定目录中的每个文件创建文档。

```python
from llama_index.core import SimpleDirectoryReader
# 从 data 目录中加载数据
documents = SimpleDirectoryReader("./data").load_data()
```



### 从数据库查询数据

可以使用 DatabaseReader 连接器，该连接器对 SQL 数据库运行查询并将结果的每一行作为文档返回。

```python
import os
from llama_index.core import download_loader
from llama_index.readers.database import DatabaseReader

# 创建数据库连接器
reader = DatabaseReader(
    scheme=os.getenv("DB_SCHEME"),
    host=os.getenv("DB_HOST"),
    port=os.getenv("DB_PORT"),
    user=os.getenv("DB_USER"),
    password=os.getenv("DB_PASS"),
    dbname=os.getenv("DB_NAME"),
)

query = "SELECT * FROM users"
documents = reader.load_data(query=query)
```



## 如何处理数据

加载数据后，您需要处理和转换数据，然后再将其放入存储系统。这些转换包括分块、提取元数据和嵌入每个块。这是确保 LLM 能够检索和最佳使用数据所必需的。
处理和转换输入/输出是 Node 对象（Document 是 Node 的子类），也可以对它们进行堆叠和重新排序。

加载数据的代码：

```python
from llama_index.core import VectorStoreIndex

vector_index = VectorStoreIndex.from_documents(documents)
vector_index.as_query_engine()
```

上面的代码是没有经过文本分割处理的，from_documents() 方法可以接受 Document 对象数组，并正确解析它们并将它们分块。但是，有时您需要更好地控制文档的拆分方式，此时可以自定义文本拆分对象：

```python
# 定义文档拆分对象
text_splitter = SentenceSplitter(chunk_size=512, chunk_overlap=10)

from llama_index.core import Settings
from llama_index.core import VectorStoreIndex

Settings.text_splitter = text_splitter

index = VectorStoreIndex.from_documents(
    documents, transformations=[text_splitter]
```



## 创建索引index

加载数据后，我们已经拥有文档对象列表（或节点列表）。是时候为这些对象构建索引了，以便您可以开始查询它们。
在 LlamaIndex 中，索引是由 Document 对象组成的数据结构，旨在支持 LLM 进行查询。您的索引旨在补充您的查询策略。
LlamaIndex 提供了几种不同的索引类型，包括：VectorStoreIndex、SummaryIndex。

VectorStoreIndex 是最常见的索引类型。矢量存储索引获取您的文档并将它们分成节点。然后，它创建每个节点文本的向量嵌入，以供 LLM 查询。
VectorStoreIndex 的使用：

```python
from llama_index.core import VectorStoreIndex
index = VectorStoreIndex.from_documents(documents)
```



什么是向量嵌入？
向量嵌入是 LLM 应用程序运行的核心。
向量嵌入（通常简称为嵌入）是文本语义或含义的数字表示，具有相似含义的两段文本将具有数学上相似的嵌入，即使实际文本完全不同。
这种数学关系支持语义搜索，用户提供查询术语，LlamaIndex 可以定位与查询术语含义相关的文本，而不是简单的关键字匹配。这是检索增强生成的工作原理以及 LLM 的一般运作方式的重要组成部分。
嵌入有多种类型，它们的效率、有效性和计算成本各不相同。默认情况下，LlamaIndex 使用 text-embedding-ada-002，这是 OpenAI 使用的默认嵌入。如果您使用不同的 LLM，您通常会想要使用不同的嵌入。
VectorStoreIndex 使用 LLM 的 API 将所有文本转换为嵌入；这就是我们说它“嵌入您的文本”的意思。如果您有大量文本，则生成嵌入可能需要很长时间，因为它涉及许多往返 API 调用。
当您想要搜索嵌入时，您的查询本身会转换为向量嵌入，然后由 VectorStoreIndex 执行数学运算，根据所有嵌入与您的查询在语义上的相似程度对它们进行排名。
排名完成后，VectorStoreIndex 会返回最相似的嵌入作为相应的文本块。它返回的嵌入数量称为 k，因此控制返回多少嵌入的参数称为 top_k。因此，整个类型的搜索通常被称为“top-k 语义检索”。Top-k 检索是查询向量索引的最简单形式。
也可以查看一下 OpenAI 关于 embeddings 的文档：https://platform.openai.com/docs/guides/embeddings

SummaryIndex 是一种更简单的索引形式，最适合查询，顾名思义，您试图生成文档中文本的摘要。它只是存储所有文档并将它们全部返回到您的查询引擎。



## 存储数据

加载数据并建立索引后，您可能需要存储它以避免重新索引的时间和成本。默认情况下，索引数据仅存储在内存中，也可以将数据存储到磁盘中或者存储到向量数据库中。

### 使用磁盘存储

存储索引数据的最简单方法是使用每个 Index 的内置 .persist() 方法，该方法将所有数据写入磁盘的指定位置。这适用于任何类型的索引。

```python
index.storage_context.persist(persist_dir="<persist_dir>")
```

然后，你可以通过加载持久索引来避免重新加载和重新索引数据：

```python
from llama_index.core import StorageContext, load_index_from_storage
storage_context = StorageContext.from_defaults(persist_dir="<persist_dir>")
index = load_index_from_storage(storage_context)
```

### 使用向量存储

正如索引中所讨论的，最常见的索引类型之一是 VectorStoreIndex。在 VectorStoreIndex 中创建嵌入的 API 调用在时间和金钱方面可能会很昂贵，因此您需要存储它们以避免不断地重新索引。
LlamaIndex 支持大量向量存储，这些向量存储的架构、复杂性和成本各不相同。下面将使用 Chroma，一个开源矢量存储数据库。
安装 Chroma：

```python
pip install chromadb
```

使用 Chroma 存储 VectorStoreIndex 中嵌入的步骤：
1. 初始化 Chroma 客户端 
2. 创建一个集合来将您的数据存储在 Chroma 中 
3. 将 Chroma 指定为 StorageContext 中的 vector_store
4. 使用 StorageContext 初始化您的 VectorStoreIndex

```python
import chromadb
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader
from llama_index.vector_stores.chroma import ChromaVectorStore
from llama_index.core import StorageContext

# 加载数据
documents = SimpleDirectoryReader("./data").load_data()

# 初始化 Chroma 客户端
db = chromadb.PersistentClient(path="./chroma_db")

# 创建集合或者获取集合
chroma_collection = db.get_or_create_collection("quickstart")

# 设置 chroma 作为 vector_store
vector_store = ChromaVectorStore(chroma_collection=chroma_collection)
storage_context = StorageContext.from_defaults(vector_store=vector_store)

# 创建索引
index = VectorStoreIndex.from_documents(
    documents, storage_context=storage_context
)

# 执行查询
query_engine = index.as_query_engine()
response = query_engine.query("What is the meaning of life?")
print(response)
```



## 查询数据

现在已经加载了数据，构建了索引，并存储了该索引以供以后使用，可以进行查询了。
LlamaIndex 中查询就是调用 LLM 时传入一个 prompt：它可以是一个问题并获得答案，或者是一个总结请求，或者是一个更复杂的指令。
更复杂的查询可能涉及重复/链式的 prompt + LLM 调用，甚至跨多个组件的推理循环。
所有查询的基础是 QueryEngine。获取 QueryEngine 最简单的方法是让索引为您创建一个，如下所示：

```python
query_engine = index.as_query_engine()
response = query_engine.query(
    "Write an email to the user given their background information."
)
print(response)
```

查询由三个不同的阶段组成：
1. Retrieval 是指您从索引中找到并返回与您的查询最相关的文档。最常见的 Retrieval 类型是“top-k”语义检索，但还有许多其他检索策略。
2. Postprocessing 是指对检索到的数据进行可选的重新排序、转换或过滤，例如要求它们具有特定的元数据，例如附加的关键字。
3. Response Synthesis 是将您的查询、最相关的数据和提示组合起来并发送给您的法学硕士以返回响应。

LlamaIndex 有更低级的 API，可以对上面查询的三个阶段进行精细控制。下面是例子：

```python
from llama_index.core import VectorStoreIndex, get_response_synthesizer
from llama_index.core.retrievers import VectorIndexRetriever
from llama_index.core.query_engine import RetrieverQueryEngine
from llama_index.core.postprocessor import SimilarityPostprocessor

# 创建索引
index = VectorStoreIndex.from_documents(documents)

# 配置 retriever
retriever = VectorIndexRetriever(
    index=index,   # 配置 index
    similarity_top_k=10,   # 设置返回最相似的10个embedding
)

# 配置 response
response_synthesizer = get_response_synthesizer()

# 组成一个 query engine
query_engine = RetrieverQueryEngine(
    retriever=retriever,
    response_synthesizer=response_synthesizer,
    node_postprocessors=[SimilarityPostprocessor(similarity_cutoff=0.7)],  # 设置检索到的节点达到要包含的最小相似度分数。
)

# 执行查询
response = query_engine.query("What did the author do growing up?")
print(response)
```



### 配置Postprocessor

上面的例子中，使用到了 node_postprocessors，在 LlamaIndex 中支持高级节点过滤和增强，可以进一步提高检索到的节点对象的相关性。这可以帮助减少 LLM 执行的时间/次数/成本或提高响应质量。
node 的 Postprocessor 有下面几个：

```txt
KeywordNodePostprocessor: 通过required_keywords和exclusion_keywords过滤节点。
SimilarityPostprocessor: 通过设置相似度分数的阈值来过滤节点（因此仅受基于嵌入的检索器支持）
PrevNextNodePostprocessor：使用基于节点关系的附加相关上下文来增强检索到的节点对象。
```

配置一个 KeywordNodePostprocessor 的例子：

```python
node_postprocessors = [
    KeywordNodePostprocessor(
        required_keywords=["Combinator"], exclude_keywords=["Italy"]
    )
]
query_engine = RetrieverQueryEngine.from_args(
    retriever, node_postprocessors=node_postprocessors
)
response = query_engine.query("What did the author do growing up?")
```



### 配置**Response synthesis**

检索器获取相关节点后，BaseSynthesizer 通过组合信息来合成最终响应，可以通过一下代码配置它：

```python
query_engine = RetrieverQueryEngine.from_args(
    retriever, response_mode=response_mode
)
```

上面的配置中，我们的 response synthesis 使用的是默认的 BaseSynthesizer，response_mode 参数有如下可选项：

```txt
default：通过顺序遍历每个检索到的节点来“创建和完善”答案；这使得每个节点都有一个单独的 LLM 调用。适合更详细的答案。
compact：在每次 LLM 调用期间“压缩”提示，通过填充尽可能多的节点文本块来适应最大提示大小。如果一个提示中的内容太多，无法填充，通过多个提示来“创建和完善”答案。
tree_summarize：给定一组 Node 对象和查询，递归构造一棵树并返回根节点作为响应。适合总结目的。
no_text：仅运行检索器来获取本应发送到 LLM 的节点，而不实际发送它们。然后可以通过检查response.source_nodes来检查。
accumulate：给定一组 Node 对象和查询，将查询应用于每个 Node 文本块，同时将响应累积到数组中。返回所有响应的串联字符串。适合当您需要对每个文本块单独运行相同的查询时。
```



上面简单介绍了一下 LlamaIndex 的基本使用流程。

参考文档：https://docs.llamaindex.ai/en/stable/
