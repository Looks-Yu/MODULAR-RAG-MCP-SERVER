# DEV_SPEC
## 模块化 RAG + MCP Server + Agent 扩展 AI 系统开发规范

## 1. 项目概述
本项目是一个本地优先、模块化、可插拔的 AI 系统工程，核心由三部分组成：

1. RAG 系统：负责文档摄取、检索、重排、上下文构建与回答生成。
2. MCP Server：基于 Python MCP SDK，以 `stdio transport` 对外暴露工具，供 GitHub Copilot、Claude Desktop 等客户端调用。
3. Agent 扩展层：在 RAG 之上预留 Tool Use、Memory、单 Agent 与 Multi-Agent 能力。

项目目标不是做一个一次性 Demo，而是形成一套可讲解、可扩展、可测试、可持续迭代的工程骨架，既可作为求职项目，也可作为教学与面试拆解素材。

## 2. 核心特色与系统亮点
本节用于在文档前部先回答“这个项目最值得讲的是什么”。后续章节再分别展开实现细节、架构取舍与工程落地问题。

### 2.1 RAG 策略与设计
1. 本项目不是简单的“向量检索 + LLM 回答”，而是完整的工程化 RAG 链路，包括 Ingestion、Hybrid Retrieval、RRF 融合、Rerank、Context Build、Evaluation。
2. RAG 的重点不是单个算法，而是把文档处理、检索召回、重排过滤、上下文构建和回答生成串成稳定闭环。
3. 设计亮点在于把每个阶段都显式拆开，使系统既能调优，也能讲清楚。

设计思想与优势：
1. 通过 Hybrid Retrieval 兼顾关键词精确匹配和语义召回。
2. 通过 Rerank 将“能召回”提升为“能用于回答”。
3. 通过 Context Builder 将 metadata、图片描述、memory 统一纳入最终 prompt 组装。

工程化难点：
1. 文档解析噪声会直接传导到检索效果。
2. 检索命中不等于生成正确，Context Build 仍是关键。
3. 每个阶段都需要可观测和可评估，否则难以落地优化。

### 2.2 全链路可插拔架构
1. LLM、Embedding、Reranker、VectorStore、Splitter、Evaluator、MemoryStore、Tool 都必须支持抽象与替换。
2. 所有组件替换通过配置驱动完成，而不是在业务逻辑中写 provider 分支。
3. 这使项目具备长期演进能力，而不是绑定单一技术栈的短期 Demo。

设计思想与优势：
1. 抽象接口保证系统边界清晰。
2. 工厂模式保证 provider 切换成本低。
3. 配置驱动保证实验、教学、部署时的可控性。

工程化难点：
1. 抽象做得太薄会失去可替换价值，做得太厚会导致接口失真。
2. 不同 provider 的能力边界不同，需要统一最小公共契约。
3. 插拔能力必须和测试体系配套，否则切换后容易失控。

### 2.3 MCP 生态集成
1. 本项目将内部能力封装为 MCP Tools，而不是只做一个本地脚本或传统 HTTP 服务。
2. 通过 MCP，可以直接接入 Copilot、Claude Desktop 等客户端，体现更强的 AI 工程集成能力。
3. `stdio transport` 让系统以本地子进程形式运行，降低部署复杂度并贴合编辑器集成场景。

设计思想与优势：
1. 一次开发，多客户端复用。
2. 能直接融入 AI 助手工作流，而不是额外再造一个前端。
3. 更适合作为教学和求职项目展示“协议层集成能力”。

工程化难点：
1. tool 边界必须清晰，过大过小都不利于实际使用。
2. 输入输出 schema 必须稳定，否则客户端适配成本高。
3. tool 错误必须可恢复、可追踪。

### 2.4 多模态、可观测性、可视化管理与评估体系
1. 多模态上，本项目采用 Vision LLM Captioning，而不是引入复杂的图像向量检索体系。
2. 可观测性上，要求 Ingestion Trace、Query Trace、Agent Trace 全链路留痕。
3. 管理上，通过 Streamlit Dashboard 展示系统总览、数据浏览、摄取状态、Trace 与评估结果。
4. 评估上，通过 Ragas 与自定义指标形成回归闭环。

设计思想与优势：
1. 多模态策略强调“低复杂度集成进现有文本 RAG 链路”。
2. 可观测性让系统从黑盒变为白盒。
3. Dashboard 与评估体系让优化从主观感受转为数据驱动。

工程化难点：
1. 图片描述注入位置会影响检索质量。
2. 没有 Trace 很难定位问题阶段。
3. 没有评估基线就无法做持续优化与回归校验。

### 2.5 可扩展性与未来演进
1. 本项目首版聚焦单机、本地优先，但必须预留生产化迁移路径。
2. 当前架构要支持后续扩展到新向量库、新模型、新评估器、新加载器。
3. 在能力上要支持从 RAG 扩展到 Agentic RAG、Memory、Multi-Agent。

设计思想与优势：
1. 首版先做最小闭环，但不阻断后续演进。
2. 将系统拆成多个稳定模块，便于局部替换。
3. 为教学和面试提供自然的“从 v1 到 v2”的演进叙事。

工程化难点：
1. 需要避免为了未来扩展而过度设计。
2. 需要在局部简单和整体演进之间保持平衡。
3. 扩展点必须真实可落地，而不是只停留在概念预留。

### 2.6 Agent Loop 与智能体扩展
1. 本项目不把 Agent 作为独立附属物，而是作为 RAG 之上的自然扩展层。
2. 首版提供最小 Thought → Action → Observation 闭环。
3. 后续扩展到 Planner、Executor、Retriever、Evaluator 等多角色协作。

设计思想与优势：
1. 先做最小可解释闭环，保证能调试、能讲清。
2. Tool System 与 Memory System 复用同一底层架构，避免体系分裂。
3. Agent Trace 为后续 trajectory 评估与故障分析奠定基础。

工程化难点：
1. 工具调度、状态管理、循环终止条件都容易出问题。
2. Agent 失败不只是模型问题，常常是系统设计问题。
3. 若没有统一 Tool 抽象和 Trace，很难真正扩展到 Multi-Agent。

## 3. 设计原则
1. 本地优先：默认本地运行、本地存储、本地调试，核心持久化使用 SQLite，本地向量库存储优先使用 Chroma。
2. 强约束分层：严格区分 `application / domain / infrastructure / interface / observability / evaluation / agent`，禁止跨层直接耦合。
3. 可插拔优先：LLM、Embedding、Reranker、VectorStore、Splitter、Evaluator、MemoryStore 必须全部抽象化。
4. 配置驱动：所有后端切换通过 `settings.yaml` 和环境变量完成，禁止在业务代码中硬编码 provider 分支。
5. 可观测优先：所有核心链路必须产出结构化 Trace 和 JSON Lines 日志。
6. 面试友好：每个模块必须说明设计动机、替换点、扩展点。
7. 测试先行：核心领域逻辑优先 TDD，所有回归问题必须附带测试。
8. 不引入重框架：禁止使用 LangChain、LlamaIndex 做编排，仅允许 `RecursiveCharacterTextSplitter`。

## 4. 非目标
1. 不提供 HTTP 服务。
2. 不做在线多租户 SaaS。
3. 首版不实现完整 Multi-Agent，只提供接口与预留架构。
4. 首版不做分布式部署，不做复杂权限系统。

## 5. 总体架构
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

## 6. 目录结构
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

## 7. 模块职责表
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

## 8. 核心领域模型
### 8.1 Document
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

### 8.2 Chunk
```python
from pydantic import BaseModel, Field
from typing import Any


class Chunk(BaseModel):
    chunk_id: str
    doc_id: str
    chunk_index: int
    raw_text: str
    normalized_text: str
    enriched_text: str | None = None
    title_path: list[str] = Field(default_factory=list)
    page_range: list[int] = Field(default_factory=list)
    image_captions: list[str] = Field(default_factory=list)
    metadata: dict[str, Any] = Field(default_factory=dict)
    sparse_text: str
    dense_text: str
    token_count: int
    content_hash: str
    embedding_model_version: str
    embedding_fingerprint: str
    is_deleted: bool = False
```

### 8.3 QueryContext
```python
class QueryRequest(BaseModel):
    query: str
    collection: str
    collection_ids: list[str] | None = None
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

## 9. 可插拔抽象接口
### 9.1 LLM
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

### 9.2 Embedding
```python
class BaseEmbedding(ABC):
    @abstractmethod
    def embed_texts(self, texts: list[str]) -> list[list[float]]: ...

    @abstractmethod
    def embed_query(self, text: str) -> list[float]: ...
```

### 9.3 Reranker
```python
class BaseReranker(ABC):
    @abstractmethod
    def rerank(self, query: str, documents: list[str]) -> list[tuple[int, float]]: ...
```

### 9.4 VectorStore
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

### 9.5 Splitter
```python
class BaseSplitter(ABC):
    @abstractmethod
    def split_markdown(self, doc: Document) -> list[Chunk]: ...
```

### 9.6 Evaluator
```python
class BaseEvaluator(ABC):
    @abstractmethod
    def evaluate(self, dataset_path: str, run_id: str) -> dict: ...
```

### 9.7 Tool
```python
class BaseTool(ABC):
    name: str
    description: str

    @abstractmethod
    def schema(self) -> dict: ...

    @abstractmethod
    def run(self, payload: dict) -> dict: ...
```

### 9.8 MemoryStore
```python
class BaseMemoryStore(ABC):
    @abstractmethod
    def add_memory(self, memory: dict) -> str: ...

    @abstractmethod
    def search_memory(self, query: str, user_id: str, top_k: int) -> list[dict]: ...

    @abstractmethod
    def summarize_memory(self, conversation_id: str) -> dict: ...
