---
title: WeKnora 项目介绍与个人知识库搭建指南
date: 2026-04-22
tags:
  - AI
  - RAG
  - 知识库
  - WeKnora
  - 腾讯
  - 开源
  - LLM
  - Docker
---

# WeKnora - LLM 驱动的文档理解与语义检索框架

> **项目地址**：https://github.com/Tencent/WeKnora
> **官方网站**：https://weknora.weixin.qq.com
> **协议**：MIT License
> **当前版本**：v0.4.0

---

## 1. 项目概述

**WeKnora** 是腾讯开源的一款基于大语言模型（LLM）的智能知识管理与问答框架，专为深度文档理解和语义检索而设计。它采用模块化架构，融合了多模态预处理、语义向量索引、智能检索和大语言模型推理等核心能力，遵循 **RAG（Retrieval-Augmented Generation，检索增强生成）** 范式，通过将相关文档块与模型推理相结合，生成高质量、上下文感知的回答。

简单来说，WeKnora 能帮你把 PDF、Word、图片、Excel 等各种文档变成一个"可对话"的智能知识库，你可以用自然语言向它提问，它会从你的文档中找到答案。

---

## 2. 核心特性

### 2.1 双模式问答引擎

| 模式 | 引擎 | 适用场景 |
|------|------|----------|
| **快速问答** | RAG 流水线 | 日常知识查询，追求速度 |
| **智能推理** | ReACT Agent | 多源综合、复杂任务，追求深度 |

- **快速问答**：通过 RAG 管线快速检索相关内容块并生成答案
- **智能推理**：通过 ReACT Agent 引擎采用渐进策略，自主编排知识检索、MCP 工具和网络搜索，通过多轮迭代和反思得出结论
- **自定义 Agent**：支持灵活配置专属知识库、工具集和系统提示词

### 2.2 多类型知识库

- **FAQ 知识库**：问答对形式，适合结构化知识
- **文档知识库**：上传文档自动解析建索引，适合非结构化知识
- **多渠道导入**：支持拖拽上传、文件夹导入、URL 导入、在线录入、飞书/Notion 自动同步

### 2.3 丰富的文档格式支持

支持 **10+ 种文档格式**：

`PDF` / `Word` / `Txt` / `Markdown` / `HTML` / `图片（OCR/描述）` / `CSV` / `Excel` / `PPT` / `JSON`

### 2.4 混合检索策略

- **BM25 稀疏检索**：基于关键词匹配
- **稠密向量检索**：基于语义相似度
- **GraphRAG**：基于知识图谱的检索增强
- **父子分块策略**：层级化分块，提升上下文管理精度
- **跨知识库检索**：可同时检索多个知识库

### 2.5 全面的 LLM 兼容性

| 类别 | 支持列表 |
|------|----------|
| **LLM 提供商** | OpenAI / Azure OpenAI / DeepSeek / Qwen（阿里云）/ 智谱 / 混元 / 豆包（火山引擎）/ Gemini / MiniMax / NVIDIA / Novita AI / SiliconFlow / OpenRouter / Ollama |
| **Embedding 模型** | Ollama / BGE / GTE / OpenAI 兼容 API |
| **向量数据库** | PostgreSQL (pgvector) / Elasticsearch / Milvus / Weaviate / Qdrant |
| **对象存储** | Local / MinIO / AWS S3 / 火山引擎 TOS / 阿里云 OSS |

### 2.6 IM 渠道集成

支持将知识库问答能力直接接入即时通讯工具：

- 企业微信 / 飞书 / Slack / Telegram / 钉钉 / Mattermost / 微信

### 2.7 其他亮点

- **MCP 工具集成**：通过 Model Context Protocol 扩展 Agent 能力
- **网络搜索**：内置 DuckDuckGo / Google / Bing / 百度 / Tavily 等搜索引擎
- **Chrome 扩展**：浏览器一键采集网页内容到知识库
- **共享空间**：支持团队协作，租户隔离检索
- **知识图谱**：基于 Neo4j 的文档关系图谱
- **E2E 测试**：全流程可视化评估，支持 BLEU / ROUGE 指标

---

## 3. 系统架构

