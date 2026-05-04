# DEV_SPEC
## 模块化 RAG + MCP Server + Agent 扩展 AI 系统开发规范

## 1. 项目概述
本项目是一个本地优先、模块化、可插拔的 AI 系统工程，核心由三部分组成：

1. RAG 系统：负责文档摄取、检索、重排、上下文构建与回答生成。
2. MCP Server：基于 Python MCP SDK，以 `stdio transport` 对外暴露工具，供 GitHub Copilot、Claude Desktop 等客户端调用。
3. Agent 扩展层：在 RAG 之上预留 Tool Use、Memory、单 Agent 与 Multi-Agent 能力。

项目目标不是做一个一次性 Demo，而是形成一套可讲解、可扩展、可测试、可持续迭代的工程骨架，既可作为求职项目，也可作为教学与面试拆解素材。

## 2. 设计原则
1. 本地优先：默认本地运行、本地存储、本地调试，核心持久化使用 SQLite，本地向量库存储优先使用 Chroma。
2. 强约束分层：严格区分 `application / domain / infrastructure / interface / observability / evaluation / agent`，禁止跨层直接耦合。
3. 可插拔优先：LLM、Embedding、Reranker、VectorStore、Splitter、Evaluator、MemoryStore 必须全部抽象化。
4. 配置驱动：所有后端切换通过 `settings.yaml` 和环境变量完成，禁止在业务代码中硬编码 provider 分支。
5. 可观测优先：所有核心链路必须产出结构化 Trace 和 JSON Lines 日志。
6. 面试友好：每个模块必须说明设计动机、替换点、扩展点。
7. 测试先行：核心领域逻辑优先 TDD，所有回归问题必须附带测试。
8. 不引入重框架：禁止使用 LangChain、LlamaIndex 做编排，仅允许 `RecursiveCharacterTextSplitter`。

## 3. 非目标
1. 不提供 HTTP 服务。
2. 不做在线多租户 SaaS。
3. 首版不实现完整 Multi-Agent，只提供接口与预留架构。
4. 首版不做分布式部署，不做复杂权限系统。

## 4. 总体架构
```text
+------------------- MCP Client -------------------+
| Copilot / Claude Desktop / Local Agent Runtime   |
+--------------------------+-----------------------+
                           |
                           v
+---------------------- MCP Server ----------------------+
| tools: query_knowledge_hub / list_collections /        |
|        get_document_summary                            |
+--------------------------+-----------------------------+
                           |
                           v
+---------------------- Application ---------------------+
| QueryService | IngestionService | SummaryService       |
| AgentService | MemoryService    | EvaluationService    |
+--------+-------------+-------------+-------------------+
         |             |             |
         v             v             v
+------------------ Domain / Ports ----------------------+
| BaseLLM | BaseEmbedding | BaseReranker | BaseSplitter  |
| BaseVectorStore | BaseEvaluator | BaseTool            |
| BaseMemoryStore | Trace Models | Chunk / Query Models |
+--------+-------------+-------------+-------------------+
         |             |             |
         v             v             v
+---------------- Infrastructure ------------------------+
| Azure/OpenAI/Ollama/DeepSeek adapters                 |
| ChromaStore | BM25Index | SQLiteRepo | Vision Adapter |
| MarkItDown Loader | Streamlit Dashboard               |
+-------------------------------------------------------+
```