```

## 10. Provider Factory
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

## 11. 配置设计
### 11.1 settings.yaml
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
  score_threshold: 0.35
  max_seq_length: 512
  rerank_text_max_chars: 2000
  rerank_context_preview_chars: 200

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
  context_budget_tokens: 512

retrieval:
  dense_top_k_multiplier: 3
  rrf_k: 60
  retrieval_budget_tokens: 2048
  enable_multi_collection_interface: true
  allow_degraded_search: true
  enable_parallel_recall: true

evaluation:
  provider: ragas
  dataset_dir: ./data/eval
  metrics: [hit_rate, mrr, faithfulness, answer_relevancy]

bm25:
  artifact_dir: ./data/processed/bm25
  active_artifact: bm25_active.pkl
  building_artifact: bm25_building.pkl
  meta_file: bm25_meta.json
  rebuild_debounce_seconds: 30
  deleted_ratio_rebuild_threshold: 0.2
  deleted_count_rebuild_threshold: 1000
  search_prefetch_multiplier: 4
  enable_force_rebuild: true

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

### 11.2 Settings 规则
1. 所有敏感项只能从环境变量读取。
2. `settings.yaml` 只保存结构和默认值。
3. 配置加载顺序：默认值 < `settings.yaml` < `.env` < 环境变量。
4. 启动时必须做配置完整性校验，缺关键字段直接失败。

## 12. Ingestion Pipeline
### 12.1 流程
`PDF -> Markdown -> Split -> LLM Enrich -> Dual Embedding -> Upsert -> Trace -> Incremental Record`

### 12.2 详细步骤
1. 文件发现：支持 `单文件导入` 与 `目录批量导入` 两种入口，目录模式复用单文件处理链路。
2. 哈希计算：对原始文件计算 `SHA256`。
3. 增量跳过：查询 `ingestion_history`，若哈希已成功处理则直接跳过。
4. PDF 转 Markdown：调用 MarkItDown，产出统一 Markdown 文本。
5. 图片抽取：识别页面图片并获取二进制数据或引用。
6. Markdown 规范化：在 Loader 阶段完成基础清噪与结构规范化，包括页眉页脚、页码噪声、重复标题清理，并尽量保留 `# / ##` 标题层级与块级结构。
7. 智能分块：使用 `RecursiveCharacterTextSplitter` 按 Markdown 标题与段落切分，默认 `chunk_size=900`、`chunk_overlap=180`。
8. LLM 增强：
   - 增强粒度为 `chunk-level processing with local context window`，即“先切 chunk，再以当前 chunk 为中心，附带邻接上下文做增强”。
   - 重写：仅做 retrieval-oriented enrichment，清理碎片、补足上下文指代，不允许自由摘要式改写。
   - 元数据注入：标题路径、页码范围、来源文档等。
   - 图片描述：Vision LLM 对图片生成中文描述。
   - 降级策略：若增强失败，则回退到 `normalized_text`，不得阻断整条链路。
8. 双路文本构建：
   - `sparse_text = raw_text/normalized_text + 标题 + 关键词`
   - `dense_text = enriched_text + 图片描述 + 元数据上下文`
10. Embedding：仅对 `dense_text` 做向量化，并写入 `embedding_model_version` 与 `embedding_fingerprint`。
11. Upsert：将 chunk、metadata、vector 写入 Chroma。
12. BM25 建索引：以 SQLite `chunks` 表为唯一事实源，异步维护本地 BM25 artifact。
13. 中间产物持久化：保留原始 Markdown、规范化 Markdown、chunk 切分结果、enriched 结果，供 Dashboard、调试与教学使用。
14. 记录 Trace、日志与 ingestion_history。
15. 若触发删除链路，则执行反向同步：删除 Chroma 数据、软删除 SQLite chunks、更新 BM25 artifact，并逻辑保留 trace。

### 12.3 Ingestion I/O
```python
class IngestionResult(BaseModel):
    doc_id: str
    file_name: str
    chunk_count: int
    skipped: bool
    trace_id: str
```

### 12.4 ingestion_history 表
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

### 12.5 设计说明
这样设计的原因是把“是否需要重跑”前置到最便宜的阶段，避免重复调用 MarkItDown、Vision LLM、Embedding。面试时可强调“零成本增量更新”和“幂等摄取”。

### 12.6 BM25 索引生命周期设计
1. SQLite `chunks` 表是 BM25 的唯一事实源，BM25 `.pkl` 只是派生索引，不是真实数据源。
2. 进程内维护 `BM25Searcher` 活跃实例，查询线程只读，不参与重建过程。
3. 磁盘上至少维护：
   - `bm25_active.pkl`
   - `bm25_building.pkl`
   - `bm25_meta.json`
4. `pickle` artifact 必须同时保存：
   - `bm25 model`
   - `row_index -> chunk_id`
   - `chunk_id -> row_index`
   - `index_version / build metadata`
5. 新增或更新文档后，采用“后台全量重建 + 内存原子替换”的双缓冲方案。
6. 删除文档后，优先在查询结果层按 `is_deleted` 过滤，达到阈值后再触发彻底重建。
7. 原子切换前，必须对 `bm25_building.pkl` 做一次 `load_test`，确认新 artifact 可被正常读取，防止坏文件上线。
8. 重建过程不得阻塞搜索请求，最终语义为“查询不停机 + 最终一致”。

### 12.7 BM25 重建调度策略
1. 默认启用“防抖自动触发”，保证普通用户无需手动维护索引即可在短时间内达到最终一致。
2. 同时提供“显式强制重建”，用于批量导入后立即确认、异步任务异常恢复、分词/索引逻辑迁移等场景。
3. 自动触发与强制触发必须通过 `Rebuild Lock` 互斥：
   - 自动请求只会启动或重置定时器。
   - 强制请求会取消现有定时器，并立即发起后台重建任务。
4. 若已有重建线程在运行，新的请求不能并发打架，应等待完成或将旧任务标记失效。
5. 该设计兼顾系统自觉性与高级用户掌控感，是首版可落地且易讲清楚的折中方案。

### 12.8 SQLite 元数据表设计总则
1. `documents / chunks / document_images / ingestion_history` 是 Ingestion 首版的四张核心元数据表。
2. `documents / chunks / document_images` 采用软删除策略；`ingestion_history` 永久保留，不做物理删除。
3. 大型中间产物不直接内嵌存储在 SQLite 中，而是落盘到 `data/processed/`，数据库仅保存引用路径。
4. 所有字段在设计时分为三类：
   - `首版必须保留`：首版实现、删除反向链路、BM25/Chroma 重建、Dashboard、教学演示都依赖。
   - `首版建议保留`：首版不一定作为主流程强依赖，但保留后能显著提升调试性、可讲解性与后续扩展质量。
   - `扩展保留`：首版可以不参与核心逻辑，但建议在 schema 中预留或在文档中明确保留意图。
5. `logical_doc_id + version` 是文档版本演进的核心设计，必须写入首版 schema。它不仅服务工程更新语义，也服务教学演示、实验对比与面试表达。
6. `embedding_fingerprint` 是首版必须保留的高级字段，用于解决 embedding 模型静默升级、增强逻辑变化、dense_text 变化后向量状态失真但系统无感知的问题。

### 12.9 documents 表设计
```sql
CREATE TABLE documents (
  doc_id TEXT PRIMARY KEY,
  logical_doc_id TEXT NOT NULL,
  version INTEGER NOT NULL DEFAULT 1,

  collection_name TEXT NOT NULL,
  source_path TEXT NOT NULL,
  file_name TEXT NOT NULL,
  file_ext TEXT NOT NULL DEFAULT 'pdf',

  sha256 TEXT NOT NULL,
  file_size_bytes INTEGER,
  mime_type TEXT DEFAULT 'application/pdf',

  title TEXT,
  author TEXT,
  language TEXT DEFAULT 'zh',
  page_count INTEGER DEFAULT 0,

  raw_markdown_path TEXT,
  normalized_markdown_path TEXT,

  loader_name TEXT NOT NULL,
  loader_version TEXT,
  normalization_version TEXT,

  status TEXT NOT NULL,
  is_active INTEGER NOT NULL DEFAULT 1,
  is_deleted INTEGER NOT NULL DEFAULT 0,

  deleted_at TEXT,
  deleted_reason TEXT,

  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  indexed_at TEXT,

  UNIQUE(logical_doc_id, version)
);
```

字段设计说明：

| 字段 | 含义 | 保留级别 | 设计备注 |
|---|---|---|---|
| `doc_id` | 当前文档版本实例 ID | 首版必须保留 | 物理文档主键，建议包含版本信息。 |
| `logical_doc_id` | 逻辑文档稳定 ID | 首版必须保留 | 同一份文档多次更新仍保持不变；用于版本演进、对比实验和删除反向链路。 |
| `version` | 当前逻辑文档的版本号 | 首版必须保留 | 支持“同一文档不同切分/增强/索引策略”的教学与实验演示。 |
| `collection_name` | 所属知识库集合 | 首版必须保留 | 便于按 collection 做导入、删除、重建与 Dashboard 展示。 |
| `source_path` | 原始文件路径 | 首版必须保留 | 用于追踪来源与重复导入判断。 |
| `file_name` | 文件名 | 首版必须保留 | Dashboard 与引用展示直接使用。 |
| `file_ext` | 文件扩展名 | 首版必须保留 | 首版实现只处理 PDF，但接口要支持未来扩展。 |
| `sha256` | 文件级哈希 | 首版必须保留 | 文件级幂等跳过核心字段。 |
| `file_size_bytes` | 文件大小 | 首版必须保留 | 便于导入统计、异常排查与 UI 展示。 |
| `mime_type` | MIME 类型 | 首版建议保留 | 首版默认 PDF，后续扩展 HTML/Markdown 时更通用。 |
| `title` | 文档标题 | 首版建议保留 | 可从内容推断或手工填充，便于展示。 |
| `author` | 文档作者 | 扩展保留 | 首版不强依赖，后续可用于文档属性增强。 |
| `language` | 语言标记 | 首版建议保留 | 后续分词、提示词和评估策略可参考。 |
| `page_count` | 总页数 | 首版必须保留 | 与图片、页码引用、Trace 展示密切相关。 |
| `raw_markdown_path` | 原始 Markdown 产物路径 | 首版必须保留 | 用于区分解析错误和规范化错误，是教学与调试关键资产。 |
| `normalized_markdown_path` | 规范化 Markdown 路径 | 首版必须保留 | 是 Splitter 的直接输入引用。 |
| `loader_name` | 使用的 Loader 名称 | 首版必须保留 | 首版通常为 MarkItDown PDF Loader。 |
| `loader_version` | Loader 版本 | 首版建议保留 | 首版不强依赖，但解析器升级后便于识别历史文档。 |
| `normalization_version` | 规范化逻辑版本 | 首版必须保留 | 页眉页脚清理、标题去重等策略变化后可用于重跑判定。 |
| `status` | 当前文档状态 | 首版必须保留 | 建议枚举：`pending / processing / success / failed / deleted`。 |
| `is_active` | 是否为当前活跃版本 | 首版必须保留 | 文档更新采用“旧版本失活 + 新版本插入”。 |
| `is_deleted` | 是否逻辑删除 | 首版必须保留 | 反向删除链路与查询过滤依赖。 |
| `deleted_at` | 删除时间 | 首版建议保留 | 有助于审计与清理任务。 |
| `deleted_reason` | 删除原因 | 扩展保留 | 首版通常是用户手动删除，不是关键逻辑字段，但建议预留口径。 |
| `created_at` | 创建时间 | 首版必须保留 | 审计和排序展示必需。 |
| `updated_at` | 更新时间 | 首版必须保留 | 版本和状态变化追踪。 |
| `indexed_at` | 最近一次索引完成时间 | 首版必须保留 | 便于判断摄取与检索资产是否已同步。 |

