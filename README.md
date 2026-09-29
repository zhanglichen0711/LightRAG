# LightRAG 中文改造版 · DeepSeek + 本地 Embedding 快速上手

> **本仓库来源**：Fork 自 [HKUDS/LightRAG](https://github.com/HKUDS/LightRAG)（commit `453dce8`，2026-09-26）。
> 原项目是 EMNLP 2025 论文「LightRAG: Simple and Fast Retrieval-Augmented Generation」的官方实现，
> 一个轻量的**知识图谱 RAG 框架**，微软 GraphRAG 的平替。
> **MIT License 原样保留（见 [LICENSE](LICENSE)），版权归 LightRAG Team 所有**，本仓库的改动同样以 MIT 发布。
> 原版英文 README 与官方中文翻译见 [README_UPSTREAM.md](README_UPSTREAM.md) 与 [README-zh.md](README-zh.md)。

## 本仓库改了什么

只是轻改造，核心代码一行没动：

| 改动 | 说明 |
|---|---|
| 新增 `README.md`（本文件） | 面向国内用户的上手指南：DeepSeek 做 LLM、本地 Ollama 做 Embedding |
| 新增 `env.deepseek.example` | 可直接复制的环境变量模板，绕开「Embedding 必填」的启动报错 |
| 原 `README.md` 重命名为 `README_UPSTREAM.md` | 内容未动，方便对照 |

适合谁：想本地跑通一个**图谱 RAG**、LLM 走 DeepSeek API、Embedding 不想花钱走本地的开发者。

## 为什么值得跑一遍：图谱 RAG 和向量 RAG 差在哪

普通的向量 RAG（包括我博客里写的那套）：文档切块 → embedding → 向量库召回 top-k。
它擅长「找原文」，但回答「A 和 B 有什么关系」这类跨文档问题时容易碎。

LightRAG 在切块之后多做一步：**用 LLM 抽取实体和关系，构建知识图谱**，检索时同时走
「实体邻域（local）」和「全局主题（global）」两条路再融合。代价是入库时要多花 LLM 调用，
收益是多跳关系类问题的答案明显更完整。同一个语料两种方案各跑一遍，对比答案，比看十篇评测文章直观。

## 快速上手

### 1. 准备两个服务

- **DeepSeek API Key**：[platform.deepseek.com](https://platform.deepseek.com) 申请（LLM 用）
- **本地 Ollama**：[ollama.com](https://ollama.com) 安装后拉一个 embedding 模型：

```bash
ollama pull bge-m3        # 1024 维，中文效果好
```

### 2. 安装并启动

```bash
# 方式一：pip 安装（推荐）
pip install "lightrag-hku[api]"

# 方式二：源码运行（本仓库）
pip install -e ".[api]"

# 配置环境变量
cp env.deepseek.example .env
# 编辑 .env，填入你的 DEEPSEEK_API_KEY

# 启动服务（自带 WebUI）
lightrag-server
```

浏览器打开 `http://localhost:9621`（WebUI），上传文档 → 等图谱构建完 → 切换
local / global / hybrid / mix 四种查询模式对比答案。

### 3. Docker 方式（可选）

```bash
cp env.deepseek.example .env
docker compose up
```

## 关键配置说明（详见 `env.deepseek.example`）

| 变量 | 值 | 说明 |
|---|---|---|
| `LLM_BINDING` | `openai` | DeepSeek 是 OpenAI 兼容接口，走 openai 绑定 |
| `LLM_BINDING_HOST` | `https://api.deepseek.com/v1` | 注意带 `/v1` |
| `LLM_BINDING_API_KEY` | 你的 DeepSeek Key | |
| `LLM_MODEL` | `deepseek-chat` | 也可以换 `deepseek-reasoner` |
| `EMBEDDING_BINDING` | `ollama` | **DeepSeek 不提供 embedding 接口**，用本地 Ollama |
| `EMBEDDING_MODEL` | `bge-m3` | 中文场景推荐 |
| `EMBEDDING_DIM` | `1024` | 必须和模型实际维度一致，否则检索静默错乱 |

## 踩坑提示

1. **`EMBEDDING_MODEL` 不设置服务直接拒启**——这是新手第一坑，env 里必须显式写。
2. **`EMBEDDING_DIM` 和模型不匹配不报错**，只是召回结果莫名变差，排查时优先对维度。
3. **图谱构建阶段 LLM 调用量大**（每个 chunk 都要抽实体），先用几篇小文档试跑，
   观察 DeepSeek 账单，再决定要不要上大语料。`MAX_ASYNC_LLM` 控制并发，默认 4。
4. 默认存储后端是内存库（JsonKVStorage），重启数据就没了——本地玩可以接受，
   想持久化按官方 README 切 PostgreSQL。

## 同步上游

本仓库不带原项目 git 历史，想同步上游更新：

```bash
git remote add upstream https://github.com/HKUDS/LightRAG.git
git fetch upstream main --depth 1
git merge FETCH_HEAD   # 有冲突时优先保留上游代码，改动只集中在 README 和 env 模板
```

## 相关

- 原项目：https://github.com/HKUDS/LightRAG （给原作者点 star）
- 论文：LightRAG: Simple and Fast Retrieval-Augmented Generation (EMNLP 2025)
- 我的博客（RAG 工程实践系列）：https://lczhang.blog.cn