## 5. 目录结构
```text
project/
├── README.md
├── DEV_SPEC.md
├── pyproject.toml
├── settings.yaml
├── .env.example
├── src/
│   ├── main.py
│   ├── mcp_server/
│   │   ├── server.py
│   │   ├── tool_handlers.py
│   │   └── schemas.py
│   ├── application/
│   │   ├── services/
│   │   │   ├── ingestion_service.py
│   │   │   ├── retrieval_service.py
│   │   │   ├── query_service.py
│   │   │   ├── summary_service.py
│   │   │   ├── memory_service.py
│   │   │   ├── agent_service.py
│   │   │   └── evaluation_service.py
│   │   └── factories/
│   │       └── provider_factory.py
│   ├── domain/
│   │   ├── models/
│   │   │   ├── document.py
│   │   │   ├── chunk.py
│   │   │   ├── query.py
│   │   │   ├── memory.py
│   │   │   ├── trace.py
│   │   │   └── tool.py
│   │   ├── ports/
│   │   │   ├── llm.py
│   │   │   ├── embedding.py
│   │   │   ├── reranker.py
│   │   │   ├── vector_store.py
│   │   │   ├── splitter.py
│   │   │   ├── evaluator.py
│   │   │   ├── memory_store.py
│   │   │   └── tool.py
│   │   └── exceptions.py
│   ├── infrastructure/
│   │   ├── loaders/
│   │   │   └── pdf_markitdown_loader.py
│   │   ├── splitters/
│   │   │   └── recursive_splitter.py
│   │   ├── llms/
│   │   ├── embeddings/
│   │   ├── rerankers/
│   │   ├── vectorstores/
│   │   │   └── chroma_store.py
│   │   ├── retrieval/
│   │   │   ├── bm25_index.py
│   │   │   └── rrf.py
│   │   ├── memories/
│   │   │   ├── sqlite_memory_repo.py
│   │   │   └── vector_memory_store.py
│   │   ├── tools/
│   │   │   └── local_tools.py
│   │   └── persistence/
│   │       ├── sqlite.py
│   │       └── repositories.py
│   ├── observability/
│   │   ├── logger.py
│   │   ├── trace_recorder.py
│   │   └── trace_repository.py
│   ├── dashboard/
│   │   ├── app.py
│   │   └── pages/
│   ├── evaluation/
│   │   ├── ragas_runner.py
│   │   └── custom_metrics.py
│   └── shared/
│       ├── settings.py
│       ├── enums.py
│       └── utils.py
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── e2e/
│   └── fixtures/
└── data/
    ├── raw/
    ├── processed/
    ├── chroma/
    ├── sqlite/
    └── traces/
```

## 6. 模块职责表
| 模块 | 职责 | 不负责 |
|---|---|---|
| `mcp_server` | MCP tools 注册、输入输出校验、调用应用服务 | 不写业务逻辑 |
| `application.services` | 编排用例、调用端口、聚合结果 | 不依赖具体 SDK |
| `domain.models` | 领域对象、数据结构、状态约束 | 不做 IO |
| `domain.ports` | 定义抽象接口 | 不做实现 |
| `infrastructure` | 对接 LLM、向量库、SQLite、MarkItDown、BM25 | 不承载业务规则 |
| `observability` | 日志、Trace、埋点输出 | 不做业务决策 |
| `dashboard` | 可视化查看数据、Trace、评估结果 | 不直接实现核心逻辑 |
| `evaluation` | 跑评估、产出指标 | 不处理查询主链路 |
| `agent` | Thought-Action-Observation 编排、工具调度、记忆读写 | 不绑定具体工具实现 |

## 7. 核心领域模型
### 7.1 Document
```python
from pydantic import BaseModel
from typing import Any


class Document(BaseModel):
    doc_id: str
    source_path: str
    file_name: str
    sha256: str
    markdown_text: str
    metadata: dict[str, Any]
```

### 7.2 Chunk
```python
from pydantic import BaseModel, Field
from typing import Any


class Chunk(BaseModel):
    chunk_id: str
    doc_id: str
    chunk_index: int
    text: str
    rewritten_text: str | None = None
    title_path: list[str] = Field(default_factory=list)
    page_range: list[int] = Field(default_factory=list)
    image_captions: list[str] = Field(default_factory=list)
    metadata: dict[str, Any] = Field(default_factory=dict)
    sparse_text: str
    dense_text: str
    token_count: int
    content_hash: str
```

### 7.3 QueryContext
```python
class QueryRequest(BaseModel):
    query: str
    collection: str
    top_k_retrieval: int = 20
    top_k_rerank: int = 8
    use_llm_rerank: bool = False
    conversation_id: str | None = None
    user_id: str | None = None


class RetrievedChunk(BaseModel):
    chunk_id: str
    score: float
    source: str
    stage: str
    chunk: Chunk


class QueryResponse(BaseModel):
    answer: str
    citations: list[dict[str, Any]]
    used_chunks: list[RetrievedChunk]
    trace_id: str
```

## 8. 可插拔抽象接口
### 8.1 LLM
```python
from abc import ABC, abstractmethod


class BaseLLM(ABC):
    @abstractmethod
    def generate(self, prompt: str, system_prompt: str | None = None) -> str: ...

    @abstractmethod
    def chat(self, messages: list[dict]) -> str: ...

    @abstractmethod
    def vision_caption(self, image_bytes: bytes, prompt: str) -> str: ...
```