### 12.10 chunks 表设计
```sql
CREATE TABLE chunks (
  chunk_id TEXT PRIMARY KEY,
  doc_id TEXT NOT NULL,
  logical_doc_id TEXT NOT NULL,
  collection_name TEXT NOT NULL,

  chunk_index INTEGER NOT NULL,
  parent_section_id TEXT,
  section_path TEXT,
  title_path_json TEXT NOT NULL,

  page_start INTEGER,
  page_end INTEGER,

  raw_text TEXT NOT NULL,
  normalized_text TEXT NOT NULL,
  enriched_text TEXT,

  sparse_text TEXT NOT NULL,
  dense_text TEXT NOT NULL,

  token_count INTEGER DEFAULT 0,
  char_count INTEGER DEFAULT 0,

  content_hash TEXT NOT NULL,
  chunking_version TEXT NOT NULL,
  enrichment_version TEXT NOT NULL,
  caption_version TEXT,

  embedding_provider TEXT NOT NULL,
  embedding_model TEXT NOT NULL,
  embedding_model_version TEXT NOT NULL,
  embedding_dimension INTEGER,
  embedding_fingerprint TEXT NOT NULL,

  vector_store_provider TEXT DEFAULT 'chroma',
  vector_store_collection TEXT,
  bm25_index_version TEXT,

  image_count INTEGER DEFAULT 0,
  has_image INTEGER NOT NULL DEFAULT 0,
  synthetic_image_chunk INTEGER NOT NULL DEFAULT 0,

  is_active INTEGER NOT NULL DEFAULT 1,
  is_deleted INTEGER NOT NULL DEFAULT 0,
  deleted_at TEXT,

  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,

  FOREIGN KEY(doc_id) REFERENCES documents(doc_id)
);
```

字段设计说明：

| 字段 | 含义 | 保留级别 | 设计备注 |
|---|---|---|---|
| `chunk_id` | chunk 主键 | 首版必须保留 | 建议全局唯一，服务 Chroma、BM25 映射、引用与删除。 |
| `doc_id` | 所属文档版本 ID | 首版必须保留 | 关联当前物理文档版本。 |
| `logical_doc_id` | 所属逻辑文档 ID | 首版必须保留 | 支持版本级批量操作与教学对比。 |
| `collection_name` | 所属集合 | 首版必须保留 | 冗余字段，但能明显降低 BM25 全量重建、Dashboard 过滤、局部删除的 join 成本。 |
| `chunk_index` | 文档内顺序号 | 首版必须保留 | 用于还原顺序、拼接局部上下文窗口。 |
| `parent_section_id` | 父 section ID | 首版建议保留 | 为后续 parent-child retrieval 预留。 |
| `section_path` | section 层级路径 | 首版建议保留 | 便于章节级聚合与调试。 |
| `title_path_json` | 标题路径 | 首版必须保留 | 是语义分块和引用展示的核心元数据。 |
| `page_start` | 起始页 | 首版必须保留 | 用于定位和引用。 |
| `page_end` | 结束页 | 首版必须保留 | 用于定位和引用。 |
| `raw_text` | 原始块文本 | 首版必须保留 | 直接反映解析后块级内容，是 debug 基线。 |
| `normalized_text` | 规范化文本 | 首版必须保留 | 是结构清洗后的稳定文本。 |
| `enriched_text` | 增强文本 | 首版必须保留 | retrieval-oriented enrichment 的结果；失败时允许为空并回退到 `normalized_text`。 |
| `sparse_text` | 稀疏检索文本 | 首版必须保留 | 服务 BM25，可包含原文、标题、关键词。 |
| `dense_text` | 稠密检索文本 | 首版必须保留 | 服务 embedding 检索，可包含增强结果、图片描述和上下文元数据。 |
| `token_count` | token 数 | 首版必须保留 | 便于控制检索与上下文预算。 |
| `char_count` | 字符数 | 首版必须保留 | 便于快速统计与调试。 |
| `content_hash` | chunk 内容哈希 | 首版必须保留 | 为未来 chunk 级 diff 和局部重建预留。 |
| `chunking_version` | 切分逻辑版本 | 首版必须保留 | `chunk_size`、`overlap`、separator 改变后用于识别脏 chunk。 |
| `enrichment_version` | 增强逻辑版本 | 首版必须保留 | prompt、局部上下文窗口、元数据注入变化后可用于重建。 |
| `caption_version` | 图片描述逻辑版本 | 首版建议保留 | 首版可不强依赖，但多模态策略迭代时很有帮助。 |
| `embedding_provider` | embedding 服务商 | 首版必须保留 | 与向量重建和问题排查直接相关。 |
| `embedding_model` | embedding 模型名 | 首版必须保留 | 向量状态的基本标识。 |
| `embedding_model_version` | embedding 模型版本 | 首版必须保留 | 更换模型或维度时可识别旧 chunk。 |
| `embedding_dimension` | 向量维度 | 首版必须保留 | 有助于迁移和完整性校验。 |
| `embedding_fingerprint` | 向量状态指纹 | 首版必须保留 | 由 `dense_text + provider + model + model_version + enrichment_version` 组合生成，是识别“静默脏向量”的核心字段。 |
| `vector_store_provider` | 向量库类型 | 首版建议保留 | 首版默认 Chroma，后续迁移到其他向量库时可复用。 |
| `vector_store_collection` | 向量库 collection 名称 | 首版建议保留 | 便于删除和重建时定位。 |
| `bm25_index_version` | 当前归属 BM25 artifact 版本 | 首版建议保留 | 首版不是硬依赖，但对 BM25 一致性调试和教学展示很有帮助。 |
| `image_count` | 关联图片数 | 首版必须保留 | 多模态展示和分析直接需要。 |
| `has_image` | 是否含图 | 首版必须保留 | 便于快速筛选和调试。 |
| `synthetic_image_chunk` | 是否为图片辅助 chunk | 首版建议保留 | 当图片无法自然归属时生成的辅助块标记。 |
| `is_active` | 是否活跃 | 首版必须保留 | 文档更新采用“旧版本失活 + 新版本插入”。 |
| `is_deleted` | 是否逻辑删除 | 首版必须保留 | BM25 删除过滤与查询侧一致性依赖。 |
| `deleted_at` | 删除时间 | 首版必须保留 | 删除审计与阈值重建分析需要。 |
| `created_at` | 创建时间 | 首版必须保留 | 审计与排序展示。 |
| `updated_at` | 更新时间 | 首版必须保留 | 状态变化追踪。 |

### 12.11 document_images 表设计
```sql
CREATE TABLE document_images (
  image_id TEXT PRIMARY KEY,
  doc_id TEXT NOT NULL,
  logical_doc_id TEXT NOT NULL,
  chunk_id TEXT,

  page_number INTEGER NOT NULL,
  image_index_on_page INTEGER DEFAULT 0,

  image_path TEXT,
  image_sha256 TEXT,

  caption_text TEXT,
  caption_model_provider TEXT,
  caption_model_name TEXT,
  caption_model_version TEXT,
  caption_prompt_version TEXT,

  placement_strategy TEXT NOT NULL,
  placement_confidence REAL DEFAULT 0.0,

  width INTEGER,
  height INTEGER,

  is_orphan INTEGER NOT NULL DEFAULT 0,
  is_deleted INTEGER NOT NULL DEFAULT 0,
  deleted_at TEXT,

  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,

  FOREIGN KEY(doc_id) REFERENCES documents(doc_id),
  FOREIGN KEY(chunk_id) REFERENCES chunks(chunk_id)
);
```

字段设计说明：

| 字段 | 含义 | 保留级别 | 设计备注 |
|---|---|---|---|
| `image_id` | 图片主键 | 首版必须保留 | 图片级追踪与 caption 管理基础。 |
| `doc_id` | 所属文档版本 ID | 首版必须保留 | 绑定当前文档版本。 |
| `logical_doc_id` | 所属逻辑文档 ID | 首版必须保留 | 支持版本级图片追踪。 |
| `chunk_id` | 归属 chunk ID | 首版必须保留 | 入库时就确定归属，避免查询阶段重复猜测。 |
| `page_number` | 图片页码 | 首版必须保留 | 与页级定位强相关。 |
| `image_index_on_page` | 页内图片序号 | 首版必须保留 | 同页多图场景需要。 |
| `image_path` | 图片文件路径 | 首版必须保留 | Dashboard 展示与重处理依赖。 |
| `image_sha256` | 图片哈希 | 首版必须保留 | 去重与缓存可依赖。 |
| `caption_text` | 图片描述文本 | 首版必须保留 | 多模态能力核心资产。 |
| `caption_model_provider` | caption 模型服务商 | 首版必须保留 | 排查 caption 效果问题时需要。 |
| `caption_model_name` | caption 模型名 | 首版必须保留 | 与 provider 一起构成 caption 来源。 |
| `caption_model_version` | caption 模型版本 | 首版建议保留 | 首版不强依赖，但后续模型更替时有价值。 |
| `caption_prompt_version` | caption prompt 版本 | 首版建议保留 | 便于复盘 caption 策略变化。 |
| `placement_strategy` | 图片归属策略 | 首版必须保留 | 建议枚举：`inline_nearest_text / nearest_heading_block / synthetic_image_chunk`。 |
| `placement_confidence` | 图片归属置信度 | 扩展保留 | 首版通常没有成熟打分算法，但可预留给未来更智能的归属策略。 |
| `width` | 图片宽度 | 首版建议保留 | 成本低，后续图片展示和规则调优有用。 |
| `height` | 图片高度 | 首版建议保留 | 与 `width` 同理。 |
| `is_orphan` | 是否未自然归属 | 首版必须保留 | 后续生成 `synthetic_image_chunk` 或人工复查时很关键。 |
| `is_deleted` | 是否逻辑删除 | 首版必须保留 | 与文档删除链路保持一致。 |
| `deleted_at` | 删除时间 | 首版建议保留 | 有助于删除审计。 |
| `created_at` | 创建时间 | 首版必须保留 | 审计用途。 |
| `updated_at` | 更新时间 | 首版必须保留 | 状态变化追踪。 |