WeKnora 采用现代模块化设计，构建完整的文档理解与检索流水线：

```
┌─────────────────────────────────────────────────────────┐
│                     用户接口层                           │
│         Web UI  /  RESTful API  /  Chrome Extension     │
└───────────────────────┬─────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────┐
│                   对话管理引擎                           │
│     快速问答（RAG）  /  智能推理（ReACT Agent）          │
└───────────────────────┬─────────────────────────────────┘
                        │
┌───────────┬───────────▼───────────┬─────────────────────┐
│  文档解析  │    语义向量索引        │    检索引擎          │
│  DocReader │  Embedding Models     │  BM25 + Dense +     │
│  多格式支持 │  BGE / GTE / Ollama   │  GraphRAG           │
└───────────┴───────────────────────┴─────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────┐
│              向量数据库 & 对象存储                        │
│  pgvector / ES / Milvus / Weaviate / Qdrant             │
│  Local / MinIO / S3 / TOS / OSS                         │
└─────────────────────────────────────────────────────────┘
```

每个组件都可以灵活配置和替换，实现高度可定制化。

---

## 4. 应用场景

| 场景 | 典型应用 | 核心价值 |
|------|----------|----------|
| 企业知识管理 | 内部文档检索、政策问答、操作手册查询 | 提升知识发现效率，降低培训成本 |
| 学术研究分析 | 论文检索、研究报告分析、学术资料整理 | 加速文献综述，辅助研究决策 |
| 产品技术支持 | 产品手册问答、技术文档搜索、故障排查 | 提升客服质量，减轻支持负担 |
| 法律合规审查 | 合同条款检索、法规政策搜索、案例分析 | 提高合规效率，降低法律风险 |
| 医疗知识辅助 | 医学文献检索、治疗指南搜索、病例分析 | 支持临床决策，提高诊断质量 |

---

## 5. 个人知识库搭建指南

以下是从零开始搭建个人知识库的完整步骤。

### 5.1 环境准备

**硬件要求**：
- CPU：4 核及以上（推荐 8 核）
- 内存：8GB 及以上（推荐 16GB+）
- 磁盘：20GB 可用空间（视知识库大小而定）