### 8.2 Embedding
```python
class BaseEmbedding(ABC):
    @abstractmethod
    def embed_texts(self, texts: list[str]) -> list[list[float]]: ...

    @abstractmethod
    def embed_query(self, text: str) -> list[float]: ...
```

### 8.3 Reranker
```python
class BaseReranker(ABC):
    @abstractmethod
    def rerank(self, query: str, documents: list[str]) -> list[tuple[int, float]]: ...
```

### 8.4 VectorStore
```python
class BaseVectorStore(ABC):
    @abstractmethod
    def upsert_chunks(self, collection: str, chunks: list[Chunk], vectors: list[list[float]]) -> None: ...

    @abstractmethod
    def similarity_search(self, collection: str, query_vector: list[float], top_k: int) -> list[RetrievedChunk]: ...

    @abstractmethod
    def get_document_summary(self, doc_id: str) -> dict: ...

    @abstractmethod
    def list_collections(self) -> list[str]: ...
```

### 8.5 Splitter
```python
class BaseSplitter(ABC):
    @abstractmethod
    def split_markdown(self, doc: Document) -> list[Chunk]: ...
```

### 8.6 Evaluator
```python
class BaseEvaluator(ABC):
    @abstractmethod
    def evaluate(self, dataset_path: str, run_id: str) -> dict: ...
```

### 8.7 Tool
```python
class BaseTool(ABC):
    name: str
    description: str

    @abstractmethod
    def schema(self) -> dict: ...

    @abstractmethod
    def run(self, payload: dict) -> dict: ...
```

### 8.8 MemoryStore
```python
class BaseMemoryStore(ABC):
    @abstractmethod
    def add_memory(self, memory: dict) -> str: ...

    @abstractmethod
    def search_memory(self, query: str, user_id: str, top_k: int) -> list[dict]: ...

    @abstractmethod
    def summarize_memory(self, conversation_id: str) -> dict: ...
```

## 9. Provider Factory
```python
class ProviderFactory:
    def __init__(self, settings):
        self.settings = settings

    def create_llm(self) -> BaseLLM: ...
    def create_embedding(self) -> BaseEmbedding: ...
    def create_reranker(self) -> BaseReranker: ...
    def create_vector_store(self) -> BaseVectorStore: ...
    def create_splitter(self) -> BaseSplitter: ...
    def create_evaluator(self) -> BaseEvaluator: ...
    def create_memory_store(self) -> BaseMemoryStore: ...
```

规则：
1. 所有 provider 创建逻辑集中在工厂层。
2. 服务层只能依赖抽象接口。
3. 工厂选择 provider 的唯一依据是 `settings.yaml`。

## 10. 配置设计
### 10.1 settings.yaml
```yaml
project:
  name: modular-rag-mcp-server
  env: dev
  data_dir: ./data

llm:
  provider: azure_openai
  model: gpt-4.1-mini
  api_base: ${AZURE_OPENAI_ENDPOINT}
  api_key: ${AZURE_OPENAI_API_KEY}
  api_version: "2024-12-01-preview"
  temperature: 0.1
  max_tokens: 2048

embedding:
  provider: openai
  model: text-embedding-3-large
  api_key: ${OPENAI_API_KEY}
  batch_size: 32

reranker:
  provider: cross_encoder
  model: BAAI/bge-reranker-base
  llm_fallback_provider: openai

vector_store:
  provider: chroma
  persist_dir: ./data/chroma
  collection_prefix: knowledge_hub_

splitter:
  provider: recursive_character
  chunk_size: 900
  chunk_overlap: 180
  separators: ["\n## ", "\n### ", "\n\n", "\n", "。", " "]

loader:
  pdf_to_markdown: markitdown
  image_caption_prompt: "请用中文描述该图片中与知识检索相关的信息。"

memory:
  short_term_window: 12
  summary_trigger_turns: 10
  sqlite_path: ./data/sqlite/memory.db
  vector_collection: user_memory

evaluation:
  provider: ragas
  dataset_dir: ./data/eval
  metrics: [hit_rate, mrr, faithfulness, answer_relevancy]

observability:
  log_path: ./data/traces/app.jsonl
  trace_db: ./data/sqlite/traces.db
  enable_console_log: true

mcp:
  server_name: modular-rag-mcp
  transport: stdio
  enabled_tools:
    - query_knowledge_hub
    - list_collections
    - get_document_summary
```