### 12.12 ingestion_history 表设计
```sql
CREATE TABLE ingestion_history (
  ingestion_id TEXT PRIMARY KEY,

  logical_doc_id TEXT,
  doc_id TEXT,

  collection_name TEXT NOT NULL,
  source_path TEXT NOT NULL,
  file_name TEXT NOT NULL,
  sha256 TEXT NOT NULL,

  trigger_type TEXT NOT NULL,
  operation_type TEXT NOT NULL,

  status TEXT NOT NULL,
  error_stage TEXT,
  error_message TEXT,

  raw_markdown_path TEXT,
  normalized_markdown_path TEXT,
  chunk_snapshot_path TEXT,
  enriched_snapshot_path TEXT,

  page_count INTEGER DEFAULT 0,
  image_count INTEGER DEFAULT 0,
  chunk_count INTEGER DEFAULT 0,
  affected_chunks_count INTEGER DEFAULT 0,

  skipped_by_hash INTEGER NOT NULL DEFAULT 0,
  skipped_reason TEXT,

  bm25_rebuild_requested INTEGER NOT NULL DEFAULT 0,
  bm25_rebuild_mode TEXT,
  bm25_index_version_before TEXT,
  bm25_index_version_after TEXT,

  trace_id TEXT,
  started_at TEXT NOT NULL,
  finished_at TEXT,
  duration_ms INTEGER
);
```

字段设计说明：

| 字段 | 含义 | 保留级别 | 设计备注 |
|---|---|---|---|
| `ingestion_id` | 摄取任务 ID | 首版必须保留 | 每一次任务的唯一主键，而不是只记录文档最终状态。 |
| `logical_doc_id` | 逻辑文档 ID | 首版必须保留 | 便于跨版本追踪任务。 |
| `doc_id` | 物理文档版本 ID | 首版必须保留 | 与当次处理的具体文档绑定。 |
| `collection_name` | 目标集合 | 首版必须保留 | Dashboard 和重建调度需要。 |
| `source_path` | 输入文件路径 | 首版必须保留 | 排错和重放任务基础。 |
| `file_name` | 文件名 | 首版必须保留 | 展示与筛选。 |
| `sha256` | 文件哈希 | 首版必须保留 | 与幂等跳过直接相关。 |
| `trigger_type` | 任务触发来源 | 首版必须保留 | 建议枚举：`manual_file / manual_directory / scheduled_sync / reindex / delete`。 |
| `operation_type` | 任务操作类型 | 首版必须保留 | 建议枚举：`insert / update / delete / rebuild`。 |
| `status` | 当前任务状态 | 首版必须保留 | 建议枚举：`pending / processing / success / failed / skipped`。 |
| `error_stage` | 失败阶段 | 首版必须保留 | 建议记录：`hash_check / load_markdown / normalize_markdown / split / caption / enrich / embed / vector_upsert / sqlite_commit / bm25_rebuild`。 |
| `error_message` | 错误信息 | 首版必须保留 | 失败排查关键字段。 |
| `raw_markdown_path` | 原始 Markdown 快照路径 | 首版必须保留 | Trace 和教学演示直接需要。 |
| `normalized_markdown_path` | 规范化 Markdown 路径 | 首版必须保留 | 对比结构清洗效果。 |
| `chunk_snapshot_path` | chunk 快照路径 | 首版必须保留 | 用于查看切块结果。 |
| `enriched_snapshot_path` | 增强结果快照路径 | 首版必须保留 | 用于查看 enrichment 效果。 |
| `page_count` | 处理页数 | 首版必须保留 | 任务统计与异常排查。 |
| `image_count` | 图片数 | 首版必须保留 | 多模态统计。 |
| `chunk_count` | 产出 chunk 数 | 首版必须保留 | 描述本次生成了多少碎片。 |
| `affected_chunks_count` | 实际影响 chunk 数 | 首版必须保留 | 与 `chunk_count` 语义不同，更适合 Dashboard 展示更新/删除任务影响范围。 |
| `skipped_by_hash` | 是否因哈希跳过 | 首版必须保留 | 增量逻辑透明化关键字段。 |
| `skipped_reason` | 跳过原因 | 首版必须保留 | 便于向用户解释为什么没有重跑。 |
| `bm25_rebuild_requested` | 是否请求 BM25 重建 | 首版必须保留 | 记录任务是否影响稀疏索引。 |
| `bm25_rebuild_mode` | BM25 重建模式 | 首版必须保留 | 建议枚举：`debounced_auto / forced_manual / delete_threshold / full_rebuild`，完整体现“自动防抖 + 强制重建”的调度设计。 |
| `bm25_index_version_before` | 重建前 BM25 版本 | 首版建议保留 | 对比重建前后状态很有帮助。 |
| `bm25_index_version_after` | 重建后 BM25 版本 | 首版必须保留 | 审计与 Dashboard 展示可直接使用。 |
| `trace_id` | 关联 Trace ID | 首版必须保留 | 将任务记录与可观测链路串起来。 |
| `started_at` | 开始时间 | 首版必须保留 | 审计和任务排序基础。 |
| `finished_at` | 结束时间 | 首版必须保留 | 计算耗时与状态判定。 |
| `duration_ms` | 总耗时 | 首版必须保留 | Dashboard 和性能调优直接使用。 |

### 12.13 中间产物存储策略
1. 下列中间产物必须保留，并以文件形式落盘到 `data/processed/`：
   - 原始 Markdown
   - 规范化 Markdown
   - chunk 快照
   - enriched chunk 快照
2. SQLite 只保存这些中间产物的路径引用，不直接保存大段文本快照。
3. 建议目录结构：

```text
data/processed/
  documents/{doc_id}/raw.md
  documents/{doc_id}/normalized.md
  documents/{doc_id}/chunks.jsonl
  documents/{doc_id}/enriched_chunks.jsonl
  bm25/
    bm25_active.pkl
    bm25_building.pkl
    bm25_meta.json
```

### 12.14 文档更新与删除语义
1. 文档更新采用“旧版本失活 + 新版本插入”的版本演进语义，而不是原地覆盖。
2. 当新版本插入成功后：
   - 旧 `documents.is_active = 0`
   - 旧 `chunks.is_deleted = 1`
   - 新版本写入新的 `doc_id` 和新的 chunk 集合
3. 文档删除采用“软删除 + 外部索引物理删除”的组合语义：
   - SQLite 中 `documents / chunks / document_images` 做软删除
   - Chroma 中对应向量物理删除
   - BM25 查询侧先过滤软删除项，再由异步重建修复索引状态
4. `trace` 与 `ingestion_history` 在删除场景中逻辑保留，不物理删除。

## 13. Retrieval Pipeline
### 13.1 查询主链路
`Query Normalize -> Memory Recall -> Dense Retrieve -> BM25 Retrieve -> RRF Fusion -> Rerank -> Context Build -> Answer Generate`

### 13.2 详细步骤
1. 查询标准化：去空白、统一大小写、保留原始 query。
2. 检索边界：首版实际执行仅在单 `collection_name` 内检索，但接口层预留 `collection_ids: list[str]`，为“个人库 + 公共库”等多集合检索场景留接口。
3. Memory 处理：首版不做 query rewrite，不让 memory 参与召回阶段；memory 仅在 Context Build 阶段以 `Memory-Augmented Context` 方式注入。
4. 并行召回：Dense 与 Sparse 检索必须通过 `asyncio.gather()` 并行启动，避免串行等待导致 TTFT 近似翻倍。
5. Dense 检索：向量检索默认获取 `max(top_k_retrieval, top_k_rerank * dense_top_k_multiplier)` 个候选，仅检索当前 collection 内 `is_deleted = 0` 且 `is_active = 1` 的 chunk。
6. Sparse 检索：BM25 初始获取比目标更大的候选集，默认 `top_k_retrieval * search_prefetch_multiplier`，然后过滤软删除项与无效项。
7. RRF 融合：使用统一 chunk_id 去重融合，并保留每个候选的召回来源、各自 rank、原始分数与融合分数。
8. 融合统计：计算 `fused_overlap_count`，记录有多少 chunk 同时被 Dense 和 Sparse 命中，用于衡量双路一致性。
9. 精排：
   - 默认使用 `BGE-Reranker` 或同类开源 Cross-Encoder。
   - 配置开启时可使用 LLM Rerank，但它始终是高成本实验路径或高价值查询开关，而非默认路径。
   - Rerank 输入不直接传散装 metadata，而是由 Retrieval 层构造统一 `rerank_text`。
10. 阈值判断：
   - 记录 `rerank_top_score`。
   - 基于绝对 `score_threshold` 产出 `rerank_threshold_passed`。
   - 若阈值未通过，RetrievalService 只标记“相关性不足”，最终是否拒答由 QueryService 决定。
11. Top-K 截断：保留前 `top_k_rerank` 候选，并以结构化结果返回。
12. Context Build：在 QueryService 中按“两段式 Token Budget”组装上下文：
   - `memory_budget_tokens`
   - `retrieval_budget_tokens`
13. 答案生成：LLM 根据上下文与约束 prompt 作答。
14. 输出引用：返回 chunk 来源、页码、文档名。
15. 写入 Query Trace 与会话短期记忆。

### 13.3 RRF
```python
def rrf_fuse(rank_lists: list[list[str]], k: int = 60) -> dict[str, float]:
    scores = {}
    for rank_list in rank_lists:
        for rank, chunk_id in enumerate(rank_list, start=1):
            scores[chunk_id] = scores.get(chunk_id, 0.0) + 1.0 / (k + rank)
    return scores
```

### 13.4 精排策略
1. Cross-Encoder 适合作为默认精排，成本更稳定。
2. LLM Rerank 仅用于高价值查询或实验开关。
3. 重排输入必须包含 `query + rerank_text`，不得只看裸文本。
4. `rerank_text` 以 `enriched_text` 为主，并拼接标题路径、页码范围、图片描述摘要等信息，保证 Cross-Encoder 能感知结构上下文。
5. 若 `enriched_text` 过长，必须在 Retrieval 层做预截断，必要时附带邻接内容预览，避免超过 reranker 的 `max_seq_length`。
6. 若 Cross-Encoder 失败，则自动降级到 fused 排序；若 LLM Rerank 失败，则自动回退到 Cross-Encoder，再失败则回退 fused 排序。

### 13.5 设计说明
Hybrid Retrieval 解决关键词匹配与语义召回互补问题，RRF 保证融合算法简单可解释，Rerank 负责把候选集合提升到可生成答案的精度。

### 13.6 Retrieval 职责边界
1. `RetrievalService` 只负责：
   - Dense recall
   - Sparse recall
   - RRF fusion
   - Rerank
   - 结构化返回候选、中间分数、降级状态和质量判断
2. `QueryService` 负责：
   - Memory-Augmented Context 组装
   - Token budget 截断
   - 最终回答/拒答策略
   - LLM 生成与引用输出
3. 这种拆分的意义是保证“检索质量判断”和“最终回答策略判断”分层明确，便于调试、评估与单元测试。