**软件要求**：
- 操作系统：Windows / macOS / Linux 均可
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)（包含 Docker Engine 和 Docker Compose）
- [Git](https://git-scm.com/)
- （可选）[Ollama](https://ollama.ai/) 用于本地运行大模型

### 5.2 安装步骤

#### 第 1 步：克隆项目

```bash
git clone https://github.com/Tencent/WeKnora.git
cd WeKnora
```

#### 第 2 步：配置环境变量

```bash
# 复制示例配置文件
cp .env.example .env
```

编辑 `.env` 文件，根据需要修改配置。对于初次使用的个人用户，大部分默认值即可使用，重点关注以下配置：

```env
# 如果使用本地 Ollama，确保以下配置正确
OLLAMA_BASE_URL=http://host.docker.internal:11434

# 数据库配置（默认即可）
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
```

#### 第 3 步：启动核心服务

**最小化启动**（推荐个人用户首次使用）：

```bash
docker compose up -d
```

这会启动核心服务：Web 前端、后端 API、PostgreSQL 数据库、Redis 消息队列。

**完整功能启动**：

```bash
docker compose --profile full up -d
```

#### 第 4 步：（可选）启动本地 Ollama

如果想在本地运行大模型而不依赖外部 API：

```bash
# 安装并启动 Ollama
ollama serve > /dev/null 2>&1 &

# 拉取推荐模型
ollama pull qwen2.5:7b        # 通用对话模型（推荐）
ollama pull bge-m3             # Embedding 模型
```

> **模型选择建议**：
> - 如果显存 >= 8GB，可使用 `qwen2.5:14b` 获得更好效果
> - 如果显存有限，使用 `qwen2.5:3b` 或接入云端 API（DeepSeek / OpenAI 等）

#### 第 5 步：访问 Web 界面

打开浏览器访问：**http://localhost**

首次访问会自动跳转到注册/登录页面。完成注册后即可开始使用。

### 5.3 配置模型

登录后，进入 **设置** 页面，配置 LLM 和 Embedding 模型：

#### 方案 A：使用本地 Ollama（免费，需 GPU）

| 配置项 | 推荐值 |
|--------|--------|
| Chat Model Provider | Ollama |
| Chat Model | qwen2.5:7b |
| Embedding Provider | Ollama |
| Embedding Model | bge-m3 |

#### 方案 B：使用云端 API（效果更好，需付费）

| 配置项 | 推荐值 |
|--------|--------|
| Chat Model Provider | DeepSeek / OpenAI |
| Chat Model | deepseek-chat / gpt-4o-mini |
| Embedding Provider | OpenAI / BGE API |
| Embedding Model | text-embedding-3-small / bge-m3 |

> **提示**：国内用户推荐 DeepSeek，性价比极高；Embedding 推荐使用 BGE 系列模型。

### 5.4 创建知识库

1. 点击 **"知识库"** → **"新建知识库"**
2. 选择类型：
   - **文档知识库**：上传 PDF、Word、Markdown 等文件（推荐）
   - **FAQ 知识库**：导入问答对
3. 上传文档：支持拖拽、文件夹导入、URL 导入
4. 等待文档解析和索引构建完成

> **文档解析过程**：上传后，系统会自动进行文档拆分 → 语义分块 → 向量化 → 索引存储。处理速度取决于文档数量和模型性能。

### 5.5 开始问答

- 在对话界面，选择 **"快速问答"** 模式进行简单查询
- 切换到 **"智能推理"** 模式进行复杂分析
- 可以通过 `@` 符号指定检索的知识库

### 5.6 高级功能配置

#### 启用知识图谱（Neo4j）

```bash
docker compose --profile neo4j up -d
```

知识图谱可以将文档内容转化为结构化的语义关联网络，提升检索的广度和相关性。

#### 启用对象存储（MinIO）

```bash
docker compose --profile minio up -d
```

当知识库文件较多时，推荐使用 MinIO 作为文件存储后端。

#### MCP Server 配置

在 Claude Desktop、Cursor 等 MCP 客户端中添加 WeKnora MCP 服务：

```json
{
  "mcpServers": {
    "weknora": {
      "command": "python",
      "args": ["path/to/WeKnora/mcp-server/run_server.py"],
      "env": {
        "WEKNORA_API_KEY": "你的 API Key（在浏览器开发者工具中查看请求头 x-api-key）",
        "WEKNORA_BASE_URL": "http://localhost:8080/api/v1"
      }
    }
  }
}
```

或使用 pip 安装：

```bash
pip install weknora-mcp-server
python -m weknora-mcp-server
```

---

## 6. 版本演进

| 版本 | 核心特性 |
|------|----------|
| **v0.4.0** (最新) | 知识助手云端服务、WeKnora Cloud、Chrome 扩展、ClawHub 技能市场、微信 IM 集成、Notion 数据源、Azure OpenAI 支持 |
| **v0.3.6** | ASR 语音识别、飞书数据源自动同步、OIDC 认证、IM 引用上下文、文档摘要、并行工具调用 |
| **v0.3.5** | Telegram/钉钉/Mattermost IM 集成、Slash 命令、建议问题、VLM 自动描述图片 |
| **v0.3.4** | 企业微信/飞书/Slack IM 集成、多模态图片、Weaviate 向量库、AWS S3 存储、AES-256 加密 |
| **v0.3.3** | 父子分块、知识库置顶、Fallback 响应、Milvus 向量库 |
| **v0.3.0** | 共享空间、Agent 技能系统、自定义 Agent、数据分析 Agent、思考模式、Kubernetes Helm |
| **v0.2.0** | ReACT Agent 模式、多类型知识库、对话策略、网络搜索、MCP 工具集成 |

---

## 7. 个人知识库最佳实践

### 7.1 知识库组织建议

```
知识库结构建议：
├── 技术文档库（文档类型）
│   ├── API 文档
│   ├── 架构设计文档
│   └── 运维手册
├── 学习笔记库（文档类型）
│   ├── 读书笔记
│   ├── 课程资料
│   └── 研究报告
├── FAQ 库（FAQ 类型）
│   ├── 常见问题
│   └── 最佳实践
└── 项目资料库（文档类型）
    ├── 需求文档
    ├── 会议纪要
    └── 决策记录
```

### 7.2 文档格式建议

1. **Markdown 最佳**：结构清晰，解析效果最好
2. **PDF 亦可**：确保文字可选中（非扫描件），扫描件会自动 OCR
3. **图片文档**：系统支持 OCR 和图片描述，但纯文本效果更好
4. **表格数据**：CSV/Excel 支持良好，可通过数据分析 Agent 进行分析

### 7.3 检索优化建议

1. **合理分块**：在知识库设置中调整 chunk 大小，一般 512-1024 tokens 效果较好
2. **启用 Rerank**：如果配置了 Rerank 模型，可以显著提升检索精度
3. **混合检索**：同时启用关键词和向量检索，覆盖不同类型的查询
4. **Prompt 定制**：在对话策略中定制 Prompt，引导模型按照你的期望回答

### 7.4 安全建议

- 本地部署是最安全的方式，数据不离开你的机器
- 生产环境请配置防火墙，不要将服务直接暴露在公网
- 定期更新到最新版本获取安全补丁
- 启用 `DISABLE_REGISTRATION` 关闭公开注册

---

## 8. 与其他知识库工具对比

| 特性 | WeKnora | MaxKB | Dify | FastGPT |
|------|---------|-------|------|---------|
| 开源协议 | MIT | Apache 2.0 | Apache 2.0 | Apache 2.0 |
| RAG 检索 | BM25 + Dense + GraphRAG | Dense + BM25 | Dense + BM25 | Dense + BM25 |
| Agent 模式 | ReACT Agent | 简单工作流 | 深度工作流 | 简单工作流 |
| IM 集成 | 7+ 渠道 | 有限 | 有限 | 有限 |
| MCP 支持 | 内建 | 无 | 部分支持 | 无 |
| 知识图谱 | Neo4j GraphRAG | 无 | 无 | 无 |
| 数据源同步 | 飞书/Notion | 无 | 无 | 无 |
| 部署复杂度 | Docker 一键部署 | Docker | Docker | Docker |

WeKnora 的优势在于：**Agent 能力强**（ReACT 渐进推理）、**集成渠道丰富**（7+ IM 渠道）、**检索策略全面**（BM25 + Dense + GraphRAG 三重检索）、**数据源自动同步**（飞书/Notion）。

---

## 9. 常见问题

### Q: 没有 GPU 可以用吗？
可以。使用云端 API（如 DeepSeek、OpenAI）作为 LLM 和 Embedding 后端即可，不需要本地 GPU。

### Q: 支持中文吗？
完全支持。WeKnora 对中文文档有良好的解析和检索能力，推荐使用 Qwen 或 DeepSeek 系列模型以获得最佳中文效果。

### Q: 数据存储在哪里？
默认存储在 Docker 容器的 PostgreSQL 数据库和本地文件系统中。你的数据完全在本地，不会上传到任何外部服务。

### Q: 如何备份数据？
```bash
# 备份 PostgreSQL 数据
docker exec postgres pg_dump -U postgres weknora > backup.sql

# 备份上传文件
cp -r ./data/uploads ./backup_uploads
```

### Q: 如何更新到最新版本？
```bash
git pull origin main
docker compose down
docker compose up -d
# 数据库会自动迁移
```

---

## 10. 参考资源

- **GitHub 仓库**：https://github.com/Tencent/WeKnora
- **官方网站**：https://weknora.weixin.qq.com
- **微信对话开放平台**：https://chatbot.weixin.qq.com
- **API 文档**：项目内 `docs/api/README.md`
- **知识图谱配置**：项目内 `docs/KnowledgeGraph.md`
- **MCP 配置指南**：项目内 `mcp-server/MCP_CONFIG.md`
- **开发指南**：项目内 `docs/开发指南.md`
- **路线图**：项目内 `docs/ROADMAP.md`

---

> 本文基于 WeKnora v0.4.0 版本撰写，项目持续迭代中，请关注 [GitHub Releases](https://github.com/Tencent/WeKnora/releases) 获取最新动态。