### 10.2 Settings 规则
1. 所有敏感项只能从环境变量读取。
2. `settings.yaml` 只保存结构和默认值。
3. 配置加载顺序：默认值 < `settings.yaml` < `.env` < 环境变量。
4. 启动时必须做配置完整性校验，缺关键字段直接失败。

## 11. Ingestion Pipeline
### 11.1 流程
`PDF -> Markdown -> Split -> LLM Enrich -> Dual Embedding -> Upsert -> Trace -> Incremental Record`

### 11.2 详细步骤
1. 文件发现：读取待导入 PDF。
2. 哈希计算：对原始文件计算 `SHA256`。
3. 增量跳过：查询 `ingestion_history`，若哈希已成功处理则直接跳过。
4. PDF 转 Markdown：调用 MarkItDown，产出统一 Markdown 文本。
5. 图片抽取：识别页面图片并获取二进制数据或引用。
6. 智能分块：使用 `RecursiveCharacterTextSplitter` 按 Markdown 标题与段落切分。
7. LLM 增强：
   - 重写：清理碎片、补足上下文指代。
   - 元数据注入：标题路径、页码范围、来源文档等。
   - 图片描述：Vision LLM 对图片生成中文描述。
8. 双路文本构建：
   - `sparse_text = 原文 + 标题 + 关键词`
   - `dense_text = 重写文本 + 图片描述 + 元数据上下文`
9. Embedding：仅对 `dense_text` 做向量化。
10. Upsert：将 chunk、metadata、vector 写入 Chroma。
11. BM25 建索引：将 `sparse_text` 写入本地 BM25 索引。
12. 记录 Trace、日志与 ingestion_history。

### 11.3 Ingestion I/O
```python
class IngestionResult(BaseModel):
    doc_id: str
    file_name: str
    chunk_count: int
    skipped: bool
    trace_id: str
```

### 11.4 ingestion_history 表
```sql
CREATE TABLE ingestion_history (
  file_hash TEXT PRIMARY KEY,
  doc_id TEXT NOT NULL,
  file_path TEXT NOT NULL,
  status TEXT NOT NULL,
  chunk_count INTEGER DEFAULT 0,
  processed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  error_msg TEXT
);
```

### 11.5 设计说明
这样设计的原因是把“是否需要重跑”前置到最便宜的阶段，避免重复调用 MarkItDown、Vision LLM、Embedding。面试时可强调“零成本增量更新”和“幂等摄取”。

## 12. Retrieval Pipeline
### 12.1 查询主链路
`Query Normalize -> Memory Recall -> Dense Retrieve -> BM25 Retrieve -> RRF Fusion -> Rerank -> Context Build -> Answer Generate`

### 12.2 详细步骤
1. 查询标准化：去空白、统一大小写、保留原始 query。
2. Memory 检索：若有 `conversation_id/user_id`，先召回短期摘要和长期记忆。
3. Dense 检索：向量检索获取 `top_k_retrieval`。
4. Sparse 检索：BM25 获取 `top_k_retrieval`。
5. RRF 融合：使用统一 chunk_id 去重融合。
6. 精排：
   - 默认 Cross-Encoder 重排。
   - 配置开启时可使用 LLM Rerank。
7. Top-K 截断：保留前 `top_k_rerank`。
8. 上下文组装：拼接 chunk 文本、标题路径、图片描述、记忆摘要。
9. 答案生成：LLM 根据上下文与约束 prompt 作答。
10. 输出引用：返回 chunk 来源、页码、文档名。
11. 写入 Query Trace 与会话短期记忆。

### 12.3 RRF
```python
def rrf_fuse(rank_lists: list[list[str]], k: int = 60) -> dict[str, float]:
    scores = {}
    for rank_list in rank_lists:
        for rank, chunk_id in enumerate(rank_list, start=1):
            scores[chunk_id] = scores.get(chunk_id, 0.0) + 1.0 / (k + rank)
    return scores
```

### 12.4 精排策略
1. Cross-Encoder 适合作为默认精排，成本更稳定。
2. LLM Rerank 仅用于高价值查询或实验开关。
3. 重排输入必须包含 `query + chunk dense_text`，不得只看裸文本。

### 12.5 设计说明
Hybrid Retrieval 解决关键词匹配与语义召回互补问题，RRF 保证融合算法简单可解释，Rerank 负责把候选集合提升到可生成答案的精度。

## 13. Context 构建策略
上下文由四部分组成：

1. 检索片段正文。
2. 元数据：文档名、标题路径、页码范围、chunk 序号。
3. 图片描述：作为正文附加段落插入。
4. 记忆内容：短期对话摘要、长期偏好记忆、任务回顾记忆。