### 13.7 Retrieval 输出结构
```python
class RetrievedCandidate(BaseModel):
    chunk_id: str
    collection_name: str
    source_type: str  # dense / sparse
    raw_score: float
    rank: int
    chunk: Chunk
    matched_terms: list[str] | None = None


class FusedCandidate(BaseModel):
    chunk_id: str
    collection_name: str
    dense_rank: int | None = None
    sparse_rank: int | None = None
    dense_score: float | None = None
    sparse_score: float | None = None
    rrf_score: float
    sources: list[str]
    chunk: Chunk


class RerankedCandidate(BaseModel):
    chunk_id: str
    collection_name: str
    rerank_score: float
    rrf_score: float
    rerank_text: str
    chunk: Chunk


class RetrievalResult(BaseModel):
    query: str
    collection_name: str
    collection_ids: list[str]

    retrieval_status: str
    status_reason: str | None = None
    degraded_from: list[str]
    applied_pre_filters: list[str] = []
    applied_post_filters: list[str] = []

    dense_available: bool
    sparse_available: bool

    dense_candidates: list[RetrievedCandidate]
    sparse_candidates: list[RetrievedCandidate]
    fused_candidates: list[FusedCandidate]
    reranked_candidates: list[RerankedCandidate]

    fused_overlap_count: int
    rerank_top_score: float | None = None
    rerank_threshold_passed: bool = False

    selected_candidates: list[RerankedCandidate]
    trace_id: str
```

说明：
1. `retrieval_status` 同时表达“系统执行状态”和“结果质量判断”，建议枚举：
   - `success`
   - `dense_only`
   - `sparse_only`
   - `hybrid_without_rerank`
   - `no_relevant_result`
   - `index_not_ready`
   - `failed`
2. `dense_available / sparse_available` 用于显式表达索引可用性，便于 Dashboard、故障定位和后台自愈调度。
3. `matched_terms` 首版暂不强求，但建议在结构上预留。
4. `fused_overlap_count` 用于衡量双路检索一致性。若长期接近 0，说明稀疏检索与稠密检索在“各说各话”，需要回头检查分词策略、chunk 质量或 embedding 模型。
5. `applied_pre_filters / applied_post_filters` 用于记录本次查询实际命中的前置和后置过滤规则，便于 bad case 复盘与 Dashboard 调试。

### 13.8 rerank_text 构造策略
1. `rerank_text` 必须由 Retrieval 层统一构造，而不是把 metadata 散装传入 reranker。
2. 推荐模板：

```text
[Document]
File: {file_name}
Titles: {title_path}
Pages: {page_start}-{page_end}

[Content]
{enriched_text_truncated}

[Context Preview]
Prev: {prev_preview}
Next: {next_preview}

[Image Notes]
{image_caption_summary}
```

3. 构造原则：
   - 正文以 `enriched_text` 为中心，因为它已经补足了主语、省略和局部上下文，更适合 Cross-Encoder。
   - 若 `enriched_text` 过长，先按 `rerank_text_max_chars` 做预截断。
   - 可附带邻接 chunk 的前后预览 `context_window preview`，但不应喧宾夺主。
   - 图片描述只作为辅助信号，不应压过正文内容。

### 13.9 分数阈值与拒答协作
1. RetrievalService 负责基于绝对 `score_threshold` 产出：
   - `rerank_top_score`
   - `rerank_threshold_passed`
2. QueryService 负责根据这些信号，结合 Memory、Prompt Policy 与回答策略，决定：
   - 正常回答
   - 弱回答（说明证据不足）
   - 明确拒答（如“未检索到足够相关知识”）
3. 该分层可以避免检索模块越权直接决定最终文案，但又能让检索质量判断结构化沉淀。

### 13.10 Token Budget 截断策略
1. Context 组装必须采用“两段式 Token Budget”：
   - `memory_budget_tokens`
   - `retrieval_budget_tokens`
2. 截断基于最终 context block 的 token 数，而不是单独的 `dense_text` token 数。
3. 优先级策略：
   - 先保留 memory 摘要预算
   - 再按 rerank 顺序累加检索块
   - 一旦超过预算，后续 chunk 直接舍弃
4. 首版不做 chunk 内部再切割，保持逻辑简单、行为可解释。

### 13.11 降级与自愈策略
1. 整体语义：优先 Hybrid，逐层降级，查询不中断。
2. 允许的降级路径：
   - Dense 失败 -> 退化为 Sparse Only
   - Sparse 失败 -> 退化为 Dense Only
   - Rerank 失败 -> 退化为 Fused 排序
3. 若 `sparse_available = False`，系统应在后台自动尝试 BM25 artifact 重载或重建，而不是长期维持不可用状态。
4. 若 `dense_available = False`，应记录 collection 缺失、向量未建或 Chroma 异常的原因，并继续尝试单路稀疏检索。
5. 降级状态必须通过 `retrieval_status` 和 `status_reason` 返回，而不能只写日志。

### 13.12 过滤策略总则
1. 过滤原则：先解析、能前置则前置、无法前置则后置兜底。
2. 若底层索引支持且属于硬约束（Hard Filter），则在 Dense / Sparse 检索阶段做 Pre-filter，以缩小候选集、降低成本。
3. 无法前置的过滤（索引不支持、字段缺失或字段质量不稳定）在 Rerank 前统一做 Post-filter，作为 safety net。
4. 对缺失字段默认采取 `missing -> include` 的宽松策略，避免过早误杀召回。
5. 软偏好（Soft Preference，例如“更近期更好”“更高质量文档优先”）不做硬过滤，而应作为排序信号在融合或重排阶段加权。
6. 该策略的目标是：保证召回尽可能全，同时把错误和脏数据挡在最终答案之前。

## 14. Context 构建策略
上下文由四部分组成：

1. 记忆内容：短期对话摘要、长期偏好记忆、任务回顾记忆。
2. 检索片段正文。
3. 元数据：文档名、标题路径、页码范围、chunk 序号。
4. 图片描述：作为正文附加段落插入。

上下文模板：
```text
[Memory Summary]
{memory_summary}

[Retrieved Context 1]
[1] {doc_ref_id}
Source: {file_name} | Titles: {title_path} | Pages: {page_range}
Text: {dense_text}
Image Notes: {image_captions}

[Retrieved Context 2]
...
```

规则：
1. 上下文构建必须保留来源信息，便于引用。
2. 图像描述默认和所属 chunk 同级拼接，不单独建独立回答上下文。
3. 首版采用 `Memory-Augmented Context`：`Memory Summary + Retrieved Chunks`，不让 memory 改写 query。
4. 超长上下文必须按 token budget 截断，禁止无上限堆叠，也不使用固定 chunk 条数。
5. Context Build 阶段必须为每个注入的 chunk 生成唯一 `doc_ref_id`，并在 Retrieved Knowledge 中以 `[1]`, `[2]` 这类编号块形式展示，便于 LLM 学习并输出稳定引用。
6. 首版 Memory 注入只实现 `conversation summary`；`relevant long-term memory bullets` 作为 V2 扩展，在长期记忆向量检索能力成熟后再接入。

## 15. 多模态处理
### 15.1 处理策略
1. 不使用 CLIP。
2. 只做 `Image -> Vision LLM -> Text Caption`。
3. 将 caption 注入 `dense_text` 和 `image_captions` 字段。

### 15.2 融合方式
1. 若图片属于某段落区域，则并入对应 chunk。
2. 若图片跨多个段落，则并入最邻近标题块。
3. 若图片无法定位，则生成独立辅助 chunk，并标记 `metadata["synthetic_image_chunk"]=True`。

### 15.3 设计说明
该方案牺牲了图像向量检索的细粒度能力，但大幅降低系统复杂度，且与纯文本 RAG 管线兼容，适合教学和求职项目。

## 16. Memory System
### 16.1 三层结构
1. Short-term memory：当前会话最近 N 轮消息和摘要。
2. Long-term memory：用户偏好、稳定事实、历史有用知识，向量化存储。
3. Episodic memory：一次任务的目标、步骤、结果、反思。

### 16.2 数据结构
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

### 16.3 存储设计
1. SQLite：存原始内容、摘要、类型、用户、时间、重要度。
2. Chroma：仅对长期记忆和 episodic summary 存向量。
3. 短期记忆默认仅存 SQLite。

### 16.4 检索策略
1. 短期记忆：按会话 ID 和时间窗口直接取最近 N 条。
2. 长期记忆：按 query embedding + user_id filter 检索。
3. Episodic memory：优先按任务标签过滤，再做向量检索。

### 16.5 更新策略
1. 每轮对话落短期记忆。
2. 达到 `summary_trigger_turns` 时自动摘要压缩。
3. 高重要信息由 LLM 分类后提升为长期记忆。
4. 任务结束时生成 episodic summary。
5. 定期运行记忆压缩任务，合并语义重复记忆。

### 16.6 面试点
可以强调“短期-长期-事件”分层让系统同时兼顾实时上下文、稳定偏好和任务复盘，且通过 SQLite + Chroma 保持实现简单。

## 17. Tool System
### 17.1 Tool Registry
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

### 17.2 调度逻辑
1. MCP tools 和本地 tools 都注册到统一 Registry。
2. Agent 只依赖 Registry，不关心工具来自 MCP 还是本地函数。
3. 每次调用必须记录 `tool_name / input / output / latency / status`。

### 17.3 设计说明
统一 Tool System 能避免后续 Agent 和 MCP 两套调用体系分裂。

## 18. Agent Framework
### 18.1 最小循环
`Thought -> Action -> Observation -> Thought ... -> Final Answer`

### 18.2 状态结构
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

### 18.3 控制规则
1. 每轮先让 LLM 输出结构化 Thought/Action。
2. 若 Action 是工具调用，则通过 ToolRegistry 执行。
3. 将 Observation 追加回消息上下文。
4. 达到 `max_steps` 或生成 `final_answer` 时终止。
5. 工具异常必须进入 Observation，而不是直接崩溃。

### 18.4 预留 Multi-Agent
定义接口但首版不实现编排：
```python
class PlannerAgent: ...
class ExecutorAgent: ...
class RetrieverAgent: ...
class EvaluatorAgent: ...
```

### 18.5 面试点
该设计展示了“单 Agent 最小闭环 + 多 Agent 预留端口”的渐进式工程思路，避免过度设计。

## 19. MCP Server 设计
### 19.1 工具定义
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

### 19.2 调用链路
`MCP Client -> MCP Server -> Handler -> Application Service -> Domain Ports -> Infrastructure -> Response`

### 19.3 实现约束
1. 仅 `stdio transport`。
2. Handler 负责 schema 校验和错误包装。
3. Application Service 负责业务编排。
4. 所有 tool 调用都必须产出 trace_id。

## 20. 可观测性
### 20.1 Trace 类型
1. Ingestion Trace
2. Query Trace
3. Agent Trace

### 20.2 Trace 结构
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

### 20.3 JSON Lines 日志格式
```json
{"ts":"2026-05-04T12:00:00Z","level":"INFO","trace_id":"t1","module":"query_service","event":"dense_retrieval_done","provider":"chroma","latency_ms":42,"payload":{"top_k":20}}
```

字段要求：
`ts, level, trace_id, module, event, provider, latency_ms, payload`

### 20.4 设计说明
JSON Lines 便于本地 grep、后续接 ELK/ClickHouse，也适合教学演示单次链路。

## 21. Dashboard 设计
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

## 22. 评估体系
### 22.1 支持指标
1. Ragas：`faithfulness`, `answer_relevancy`, `context_precision`, `context_recall`
2. 自定义：`hit_rate`, `mrr`

### 22.2 hit_rate
命中定义：目标答案对应文档或 chunk 出现在 Top-K 中。
```python
hit_rate = hit_count / total_queries
```

### 22.3 MRR
```python
MRR = mean(1 / rank_of_first_relevant_doc)
```

### 22.4 扩展预留
1. Agent trajectory correctness
2. Tool call success rate
3. Tool selection accuracy

### 22.5 面试点
评估模块能体现“不是拍脑袋调参，而是数据驱动优化”。

## 23. 数据流说明
### 23.1 Ingestion Flow
1. 读 PDF
2. 算 SHA256
3. 查增量表
4. 转 Markdown
5. 规范化 Markdown 与基础清噪
6. 抽图并 caption
7. 分块
8. 基于局部上下文窗口做 chunk 级增强
9. 构造 sparse/dense 文本
10. 写向量库
11. 写 SQLite 元数据与中间产物
12. 异步刷新 BM25 artifact
13. 写 Trace

### 23.2 Query Flow
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

### 23.3 Agent Flow
1. 收到目标
2. 读取短期与长期记忆
3. 生成 Thought/Action
4. 调度工具
5. 写 Observation
6. 循环直到结束
7. 写 episodic memory 和 Trace

## 24. 测试方案
### 24.1 总原则
1. 核心领域逻辑优先 TDD。
2. 所有 provider 适配器必须可 mock。
3. 单元、集成、E2E 分层执行，不能混淆。

### 24.2 单元测试
覆盖：
1. Chunk 构造与 metadata 注入
2. RRF 融合
3. Memory 压缩与筛选逻辑
4. Tool Registry
5. Agent loop 终止条件
6. settings 校验

### 24.3 集成测试
覆盖：
1. MarkItDown -> Split -> Upsert 链路
2. Chroma 检索 + BM25 融合
3. SQLite 记忆读写
4. MCP tool handler 到 service 的串联

### 24.4 E2E 测试
覆盖：
1. 导入一个 PDF 后可成功查询
2. `query_knowledge_hub` 返回 answer、citations、trace_id
3. 记忆写入后下一轮问答可正确利用记忆
4. Dashboard 能显示最新 trace 和评估结果

### 24.5 RAG 专项测试
1. retrieval 质量：固定样本集评估 `hit_rate/MRR`
2. rerank 效果：比较 rerank 前后相关文档排名变化

### 24.6 Agent 预留测试
1. trajectory 测试
2. tool 调用序列测试
3. max_steps 安全测试

## 25. 开发约束
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

## 26. 项目排期
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

## 27. 进度跟踪表
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

## 28. 面试回答点与扩展点
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

## 29. 开发起始顺序
建议严格按 `A -> B -> C -> D -> E -> F -> G -> H -> I` 推进，任何阶段未验收通过，不进入下一阶段。首个可演示里程碑是阶段 F，首个完整工程里程碑是阶段 I。

## 30. 架构补充：项目核心特色与工程亮点
本节保留为“厚版总结”，用于在整体设计梳理完成后，再从全局视角回看项目价值。前文第 2 章负责快速建立阅读认知，本节负责做更完整的归纳总结。

### 30.1 一套工程骨架，同时服务三类目标
1. 工程目标：形成可持续开发、可测试、可扩展的 AI 系统基础设施。
2. 教学目标：每个模块都能独立拆出来讲清楚设计思路、技术选型与演进路径。
3. 求职目标：每个模块都能映射到简历亮点、面试问题和实际项目经验表达。

### 30.2 模块化而非黑盒集成
1. RAG 主链路中的 Loader、Splitter、Embedding、Retriever、Reranker、Evaluator 全部抽象化。
2. Agent、Memory、Tool System 与 RAG 共享底层基础设施，而不是额外再起一套框架。
3. 整体遵循“统一领域模型 + 可插拔实现 + 配置切换”的设计，避免项目后期出现体系分裂。

### 30.3 兼顾学习价值与工业现实
1. 学习价值体现在链路透明：能清楚说明每个阶段输入、输出、边界和失败模式。
2. 工业现实体现在工程约束：增量摄取、可观测性、评估闭环、配置驱动、分层测试。
3. 不追求首版过度复杂，但关键架构点必须一步到位预留扩展空间。

### 30.4 面向未来能力扩展
本项目首版聚焦单机、本地优先、单 Agent 最小闭环，但架构上必须为以下方向预留明确演进路径：
1. Agentic RAG
2. Multi-Agent 协作
3. 长期记忆与个性化
4. 新模型与新检索后端切换
5. 更细粒度的评估与自动回归体系

## 31. 架构补充：技术选型说明与架构取舍
本节用于解释“为什么这样选”，它是技术选型说明、架构设计文档和面试回答素材的统一来源。

### 31.1 总体取舍原则
1. 首版优先保证透明性与可解释性，而不是最短路径接 SDK 拼装。
2. 首版优先保证本地可运行、可调试、可教学，而不是直接追求云原生复杂部署。
3. 首版优先保证模块边界清晰，而不是为了省代码把逻辑堆在一起。

### 31.2 为什么选择 MCP + stdio，而不是 HTTP API
1. MCP 能直接接入 Copilot、Claude Desktop 等 AI 客户端，具备更强的演示与实用价值。
2. `stdio transport` 更适合本地子进程模式，避免引入额外服务治理与网络部署成本。
3. 对求职项目而言，MCP 比普通 HTTP Chat API 更能体现对新型 AI 应用协议的理解。

### 31.3 为什么禁止用 LangChain / LlamaIndex 做主编排
1. 黑盒太多，不利于讲清核心链路。
2. 工程边界不够透明，不利于做强约束架构设计。
3. 面试时更难讲明白“你自己到底设计了什么”。
4. 仅保留 `RecursiveCharacterTextSplitter`，因为它在文本切分上足够成熟且替代成本低。

### 31.4 为什么向量库首版选择 Chroma
1. 本地优先，易启动，适合教学和单机项目。
2. 对首版数据规模足够，且能快速完成端到端闭环。
3. 保留 `BaseVectorStore` 后，后续可平滑迁移到 Qdrant、Milvus、pgvector。

### 31.5 为什么持久化首版选择 SQLite
1. 适合作为轻量元数据存储、trace 索引存储、memory 基础存储。
2. 对本地开发和教学演示十分友好。
3. 后续替换为 PostgreSQL 时，上层仓储接口可保持不变。

### 31.6 为什么选择 Hybrid Retrieval + RRF + Rerank
1. BM25 擅长关键词匹配和专有名词查找。
2. Dense Retrieval 擅长语义相似召回。
3. RRF 实现简单、可解释性强、调参成本低。
4. Rerank 将粗排结果进一步收敛为“适合生成答案”的上下文集合。

### 31.7 为什么采用三层 Memory
1. 对话上下文、长期偏好、任务经历三类信息生命周期不同。
2. 若统一存在一个池子里，会出现检索污染和无关上下文膨胀。
3. 三层记忆使系统更接近真实 Agent 应用，而不是一次性问答机器人。

## 32. 工业落地性与关键难点
本项目虽然以本地优先为首版路线，但必须从一开始考虑工业落地时真正会遇到的问题。

### 32.1 文档摄取落地难点
1. PDF 结构噪声大，标题层级、页眉页脚、表格、图片位置经常不稳定。
2. 图片与文本的对齐关系不总是明确，caption 注入位置容易影响检索质量。
3. 同一文档重复导入、版本更新、局部修改会带来索引幂等和增量更新问题。

### 32.2 检索落地难点
1. 业务查询往往是口语化、模糊表达，不一定与文档写法一致。
2. 纯向量检索容易漏专有名词，纯 BM25 容易漏语义变体。
3. 检索不是最终目标，真正目标是为后续回答生成提供正确上下文。

### 32.3 生成落地难点
1. 检索到“相关”上下文不代表生成一定正确。
2. 多段上下文拼接会出现上下文冲突、冗余和 token 浪费。
3. 若无引用与 trace，很难定位幻觉来源。

### 32.4 Agent 落地难点
1. Agent 的失败常常不是“模型能力不够”，而是工具边界不清、状态管理混乱。
2. 记忆写得太多会污染后续推理，写得太少又失去个性化与连续性。
3. 缺乏 trajectory 级 trace 时，很难调试 Thought-Action-Observation 的失败原因。

### 32.5 评估落地难点
1. 离线评估分数高，不代表真实用户问题一定表现好。
2. 单看 answer quality 不足以定位到底是 retrieval、rerank 还是 prompt 的问题。
3. 若没有回归集，模型或参数变更后很容易“优化一处，退化一片”。

### 32.6 本项目的落地性回答口径
在面试或汇报中，应明确：
1. 首版系统选择本地优先，是为了降低复杂度并尽快形成可验证闭环。
2. 架构上已经通过接口抽象、配置驱动、数据分层，为后续生产化迁移保留路径。
3. 真正的工程价值不在于“堆更多组件”，而在于把链路做透明、可测、可迭代。

## 33. 模块设计沟通原则
从当前阶段开始，后续所有模块细化都应以“架构设计讨论优先”而不是“先写代码”为原则。

### 33.1 每个模块在细化时必须回答六个问题
1. 这个模块的职责边界是什么。
2. 这个模块与上下游的输入输出契约是什么。
3. 这个模块的关键技术难点是什么。
4. 这个模块为什么这样设计，还有哪些替代方案。
5. 这个模块未来如何扩展。
6. 这个模块如何转化为面试表达与简历亮点。

### 33.2 每个模块的补充结构模板
后续细化时，统一采用如下结构：
1. 模块目标
2. 职责边界
3. 核心流程
4. 关键难点
5. 架构取舍
6. 可扩展方向
7. 知识点清单
8. 高频面试题
9. 简历撰写建议

## 34. 模块详解：Ingestion Pipeline
### 34.1 模块目标
将原始 PDF 文档转换为适合后续检索与生成使用的高质量 chunk 资产，并以可追踪、可增量、可复用的方式写入本地知识库。

### 34.2 架构设计亮点
1. 采用“文件级去重 + chunk 级标准化 + trace 记录”的工程化摄取链路。
2. 使用 Markdown 作为中间统一表示，降低后续 Splitter 和增强逻辑的复杂度。
3. 将多模态处理纳入摄取阶段，而不是查询阶段临时处理，减少查询时延。
4. BM25 采用“SQLite 事实源 + 双缓冲 artifact + 逻辑删除过滤”的组合策略，在实现复杂度和工程稳定性之间取得平衡。