上下文模板：
```text
[Memory Summary]
{memory_summary}

[Retrieved Context 1]
Source: {file_name} | Titles: {title_path} | Pages: {page_range}
Text: {dense_text}
Image Notes: {image_captions}

[Retrieved Context 2]
...
```

规则：
1. 上下文构建必须保留来源信息，便于引用。
2. 图像描述默认和所属 chunk 同级拼接，不单独建独立回答上下文。
3. 超长上下文必须按分数截断，禁止无上限堆叠。

## 14. 多模态处理
### 14.1 处理策略
1. 不使用 CLIP。
2. 只做 `Image -> Vision LLM -> Text Caption`。
3. 将 caption 注入 `dense_text` 和 `image_captions` 字段。

### 14.2 融合方式
1. 若图片属于某段落区域，则并入对应 chunk。
2. 若图片跨多个段落，则并入最邻近标题块。
3. 若图片无法定位，则生成独立辅助 chunk，并标记 `metadata["synthetic_image_chunk"]=True`。

### 14.3 设计说明
该方案牺牲了图像向量检索的细粒度能力，但大幅降低系统复杂度，且与纯文本 RAG 管线兼容，适合教学和求职项目。

## 15. Memory System
### 15.1 三层结构
1. Short-term memory：当前会话最近 N 轮消息和摘要。
2. Long-term memory：用户偏好、稳定事实、历史有用知识，向量化存储。
3. Episodic memory：一次任务的目标、步骤、结果、反思。

### 15.2 数据结构
```python
class MemoryRecord(BaseModel):
    memory_id: str
    user_id: str
    conversation_id: str | None = None
    memory_type: str  # short_term / long_term / episodic
    content: str
    summary: str | None = None
    metadata: dict
    importance_score: float = 0.5
    created_at: str
```

### 15.3 存储设计
1. SQLite：存原始内容、摘要、类型、用户、时间、重要度。
2. Chroma：仅对长期记忆和 episodic summary 存向量。
3. 短期记忆默认仅存 SQLite。

### 15.4 检索策略
1. 短期记忆：按会话 ID 和时间窗口直接取最近 N 条。
2. 长期记忆：按 query embedding + user_id filter 检索。
3. Episodic memory：优先按任务标签过滤，再做向量检索。

### 15.5 更新策略
1. 每轮对话落短期记忆。
2. 达到 `summary_trigger_turns` 时自动摘要压缩。
3. 高重要信息由 LLM 分类后提升为长期记忆。
4. 任务结束时生成 episodic summary。
5. 定期运行记忆压缩任务，合并语义重复记忆。

### 15.6 面试点
可以强调“短期-长期-事件”分层让系统同时兼顾实时上下文、稳定偏好和任务复盘，且通过 SQLite + Chroma 保持实现简单。

## 16. Tool System
### 16.1 Tool Registry
```python
class ToolRegistry:
    def __init__(self):
        self._tools: dict[str, BaseTool] = {}

    def register(self, tool: BaseTool) -> None:
        self._tools[tool.name] = tool

    def get(self, name: str) -> BaseTool:
        return self._tools[name]

    def list_tools(self) -> list[dict]:
        return [{"name": t.name, "description": t.description, "schema": t.schema()} for t in self._tools.values()]
```

### 16.2 调度逻辑
1. MCP tools 和本地 tools 都注册到统一 Registry。
2. Agent 只依赖 Registry，不关心工具来自 MCP 还是本地函数。
3. 每次调用必须记录 `tool_name / input / output / latency / status`。

### 16.3 设计说明
统一 Tool System 能避免后续 Agent 和 MCP 两套调用体系分裂。

## 17. Agent Framework
### 17.1 最小循环
`Thought -> Action -> Observation -> Thought ... -> Final Answer`

### 17.2 状态结构
```python
class AgentState(BaseModel):
    session_id: str
    user_goal: str
    thoughts: list[str]
    actions: list[dict]
    observations: list[dict]
    final_answer: str | None = None
    max_steps: int = 6
    current_step: int = 0
```

### 17.3 控制规则
1. 每轮先让 LLM 输出结构化 Thought/Action。
2. 若 Action 是工具调用，则通过 ToolRegistry 执行。
3. 将 Observation 追加回消息上下文。
4. 达到 `max_steps` 或生成 `final_answer` 时终止。
5. 工具异常必须进入 Observation，而不是直接崩溃。

### 17.4 预留 Multi-Agent
定义接口但首版不实现编排：
```python
class PlannerAgent: ...
class ExecutorAgent: ...
class RetrieverAgent: ...
class EvaluatorAgent: ...
```

### 17.5 面试点
该设计展示了“单 Agent 最小闭环 + 多 Agent 预留端口”的渐进式工程思路，避免过度设计。

## 18. MCP Server 设计
### 18.1 工具定义
#### `query_knowledge_hub`
输入：
```json
{
  "query": "RRF 是什么",
  "collection": "default",
  "top_k_retrieval": 20,
  "top_k_rerank": 8,
  "use_llm_rerank": false,
  "conversation_id": "conv_001",
  "user_id": "user_001"
}
```
输出：
```json
{
  "answer": "RRF 是一种排序融合算法...",
  "citations": [{"doc_id":"d1","chunk_id":"c3","page_range":[2,3]}],
  "trace_id": "trace_xxx"
}
```

#### `list_collections`
输入：`{}`
输出：
```json
{"collections": ["default", "finance", "legal"]}
```

#### `get_document_summary`
输入：
```json
{"doc_id": "doc_001"}
```
输出：
```json
{
  "doc_id": "doc_001",
  "file_name": "rag_intro.pdf",
  "summary": "该文档主要介绍...",
  "chunk_count": 42
}
```

### 18.2 调用链路
`MCP Client -> MCP Server -> Handler -> Application Service -> Domain Ports -> Infrastructure -> Response`

### 18.3 实现约束
1. 仅 `stdio transport`。
2. Handler 负责 schema 校验和错误包装。
3. Application Service 负责业务编排。
4. 所有 tool 调用都必须产出 trace_id。

## 19. 可观测性
### 19.1 Trace 类型
1. Ingestion Trace
2. Query Trace
3. Agent Trace

### 19.2 Trace 结构
```python
class TraceEvent(BaseModel):
    trace_id: str
    span_id: str
    parent_span_id: str | None = None
    trace_type: str
    stage: str
    provider: str | None = None
    method: str | None = None
    input_payload: dict
    output_payload: dict
    latency_ms: int
    status: str
    error_message: str | None = None
    created_at: str
```

### 19.3 JSON Lines 日志格式
```json
{"ts":"2026-05-04T12:00:00Z","level":"INFO","trace_id":"t1","module":"query_service","event":"dense_retrieval_done","provider":"chroma","latency_ms":42,"payload":{"top_k":20}}
```

字段要求：
`ts, level, trace_id, module, event, provider, latency_ms, payload`

### 19.4 设计说明
JSON Lines 便于本地 grep、后续接 ELK/ClickHouse，也适合教学演示单次链路。

## 20. Dashboard 设计
页面必须包括：

1. 系统总览  
展示当前 provider 配置、文档数、chunk 数、最近查询数、最近摄取数。

2. 数据浏览  
支持按 collection、doc_id、标题、关键词查看 chunk 和 metadata。

3. Ingestion 管理  
支持选择文件导入、查看状态、跳过记录、失败原因。

4. Trace 查看  
按 trace_type、stage、trace_id 检索链路，显示耗时瀑布和输入输出。

5. 评估面板  
运行 Ragas、自定义指标任务，查看历史结果。

约束：
1. Dashboard 只读核心数据，避免直接写复杂业务逻辑。
2. 页面数据全部从 SQLite、Trace Repo、VectorStore 读取。

## 21. 评估体系
### 21.1 支持指标
1. Ragas：`faithfulness`, `answer_relevancy`, `context_precision`, `context_recall`
2. 自定义：`hit_rate`, `mrr`

### 21.2 hit_rate
命中定义：目标答案对应文档或 chunk 出现在 Top-K 中。
```python
hit_rate = hit_count / total_queries
```

### 21.3 MRR
```python
MRR = mean(1 / rank_of_first_relevant_doc)
```

### 21.4 扩展预留
1. Agent trajectory correctness
2. Tool call success rate
3. Tool selection accuracy

### 21.5 面试点
评估模块能体现“不是拍脑袋调参，而是数据驱动优化”。

## 22. 数据流说明
### 22.1 Ingestion Flow
1. 读 PDF
2. 算 SHA256
3. 查增量表
4. 转 Markdown
5. 抽图并 caption
6. 分块
7. LLM 增强
8. 构造 sparse/dense 文本
9. 写向量库
10. 写 BM25
11. 写 SQLite 元数据
12. 写 Trace