### 34.3 关键技术难点
1. PDF 转 Markdown 后可能出现结构噪声。
2. chunk 切分过细会丢上下文，过粗会影响召回精度与 token 成本。
3. LLM 重写虽能提升语义完整性，但也可能引入改写偏差。
4. 图片 caption 若注入不当，会污染原始语义。
5. BM25 与 SQLite/Chroma 的一致性维护，是本地优先知识库系统最容易被忽视但最影响稳定性的部分。

### 34.4 扩展方向与建议
1. 增加更多 Loader，如 Markdown、HTML、Code Repo。
2. 支持 chunk 质量评估与自动清洗。
3. 支持文档版本对比与局部重建索引。
4. 支持 OCR 与表格结构化抽取。
5. 支持基于 `embedding_fingerprint` 的批量重嵌入与索引迁移。

### 34.5 知识点清单
1. PDF 解析与文本结构化
2. Markdown 作为中间表示的优势
3. 智能切分策略与 chunk granularity
4. 多模态 captioning 的基本原理
5. 增量索引与幂等摄取

### 34.6 高频面试题
1. 为什么先把 PDF 转成 Markdown，而不是直接对 PDF 提取纯文本切块？  
参考回答：Markdown 保留了更多结构信息，如标题、列表、段落层级，更适合作为后续语义分块和元数据注入的中间层。

2. 为什么要做 LLM 重写，而不是直接拿原文做 embedding？  
参考回答：原文经常存在断句、指代缺失、表格残缺等问题，重写可提升语义完整性，但必须与原文并存，不能完全覆盖原文。

3. 增量摄取为什么重要？  
参考回答：真实业务中知识库会反复更新，如果每次全量重跑，成本和延迟都不可接受，因此需要哈希跳过与幂等 upsert。

4. 为什么 BM25 采用“自动防抖 + 强制重建”双机制？  
参考回答：自动防抖保证普通用户上传后无需人工干预也能最终一致；强制重建则用于批量导入后的即时确认、异步异常恢复和索引逻辑迁移，这体现的是工程系统的“自觉性 + 掌控感”。

### 34.7 简历撰写建议
可写为：
“设计并实现模块化文档摄取链路，支持 PDF→Markdown→结构规范化→语义分块→多模态增强→向量化的端到端处理；通过 SHA256 增量跳过、BM25 双缓冲重建与可观测 Trace 机制提升知识库更新效率、一致性与可调试性。”

## 35. 模块详解：Retrieval / Rerank / Context
### 35.1 模块目标
在查询阶段以低延迟、高召回、强可解释性的方式找到最相关的知识片段，并构造成适合 LLM 使用的上下文。

### 35.2 架构设计亮点
1. Hybrid Retrieval 同时覆盖关键词精确匹配与语义检索。
2. RRF 作为融合层，兼具实现简单和面试可解释性。
3. Rerank 作为精排层，使检索链路具备“粗排召回 + 精排过滤”的工业形态。
4. Context Builder 将 metadata、图片描述、memory 统一纳入上下文编排。
5. Retrieval 输出结构化中间结果与降级状态，而不是只返回最终 chunks，便于 Dashboard、评估和坏案例分析。

### 35.3 关键技术难点
1. Query 与文档语义空间不一致时，Dense Retrieval 也可能失效。
2. BM25 和 Dense 分数不可直接比较，因此需要融合层。
3. Rerank 提升精度的同时也引入额外成本。
4. Context 构建不只是“拼字符串”，而是信息压缩与冲突控制问题。
5. 当精排最高分也很低时，系统必须有能力“拒绝胡答”，而不是继续把低质量上下文喂给生成模型。

### 35.4 扩展方向与建议
1. 增加 query rewrite、self-query、multi-query retrieval。
2. 增加更细粒度 rerank 策略与预算控制。
3. 支持 chunk window 扩展与 parent-child retrieval。
4. 支持基于用户画像或记忆的 personalized retrieval。
5. 基于 `rerank_top_score` 和坏案例统计，进一步推导相对阈值与自适应拒答策略。

### 35.5 知识点清单
1. BM25 原理
2. Dense Retrieval 与 embedding space
3. Bi-Encoder vs Cross-Encoder
4. Reciprocal Rank Fusion
5. Context window 预算与 prompt packing

### 35.6 高频面试题
1. 为什么 BM25 和 Dense 要一起用？  
参考回答：两者擅长的匹配模式不同，BM25 对专有名词、代码名、缩写更敏感，Dense 对语义近义表达更强，组合后查全率更高。

2. 为什么用 RRF，而不是简单加权分数？  
参考回答：不同检索器的原始分数分布不统一，直接加权不稳定，RRF 基于排名位置融合，更稳健、实现也更简单。

3. Cross-Encoder 和 Bi-Encoder 的差异是什么？  
参考回答：Bi-Encoder 适合大规模召回，Cross-Encoder 适合小规模精排；前者强调效率，后者强调匹配精度。

4. 检索做好了为什么还会答错？  
参考回答：因为 RAG 是“检索 + 上下文构建 + 生成”的系统，检索正确不代表上下文拼装和生成一定正确。

5. 为什么 RetrievalResult 要保留中间态而不是只返回最终 Top-K？  
参考回答：因为真实工程里需要解释“为什么命中了这些结果”“为什么退化成单路检索”“为什么拒答”，中间态是可观测性、Dashboard、评估和坏案例分析的基础。

6. 为什么要做 `score threshold`？  
参考回答：因为不是所有检索结果都值得交给生成模型。如果 rerank 最高分都很低，说明检索到的上下文相关性不足，此时继续生成只会放大幻觉风险。

### 35.7 简历撰写建议
可写为：
“构建 Hybrid Retrieval 检索链路，结合 BM25、Dense Embedding、RRF 融合与 Cross-Encoder/LLM Rerank，实现粗排召回与精排过滤两阶段架构；设计结构化 RetrievalResult、分数阈值拒答机制与 Memory-Augmented Context，提升系统可解释性、稳定性与坏案例分析能力。”

## 36. 模块详解：Memory System
### 36.1 模块目标
为 RAG 与 Agent 提供持续性上下文，使系统具备短期连续对话能力、长期偏好记忆能力和任务级经历积累能力。

### 36.2 架构设计亮点
1. 采用短期、长期、事件三层结构，显式区分不同生命周期信息。
2. 同时使用 SQLite 与向量存储，兼顾结构化过滤与语义检索。
3. 引入摘要与压缩机制，控制记忆规模膨胀。

### 36.3 关键技术难点
1. 什么信息应该被记住，什么应该被丢弃。
2. 记忆检索相关性如何定义，如何避免污染当前任务。
3. 摘要压缩后如何保证关键信息不丢失。

### 36.4 扩展方向与建议
1. 增加 memory importance scoring。
2. 增加用户画像抽取与偏好学习。
3. 增加记忆衰减、过期与冲突消解机制。
4. 增加 episodic reflection 与任务复盘。

### 36.5 知识点清单
1. Memory 分类方法
2. 记忆检索与向量索引
3. 摘要压缩与信息保真
4. 个性化推荐与用户画像基础

### 36.6 高频面试题
1. 为什么不用一个统一 memory 表解决所有问题？  
参考回答：不同类型记忆的生命周期、召回方式和使用场景不同，混用会造成上下文污染与检索噪声。

2. 记忆系统的核心难点是什么？  
参考回答：不是“存下来”本身，而是何时写入、何时压缩、何时召回，以及如何避免无关信息干扰当前推理。

### 36.7 简历撰写建议
可写为：
“设计三层 Memory System，结合 SQLite 结构化存储与向量检索实现短期对话、长期偏好和任务经历管理，并引入自动摘要压缩机制控制上下文膨胀。”

## 37. 模块详解：Tool System 与 Agent Framework
### 37.1 模块目标
构建统一工具抽象与最小 Agent 执行闭环，为后续 Tool Use、ReAct、Multi-Agent 协作提供基础。

### 37.2 架构设计亮点
1. Tool 抽象独立于 MCP，使本地工具与 MCP 工具能统一调度。
2. Agent Loop 采用最小 Thought-Action-Observation 闭环，结构清晰，便于教学与调试。
3. Tool Trace 与 Agent Trace 统一记录，便于排障和评估。

### 37.3 关键技术难点
1. 工具 schema 设计不清晰会导致模型调用失败率高。
2. Agent 的失败常常来自状态不一致，而非单次模型输出错误。
3. 如果没有步骤限制和异常观察机制，Agent 很容易失控。

### 37.4 扩展方向与建议
1. 增加结构化 Thought/Action 输出协议。
2. 增加 planner-executor 模式。
3. 增加反思（reflection）与重试策略。
4. 增加 tool choice evaluation。

### 37.5 知识点清单
1. ReAct 思想
2. Tool Calling 机制
3. Function schema 设计
4. Agent trajectory 与状态机

### 37.6 高频面试题
1. 为什么要把 MCP tools 和本地 tools 统一抽象？  
参考回答：后续 Agent 不应该关心工具来源，只应该依赖统一调用协议，这样系统扩展性更强。

2. Agent 和普通 RAG 的区别是什么？  
参考回答：普通 RAG 主要是一次性检索增强生成，Agent 则具备目标驱动、工具调用、状态演化与多步推理能力。

### 37.7 简历撰写建议
可写为：
“设计统一 Tool System 与最小 ReAct Agent Loop，将 MCP tools 与本地 tools 纳入统一注册和调度体系，为后续 Multi-Agent 与任务自动化扩展提供基础。”

## 38. 模块详解：MCP Server
### 38.1 模块目标
将内部 RAG/Memory/Agent 能力以标准 MCP Tool 形式暴露给外部 AI 客户端，实现一次开发、多客户端复用。

### 38.2 架构设计亮点
1. 面向 MCP 标准设计，而不是单一 UI 或单一 API。
2. 使用 `stdio transport` 贴合本地 Agent/编辑器集成场景。
3. MCP Handler 与业务服务解耦，保证协议层和业务层边界清晰。

### 38.3 关键技术难点
1. tool 输入输出 schema 必须稳定，否则客户端适配成本高。
2. 若错误结构不统一，客户端难以恢复与调试。
3. tool 的语义边界如果不清晰，会造成“一个 tool 过大”或“tool 过碎”。

### 38.4 扩展方向与建议
1. 增加更多资源型接口与只读知识资源。
2. 增加 memory、agent、evaluation 相关 MCP tools。
3. 增加 capability discovery 和工具版本兼容策略。