### 22.2 Query Flow
1. 收到 MCP tool 请求
2. 参数校验
3. 召回记忆
4. Dense/BM25 双检索
5. RRF 融合
6. Rerank
7. 构造上下文
8. 生成答案
9. 输出引用
10. 写记忆与 Trace

### 22.3 Agent Flow
1. 收到目标
2. 读取短期与长期记忆
3. 生成 Thought/Action
4. 调度工具
5. 写 Observation
6. 循环直到结束
7. 写 episodic memory 和 Trace

## 23. 测试方案
### 23.1 总原则
1. 核心领域逻辑优先 TDD。
2. 所有 provider 适配器必须可 mock。
3. 单元、集成、E2E 分层执行，不能混淆。

### 23.2 单元测试
覆盖：
1. Chunk 构造与 metadata 注入
2. RRF 融合
3. Memory 压缩与筛选逻辑
4. Tool Registry
5. Agent loop 终止条件
6. settings 校验

### 23.3 集成测试
覆盖：
1. MarkItDown -> Split -> Upsert 链路
2. Chroma 检索 + BM25 融合
3. SQLite 记忆读写
4. MCP tool handler 到 service 的串联

### 23.4 E2E 测试
覆盖：
1. 导入一个 PDF 后可成功查询
2. `query_knowledge_hub` 返回 answer、citations、trace_id
3. 记忆写入后下一轮问答可正确利用记忆
4. Dashboard 能显示最新 trace 和评估结果

### 23.5 RAG 专项测试
1. retrieval 质量：固定样本集评估 `hit_rate/MRR`
2. rerank 效果：比较 rerank 前后相关文档排名变化

### 23.6 Agent 预留测试
1. trajectory 测试
2. tool 调用序列测试
3. max_steps 安全测试

## 24. 开发约束
1. Python 版本固定 `3.11+`
2. 测试框架固定 `pytest`
3. 格式化和 lint 使用 `ruff`
4. 类型检查使用 `mypy`
5. 数据模型使用 `pydantic`
6. 依赖管理推荐 `uv`
7. 不允许直接在 service 中 import 某具体 provider 类
8. 不允许把 SQLite SQL 拼接散落在多个服务中，统一通过 repository
9. 所有公共函数必须有类型标注
10. 任何新功能都必须补测试和文档

## 25. 项目排期
### 阶段 A：骨架初始化
| 任务 | 文件 | 类/函数 | 验收标准 | 测试 |
|---|---|---|---|---|
| A1 项目结构初始化 | `pyproject.toml`, `src/`, `tests/` | N/A | 目录树可用 | `pytest -q` 空跑 |
| A2 配置系统 | `shared/settings.py` | `load_settings` | 能加载 yaml 和 env | 单测配置覆盖 |
| A3 领域模型 | `domain/models/*` | `Document`, `Chunk`, `TraceEvent` | 模型校验通过 | Pydantic 单测 |

### 阶段 B：Provider 抽象
| 任务 | 文件 | 类/函数 | 验收标准 | 测试 |
|---|---|---|---|---|
| B1 定义 ports | `domain/ports/*` | Base 接口 | 接口完整 | 抽象类单测 |
| B2 工厂实现 | `application/factories/provider_factory.py` | `create_*` | 可按配置返回 provider | mock 测试 |
| B3 LLM 适配器骨架 | `infrastructure/llms/*` | provider classes | 至少 4 个骨架 | 初始化测试 |

### 阶段 C：Ingestion 基线
| 任务 | 文件 | 类/函数 | 验收标准 | 测试 |
|---|---|---|---|---|
| C1 PDF Loader | `pdf_markitdown_loader.py` | `load_pdf` | PDF 转 Markdown | fixture PDF 集成测 |
| C2 Splitter | `recursive_splitter.py` | `split_markdown` | 产生 chunk | 分块单测 |
| C3 增量表 | `repositories.py` | `save_ingestion_history` | 可跳过重复文件 | SQLite 集成测 |

### 阶段 D：增强与向量化
| 任务 | 文件 | 类/函数 | 验收标准 | 测试 |
|---|---|---|---|---|
| D1 Vision Caption | `llms/*` | `vision_caption` | 可生成图片描述 | mock LLM 测试 |
| D2 Chunk Enrich | `ingestion_service.py` | `enrich_chunks` | 注入 metadata 和 caption | 单测 |
| D3 Embedding + Upsert | `embeddings/*`, `chroma_store.py` | `embed_texts`, `upsert_chunks` | 数据写入 Chroma | 集成测 |

### 阶段 E：检索链路
| 任务 | 文件 | 类/函数 | 验收标准 | 测试 |
|---|---|---|---|---|
| E1 BM25 | `bm25_index.py` | `search` | 稀疏检索可用 | 检索单测 |
| E2 Dense Search | `chroma_store.py` | `similarity_search` | 稠密检索可用 | 集成测 |
| E3 RRF | `rrf.py` | `rrf_fuse` | 融合正确 | 单测 |
| E4 Rerank | `rerankers/*` | `rerank` | 排序可输出 | mock 测试 |

### 阶段 F：Query 与 MCP
| 任务 | 文件 | 类/函数 | 验收标准 | 测试 |
|---|---|---|---|---|
| F1 QueryService | `query_service.py` | `query` | 返回 answer/citations/trace | 集成测 |
| F2 MCP Schemas | `schemas.py` | request/response models | 输入输出固定 | schema 单测 |
| F3 MCP Tools | `tool_handlers.py`, `server.py` | 3 个 tools | 客户端可调用 | E2E 测试 |

### 阶段 G：Observability 与 Dashboard
| 任务 | 文件 | 类/函数 | 验收标准 | 测试 |
|---|---|---|---|---|
| G1 JSONL Logger | `logger.py` | `log_event` | 写入标准 jsonl | 日志单测 |
| G2 Trace Repo | `trace_repository.py` | `save_trace` | 可查 trace | SQLite 测试 |
| G3 Dashboard | `dashboard/pages/*` | pages | 页面可显示数据 | Streamlit smoke test |

### 阶段 H：Memory 与 Agent
| 任务 | 文件 | 类/函数 | 验收标准 | 测试 |
|---|---|---|---|---|
| H1 Memory Repo | `sqlite_memory_repo.py` | CRUD/search | 三层记忆可用 | 集成测 |
| H2 MemoryService | `memory_service.py` | summarize/promote/search | 可自动摘要压缩 | 单测 |
| H3 Agent Loop | `agent_service.py` | `run_agent_loop` | Thought-Action-Observation 可运行 | trajectory 测试 |

### 阶段 I：评估与收尾
| 任务 | 文件 | 类/函数 | 验收标准 | 测试 |
|---|---|---|---|---|
| I1 Ragas Runner | `ragas_runner.py` | `run` | 可生成评估报告 | 集成测 |
| I2 Custom Metrics | `custom_metrics.py` | `hit_rate`, `mrr` | 指标正确 | 单测 |
| I3 文档与样例数据 | `README.md`, `data/eval` | N/A | 可演示完整流程 | E2E 回归 |

## 26. 进度跟踪表
| 阶段 | 状态 | 完成标准 |
|---|---|---|
| A | 未开始 | 项目骨架和配置可运行 |
| B | 未开始 | 所有 provider 抽象完整 |
| C | 未开始 | 可导入 PDF 并分块 |
| D | 未开始 | 可写入向量库 |
| E | 未开始 | Hybrid Retrieval 全链路打通 |
| F | 未开始 | MCP tools 可调用 |
| G | 未开始 | Trace 和 Dashboard 可用 |
| H | 未开始 | Memory 和 Agent 最小闭环 |
| I | 未开始 | 评估闭环与演示完成 |

## 27. 面试回答点与扩展点
1. 为什么不用 LangChain/LlamaIndex  
为了保持架构透明、降低黑盒依赖、便于讲解与控制每个环节。

2. 为什么用 Hybrid Retrieval  
BM25 擅长专有名词，Dense 擅长语义匹配，RRF 简洁可解释。

3. 为什么 Memory 分三层  
对话、偏好、任务信息生命周期不同，混在一起会导致检索污染。

4. 为什么统一 Tool System  
MCP 和 Agent 都依赖工具调用，统一抽象能降低未来扩展成本。

5. 如何扩展  
可扩展到新 VectorStore、新 LLM provider、Agent 反思机制、Planner/Executor 多 Agent、外部知识同步任务。

## 28. 开发起始顺序
建议严格按 `A -> B -> C -> D -> E -> F -> G -> H -> I` 推进，任何阶段未验收通过，不进入下一阶段。首个可演示里程碑是阶段 F，首个完整工程里程碑是阶段 I。