### 38.5 知识点清单
1. MCP 协议基础
2. Tool schema 设计
3. stdio transport 工作方式
4. 协议层与业务层分离

### 38.6 高频面试题
1. 为什么这个项目不用 HTTP，而选择 MCP stdio？  
参考回答：因为项目目标是与 AI 助手深度集成，MCP 在这类场景下更自然，stdio 也更适合本地子进程模式。

2. MCP tool 设计的关键是什么？  
参考回答：关键是边界稳定、输入输出清晰、错误可恢复、调用链路可追踪。

### 38.7 简历撰写建议
可写为：
“基于 Python MCP SDK 设计本地 `stdio` 模式 MCP Server，将模块化 RAG 能力封装为标准 tools，支持外部 AI 客户端直接调用，提升系统复用性与协议集成能力。”

## 39. 模块详解：可观测性、Dashboard 与评估
### 39.1 模块目标
让系统不仅“能跑”，还要能解释、能回放、能评估、能持续迭代优化。

### 39.2 架构设计亮点
1. Trace 同时覆盖 Ingestion、Query、Agent 三类主链路。
2. Dashboard 不直接承载主逻辑，只负责把系统状态和链路透明化。
3. 评估模块与运行链路解耦，可独立执行离线评测和回归分析。

### 39.3 关键技术难点
1. 没有 Trace 时很难定位问题是在解析、检索、重排还是生成。
2. 没有 Dashboard 时系统虽能工作，但调试效率很低。
3. 没有评估闭环时，系统优化很容易变成主观调参。

### 39.4 扩展方向与建议
1. 增加 trace 可视化瀑布图与阶段对比。
2. 增加评估任务历史对比与回归预警。
3. 增加 tool use correctness 和 agent trajectory 评估。
4. 增加日志接入 ELK、ClickHouse 或 OpenTelemetry。

### 39.5 知识点清单
1. 结构化日志
2. Trace 与 Span 基础
3. Ragas 指标含义
4. Hit Rate / MRR 的适用场景
5. 离线评估与在线反馈差异

### 39.6 高频面试题
1. 为什么 RAG 项目一定要做可观测性？  
参考回答：因为 RAG 是多阶段流水线，没有中间态可视化就无法定位问题，更无法稳定迭代。

2. 为什么评估不能只看最终回答质量？  
参考回答：因为最终回答质量只是结果指标，无法帮助定位问题究竟出在 retrieval、rerank 还是 context build。

### 39.7 简历撰写建议
可写为：
“搭建 RAG 系统可观测与评估闭环，设计 Ingestion/Query/Agent Trace、Streamlit Dashboard 及 Ragas + 自定义指标体系，支撑数据驱动的质量优化与回归测试。”

## 40. 教学、面试与简历输出规范
本项目每个核心模块都必须产出三类附属材料，作为后续持续完善的一部分。

### 40.1 知识点清单产出规范
每个模块都要列出：
1. 核心原理
2. 常见替代方案
3. 工程实现中的关键取舍
4. 与项目代码的对应关系

### 40.2 高频面试题产出规范
每个模块至少准备：
1. 原理题
2. 设计题
3. 工程权衡题
4. 故障定位题

### 40.3 简历撰写建议产出规范
每个模块都要形成：
1. 一条偏架构亮点的简历描述
2. 一条偏工程落地的简历描述
3. 一条偏 AI 系统特色的简历描述

### 40.4 后续细化方式
从下一轮开始，逐个模块深挖时，除补充架构与流程外，还必须同步补充：
1. 该模块知识点清单
2. 高频面试题与参考回答
3. 简历撰写建议
4. 模块扩展路线图

## 41. QueryService 设计
### 41.1 模块目标
`QueryService` 是问答主控层，负责把用户问题、检索结果、记忆上下文和回答策略整合为最终可返回的结构化响应。它不是“简单拼 prompt 再调用 LLM”，而是整个问答链路的决策中心。

### 41.2 职责边界
1. 接收 MCP 层查询请求并做参数归一化。
2. 并行触发 `RetrievalService` 和 Memory 读取。
3. 根据 Retrieval 结果与质量阈值，决定正常回答、弱回答、拒答或降级回答。
4. 基于 `Memory-Augmented Context` 组装最终 prompt。
5. 调用 LLM 生成答案，并构建 citations。
6. 记录 Query Trace，并追加 short-term memory。
7. `QueryService` 不负责底层召回、RRF、Rerank 实现细节，这些属于 `RetrievalService`。

### 41.3 QueryService 输入输出
```python
class QueryServiceRequest(BaseModel):
    query: str
    collection_name: str
    collection_ids: list[str] | None = None
    conversation_id: str | None = None
    user_id: str | None = None
    top_k_retrieval: int = 20
    top_k_rerank: int = 8
    use_llm_rerank: bool = False
    answer_mode: str = "default"
    allow_fallback_answer: bool = True


class QueryServiceResponse(BaseModel):
    answer: str
    answer_status: str
    retrieval_status: str
    citations: list[dict[str, Any]]
    used_memory: dict[str, Any] | None = None
    used_chunk_ids: list[str]
    refusal_reason: str | None = None
    trace_id: str
```

说明：
1. `answer_status` 建议枚举：
   - `answered`
   - `weak_answered`
   - `refused`
   - `degraded_answered`
   - `failed`
2. `retrieval_status` 直接透传 Retrieval 层的结构化状态，避免查询层吞掉底层退化信息。
3. `used_memory` 与 `used_chunk_ids` 是教学、调试、Dashboard 和坏案例分析的重要数据。

### 41.4 异步执行模型
1. `QueryService` 首版必须采用 async 主链路。
2. 原因：
   - LLM API 通常需要秒级到十秒级等待。
   - 若不用 async，MCP Server 会在等待期间阻塞，影响其他请求、心跳与整体交互体验。
3. 推荐执行模型：

```python
memory_task = load_memory_summary(...)
retrieval_task = retrieval_service.retrieve(...)
memory_result, retrieval_result = await asyncio.gather(memory_task, retrieval_task)
```

4. Memory 与 Retrieval 可以并行，因为首版 memory 不参与 query rewrite，不构成前置依赖。

### 41.5 回答策略与拒答语义
1. 当 `rerank_threshold_passed = True` 时，默认正常回答。
2. 当 `rerank_threshold_passed = False` 时，首版采用“弱拒答”策略：
   - 明确说明知识库中未检索到足够相关内容。
   - 不编造确定性结论。
   - 可提示用户换问法、补充上下文或确认文档是否已导入。
3. Retrieval 完全失败时，不允许仅依赖 memory 直接回答知识型问题；memory 只能辅助理解上下文，不能越权替代知识证据。
4. 若 Retrieval 成功但 citations 构建失败，可返回 `degraded_answered`，但必须显式标记状态。

### 41.6 Prompt 结构
首版固定采用四段式 Prompt：
1. `System Instruction`
2. `Answer Policy`
3. `Memory Context`
4. `Retrieved Knowledge`

其中 `Answer Policy` 必须显式包含以下约束：
1. 优先基于检索到的知识片段回答。
2. 若检索证据不足，应明确说明“不确定”或“未找到足够相关知识”。
3. 输出时尽量附引用编号。
4. 若检索内容与记忆冲突，以检索内容为准。
5. 不允许把 memory 当作高于知识库证据的事实来源。

### 41.7 Memory 注入策略
1. 首版只注入 `conversation summary`。
2. `relevant long-term memory bullets` 作为 V2 能力，在长期记忆向量检索成熟后接入。
3. memory 在 Prompt 中只作为辅助上下文，不参与事实优先级竞争。

### 41.8 Citation 与引用锚点设计
1. Context Build 时，必须为每个注入的检索块生成唯一 `doc_ref_id`。
2. 在 `Retrieved Knowledge` 中按 `[1]`, `[2]` 形式组织 block，帮助 LLM 学习稳定的引用格式。
3. Citation 最终至少返回：
   - `doc_ref_id`
   - `doc_id`
   - `chunk_id`
   - `file_name`
   - `title_path`
   - `page_range`
4. 主响应中只返回最终实际使用到的 citations；完整候选集保留在 trace 中。

### 41.9 Query Trace 分阶段事件
首版 Query Trace 至少记录以下 stage：
1. `request_received`
2. `memory_loaded`
3. `retrieval_completed`
4. `context_built`
5. `answer_generated`
6. `citations_built`
7. `memory_appended`

### 41.10 模块详解：QueryService
#### 41.10.1 架构设计亮点
1. 将“检索质量判断”和“最终回答策略判断”显式分层。
2. 采用 async 主链路，避免 MCP Server 因等待 LLM 而整体阻塞。
3. 通过 `answer_status + retrieval_status + refusal_reason` 提升可观测性。
4. 通过编号式引用锚点，提升模型输出引用的稳定性和一致性。

#### 41.10.2 关键技术难点
1. 如何在检索不充分时拒绝胡答，而不是继续生成幻觉。
2. 如何在 memory 存在的情况下，仍保持知识库证据优先。
3. 如何在 token budget 下同时兼顾 memory、检索内容和引用完整性。
4. 如何在生成失败或 citation 失败时做优雅降级。

#### 41.10.3 扩展方向与建议
1. 增加 long-term memory bullets 注入。
2. 增加 answer mode，如严格引用模式、只基于知识库模式、简洁回答模式。
3. 增加基于 bad case 的自适应拒答策略。
4. 增加 streaming answer 与渐进式 citation 输出。

#### 41.10.4 知识点清单
1. Prompt 分层设计
2. Retrieval-Augmented Answering 与拒答策略
3. Token Budget 与上下文装配
4. 引用对齐与 citation grounding
5. 异步服务设计与非阻塞 I/O

#### 41.10.5 高频面试题
1. 为什么 QueryService 不能只是“拿结果拼 prompt”？  
参考回答：因为它还要负责回答策略、拒答逻辑、记忆注入、引用锚点和可观测状态的统一编排，是整个问答系统的控制层。

2. 为什么 memory 与知识检索冲突时，以检索内容为准？  
参考回答：因为 memory 更像历史上下文和用户偏好，而知识库检索结果才是当前问题的事实证据来源。

3. 为什么要做 async QueryService？  
参考回答：因为 LLM 调用通常是整个链路中最慢的环节，如果主链路是同步阻塞的，MCP Server 的并发体验和稳定性都会明显下降。

4. 为什么要用 `[1] [2]` 这种引用块格式？  
参考回答：因为它能帮助模型建立稳定的“证据块 -> 输出引用”映射，提升 citation 一致性与可解释性。

#### 41.10.6 简历撰写建议
可写为：
“设计异步 QueryService 作为 RAG 问答主控层，统一编排检索结果、对话记忆、Token Budget 与引用锚点机制；通过弱拒答策略、证据优先回答和结构化状态输出，提升系统稳定性与可解释性。”
